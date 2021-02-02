# Spring Cloud 知识体系

##  什么是微服务？

微服务最早由Martin Fowler与James Lewis于2014年共同提出，微服务架构风格是一种使用一套小服务来开发单个应用的方式途径，每个服务运行在自己的进程中，并使用轻量级机制通信，通常是HTTP API，这些服务基于业务能力构建，并能够通过自动化部署机制来独立部署，这些服务使用不同的编程语言实现，以及不同数据存储技术，并保持最低限度的集中式管理。

微服务是一种用于构建应用的架构方案。微服务架构有别于更为传统的单体式方案，可将应用拆分成多个核心功能。每个功能都被称为一项服务，可以单独构建和部署，这意味着各项服务在工作（和出现故障）时不会相互影响。

**“微服务”不是银弹**

微服务并不是一劳永逸的解决了所有的问题，相反的，如果不能正确的使用微服务，则有可能被微服务自身的限制拖入另一个泥潭：

- 分布式的代价。原本在单体应用中，很多简单的问题都会在分布式环境下被几何级的放大。例如分布式事务、分布式锁、远程调用等，不光要考虑如何实现他们，相关场景的异常处理也是必须要考虑到的问题。
- 协同代价。如果你经历过一个项目上线需要发布十几个应用，而这些应用又分别由多个团队在维护。你就能深刻的体会到协同是一件多么痛苦的事情了。
- 服务拆分需要很强的设计功力。微服务的各种优势，其中一个重要的基础是对服务领域的正确切分。如果使用了不合适的切分粒度，或者是错误的切分方法，都会让服务不能很好的实现高内聚低耦合的要求。

##  微服务架构 

微服务架构是目前构建互联网系统的主流架构体系。通常来说，我们认为架构的发展历程经历了这样一个历程：单体架构 -> SOA架构 -> 微服务架构。

通常来说我们看到的微服务架构如下图所示：

![Microservice_Architecture](images\Microservice_Architecture.png)

## 微服务组件

基于目前业界主流的微服务实现技术提炼了八大技术体系，包括服务通信、服务治理、服务路由、服务容错、服务网关、服务配置、服务安全和服务监控，如下图所示

![服务八大组件](images\服务八大组件.png)

上图中的每个技术体系都非常重要，下面来对它们分别展开介绍。

###  服务通信

网络通信是任何分布式系统的基础组件。网络通信本身涉及面很广，相对比较复杂，对于微服务架构而言，我们关注的是网络连接模式、I/O 模型和服务调用方式。

![服务通信](images\服务通信.png)

基于TCP协议的网络连接有两种基本方式，也就是通常所说的长连接和短连接。长连接和短连接的产生在于client和server采取的关闭策略，具体的应用场景采用具体的策略。在微服务架构中，dubbo 框架采用的是长连接rpc方式，openfeign采用短连接http方式。

I/O 模型也有阻塞式 I/O 和非阻塞式 I/O 等多种实现方式。阻塞式 I/O 实现简单，而非阻塞式 I/O 的性能更好。在微服务架构中，以服务网关而言，像Netflix 的 Zuul就是阻塞式 I/O，而Spring 自研的 Spring Cloud Gateway则采用的是非阻塞式 I/O。

服务调用方式，也会有同步和异步调用两种方式。在微服务架构中，通常会采用异步转同步的实现机制，也就是说开发人员使用同步的方式进行方法调用，而框架本身会基于 Future 等机制实现异步的远程处理。

其他通信还可能基于消息通信、事件驱动的通信，有兴趣可以去了解。

###  服务治理

在微服务架构中，服务治理可以说是最为关键的一个技术组件，因为各个微服务需要通过服务治理实现自动化的注册和发现。如果尝试着用手动的方式来给每一个客户端来配置所有服务提供者的服务列表是一件非常困难的事，而且也不利于服务的动态扩缩容、健康检查等。

![服务注册发现流程](images\服务注册发现流程.png)

