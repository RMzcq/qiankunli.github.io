---

layout: post
title: client-go informer源码分析
category: 架构
tags: Kubernetes
keywords:  kubernetes client-go

---

## 简介

* TOC
{:toc}

```
k8s.io/client-go
    /rest
    /informer 
        /core
            /v1
                /pod.go
                /interface.go
            /interface.go
        /factory.go // 定义sharedInformerFactory struct
    /tools
        /cache      // informer 机制的的重点在cache 包里
            /shared_informer.go // 定义了 sharedIndexInformer struct
            /controller.go
            /reflector.go
            /delta_fifo.go
```

[如何使用Go调用Kubernetes API？](https://mp.weixin.qq.com/s/aGwK75-NXRy3WwcAIVG8zQ)在剖析client-go本身之前，了解它的两个主要依赖项可能是一个好主意，k8s.io/api和k8s.io/apimachinery模块。这两个模块被分离出来是有原因的——它们不仅可以被客户端使用，也可以被服务器端使用，或者被处理Kubernetes对象的任何其他软件使用。
1. k8s.io/api模块：1000个以上的结构描述Kubernetes API对象，自带JSON和Protobuf注解，几乎没有算法，只有“哑”的数据结构。专注于具体的高级类型，如Deployment、Secret、Pod
2. k8s.io/apimachery
	1. 低层但更通用的数据结构。例如，Kubernetes对象的所有这些公共属性：apiVersion、kind、name、uid、ownerReferences、creationTimestamp等。TypeMeta、ObjectMeta、runtime.Object（Go 很晚才出现泛型支持，在此之前，runtime.Object是一个泛型接口，在代码库中广泛地进行类型断言和类型切换）
	2. GetOptions、ListOptions、UpdateOptions等等——这些结构体代表了客户端对资源的相应动作的参数。
	3. GroupKind、GroupVersionKind、GroupResource、GroupVersionResource等——简单的数据传输对象，包含组、版本、类型或资源字符串的元组。
	3. 对象序列化为JSON、YAML或Protobuf
	4. API错误处理

Kubernetes Controller能够知道资源对象的当前状态，通常需要访问API Server才能获得资源对象，当Controller越来越多时，会导致API Server负载过大。Kubernetes使用Informer代替Controller去访问API Server，Controller的所有操作都和Informer进行交互，而Informer并不会每次都去访问API Server。Informer使用ListAndWatch的机制，在Informer首次启动时，会调用LIST API获取所有最新版本的资源对象，然后再通过WATCH API来监听这些对象的变化，并将事件信息维护在一个只读的缓存队列中提升查询的效率，同时**降低API Server的负载**。除了ListAndWatch，Informer还可以注册相应的事件，之后如果监听到的事件变化就会调用对应的EventHandler，**实现回调**。

## 整体设计

[Kubernetes: Controllers, Informers, Reflectors and Stores](http://borismattijssen.github.io/articles/kubernetes-informers-controllers-reflectors-stores)Kubernetes offers these powerful structures to get a local representation of the API server's resources.The **Informer just a convenient wrapper** to  automagically syncs the upstream data to a downstream store and even offers you some handy event hooks.

针对同一类资源只建立一个连接，同一类资源对象（比如Pod、Deployment）都共享一个Informer（底层都用到了SharedIndexInformer），减少kube-apiserver的负载。

[Kubernetes Informer 详解](https://developer.aliyun.com/article/679508) Informer 只会调用 Kubernetes List 和 Watch 两种类型的 API，Informer 在初始化的时，先调用 Kubernetes List API 获得某种 resource 的全部 Object（真的只调了一次），缓存在内存中; 然后，调用 Watch API 去 watch 这种 resource，去维护这份缓存; 之后，Informer 就不再调用 Kubernetes 的任何 API。Kubernetes 中的组件如果要访问Kubernetes 中的Object，绝大部分会使用Informer中的Lister() 方法，而非自接调用kube-apiserver。

![](/public/upload/kubernetes/k8s_controller_model.png)

[client-go的informer的工作流程](https://cloudsre.me/2020/03/client-go-0-informer/)

informer 机制主要两个流程

1. Reflector 通过ListWatcher 同步apiserver 数据（只启动时搞一次），并watch apiserver ，将event 加入到 delta Queue 中。PS：reflector 大部分时间都是在watch event/delta
2. controller 从 delta Queue中取出event，调用Indexer进行缓存并建立索引，并触发Processor 业务层注册的 ResourceEventHandler。即processLoop。

![](/public/upload/kubernetes/informer_overview.png)

## Reflector

[client-go 之 Reflector 源码分析](https://mp.weixin.qq.com/s/VLmIK8vcGNw7fI7xb1ZQCA)

[Kubernetes client-go 源码分析 - Reflector](https://mp.weixin.qq.com/s/aZr2WkzDAaJNdjw-WstEbA) 未读

```go
// k8s.io/client-go/tools/cache/reflector.go
type Reflector struct {
  name string
  expectedTypeName string
  expectedType reflect.Type // 放到 Store 中的对象类型
  expectedGVK *schema.GroupVersionKind
  // 与 watch 源同步的目标 Store
  store Store
  // 用来执行 lists 和 watches 操作的 listerWatcher 接口（最重要的）
  listerWatcher ListerWatcher
  WatchListPageSize int64
  ...
```
Reflector 对象通过 Run 函数来启动监控并处理监控事件

```go
// k8s.io/client-go/tools/cache/reflector.go
// Run 函数反复使用 ListAndWatch 函数来获取所有对象和后续的 deltas。
// 当 stopCh 被关闭的时候，Run函数才会退出。
func (r *Reflector) Run(stopCh <-chan struct{}) {
  wait.BackoffUntil(func() {
    if err := r.ListAndWatch(stopCh); err != nil {
      utilruntime.HandleError(err)
    }
  }, r.backoffManager, true, stopCh)
}
func (r *Reflector) ListAndWatch(stopCh <-chan struct{}) error {
	var resourceVersion string
	options := metav1.ListOptions{ResourceVersion: r.relistResourceVersion()}
	if err := func() error {
		var list runtime.Object
		listCh := make(chan struct{}, 1)
		go func() {
			pager := pager.New(...)
			pager.PageSize = xx
			list, paginatedResult, err = pager.List(context.Background(), options)
			close(listCh)   //close listCh后，下面的select 会解除阻塞
		}()
		select {
		case <-stopCh:
			return nil
		case r := <-panicCh:
			panic(r)
		case <-listCh:
		}
		...
		r.setLastSyncResourceVersion(resourceVersion)
		return nil
	}()
	// 处理resync 逻辑
	for {
		options = metav1.ListOptions{...}
		w, err := r.listerWatcher.Watch(options)
		if err := r.watchHandler(start, w, &resourceVersion, resyncerrc, stopCh); err != nil {...}
	}
}
func (r *Reflector) watchHandler(start time.Time, w watch.Interface, resourceVersion *string, errc chan error, stopCh <-chan struct{}) error {
loop:
	for {
		select {
		case <-stopCh:
			return errorStopRequested
		case err := <-errc:
			return err
		case event, ok := <-w.ResultChan():
			meta, err := meta.Accessor(event.Object)
			newResourceVersion := meta.GetResourceVersion()
			switch event.Type {
			case watch.Added:
				err := r.store.Add(event.Object)
			case watch.Modified:
				err := r.store.Update(event.Object)
			case watch.Deleted:
				err := r.store.Delete(event.Object)
			case watch.Bookmark:
				// A `Bookmark` means watch has synced here, just update the resourceVersion
			default:
				utilruntime.HandleError(fmt.Errorf("%s: unable to understand watch event %#v", r.name, event))
			}
			*resourceVersion = newResourceVersion
			r.setLastSyncResourceVersion(newResourceVersion)
			if rvu, ok := r.store.(ResourceVersionUpdater); ok {
				rvu.UpdateResourceVersion(newResourceVersion)
			}
		}
	}
	return nil
}
```
Reflector.Run ==> pager.List + listerWatcher.Watch ==> Reflector.watchHandler ==> store.Add/Update/Delete ==> DeltaFIFO.Add  obj 加入DeltaFIFO。

首先通过Reflector的 relistResourceVersion 函数获得Reflector relist 的资源版本，如果资源版本非 0，则表示根据资源版本号继续获取，当传输过程中遇到网络故障或者其他原因导致中断，下次再连接时，会根据资源版本号继续传输未完成的部分。

ResourceVersion（资源版本号）非常重要，Kubernetes 中所有的资源都拥有该字段，它标识当前资源对象的版本号，每次修改（CUD）当前资源对象时，Kubernetes API Server 都会更改 ResourceVersion，这样 client-go 执行 Watch 操作时可以根据ResourceVersion 来确定当前资源对象是否发生了变化。

## DeltaFIFO

DeltaFIFO 和 FIFO 一样也是一个队列，**DeltaFIFO里面的元素是一个个 Delta**。DeltaFIFO实现了Store和 Queue Interface。生产者为Reflector，消费者为 Pop() 函数。

```go
// k8s.io/client-go/tools/cache/delta_fifo.go
type Delta struct {
	Type   DeltaType	// Added/Updated/Deleted/Replaced/Sync
	Object interface{}
}
type DeltaFIFO struct {
	items map[string]Deltas //  存储key到元素对象的Map，提供Store能力
    queue []string      	// key的队列，提供Queue能力
    ...
}
func (f *DeltaFIFO) Pop(process PopProcessFunc) (interface{}, error) {...}
// Get returns the complete list of deltas for the requested item
func (f *DeltaFIFO) Get(obj interface{}) (item interface{}, exists bool, err error) {...}
```


[Kubernetes client-go 源码分析 - DeltaFIFO](https://mp.weixin.qq.com/s/140YOECmektc_HxQbVkpjA)

![](/public/upload/kubernetes/delta_queue.jpg)

疑问：DeltaFIFO 是用来传递delta/event的，不是为了来传递obj 的，watch 得到 event 用queue 缓冲一些可以理解，为何底层要搞那么复杂呢？从设计看，**evnet 在队列里可能对堆积**，比如一个 add event 新增key=a，之后又有一个update event 将key=a改为b，其实此时可以 直接合并为一个 add event 即key=b。堆积之后 仍然让 消费者依次处理所有event(`Pop()`)，还是告诉它所有的event(`Get()`)，还是自动帮它做合并？PS: 很多能力 因为封装的太好，以至于不知道

## Indexer（未完成）

因为etcd存储的缘故，k8s的性能 没有那么优秀，假设集群有几w个pod，list 就是一个巨耗时的操作，有几种优化方式
1. list 时加上 label 限定查询范围。k8s apiserver 支持根据 label 对object 进行检索
2. 使用client-go 本地cache，再进一步，根据经常查询的label/field 建立本地index。PS：apiserver 确实对label 建了索引，但是本地并没有自动建立。

[Kubernetes client-go 源码分析 - Indexer & ThreadSafeStore](https://mp.weixin.qq.com/s/YVl4z0Yr0cDp3dFg6Kwg5Q) 未细读

[client-go 之 Indexer 的理解](https://cloud.tencent.com/developer/article/1692517) 未细读

## Watch event 消费

sharedIndexInformer.Run ==> controller.Run ==> controller.processLoop ==> for Queue.Pop 也就是 sharedIndexInformer.HandleDeltas ==> 更新LocalStore + processor.distribute

```go
func (s *sharedIndexInformer) HandleDeltas(obj interface{}) error {
	// from oldest to newest
	for _, d := range obj.(Deltas) {
		switch d.Type {
		case Sync, Added, Updated:
			isSync := d.Type == Sync
			if old, exists, err := s.indexer.Get(d.Object); err == nil && exists {
				if err := s.indexer.Update(d.Object); err != nil {...}
				s.processor.distribute(updateNotification{oldObj: old, newObj: d.Object}, isSync)
			} else {
				if err := s.indexer.Add(d.Object); err != nil {...}
				s.processor.distribute(addNotification{newObj: d.Object}, isSync)
			}
		case Deleted:
			if err := s.indexer.Delete(d.Object); err != nil {...}
			s.processor.distribute(deleteNotification{oldObj: d.Object}, false)
		}
	}
	return nil
}
```

## processor 是如何处理数据的

两条主线
1. sharedIndexInformer.HandleDeltas ==> sharedProcessor.distribute ==> 多个 processorListener.addCh 往channel 里塞数据。
2. sharedIndexInformer.Run ==> sharedProcessor.run ==> sharedProcessor.pop   消费channel数据
这里要注意的是，sharedProcessor.distribute 是将消息分发给多个processorListener， processorListener.pop 必须处理的非常快，否则就会阻塞distribute 执行。

```go
// k8s.io/client-go/tools/cache/shared_informer.go
type sharedProcessor struct {
	listenersStarted bool
	listenersLock    sync.RWMutex
	listeners        []*processorListener
	syncingListeners []*processorListener
	clock            clock.Clock
	wg               wait.Group
}
func (p *sharedProcessor) distribute(obj interface{}, sync bool) {
    for _, listener := range p.listeners {
        // 加入到processorListener 的addCh 中，随后进入pendingNotifications，因为这里不能阻塞
        listener.add(obj)    
    }
}
// k8s.io/client-go/tools/cache/shared_informer.go
type processorListener struct {
	nextCh chan interface{}
	addCh  chan interface{}
    handler ResourceEventHandler
    pendingNotifications buffer.RingGrowing
    ...
}
func (p *processorListener) add(notification interface{}) {
	p.addCh <- notification
}
func (p *sharedProcessor) run(stopCh <-chan struct{}) {
	func() {
		for _, listener := range p.listeners {
			p.wg.Start(listener.run)   // 消费nextCh     
			p.wg.Start(listener.pop)   // 消费addCh 经过 mq 转到 nextCh
		}
		p.listenersStarted = true
	}()
	...
}
```

![](/public/upload/kubernetes/client_go_processor.png)

消息流转的具体路径：addCh ==> notificationToAdd ==> pendingNotifications ==> notification ==> nextCh。 搞这么复杂的原因就是：pop作为addCh 的消费逻辑 必须非常快，而下游nextCh 的消费函数run 执行的速度看业务而定，中间要通过pendingNotifications 缓冲。

```go
func (p *processorListener) pop() {
	var nextCh chan<- interface{}
	var notification interface{}  // 用来做消息的中转，并在最开始的时候标记pendingNotifications 为空
	for {
        // select case channel 更多是事件驱动的感觉，哪个channel 来数据了或者可以 接收数据了就处理哪个 case 内逻辑
		select {
		case nextCh <- notification:
			// Notification dispatched
			notification, ok = p.pendingNotifications.ReadOne()
			if !ok { // Nothing to pop
				nextCh = nil // Disable this select case
			}
		case notificationToAdd, ok := <-p.addCh:
			if notification == nil { // No notification to pop (and pendingNotifications is empty)
				// Optimize the case - skip adding to pendingNotifications
				notification = notificationToAdd
				nextCh = p.nextCh
			} else { // There is already a notification waiting to be dispatched
				p.pendingNotifications.WriteOne(notificationToAdd)
			}
		}
	}
}
func (p *processorListener) run() {
	stopCh := make(chan struct{})
	wait.Until(func() {
		for next := range p.nextCh {
			switch notification := next.(type) {
			case updateNotification:
				p.handler.OnUpdate(notification.oldObj, notification.newObj)
			case addNotification:
				p.handler.OnAdd(notification.newObj)
			case deleteNotification:
				p.handler.OnDelete(notification.oldObj)
			default:
				utilruntime.HandleError(fmt.Errorf("unrecognized notification: %T", next))
			}
		}
		// the only way to get here is if the p.nextCh is empty and closed
		close(stopCh)
	}, 1*time.Second, stopCh)
}
```

一个eventhandler 会被封装为一个processListener，一个processListener 对应两个协程，run 协程负责 消费pendingNotifications 所有event 。pendingNotifications是一个ring buffer， 默认长度为1024，如果被塞满，则扩容至2倍大小。如果event 处理较慢，则会导致pendingNotifications 积压，event 处理的延迟增大。PS：业务实践上确实发现了 pod 因各种原因大量变更， 叠加 event 处理慢 导致pod ready 后无法及时后续处理的情况

## workqueue

[Kubernetes client-go 源码分析 - workqueue](https://mp.weixin.qq.com/s/9zHYc266cJlXGcZa-xnOFA) 讲的很详细。

![](/public/upload/kubernetes/workqueue.png)

如果想自定义控制器非常简单，我们直接注册handler就行。但是绝大部分k8s原生控制器中，handler并没有直接处理。而是统一遵守一套：add/update/del -> queue -> run -> runWorker -> syncHandler 处理的模式。有几个好处
1. chan的功能过于单一，无法满足各类场景的需求，workqueue除了一个缓冲机制外，还有错误重试、限速等机制。
2. 利用了Indexer本地缓存机制，queue里面只包括 key就行。数据indexer里有


## 细节

### watch 是如何实现的？

[K8s 如何提供更高效稳定的编排能力？K8s Watch 实现机制浅析](https://mp.weixin.qq.com/s/0H0sYPBT-9JKOle5Acd_IA)从 HTTP 说起： HTTP 发送请求 Request 或服务端 Response，会在 HTTP header 中携带 Content-Length，以表明此次传输的总数据长度。如果服务端提前不知道要传输数据的总长度，怎么办？
1. HTTP 从 1.1 开始增加了分块传输编码（Chunked Transfer Encoding），将数据分解成一系列数据块，并以一个或多个块发送，这样服务器可以发送数据而不需要预先知道发送内容的总大小。数据块长度以十六进制的形式表示，后面紧跟着 `\r\n`，之后是分块数据本身，后面也是 `\r\n`，终止块则是一个长度为 0 的分块。为了实现以流（Streaming）的方式 Watch 服务端资源变更，HTTP1.1 Server 端会在 Header 里告诉 Client 要变更 Transfer-Encoding 为 chunked，之后进行分块传输，直到 Server 端发送了大小为 0 的数据。
2. HTTP/2 并没有使用 Chunked Transfer Encoding 进行流式传输，而是引入了以 Frame(帧) 为单位来进行传输，其数据完全改变了原来的编解码方式，整个方式类似很多 RPC协议。Frame 由二进制编码，帧头固定位置的字节描述 Body 长度，就可以读取 Body 体，直到 Flags 遇到 END_STREAM。这种方式天然支持服务端在 Stream 上发送数据，不需要通知客户端做什么改变。K8s 为了充分利用 HTTP/2 在 Server-Push、Multiplexing 上的高性能 Stream 特性，在实现 RESTful Watch 时，提供了 HTTP1.1/HTTP2 的协议协商(ALPN, Application-Layer Protocol Negotiation) 机制，在服务端优先选中 HTTP2。

![](/public/upload/kubernetes/k8s_list_watch.png)

HTTP1.1例子： 当客户端调用watch API时，apiserver 在response的HTTP Header中设置Transfer-Encoding的值为chunked，表示采用分块传输编码，客户端收到该信息后，便和服务端保持该链接，并等待下一个数据块，即资源的事件信息。例如：

```sh
$ curl -i http://{kube-api-server-ip}:8080/api/v1/watch/pods?watch=yes
HTTP/1.1 200 OK
Content-Type: application/json
Transfer-Encoding: chunked
Date: Thu, 02 Jan 2019 20:22:59 GMT
Transfer-Encoding: chunked
{"type":"ADDED", "object":{"kind":"Pod","apiVersion":"v1",...}}
{"type":"ADDED", "object":{"kind":"Pod","apiVersion":"v1",...}}
{"type":"MODIFIED", "object":{"kind":"Pod","apiVersion":"v1",...}}
```

从`k8s.io/apimachinery/pkg/watch` 返回的watch.Interface 
```go
type Interface interface{
    Stop()
    ResultChan() <- Event
}
type Event struct{
    Type EventType  // ADDED/MODIFIED/DELETED/ERROR
    Object runtime.Object
}
```
[Kubernetes List-Watch 机制原理与实现 - chunked](https://mp.weixin.qq.com/s/FOVjzOtwgeSOnuC_HsQF_w)

[K8s apiserver watch 机制浅析](https://mp.weixin.qq.com/s/jp9uVNyd8jyz6dwT_niZuA) 未细读。 为了减轻etcd的压力，kube-apiserver本身对etcd实现了list-watch机制，然后再把watch 转到client-go。

![](/public/upload/kubernetes/client_go_watch.jpg)

### resync机制

[为什么需要 Resync 机制](https://github.com/cloudnativeto/sig-kubernetes/issues/11#issuecomment-670998151)

Informer 中的 Reflector 通过 List/watch 从 apiserver 中获取到集群中所有资源对象的变化事件（event），将其放入 Delta FIFO 队列中（以 Key、Value 的形式保存），触发 onAdd、onUpdate、onDelete 回调将 Key 放入 WorkQueue 中。同时将 Key 更新 Indexer 本地缓存。Control Loop 从 WorkQueue 中取到 Key，从 Indexer 中获取到该 Key 的 Value，进行相应的处理。

我们在使用 SharedInformerFactory 去创建 SharedInformer 时，需要填一个 ResyncDuration 的参数
```go
// k8s.io/client-go/informers/factory.go
// NewSharedInformerFactory constructs a new instance of sharedInformerFactory for all namespaces.
func NewSharedInformerFactory(client kubernetes.Interface, defaultResync time.Duration) SharedInformerFactory {
	return NewSharedInformerFactoryWithOptions(client, defaultResync)
}
```

这个参数指的是，多久从 Indexer 缓存中同步一次数据到 Delta FIFO 队列，重新走一遍流程

```go
type DeltaFIFO struct {
	...
	knownObjects KeyListerGetter	// 实质是indexer
}
// k8s.io/client-go/tools/cache/delta_fifo.go
// 重新同步一次 Indexer 缓存数据到 Delta FIFO 队列中
func (f *DeltaFIFO) Resync() error {
	// 遍历 indexer 中的 key，传入 syncKeyLocked 中处理
	keys := f.knownObjects.ListKeys()
	for _, k := range keys {
		f.syncKeyLocked(k)
	}
	return nil
}

func (f *DeltaFIFO) syncKeyLocked(key string) error {
	obj, exists, err := f.knownObjects.GetByKey(key)
	// 如果发现 FIFO 队列中已经有相同 key 的 event 进来了，说明该资源对象有了新的 event，
	// 在 Indexer 中旧的缓存应该失效，因此不做 Resync 处理直接返回 nil
	id, err := f.KeyOf(obj)
	if len(f.items[id]) > 0 {
		return nil
	}
    // 重新放入 FIFO 队列中
	err := f.queueActionLocked(Sync, obj)
	return nil
}
```
**为什么需要 Resync 机制呢？**因为在处理 SharedInformer 事件回调时，可能存在处理失败的情况，**定时的 Resync 让这些处理失败的事件有了重新 onUpdate 处理的机会**。那么经过 Resync 重新放入 Delta FIFO 队列的事件，和直接从 apiserver 中 watch 得到的事件处理起来有什么不一样呢？
```go
// k8s.io/client-go/tools/cache/shared_informer.go
func (s *sharedIndexInformer) HandleDeltas(obj interface{}) error {
	// from oldest to newest
	for _, d := range obj.(Deltas) {
		// 判断事件类型，看事件是通过新增、更新、替换、删除还是 Resync 重新同步产生的
		switch d.Type {
		case Sync, Replaced, Added, Updated:
			s.cacheMutationDetector.AddObject(d.Object)
			if old, exists, err := s.indexer.Get(d.Object); err == nil && exists {
				err := s.indexer.Update(d.Object)
				isSync := false
				switch {
				case d.Type == Sync:
					// 如果是通过 Resync 重新同步得到的事件则做个标记
					isSync = true
				case d.Type == Replaced:
					...
				}
				// 如果是通过 Resync 重新同步得到的事件，则触发 onUpdate 回调
				s.processor.distribute(updateNotification{oldObj: old, newObj: d.Object}, isSync)
			} else {
				err := s.indexer.Add(d.Object)
				s.processor.distribute(addNotification{newObj: d.Object}, false)
			}
		case Deleted:
			err := s.indexer.Delete(d.Object)
			s.processor.distribute(deleteNotification{oldObj: d.Object}, false)
		}
	}
	return nil
}
```

从上面对 Delta FIFO 的队列处理源码可看出，如果是从 Resync 重新同步到 Delta FIFO 队列的事件，会分发到 updateNotification 中触发 onUpdate 的回调。




## 其他

![](/public/upload/kubernetes/client_go_informer_process.png)
