---

layout: post
title: 什么是云原生
category: 技术
tags: Architecture
keywords: alibaba cloud native

---

## 简介

* TOC
{:toc}

从单机过渡到分布式，系统设计时就要考虑一致性、某个节点挂掉等问题，在云原生阶段，系统就要假设节点可能被扩缩容、从依赖计算机的操作系统和硬件变成依赖云服务 等场景。

## 什么是云原生

云是和本地相对的，传统的应用必须跑在本地服务器上，现在流行的应用都跑在云端，云包含了 IaaS、PaaS 和 SaaS（譬如邮件服务） 。原生就是土生土长的意思，我们在开始设计应用的时候就考虑到应用将来是运行在云环境上，要充分利用云资源的优点，比如️云服务的弹性和分布式优势。**云原生应用程序**源于云原生，它们构建并部署在云中，真正地访问了云基础设施的强大功能。**云计算应用程序**/云托管（Cloud Hosting）应用通常是在内部使用传统基础设施开发的，并且经过调整后可以在云中远程访问。PS：比如service mesh 的面向localhost 开发。 访问依赖资源的方式。

[解读容器 2019：把“以应用为中心”进行到底](https://www.kubernetes.org.cn/6408.html)云原生的本质是一系列最佳实践的结合；更详细的说，云原生为实践者指定了一条低心智负担的、能够以可扩展、可复制的方式最大化地利用云的能力、发挥云的价值的最佳路径。这种思想，以一言以蔽之，就是“以应用为中心”。正是因为以应用为中心，云原生技术体系才会无限强调**让基础设施能更好的配合应用**、以更高效方式为应用“输送”基础设施能力，而不是反其道而行之。而相应的， Kubernetes 、Docker、Operator 等在云原生生态中起到了关键作用的开源项目，就是让这种思想落地的技术手段。

[解读容器 2019：把“以应用为中心”进行到底](https://www.kubernetes.org.cn/6408.html)这篇文章读多少遍都不多

云原生的“上下文”是什么？阿里2015年前全部使用内部基础设施 ==> 2015-2018 活动期间 向阿里云申请“机房”（一批服务器）双十一后归还（机器资源成本下降50%）； 部分业务上云 ==> 全面上云 ==> 云原生

[传统架构 vs 云原生架构，谈谈为什么我们需要云原生架构？](https://mp.weixin.qq.com/s/zOlQ07dCLRy_cHTTai6V1g) 几个界面直观的展示了云原生一定发展阶段的 应用开发、运行、维护方式。让我想起了一个团队准备做一个校园app， 他们召集几十个学生，详细的列举了一个学生一天所有要做的事儿，然后删繁就简，最后得到一个校园app的原型。云原生也类似，列出开发维护过程中的所有痛点，解决他们，然后冠名以云原生。

[云原生关乎文化，而不是容器](https://mp.weixin.qq.com/s/GQZODViNUnjDjLZN5R0RvA)

[不是技术也能看懂云原生](https://mp.weixin.qq.com/s/csY8T02Ck8bnE3vVcZxVjQ)技术价值按重要性排序依次是：业务创新；系统稳定；成本优化；技术创新。然而这四点价值其实是有矛盾的，比如创新就要快速开发，如何同时保证稳定呢？再比如系统稳定需要做多副本的冗余，那如何成本优化呢？成本优化就要做改造，一折腾，系统可能就不稳了。再比如技术创新引入新技术，成本就会高，新技术就会不稳，咋办？如果成本控制很严，三年五年不创新，IT人员都跑了，到时候想开发新系统，找不到人，咋办？为了解决业务变而系统稳这一对矛盾，大牛们不断摸索，从而构建了一套目前看来边界尚未清晰的体系，最终命名为云原生。
2. immunitability/不可改变基础设施：所有对于部署环境的改变都要通过发布系统来改变，一旦发布就不再改变，如果要改变就重新走一次发布，而不能私自登陆到某台机器上去做改变。
1. 用云实现资源的弹性，是资源层敏捷的关键。
3. 定义了云原生各个阶段及各个阶段碰到的问题：基于虚拟化的云原生探索期；跨环境场景下的标准形成期；跨语言服务治理的标准形成期；大量业务使用云原生的规模落地期（发布、容器隔离性；大规模；网络选型；有状态存储；镜像下发；平滑升级；多集群；mesh优化）；全面发挥云原生价值的成熟深耕期



![](/public/upload/architecture/company_architecture_1.png)

![](/public/upload/architecture/company_architecture_2.png)

![](/public/upload/architecture/company_architecture_3.png)

[2021 年云原生技术发展现状及未来趋势](https://mp.weixin.qq.com/s/ZMCskVFy7WRA1DSVx7S1kg)从系统层次分可以区分出云原生基础设施【如存储、网络、管理平台 K8s】、云原生中间件、云原生应用架构以及云原生交付运维体系

[字节跳动是如何建设云的？](https://mp.weixin.qq.com/s/LUHsgD9nap3wAhexpiOYjA)  我自己感觉，云原生就是软件研发体系向前发展的一个过程。计算机刚刚被发明的时候，人们写机器代码太麻烦了，程序员都怕麻烦，于是有了高级编程语言，编译器；人们自己定义各种数据存储格式，读写存储非常麻烦，还容易出错，于是有了数据库；人们已经写了软件，想用的时候，还得买机器，租机房，部署网络，太麻烦了，于是有了云计算。那么，现在还有哪些事情太麻烦？还有很多。所以，我觉得云原生，就是考虑云上的软件开发、维护全流程，去改进各个子系统，目标就是让开发人员更聚焦在业务本身，让云平台去解决其他的“麻烦事儿”。

[云原生时代的DevOps平台设计之道](https://mp.weixin.qq.com/s/oxeNq4GHE85NUBIDcgixcg)

![](/public/upload/distribute/paas_cloud_native.png)

1. 最原始的裸机时代，并没有开发运维之分。从底层基础设施，一直到最顶层的业务系统都是同一批人在处理，这一批老程序员可以被称为真正的全栈工程师。但毫无疑问，每一个开发人员，都希望能够抛却运维工作，更专注于自己开发的代码。
2. 云计算时代的兴起以虚拟化技术为基础，软件定义基础设施变得炙手可热起来。运维人员通过建设并维护一套 IaaS 云平台，将计算资源进行池化。开发人员按需申请自己需要的虚拟机，从而得到一个操作系统界面来进行交互。与操作系统打交道，对开发人员依然是个巨大的挑战，在 IT 领域，操作系统就像一座漂浮在海上的冰山，看似只露出冰山一角，然而其庞大的知识领域“身躯”都隐藏在海平面以下。和裸机时代相比较，开发人员和运维人员已经是两个完全不同的群体，开发人员已经可以将自己的大部分精力放在业务系统上了。值得一提的是，对操作系统的掌控是不折不扣的运维领域技能。
3. 容器技术以及 Kuberentes 的横空出世，成为了云计算时代的分水岭。软件定义基础设施的技术手段已经被发挥到了极致，并且成为了现阶段运维人员的标配技能。运维人员通过建设并维护一套 PaaS 云平台，终于**将操作系统这一座最后的“大山”从开发人员的身上搬开**。开发人员可以将更多的精力放在业务系统上了，除了他们依然需要学习容器技术和 Kubernetes ，至少他们要学会如何面向 Kubernetes 编写业务系统所需的声明式配置文件。运维人员也通过 PaaS 云平台得到了自己想要的能力，容器技术和 Kubernetes 为他们带来了弹性、便捷性的巨大提升。
3. 云原生是一种面向云设计应用的思想理念，充分发挥云效能，组织内 IT 人员相互协作构建弹性可靠、松耦合、易管理、可观测的云应用系统，最终目标是提升软件交付效率，降低运维复杂度。相比上文中提到的 PaaS 平台，**起码要能够避免开发人员去编写复杂的 Kubernetes 声明式配置文件**。PS：代码变镜像，变实例。

跟随时代的变迁，我得出了一个结论：从开发人员与运维人员的关系上来看，IT 领域的演变，就是**运维人员通过不断向上接手开发人员眼中“跟开发无关的杂活”**，来不断为开发人员减负。开发人员在得到了解放后，可以将视角更多的聚焦在自己开发的业务系统上，释放出自己的创造力。那么跟随结论中的趋势，解放开发人员负担的脚步绝对不会停止。DevOps 的工作，就是以开发人员为用户群体，打造一套可以让开发人员毫无障碍的使用基础设施的“云原生平台“。

## 全面上云

在Cloud Native 时代之前，多数公司随着业务的发展，或多或少都会打造出自有、封闭的技术体系，如果是自建机房/采购物理机称为Hosting，如果是采购云厂商的资源自己搭，称之为Cloud Hosting。这一方面造成巨大的投入，使得公司的技术人才力量没有完全专注的投入到业务上，另一方面也造成了这个行业人才流动的困难。尽管很多的开源产品在一定程度上有助于解决这个问题，但还不足以**体系化**。对于云厂商而言，会提供越来越多开放、主流的技术栈的技术产品， 从而让客户有更为丰富和自主的选择权，也会做到让这些技术产品的互通性更好。

[把阿里巴巴的核心系统搬到云上，架构上的挑战与演进是什么？](https://www.infoq.cn/article/NWRrJjBWzgCQoq4QxoJw)全面上云也不只是将机器搬到云上，更重要的是 replatforming，集团的技术栈和云产品融合，应用通过神龙服务器 + 容器和 K8s 上云，数据库接入 PolarDB，中间件采用云中间件和消息产品，负载均衡采用云 SLB 产品等。

1. 自研神龙服务器
2. 存储方面，走向了全面云化存储，也即全面存储计算分离。盘古 

    ![](/public/upload/architecture/pangu.jpeg)

[一个优秀的云原生架构需要注意哪些地方](https://mp.weixin.qq.com/s/XoOcCfwHHKCb5R0Pjm8JOQ)paas在内部会去对接底层的存储、计算、网络、再加上标准的mysql、redis等开源服务，对用户提供统一的访问接口。PS：假设PaaS化了之后，公司的技术体系可以部署在任何运行k8s的地方

![](/public/upload/architecture/cloud_native_architecture.png)

PS：如何更快的开发部署运维一个服务？一开始是一个个中间件redis/mq等产品，之后是提供IaaS 省去自建机房，再之后是k8s 作为集群操作系统，一步步吃掉手工的部分，将能力可复制的提供出来。

## 上云架构未来演进：云原生

[阿里云蒋江伟：什么是真正的云原生？](https://mp.weixin.qq.com/s/UTBh1Nyv7IV7Tc6fzpEF9Q)早起狭义的理解将云原生等同于容器和服务网格，什么是广义的云原生呢？因云而生的软件、硬件、架构，就是真正的云原生。云原生更多应该从客户应用的视角来看，部署到云上的应用，必须用到了只有大规模公共云实践才能提供的三类能力的一类或多类，即弹性、API自动化部署和运维等特性；服务化的云原生产品，如RDS、EMR等；因云而生的软硬一体化架构。这，就是云原生。PS：比如现有的cpu 不能极致的支持虚拟化，AWS、微软、阿里都有自研产品。

**基础设施和业务的边界应该在哪里？真正的判断标准是业务关注的边界而非架构边界，基础设施应无须业务持续关注和维护**。例如，使用了容器服务，是否能让业务无须关注底层资源？也未必，取决于资源的弹性能力是否有充分的支持（否则业务仍需时刻关注流量和资源压力），也取决于可观测性（否则问题的排查仍需关注底层环境）的能力。

1. 节点托管的 K8s 服务
2. Service Mesh 化
3. 应用和基础设施全面解耦
4. 应用交付的标准化：OAM

解耦前

![](/public/upload/architecture/application_infrastructure_coupling.jpeg)

解耦后

![](/public/upload/architecture/application_infrastructure_decoupling.jpeg)

通过上云，最大化的使阿里巴巴的业务使用云的技术，通过技术架构的演进使业务更好的聚焦于自身的发展而无须关于通用底层技术，使业务研发效率提升，迭代速度更快是技术人的真正目标。PS：笔者的感受就是，有一阵鼓吹DevOps，代码上线之前评估流量、压测、上下游服务是否受得了等。从另一个视角看，我负责开发一个服务，那么除了我这个服务，**其它所有的一切计算、存储、上下游服务对我来说都是基础设施**，都可以假定是可用的。

[别再空谈云原生，来看实战方法论](https://mp.weixin.qq.com/s/0ztL33ZHl_UAqc4KsAzmuA)从技术的角度，云原生架构作为基于云原生技术的架构原则与设计模式的集合，旨在**将云应用中的非业务代码部分进行最大化剥离**，让云设施接管应用中原有的大量非功能特性（如弹性、韧性、安全、可观测性、灰度等），使业务不再有非功能性业务中断困扰的同时，具备轻量、敏捷、高度自动化的特性。从业务代码中剥离了大量非功能性特性 (不会是所有，比如易用性还不能剥离) 到 IaaS 和 PaaS 中，从而缩小业务代码开发人员的技术关注范围，**通过云厂商的专业性提升应用的非功能性能力**。

![](/public/upload/architecture/business_infrastructure_decouple.png)

云原生的实战过程中，**最大的挑战还是企业究竟选用哪些云原生技术去为其产生价值**。

![](/public/upload/architecture/cloud_native_maturity.png)

云原生架构成熟度模型提出的最大意义，在于对企业云原生化现状、能力和发展路径不清晰等问题，给出了评估与优化方向，帮助企业走上数字化转型“最短路径”。

## 云原生如何落地

[使用Kubernetes落地云原生困难重重](https://mp.weixin.qq.com/s/9XEVZcA8jwAEeS0TIaQ3xw) 值得细读

[蚂蚁金服 Service Mesh 深度实践](https://mp.weixin.qq.com/s/XjbmCxdJLKVcFlEUiM7Pig)

![](/public/upload/architecture/cloud_native_practice.png)

当我们回顾云计算的发展历程，会看到**基础架构**经历了从物理机到虚拟机，从虚 拟机再到容器的演进过程。在这大势之下，**应用架构**也在同步演进，从单体过渡到多层，再到当下的微服务。在变化的背后，有一股持续的动力，它来自于三个不变的追求:

1. 提高资源利用率
2. 优化开发运维体验。 [统一运维平台的思考](https://mp.weixin.qq.com/s/mLnqR2kwGNiFLL39g-Ml8Q)
3. 以及更好地支持业务发展。

[对话阿里云叔同：释放云价值，让容器成为“普适”技术](https://mp.weixin.qq.com/s/cIdQY4mWpAtDagEuvmH_1A)云原生正在通过方法论、工具集和理念重塑整个软件技术栈和生命周期

1. 过去我们常以虚拟化作为云平台和与客户交互的界面，为企业带来灵活性的同时也带来一定的管理复杂度；
2. 容器的出现，**在虚拟化的基础上向上封装了一层**，逐步成为云平台和与客户交互的新界面之一，应用的构建、分发和交付得以在这个层面上实现标准化，大幅降低了企业 IT 实施和运维成本，提升了业务创新的效率。从技术发展的维度看，开源让云计算变得越来越标准化，容器已经成为应用分发和交付的标准，可以将应用与底层运行环境解耦；
3. Kubernetes成为资源调度和编排的标准，屏蔽了底层架构的差异性，帮助应用平滑运行在不同的基础设施上；
4. 在此基础上建立的上层应用抽象如微服务和服务网格，逐步形成应用架构现代化演进的标准，开发者只需要关注自身的业务逻辑，无需关注底层实现

[云原生中间件的下一站](https://mp.weixin.qq.com/s/EWR6xwbf53XHZ8O3m2bdVA)在云原生时代，不变镜像作为核心技术的 docker 定义了不可变的单服务部署形态，统一了容器编排形态的 k8s 则定义了不变的 service 接口，二者结合定义了服务可依赖的不可变的基础设施。有了这种完备的不变的基础设置，就可以定义不可变的中间件新形态 -- **云原生中间件**。云原生时代的中间件，包含了不可变的缓存、通信、消息、事件(event) 等基础通信设施，应用只需通过本地代理即可调用所需的服务，无需关心服务能力来源。

### Service Mesh

[“下沉”的应用基础设施能力与 Service Mesh](https://www.kubernetes.org.cn/6408.html)在过去，我们编写一个应用所需要的基础设施能力，比如，数据库、分布式锁、服务注册 / 发现、消息服务等等，往往是通过引入一个中间件库来解决的。过更确切的说，中间件体系的出现，并不单单是要让“专业的人做专业的事”，而**更多是因为在过去，基础设施的能力既不强大、也不标准**。

不过，基础设施本身的演进过程，实际上也伴随着云计算和开源社区的迅速崛起。时至今日，以云为中心、以开源社区为依托的现代基础设施体系，已经彻底的打破了原先企业级基础设施能力良莠不齐、或者只能由全世界几家巨头提供的情况。

这个变化，正是云原生技术改变传统应用中间件格局的开始。更确切的说，**原先通过应用中间件提供和封装的各种基础设施能力，现在全都被 Kubernetes 项目从应用层“拽”到了基础设施层也就是 Kubernetes 本身当中**。而值得注意的是，Kubernetes 本身其实也不是这些能力的直接提供者， Kubernetes 项目扮演的角色，是通过声明式 API 和控制器模式把更底层的基础设施能力对用户“暴露”出去。这些能力，或者来自于“云”（比如 PolarDB 数据库服务）；或者来自于生态开源项目（比如 Prometheus 和 CoreDNS）。

这也是为什么 CNCF 能够基于 Kubernetes 这样一个种子迅速构建起来一个数百个开源项目组成的庞大生态的根本原因：Kubernetes 从来就不是一个简单的平台或者资源管理项目，它是一个分量十足的“接入层”，是云原生时代真正意义上的“操作系统”。PS：操作系统帮你简化访问内存、cpu、文件、外设的问题，k8s帮你简化 访问所有外界服务的问题。

可是，为什么只有 Kubernetes 能做到这一点呢？这是因为， **Kubernetes 是第一个真正尝试以“应用为中心“的基础设施开源项目**。以应用为中心，使得 Kubernetes 从第一天开始，就把声明式 API 、而不是调度和资源管理作为自己的立身之本。声明式 API 最大的价值在于“把简单留给用户，把复杂留给自己”。通过声明式 API，Kubernetes 的使用者永远都只需要关心和声明应用的终态，而不是底层基础设施比如云盘或者 Nginx 的配置方法和实现细节。

应用基础设施能力“下沉”，实际上伴随着整个云原生技术体系和 Kubernetes 项目的发展始终。比如， Kubernetes 最早提供的应用副本管理、服务发现和分布式协同能力，其实就是把构建分布式应用最迫切的几个需求，通过 Replication Controller，kube-proxy 体系和 etcd “下沉”到了基础设施当中。而 **Service Mesh ，其实就更进一步，把传统中间件里至关重要的“服务与服务间流量治理”部分也“下沉”了下来。当然，这也就意味着 Service Mesh 实际上并不一定依赖于 Sidecar ：只要能无感知的拦截下来服务与服务之间的流量即可（比如 API Gateway 和 lookaside 模式）**。

随着底层基础设施能力的日趋完善和强大，越来越多的能力都会被以各种各样的方式“下沉”下来。而在这个过程中，CRD + Operator 的出现，更是起到了关键的推进作用。 CRD + Operator，实际上把 Kubernetes 声明式 API 驱动对外暴露了出来，从而使得任何一个基础设施“能力”的开发者，都可以轻松的把这个“能力”植入到 Kubernetes 当中。

“声明式基础设施”的基础，就是让越来越多的“复杂”下沉到基础设施里，无论是插件化、接口化的 Kubernetes “民主化”的努力，还是容器设计模式，或者 Mesh 体系，这些所有让人兴奋的技术演进，最终的结果都是让 Kubernetes 本身变得越来越复杂。而声明式 API 的好处就在于，它能够在基础设施本身的复杂度以指数级攀升的同时，至少保证使用者的交互界面复杂度仍然以线性程度上升。否则的话，如今的 Kubernetes 恐怕早就已经重蹈了 OpenStack 的灾难而被人抛弃。

||linux|kubernetes|
|---|---|---|
|描述|单机操作系统|不只是集群操作系统管理一堆物理机<br>在应用和外界定义了一条线|
|作用|简化对硬件cpu、内存、外设的访问|简化对所有外部资源的访问|
|访问接口|系统调用|api|
|扩展方式|驱动|CRD/Operator|

“复杂”，是任何一个基础设施项目天生的特质而不是缺点。今天的 Linux Kernel 一定比 1991 年的第一版复杂不止几个数量级；今天的 Linux Kernel 开发人员也一定没办法像十年前那样对每一个 Module 都了如指掌。这是基础设施项目演进的必然结果。但是，**基础设施本身的复杂度，并不意味着基础设施的所有使用者都应该承担所有这些复杂度本身**。这就好比我们每一个人其实都在“使用” Linux Kernel，但我们并不会抱怨“Linux Kernel 实在是太复杂了”：很多时候，我们甚至并不知道 Linux Kernel 的存在。

Kubernetes 体系覆盖的三类技术群体：

1. 基础设施工程师，在公司里，他们往往称作“Kubernetes 团队”或者“容器团队”。他们负责部署、维护 Kubernetes 及容器项目、进行二次开发、研发新的功能和插件以暴露更多的基础设施能力。声明式 API 设计、CRD Operator 体系，就是为了方便基础设施工程师接入和构建新基础设施能力而设计的。
2. 运维工程师
3. 业务研发工程师


在下一代“以应用为中心”的基础设施当中，**业务研发通过声明式 API 定义的不再是具体的运维配置（比如：副本数是 10 个），而是“我的应用期望的最大延时是 X ms”**。接下来的事情，就全部会交给 Kubernetes 这样的应用基础设施来保证实际状态与期望状态的一致，这其中，副本数的设置只是后续所有自动化操作中的一个环节。只有让 Kubernetes 允许研发通过他自己的视角来定义应用，而不是定义 Kubernetes API 对象，才能从根本上的解决 Kubernetes 的使用者“错位”带来的各种问题。

在底层基础设施层逐渐 Serverless 化（PS：通过api操控），在 2019 年 4 月，Google Cloud 发布了 Cloud Run 服务，第一次把 Serverless Application 的概念呈现给大众。2019 年 9 月，CNCF 宣布成立应用交付领域小组，宣布将“应用交付”作为云原生生态的第一等公民。 2019 年 10 月，Microsoft 发布了基于 Sidecar 的应用中间件项目 Dapr ，**把“面向 localhost 编程”变成了现实**。而在 2019 年 10 月，阿里巴巴则联合 Microsoft 共同发布了 Open Application Mode（OAM）项目，第一次对“以应用为中心”的基础设施和构建规范进行了完整的阐述。在这场革命的终态，任何一个符合“以应用为中心“规范的软件，将能够**从研发而不是基础设施的视角自描述出自己的所有属性**，以及自己运行所需要所有的运维能力和外部依赖。这样的应用才能做到天生就是 Serverless Application

从 Serverless Infrastructure 到 Serverless Application，我们已经看到云原生技术栈正在迅速朝“应用”为中心聚拢和收敛。我们相信在不久的未来，一个应用要在世界上任何一个地方运行起来，唯一要做的，就是声明“我是什么”，“我要什么”。在这个时候，所有的基础设施概念包括 Kubernetes、Istio、Knative 都会“消失不见”：就跟 Linux Kernel 一样。

### Serverless

相 比容器技术，Serverless 可以将资源管理的粒度更加细化，使开发者更快上手云原 生，并且倡导**事件驱动模型支持业务发展**。从而帮助用户解决了资源管理复杂、低频业务资源占用等问题;实现**面向资源使用**，以取代**面向资源分配**的模式。

[Serverless 的喧哗与骚动（一）：Serverless 行业发展简史](https://www.infoq.cn/article/SXv6xredWW03P7NXaJ4m)

## 当我在说云原生时我在说什么？


云原生架构其实是一种方法论，没有对开发语言、框架、中间件等做限制，它是一些先进的设计理念的融合，包括容器、微服务、尽量解耦合、敏捷、容灾、频繁迭代、自动化等。什么是云原生架构，一个平台无关的、自动化的、具备容灾能力的敏捷的分布式业务系统。

1. 为什么要做微服务拆分？我们在讨论微服务时，其实讨论的是一个业务模块的微服务拆分是否彻底，这决定我们业务模块能否做到横向水平扩容、整体能否成为一个分布式系统。比如一个商城系统，库存服务逻辑变化了是否会影响到商品服务、订单服务，到双十一的时候，订单服务需要大幅扩容，我们能否只扩容一个服务、跟这个服务相关的数据库表而不影响其它服务。因此，我们需要尽量减少服务之间的业务逻辑耦合、数据耦合，**通过通信来进行数据共享，而不是通过共享数据来进行业务通信**，使得我们在做必要的变更时，影响的范围能降低到最小。
2. 为什么要容器化？容器是云原生架构的基础，这是容器的标准化属性来决定的，如果不使用容器，CICD、自动扩缩容就没办法做，我们不知道这些服务依赖什么配置、程序如何启动、如何停止，我们可以基于自身业务特性自己开发一套CICD系统、扩缩容系统，但是不是通用的，无法移植的。有人说没有集装箱就没有全球化，这里的容器就是集装箱，大家想是不是这个道理。容器还有很多优点，首先可以降低成本，K8s知道如何调度把容器到合适的机器上，使得集群节点使用率均衡、并且提高节点的资源使用率，可运维性也上来了。PS：**没有集装箱不耽误船来船往，但肯定效率非常低**。就像不拆分微服务也不是就不能跑，但效率低一样。

![](/public/upload/architecture/cloud_native_question.png)


[技术盘点：消息中间件的过去、现在和未来](https://mp.weixin.qq.com/s/fqPDodfS4JK5attdgK4eqg)云原生的核心问题是如何**重新设计应用**，才能充分释放云计算的技术红利，实现业务成功最短路径。PS：反过来说，我们不能假设一个应用是单机的，应用得与环境或者具体说是k8s 配合使用。以前linux binary的格式是 elf ，现在是 镜像 + yaml，尽可能是无状态的以便扩展（次之是存算分离），可以自动服务发现。

## 其它

[云原生时代，应用架构将如何演进？](https://mp.weixin.qq.com/s/Vu6jxpb_LU6dH3wS4eNtdw)未来，云上的资源会越来越丰富，在基础设施之上，云平台提供了更多的PaaS能力，就像是操作系统在提供了进程这些能力之上，还有很多的SDK。但是，这些能力目前在使用上还非常低效和不标准，使用过程也比较麻烦。**今天我们在以类似汇编的形式使用云**，云原生则在重新定义应用程序与云平台之间的契约，并围绕这个契约来构建更高级的编程语言和工具。这就是云原生时代背景下，应用架构演进非常重要的一个方向。

[面向云原生的应用/算力架构的想法](https://mp.weixin.qq.com/s/QdRKdcp-PIBUYGAgZwicAA)云的算力是分布式的，而分布式的算力是需要被「调度」起来的。在操作系统中，是在线程或是进程间通信视角去调度 CPU Core 的算力；在 IDC 中，在机架、机房（可用区）视角去调度不同服务器的算力；而在云上，我们在全球数据中心的基础上，一家小公司，也能在跨大洲、大洋的维度去调度 IDC 级别的算力。大公司当年做异地多活，要自建或者租用大的跨省 IDC，但不是每个小公司都能做这么大投入，今天在云上，只要架构合理，完全可以小成本完成。类似的，用户还可以把计算密集型业务放到收费低的偏远 IDC 降低成本。从这个意义上来说，云厂商的重资产基建实现了算力的普惠。

向阿里学习什么？

1. 一切问题 都可以变着法儿用技术问题去解决。
2. 就一个故障恢复，想不透的只能空口抱怨，若是想透了， 拉一个部门这个活儿都干不完。

![](/public/upload/kubernetes/fault_analysis_handle.png)

[微博云原生成本优化的6个最佳实践](https://mp.weixin.qq.com/s/DDkFGt2D_GKg4-TNTN6xDg)
1. 混合编排（提高单机利用率），不同业务使用的机器规格差异巨大
2. 异地部署（物理成本），机房成本不同，将高流量的上下游部署到一个偏远机房，减少跨机房跨专线的请求。
3. 自动扩缩容（降低服务常备冗余度），常备机器只需要能够扛日常峰值流量即可，使用动态扩缩容来保持冗余度 ==> 既能保证资源的有效利用，又能维持业务的常备冗余度。
4. 错峰应用（共享服务间冗余度）
5. 在离线整合 （共享服务间冗余度）
6. Intel 换成 AMD（物理成本）
这些优化手段也会带来服务稳定性的巨大挑战，单机部署更多实例可能会带来可用性风险，加大了服务和资源治理的难度，而解决这些问题则需要指标体系、调度体系、混合云、服务画像、硬件资源等方方面面的相互协作。微博还存在一些独特的业务场景，可以通过带宽优化、存储优化、冷热分离、应用优化等等手段做进一步优化