服务注册中心是保存服务调用所需的路由信息的存储仓库，也是服务提供者和服务消费者进行交互的媒介，充当着服务注册和发现服务器的作用。诸如 Dubbo、Spring Cloud 等主流的微服务框架都基于 Zookeeper、Eureka、Nacos 等分布式系统协调工具构建了服务注册中心。后续我们会对Nacos的服务注册、发现进行原理分析。

###  服务路由

当我们服务基于服务注册中心构建一个多台集群化的服务时，当客户端请求到达集群，如何确定由哪一台服务器进行请求响应呢？这就是服务路由问题。在Spring Cloud中，采用负载均衡是一种比较常见的路由方案，常见的客户端/服务器端负载均衡技术都可以完成服务路由。Spring Cloud 主要内置了 Ribbon 等客户端负载均衡组件。具体需要了解几种负载均衡算法，如何配置不同的负载均衡策略。

![注册中心与负载均衡结构示意图](images\注册中心与负载均衡结构示意图.png)



### 服务配置

在微服务架构中，配置中心也是微服务架构中的基础组件。其目的也是对服务配置信息进行统一管理。

![配置中心](images\配置中心.png)

为了满足以上要求，配置中心通常需要依赖分布式协调机制，即通过一定的方法确保配置信息在分布式环境中的各个服务中能得到实时、一致的管理。可以采用诸如 Zookeeper 等主流的开源分布式协调框架来构建配置中心。当然，像 Spring Cloud 提供了专门的配置中心实现工具 Spring Cloud Config，阿里巴巴也提供nacos进行配置中心。

###  服务网关

服务网关也叫API网关。在微服务架构中，服务网关的核心要点是，所有的客户端和消费端都通过统一的网关接入微服务，在网关层处理所有的非业务功能。

![服务网关](images\服务网关.png)

在功能设计上，服务网关在完成客户端与服务器端报文格式转换的同时，它可能还具有身份验证、监控、缓存、请求管理、静态响应处理等功能。另一方面，也可以在网关层制定灵活的路由策略。针对一些特定的 API，我们需要设置白名单、路由规则等各类限制。 Spring Cloud 提供了基于 Netflix Zuul 和 Spring Cloud Gateway 这两种网关。

### 服务容错

对于分布式环境中的服务而言，服务在自身失败引发生错误的同时，还会因为依赖其他服务而导致失败。除了比较容易想到和实现的超时、重试和异步解耦等手段之外，我们需要考虑针对各种场景的容错机制。

![服务熔断技术](images\服务熔断技术.png)

业界存在一批与服务容错相关的技术组件，包括以失效转移 Failover 为代表的集群容错策略，以线程隔离、进程隔离为代表的服务隔离机制，以滑动窗口、令牌桶算法为代表的服务限流机制，以及服务熔断机制。而从技术实现方式上看，在 Spring Cloud 中，这些机制部分包含在下面要介绍的服务网关中，而另一部分则被提炼成单独的开发框架，例如专门用于实现服务熔断的 Spring Cloud Circuit Breaker 组件。

###  服务安全

一般意义上的访问安全性，都是围绕认证和授权这两个核心概念来展开。也就是说我们首先需要确定用户身份，然后再确定这个用户是否有访问指定资源的权限。站在单个微服务的角度讲，我们系统每次服务访问都能与授权服务器进行集成以便获取访问 Token。站在多个服务交互的角度讲，我们需要确保 Token 在各个微服务之间的有效传播。在实现微服务安全访问上，我们通常使用 OAuth2 协议来实现对服务访问的授权机制，使用 JWT 技术来构建轻量级的认证体系。Spring 家族也提供了 Spring Security 和 Spring Cloud Security 框架来完整这些组件的构建。

![基于 Token 机制的服务安全结构图](images\基于 Token 机制的服务安全结构图.png)

###  服务监控

在微服务架构中，当服务数量达到一定量级时，我们难免会遇到两个核心问题。一个是如何管理服务之间的调用关系？另一个是如何跟踪业务流的处理过程和结果？这就需要构建分布式服务跟踪机制。

