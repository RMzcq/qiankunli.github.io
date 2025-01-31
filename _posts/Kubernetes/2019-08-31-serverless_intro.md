---

layout: post
title: serverless 泛谈
category: 架构
tags: Kubernetes
keywords: serverless

---

## 简介

* TOC
{:toc}

云计算相关技术的发展，往往有一个特点：云厂商的驱动性非常强，因为云厂商往往会最先感知到普遍性的用户需求，并且有足够的数据支撑其做出合理的判断与创新。所以 Serverless 架构的创新很多时候也都是由厂商驱动的；一篇讲Serverless 的文章有一个留言：Serverless是在服务网格架构之后进行业务逻辑层的改造。Serverless 的核心理念是按需使用，解耦应用和资源，自动弹性。**用户不需要容量规划，即不需要提前评估和预留容量**，在运行时按照实际需求自动伸缩。 PS：有点像云厂商直接卖pod/按pod收费，pod 也尽量不需要配cpu和mem

![](/public/upload/architecture/serverless_overview.png)

[Serverless 躁动背后的 5 大落地之痛](https://mp.weixin.qq.com/s/F8ZoUywD4ezadfQA6Wu1lg)建议细读。

[拥抱开放，Serverless 时代的下一征程](https://mp.weixin.qq.com/s/BSM_IQ14worC_qfPnVLOuw)

[谈一谈企业级弹性伸缩与优化建设](https://mp.weixin.qq.com/s/iOHr3QzML6kDV4__v29qPg) Serverless 不是特指某种技术，而是无服务器、按需弹性、按量计费、简化运维等的工具结合， 通过一个应用引擎框架集成云开发、FaaS、BaaS、托管K8s、DevOps、服务治理、弹性等帮助业务轻松上云

![](/public/upload/architecture/serverless_design.png)

毕玄：Serverless 面临的核心难题是什么？主要是弹性的速度。比如 Serverless 我要拉起一个系统，这不是几秒就能拉得起来的，但对在线业务来讲，你不可能点一下几秒都出不来，大家能接受的都是几毫秒。所以 Serverless 都只是拿来做一些像计算这种后期的事情，以前可能是一些离线的事情，都是对响应时间可以忍受的。

## 为什么Serverless 会火

Kubernetes 虽然提供了容器应用的全生命周期管理，但是太丰富、太复杂、太灵活，这既是一种优势，有时候也是一种劣势。容器和 Kubernetes 并非一定要捆绑使用。

从用户视角看：托管node ==> 托管pod ==> 托管function。从实践上也是必须这样慢慢推进的，比如一家公司不用云厂商的对象存储、数据库，然后突然一步托管到function 是不可能的。[Serverless 时代下微服务应用全托管解决方案](https://mp.weixin.qq.com/s/dxNX1pG6n72cclZD1IENmg)微服务化给运维带来了很大的挑战:
1. 效率：随着应用规模的扩张，新的研发团队需要面临很多开发和测试中的复杂性问题。在团队协作上，不同应用团队之间如何更好地形成稳定的调用链路，在几十，几百甚至上千个应用的大规模场景里如何进行调用链路上应用的快速部署和灰度。此外，如此多应用的流量的处理、调用链路的跟踪和服务鉴权也非常影响效率。
2. 稳定：微服务化之后，会出现调用链路上某核心应用出现问题，导致整体系统发生雪崩，而且有时缺少可视化、可观测性的系统来帮助快速定位分析问题，导致难以快速定位到出现问题的应用，造成长时间的损失；
3. 成本：单体应用一般只需部署几台机器；到了微服务时代，随着应用数的剧增，出于可用性的考虑需要为每个应用保持一些冗余，比如一次大促中，一个调用链路会涉及到十几个应用，为了稳定性以及调用链路的安全，会进行整个链路应用的扩容，而实际上很多应用可能长时间没有流量，服务器空闲，导致巨大的成本浪费。 
面对微服务带来的这些问题和需求， Serverless 应用引擎在这方面都做了哪些工作? 带来哪些改变？PS：或者说，这些问题如果能得到比较妥善的解决，其实不用Serverless 也可以。

[Serverless 的前世今生](https://mp.weixin.qq.com/s/_lL9AemgKggQA_k8WfKmlg) 未细读

[新兴软件研发范式崛起，云计算全面走向 Serverless 化](https://mp.weixin.qq.com/s/uiksst3d5206M7y-NlNqug)在软件研发流程中，所有工作可以分为以下三类：
1. 业务代码开发，实现业务逻辑。
2. 非功能性代码开发，包括实现容错、安全、可观测、可运维、三方软件集成等和业务逻辑无关，但又是企业应用必须具备的能力。
3. 应用基础设施管理。包括搭建开发、测试、生产环境，资源规划，安全管控等等。
这三类工作中，只有第一类是对业务带来真正价值，和企业核心竞争力密切相关的。但随着软件复杂度的提升，2、3类工作却消耗了大量的研发资源。**尽可能降低2、3类工作的复杂度，让客户专注于业务逻辑开发，是软件架构和研发模式发展的必然方向**。过去十年，无论是开源社区还是云厂商，都在不同领域将非功能性代码开发和应用基础设施管理工作抽象为标准化，可复用的软件/服务。

Serverless 是一个非常广义的概念，并不局限于计算。一般而言，同时满足以下条件的服务可以称之为 Serverless 服务：

1. 全托管服务。意味着客户使用抽象的服务化接口，而不是直接面对底层资源，也就没有安装、配置、维护或者更新软硬件的负担。全托管服务通常也提供了内置的容错，安全和可观测能力，用户通常不需要再重新构建这些能力。
2. 自适应弹性。意味着服务能够根据负载大小自动弹性伸缩，大大提升了资源使用效率。
3. 按实际用量付费。意味着只需根据实际的执行时间、流量或调用次数付费，降低了成本。过去云计算用云服务器替代了物理服务器，**但客户依旧按“几核几 G 服务器”的模式来购买云资源**，未来云计算将全面 Serverless 化，更加接近“电网”模式，按计算的调用次数付费。
因此 Serverless 服务核心价值在于尽可能消除客户非功能性代码开发，简化应用基础设施管理的工作，从而实现研发效率的飞跃。过去十年，无论是开源社区还是云厂商，**都在不同领域将非功能性代码开发和应用基础设施管理工作抽象为标准化**，可复用的软件/服务。随着厂商在存储、计算、中间件、大数据等领域推出越来越多的 Serverless 服务，并且这些服务通过事件驱动等方式紧密集成，**云逐渐变成了应用构建和运行的超级平台**。

李林峰：简而言之，云计算的发展方向是资源服务化，业务全托管。FaaS和微服务的很大区别是，在微服务时代，通信方式、通信协议、操作系统镜像升级、微服务框架升级、注册中心和服务发现、路由策略和灰度、服务治理、容灾和多活等都需要业务方结合平台的能力来构建，业务需要在核心业务功能开发之外构建大量的能力保障系统的韧性和可扩展性。到了FaaS，一切交给基础设施，完全的代码托管、自适应的弹性伸缩、按使用量付费，让开发者用云而看不到云。尽管大方向如此，但是正如微服务的发展，很多开发者都是从单体应用切换到微服务开发，最近几年刚入职的同学直接上手微服务，他们已经不需要再学习和使用单体架构体系。对于Serverless也是如此，一部分微服务开发者会切换到Serverless技能栈，当Serverless发展到一定阶段，新的同学可能直接写函数，而不需要掌握微服务开发技能。Serverless更像未来的自动驾驶网约车，它对业务完全黑盒，你不知道函数底层怎么调度、运行在什么上面，但是业务对它的质量要求会更高，当它不成熟时对业务而言简直是灾难，所以，技术的发展和演进需要周期，让时间来验证和催熟它吧～

### 资源调度的复杂性 ==> 为什么做 Serverless

[没有银弹，只有取舍 - Serverless Kubernetes 的思考与征程（一）](https://mp.weixin.qq.com/s/1aMalQs-AE2L1aA5X20gJA)

![](/public/upload/kubernetes/resource_scheduler_overview.png)

Kubernetes作为一个分布式集群管理系统，它的一个重要目标是：将适合的资源分配给适合的应用，满足对应用的QoS要求和获得最优的资源使用效率。然而，分布式系统的资源调度有着非常高的复杂性。主要挑战包括：
1. 对多形态异构资源的支持，今天应用所需的计算资源不只是简单的CPU，内存，存储等，而且包括多样化的加速设备，比如GPU、RDMA等。而且，为了考虑到计算效率的最优化，要考虑到计算资源之间的拓扑，比如CPU core在numa节点间的布局，GPU设备间NVLink拓扑等。此外随着高性能网络的的发展，GPU池化、内存池化等相继出现，给资源调度带来更多的动态性和复杂性。
2. 对多样化的工作负载的支持。从Stateless的Web应用、微服务应用，到有状态的中间件和数据应用，再到AI、大数据、HPC等计算任务类应用。他们对资源申请和使用的方式有不同的需求。
3. 对多维度的业务需求的支持。调度系统在满足应用对资源的需求的同时，也要满足不同的业务需求，比如计算效率，优先级，稳定性，利用率等等。
调度系统需要在多样化的资源和多样化的约束之间进行动态决策，整体挑战很高。而且随着时间推移，集群中逐渐出现负载不均衡的现象，资源热点会导致。如何持续调整集群负载。

Kubernetes做出了几个重要的架构选择，大大缓解了分布式集群管理系统的附属复杂性。
1. Kubernetes架构的核心就是就是控制循环 （control loops），也是一个典型的"负反馈"控制系统。当控制器观察到期望状态与当前状态存在不一致，就会持续调整资源，让当前状态趋近于期望状态。
2. 声明式API是云原生重要的设计理念，让开发者可以关注于应用自身，而非系统执行细节。这样的架构方式也有助于将整体复杂性下沉，交给基础设施实现并持续优化。PS：命令式限定了太多细节，底层优化空间就不大了
2. K8s通过一系列抽象如CNI - 容器网络接口, CSI - 容器存储接口，允许基础设施提供方提供差异化的实现，但是遵从统一的控制面接口。这帮助业务应用可以较少关注底层基础设施差异，能够在不同环境中一致管理、自由迁移；也提升了基础设施提供方的积极性，构建有竞争力的产品能力。

Kubernetes 遗留的运维复杂性。在生产环境中落地 Kubernetes，持续保障系统的稳定性，安全性和规模化成长。对绝大多数客户依然充满挑战。很多企业的K8s团队的日常工作是这个样子的
1. 日常维护集群，进行版本升级，平均每个月要进行一次小版本升级；平均每年要进行一到两次大版本升级
2. 日常更新操作系统安全补丁，平均每个月要进行一次。[KubeOS : 面向云原生场景的容器操作系统](https://mp.weixin.qq.com/s/P1Itgb6m61_0sFS-uIIuVw)
3. 解决容器集群中各种问题应急，每天n次
4. 对集群进行容量评估，手动扩缩容，按需

Kubernetes的数据面是由节点组成，节点可以是虚拟机，裸金属服务器或者物理机。K8s 控制面动态调度Pod到节点进行执行。这样架构非常自然，但也有一些天然的缺点
1. Pod与节点生命周期不同步：
    1. 节点就绪后，才能进行Pod调度，降低了弹性的效率
    2. 节点维护/下线/缩容，需要迁移所有节点上的Pod，极大增加了弹性的复杂性。
2. 同节点内部Pod共享资源：
    1. 共享内核，扩大了攻击面。用OS提供的namespace, seccomp等机制无法实现很好的安全隔离。
    2. 共享资源，产生相互影响。CPU，内存，I/O，临时存储容量等，有些无法通过cgroup进行很好的资源隔离。
3. 容器网络与节点网络独立管理：
    1. 要为节点，容器、Service 独立配置 CIDR
    2. 在跨多个可用区、混合云、或者企业网络拓扑编排等较复杂场景下，大多数客户缺乏足够的能力实现合理的网络规划。
4. 容量规划与弹性配置复杂
    1. 需要用户管理节点池，选择合适的节点规格进行扩容，优化整体资源利用率，增加了复杂性。

我们希望对 Kubernetes 进行 radical simplification，实现几个关键的
1. 免运维 - 用户无需对K8s控制面和数据面进行运维。让用户聚焦业务应用而非底层基础设施管理
2. 按需付费 - 无需预留资源，按应用实际资源使用量费。
3. 简化容量管理 - 让应用可以弹性伸缩，无需关注集群资源的调整。

### DevOps

[Knative Serverless 之道：如何 0 运维、低成本实现应用托管？](https://mp.weixin.qq.com/s/uihFHmUHeeIWuj5wwjNFrw)计算、存储和网络这三个核心要素已经被 Kubernetes 层统一了，Kubernetes 已经提供了 Pod 的无服务器支持，而应用层想要用好这个能力其实还有很多事情需要处理。

1. 弹性：缩容到零；突发流量
2. 灰度发布：如何实现灰度发布；灰度发布和弹性的关系
3. 流量管理：灰度发布的时候如何在 v1 和 v2 之间动态调整流量比例；流量管理和弹性是怎样一个关系；当有突发流量的时候如何和弹性配合，做到突发请求不丢失

我们发现虽然基础资源可以动态申请，但是应用如果要做到实时弹性、按需分配和按量付费的能力还是需要有一层编排系统来完成应用和 Kubernetes 的适配。这个适配不单单要负责弹性，还要有能力同时管理流量和灰度发布。

![](/public/upload/architecture/serverless_layer.png)

1. 资源层关注的是资源（如容器）的生命周期管理，以及安全隔离。这里是 Kubernetes 的天下，Firecracker，gVisor 等产品在做轻量级安全沙箱。这一层关注的是如何能够更快地生产资源，以及保证好安全性。
2. DevOps 层关注的是变更管理、流量调配以及弹性伸缩，还包括基于事件模型和云生态打通。这一层的核心目标是如何把运维这件事情给做没了（NoOps）。虽然所有云厂商都有自己的产品（各种 FaaS），但是我个人比较看好 Knative 这个开源产品，原因有二：

    1. 其模型非常完备；
    2. 其生态发展非常迅速和健康。很有可能未来所有云厂商都要去兼容 Knative 的标准，就像今天所有云厂商都在兼容 Kubernetes 一样。

3. 框架和运行时层呢，由于个人经验所限，我看的仅仅是 Java 领域，其实核心的还是在解决 Java 应用程序启动慢的问题（GraalVM）。当然框架如何避免 vendor lock-in 也很重要，谁都怕被一家云厂商绑定，怕换个云厂商要改代码，这方面主要是 Spring Cloud Function 在做。

[当我们在聊 Serverless 时你应该知道这些](https://mp.weixin.qq.com/s/Krfhpi7G93el4avhv9UN4g)

![](/public/upload/architecture/ali_cloud_function.png)

### 《Serverless入门课》

为什么阿里会上升到整个集团的高度来推进 Serverless 研发模式升级，对此，杜欢表示：“ 这件事本身好像是一件技术的事情，但其实它背后就是钱的事情，都是跟钱相关的。”

为什么阿里巴巴、腾讯这样的公司都在关注 ServerlessServerless 既可以满足资源最大化利用的需求，也能够调优行业内的**开发岗位分层结构。**

1. Serverless 可以有效**降低企业中中长尾应用的运营成本**。中长尾应用就是那些每天大部分时间都没有流量或者有很少流量的应用。尤其是企业在落地微服务架构后，一些边缘的微服务被调用的概率其实很低。而这个时候，我们往往又很难通过人工来控制中长尾应用，因为这里面不少应用还是被强依赖的，不可以直接下线处理。Serverless 之前，这些中长尾应用至少要独占 1 台虚拟机；现在有了 Serverless 的极速冷启动特性，企业就可以节省这部分开销。PS：核心系统可能因为极致追求性能等 还是会“重型”开发，但对于小需求 确实可以用Serverless 来支撑。
2. 其次，Serverless 可以提高研发效能。**现有的研发形态并不能最大化地发挥部分工作岗位的价值**，这同样是一种浪费。以导购类型的业务为例，开发这样一个业务通常需要前端开发工程师和后端开发工程师一起配合，但这两个开发岗位在该业务形态下并不能很好地发挥自己的全部价值。对于后端工程师来说，他在这个业务里要做的事情更多只是把现有的一些服务能力、数据组合在一起，变成一个新的数据提供给前端；而前端开发工程师负责把这个数据在页面上展示出来。这样的工作比较机械化，也没有太大的挑战，并不利于后端工程师的个人成长和岗位价值发挥；但在现有的研发模式下，由于缺乏前后端的连接点，前端工程师又不能去做这些比较简单的后端工作，业务上线也不可能给到前端工程师时间和机会去学习再实践。
Serverless 应用架构中，SFF（Serverless For Frontend）可以让前端同学自行负责数据接口的编排，微服务 BaaS 化则让我们的后端同学更加关注领域设计。可以说，这是一个颠覆性的变化，它能够**进一步放大前端工程师的价值**。

### 从技术演化看Serverless

[Serverless 的喧哗与骚动（一）：Serverless 行业发展简史](https://www.infoq.cn/article/SXv6xredWW03P7NXaJ4m)

1. 单机时代，操作系统管理了硬件资源，贴着资源层，高级语言让程序员描述业务，贴着业务层，编译器 /VM 把高级语言翻译成机器码，交给操作系统；
2. 今天的云时代，资源的单位不再是 CPU、内存、硬盘了，而是容器、分布式队列、分布式缓存、分布式文件系统。就像从汇编语言发展到高级语言后，开发者不用再去关注寄存器、信号、中断等与机器底层相关的细节

![](/public/upload/architecture/machine_vs_cloud.png)

今天我们把应用程序往云上搬的时候（a.k.a Cloud Native)，往往都会做两件事情：

1. 把巨型应用拆小，微服务化；
2. 摇身一变成为 yaml 工程师，写很多 yaml 文件来管理云上的资源。

这里存在两个巨大的 gap，这两个 gap 在图中用灰色的框表示了：

1. 编程语言和框架，目前主流的编程语言基本都是假设单机体系架构运行的
2. 编译器，程序员不应该花大量时间去写 yaml 文件，这些面向资源的 yaml 文件应该是由机器生成的，我称之为云编译器，高级编程语言用来表达业务的领域模型和逻辑，云编译器负责将语言编译成资源描述。

[喧哗的背后：Serverless 的概念及挑战](https://mp.weixin.qq.com/s/vxFRetml4Kx8WkyoSTD1tQ)Docker 和 Kubernetes，其中前者标准化了**应用分发的标准**，不论是 Spring Boot 写的应用，还是 NodeJS 写的应用，都以镜像的方式分发；而后者在前者的技术上又定义了**应用生命周期的标准**，一个应用从启动到上线，到健康检查、下线，有了统一的标准。有了应用分发的标准和生命周期的标准，云就能提供**标准化的应用托管服务**，包括应用的版本管理、发布、上线后的观测、自愈等。例如对于无状态的应用来说，一个底层物理节点的故障根本就不会影响到研发，因为应用托管服务基于标准化应用生命周期可以自动完成腾挪工作，在故障物理节点上将应用的容器下线，在新的物理节点上启动同等数量的应用容器。在此基础上，由于应用托管服务能够感知到应用运行期的数据，例如业务流量的并发、CPU load、内存占用等，业务就可以配置基于这些指标的伸缩规则，由平台执行这些规则，根据业务流量的实际情况增加或者减少容器数量，这就是最基本的 auto scaling，自动伸缩。这就能够帮助用户避免在业务低峰期限制资源，节省成本，提升运维效率。PS： **应用分发的标准 + 应用生命周期的标准 ==> 标准化的应用托管服务 + 运行数据及流量监控 ==> auto scaling ==> 由平台系统管理机器，而不是由人去管理，这就是一个很朴素的 Serverless 理解**。

如果我们把目光放到今天云的时代，那么就不能狭义地把 Serverless 仅仅理解成为不用关心服务器。云上的资源除了服务器所包含的基础计算、网络、存储资源之外，还包括各种类别的更上层的资源，例如数据库、缓存、消息等等。我们今天主流的使用云的方式不应该是未来我们使用云的方式。我认为 Serverless 的愿景应该是 Write locally, compile to the cloud，即代码只关心业务逻辑，由工具和云去管理资源。PS：这时候就又提到sql，代码就如同sql 只负责表达业务（描述做什么）而不关心怎么做。

[云原生运行时的下一个五年：函数是不是下一站？](https://mp.weixin.qq.com/s/UXsOaSMar74nGk07ANAP2A) 非常不错。

[从 Docker 讲起，深度揭秘阿里云 Serverless Kubernetes](https://mp.weixin.qq.com/s/7uBRYNUAnfqjivKqpQFgRA)无论是 k8s 还是 ServiceMesh，都在分别尝试将服务管理和流量管理下沉到基础设施中。但这些组件本身也存在管理成本，所以演化出**云上托管**。用户从云平台获取一个 kubeconfig 文件便可以直接通过 kubectl 命令行或者 Restful API 管理集群。再进一步，如果集群里面运行任务大部分都是 long run 并且资源需求是固定的任务，使用 ACK 没有问题，但如果是大量 job 类型的任务或者存在突发流量的情况，ACK 这种临时扩容虚拟机在虚拟机上启用容器方案在弹性方面有所欠缺。那么，有没有一种既能兼容 Kubernetes 使用方式，又能够秒级启动 Pod，并且**按照 Pod 维度计费**（ACK 按照 Node 维度计费）的方案呢？

### Serverfull vs Serverless

Serverfull 就是服务端运维全由我们自己负责，Serverless 则是服务端运维较少由我们自己负责，大多数的运维工作交给自动化工具负责。

1. 农耕时代，发布+看日志等工具化
2. 工业时代，资源优化和扩缩容方案也可以利用性能监控 + 流量估算解决；代码自动化发布的流水线：代码扫描 - 测试 - 灰度验证 - 上线。

免运维 NoOps 并不是说服务端运维就不存在了，而是通过全知全能的服务，覆盖研发部署需要的所有需求，让研发同学对运维的感知越来越少。另外，NoOps 是理想状态，因为我们只能无限逼近 NoOps，所以这个单词是 less，不可能是 ServerLeast 或者 ServerZero。

1. 狭义 Serverless（最常见）= Serverless computing 架构 = FaaS 架构 = Trigger（事件驱动）+ FaaS（函数即服务）+ BaaS（后端即服务，持久化或第三方服务）= FaaS + BaaS
2. 广义 Serverless = 服务端免运维 = 具备 Serverless 特性的云服务

![](/public/upload/distribute/serverless_definition.png)

[喧哗的背后：Serverless 的概念及挑战](https://mp.weixin.qq.com/s/vxFRetml4Kx8WkyoSTD1tQ)虽然说是 Serverless，但 Server（服务器）是不可能真正消失的，Serverless 里这个 **less 更确切的说是开发不用关心的意思**。这就好比现代编程语言 Java 和 Python，开发就不用手工分配和释放内存了，但内存还在哪里，只不过交给垃圾收集器管理了。称一个能帮助你管理服务器的平台为 Serverless 平台，就好比称呼 Java 和 Python 为 Memoryless 语言一样。

这里讲下笔者的一个体会，一开始在公司内搞容器化，当时觉得只要把应用以容器的方式跑起来就可以了（应用托管平台）。一开始拿测试环境试验容器化，运维把测试环境的维护工作完全交给 容器团队。这时，天天干的一个活儿是 给web 开发配域名，为此后来针对域名配置定义了一套规范、 开发了一个nginx插件自动化了（引入了网关之后，改为自动更新网关接口）。这个过程其实就是 less 的过程。


## 手感/Knative

![](/public/upload/distribute/serverless_helloworld.png)

||物理机时代|Serverless|
|---|---|---|
|运行环境|在服务端构建代码的运行环境|FaaS 应用将这一步抽象为函数服务|
|入口|负载均衡和反向代理|FaaS 应用将这一步抽象为 HTTP 函数触发器|
||上传代码和启动应用|FaaS 应用将这一步抽象为函数代码|

Knative 作为最流行的应用 Severlesss 编排引擎，其中一个核心能力就是其简洁、高效的**应用托管服务**。Knative 提供的应用托管服务可以让您免去维护底层资源的烦恼，提升应用的迭代和服务交付效率。
1. Build构建系统/Tekton：把用户定义的应用构建成容器镜像，面向kubernetes的标准化构建，区别于Dockerfile镜像构建，重点解决kubernetes环境的构建标准化问题。
2. Serving服务系统：利用Istio的部分功能，来配置应用路由，升级以及弹性伸缩。Serving中包括容器生命周期管理，容器外围对象（service，ingres）生成（恰到好处的把服务实例与访问统一在一起），监控应用请求，自动弹性负载，并且利用Virtual service和destination配置服务访问规则，**流量、灰度（版本）和弹性这三者是完美契合在一起的**。PS: Service 与普通workload 并无多大区别，结合了istio，**你不用为服务配置port 等体现Server的概念**

    ```yml
    apiVersion: serving.knative.dev/v1alpha1
    kind: Service
    metadata:
      name: stock-service-example
      namespace: default
    spec:
      template:
        metadata:
          name: stock-service-example-v2
          annotations:
            autoscaling.knative.dev/class: "kpa.autoscaling.knative.dev" # 自动扩缩容
          spec:
            containers:
            - image: registry.cn-hangzhou.aliyuncs.com/knative-sample/rest-api-go:v1
                env:
                - name: RESOURCE
                    value: v2
              readinessProbe:
                httpGet:
                  path: /
                initialDelaySeconds: 0
                periodSeconds: 3
      traffic: # 流量管理
      - tag: v1
        revisionName: stock-service-example-v1
        percent: 50
      - tag: v2
        revisionName: stock-service-example-v2
        percent: 50
    ```

3. Eventing事件系统：用于自动完成事件的绑定与触发。**事件系统与直接调用**最大的区别在于响应式设计，它允许运行服务本身不需要屏蔽了调用方与被调用方的关系。从而在业务层面能够实现业务的快速聚合，或许为后续业务编排创新提供事件。PS：这块还感受不到价值

客户端请求通过入口网关转发给Activator（此时pod实例数为0），Activator汇报指标给Autoscaler，Autoscaler创建Deployment进而创建Pod，一旦Pod Ready，Activator会将缓存的客户端请求转发给对应的Pod，网关也会将新的请求直接转发给响应的Pod。当一定周期内没有请求时，Autoscaler会将Pod replicas设置为0，同时网关将后续请求路由到Activator。 PS：可以缩容到0是因为有一个常驻的Activator

![](/public/upload/architecture/knative_autoscaler.png)

## 与FaaS的关系

Serverless不等价于FaaS。也有人故意划分为Functions Serverless和容器化的Serverless。Serverless和FaaS是不同的概念，FaaS在目前的实践中一般都会实现为Serverless，但Serverless并不局限于一定是FaaS模式，比如普通的微服务也是可以以Serverless方式部署的。

FaaS 与应用托管 PaaS（**应用托管平台**） 平台对比，**最大的区别在于资源利用率**，这也是 FaaS 最大的创新点。FaaS 的应用实例可以缩容到 0，而应用托管 PaaS 平台则至少要维持 1 台服务器或容器。FaaS 优势背后的关键点是可以极速启动，现在的云服务商，基于不同的语言特性，冷启动平均耗时基本在 100～700 毫秒之间。

**为什么 FaaS 可以极速启动，而应用托管平台 PaaS 不行？**PaaS 为了适应用户的多样性，必须支持多语言兼容，还要提供传统后台服务，例如 MySQL、Redis。这也意味着，应用托管平台 PaaS 在初始化环境时，有大量依赖和多语言版本需要兼容，而且兼容多种用户的应用代码往往也会增加应用构建过程的时间。所以通常应用托管平台 PaaS 无法抽象出轻量的可复用的层级，只能选择服务器或容器方案，从操作系统层开始构建应用实例。FaaS 设计之初就牺牲了用户的可控性和应用场景，来简化代码模型，并且通过分层结构进一步提升资源的利用率。

关于 FaaS 的 3 层结构，你可以这么想象：容器层就像是 Windows 操作系统；Runtime 就像是 Windows 里面的播放器暴风影音；你的代码就像是放在 U 盘里的电影。

[阿里云 FaaS 架构设计与创新实践](https://mp.weixin.qq.com/s/MQzdC1UT5w42oHYY4TwYEQ) 未读

[从云计算到函数计算](https://mp.weixin.qq.com/s/OBAiJg8ujAAvKmBc7qitkA)

## 实现

[Serverless Kubernetes 的 流派](https://mp.weixin.qq.com/s/1aMalQs-AE2L1aA5X20gJA) 需要继续整理。

[腾讯 Nodeless 弹性容器技术演进和实践](https://www.infoq.cn/article/2JPIXj7aGW8murb40QA7) 启动一个虚拟机，做成pod的感觉。

## 其它

Serverless 架构和之前的架构相比，最大的差异是：业务服务不再是固定的常驻进程，而是真正按需启动和关闭的服务实例。

如果基础设施和相关服务不具备实时扩缩容的能力，那么业务整体就不是弹性的。

[无服务器已死？这项技术为什么变得人人嫌弃](https://mp.weixin.qq.com/s/dyCrv-fjN9cGcQXlcl3q1A)一个良好运行的单体应用或许不应变成一个连接到八个网关、四十个队列和数十个数据库实例的一系列”函数“。因此，无服务器适用于那些尚未开发的领域。几乎没有将现有应用（架构）移植过来的案例。

[距离 Java 开发者玩转 Serverless，到底还有多远？](https://mp.weixin.qq.com/s/rhJQfnb1g3AS84ahKZr4vQ)

[云原生体系下 Serverless 弹性探索与实践](https://mp.weixin.qq.com/s/npEhYuzAqDk3tw1b58NMKQ) 几个收获
1. 互联网企业往往由于其内部应用具有显著流量特征，应用启动依赖多，速度慢，且对整体资源池容量水位，库存财务管理，离在线混部有组织上的诸多诉求，因而**更多的是以容量画像提前弹性扩容为主**，基于 Metrics 计算的容量数据作为实时修正，其目标是容量画像足够精准以至于资源利用率达到预期目标。公有云厂商服务于外部客户，提供更为通用，普适的能力，并通过可拓展性满足不同用户的差异化需求。尤其在 Serverless 场景，更强调应用应对突发流量的能力，其目标在于无需容量规划，通过指标监控配合极致弹性能力实现应用资源的近乎按需使用且整个过程服务可用。PS：互联网行业的服务可能秒级启动做不到。
2. 原地升级给 SAE 带来了诸多价值，其中最重要的是避免重调度，避免 Sidecar 容器（ARMS,SLS,AHAS）重建,使得整个部署耗时从消耗整个 Pod 生命周期到只需要拉取和创建业务容器，于此同时因为无需调度，**可以预先在 Node 上缓存新镜像** （PS：镜像push后知道往哪推了），提高弹性效率。SAE 采用阿里开源 Openkruise 项目提供的 Cloneset 作为新的应用负载，借助其提供的原地升级能力，使得整个弹性效率提升 42%。PS：没想到原地升级这么重要
3. 对于 Java 应用冷启动较慢的痛点，SAE 联合 Dragonwell 11 提供了增强的 AppCDS  启动加速策略，AppCDS 即 Application Class Data Sharing，通过这项技术可以获取应用启动时的 Classlist 并 Dump 其中的共享的类文件，当应用再次启动时可以使用共享文件来启动应用，进而有效减少冷启动耗时