分布式服务跟踪机制的建立需要完成调用链数据的生成、采集、存储及查询，同时也需要对这些调用链数据进行运算和可视化管理。这些工作不是简单一个工具和框架能全部完成，因此，在开发微服务系统时，我们通常会整合多个开发框架进行链路跟踪。例如，在 Spring Cloud 中，就提供了 Spring Cloud Sleuth 与 Zipkin 的集成方案。

![服务链路跟踪](images\服务链路跟踪.png)

[从单体架构向微服务架构转型，这9个问题需要搞明白](https://mp.weixin.qq.com/s?src=11&timestamp=1612236621&ver=2865&signature=GX6etURZ7b4bpEgiVS-vStw6GLEtKmsINr*GD4ZBIBACqIaXZFXanQX0Bzg30J6O6xejXQfjk4ZkVksGiAPG7IjdtFoQZDQnFQJbiZcyW9z4rSztO61Fb2JqBtuMey1a&new=1)

[华为架构师揭秘微服务架构（单体架构与微服务架构对比）](https://mp.weixin.qq.com/s?src=11&timestamp=1612236621&ver=2865&signature=0bJy5dParY5Wyg-nnuPdX6c9bvhYY3oYdpgMGRWSIi6or9WnluyBzBwDHAImY7eSRKZH9VrmH4Ecuns2VLGz3uZV7sMrJZ-zl3Adw7-*DZipW-2oBOLZ-FMr0w7v4lIn&new=1)

[单体架构，垂直架构，SOA架构和微服务架构的变化历程](https://mp.weixin.qq.com/s?src=11&timestamp=1612237936&ver=2865&signature=Xkc2NYHE23uY2CVt31M1Pkp64HwFGk3dQ4bfI8Yvol4g4Q2OKQkc1x6P-SgqeoKZPC0CsnIHq0yk9mWxOFxiBlUH5riA7tfHPQ0Q8NOS1lXFn2k3oSGx-Y1D4cxOB13W&new=1)

[单体架构,SOA架构,微服务架构,分布式架构,集群架构](https://mp.weixin.qq.com/s?src=11&timestamp=1612237936&ver=2865&signature=ZwJKA9llWWJBukRjwdbx0LqiIDqnWoPxS8YHCXR5ZKnmnzo0mgVI67rstv9s8C2RegHyG95au9TPVDpnQ*rpkvEXRRPWyKWYtaBzzQA4Kk6yeiROUWYbmoJxKI563Upo&new=1)

[关于微服务架构入门篇](https://mp.weixin.qq.com/s?src=11&timestamp=1612238381&ver=2865&signature=DylUQ5PlN9t6m9icK9wimBOQfe02GmnvAGKa7R6GP9qbDTmxGtnQFebKz7EkAyq25f3evk5jFMVabsZAn649jOcbcN8PR*3ymKz6M4IINU7qoXnEJgJA-ib8Qz6HE6zz&new=1)

[从单体架构到微服务架构](https://mp.weixin.qq.com/s?src=11&timestamp=1612238872&ver=2865&signature=Prvf1LathhYbjcm-NAPpBHGAt7GQZD*Tq4KRoSt9rPFVehMnv2u2qiQFQ-bSA26rH-FwY80lG5pVcHuMJ7C6Iw05TLtHitlBuIcU9Kvq6h2ipl3qI*CwzFiO3bwV4Umq&new=1)

[单体架构和微服务系统架构的优缺点](https://mp.weixin.qq.com/s?src=11&timestamp=1612236621&ver=2865&signature=GK9KHPwtc2ocpe8rmgTBGxGFKl5f23o-EBd-IlNYXpV-g0hgleM91U6s9Hh9ZPHqjf4rHVFmCziZl4uDuNohgU3lezUN3fw7ojXZfqKg2gMf2476KaD-z4kCTmyHb7QX&new=1)

