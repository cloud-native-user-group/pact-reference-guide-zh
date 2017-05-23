# Pact

### 什么是Pact?

Pact框架家族提供对[消费者驱动的契约](http://martinfowler.com/articles/consumerDrivenContracts.html)测试的支持。

### 消费者驱动的契约
`契约`是在客户端（`消费者`）与API端（`提供者`）之间的一组约定，描述了两者之间所发生的交互。

消费者驱动的契约是一种从`消费者`视角来驱动`提供者`开发的模式。

**Pact**是一种可用于确保这些`契约`被满足的测试工具。

### 为什么使用Pact

<table>
  <tr>
    <th>信心</th>
    <th>更快</th>
    <th>更不容易出错</th>
  </tr>
  <tr>
    <td>
    持续演进你的代码库，Pact将会保证契约被满足。
    </td>
    <td>
    不用再搭建端到端的环境。不用再手动测试。</td>
    <td>
    契约的生成和验证都是由Pact自动管理的。
    </td>
  </tr>
</table>

其实我也需要被[说服](faq/convinceme.html)。

### 演讲与展示

关于Pact的介绍，参见Pact作者之一的如下[关于Pact的演讲](http://www.infoq.com/presentations/pact) 及幻灯片。
<p style="text-align: center;">
<iframe src="http://www.slideshare.net/slideshow/embed_code/key/f4e6DF51EttgzJ" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="http://www.slideshare.net/bethesque/pact-44565612" title="Consumer-Driven Contracts with Pact (Sydney API Days 2015)" target="_blank">Consumer-Driven Contracts with Pact (Sydney API Days 2015)</a> </strong> from <strong><a target="_blank" href="http://www.slideshare.net/bethesque">Beth Skurrie</a></strong> </div>
</p>

具体到JVM相关的演讲，在[MelbJVM](http://www.meetup.com/en-AU/Melbourne-Java-JVM-Users-Group/)四月的meet-up和[Melbourne Microservices](http://www.meetup.com/en-AU/Melbourne-Microservices/)六月的meet-up上曾进行过名为[Deploy with Confidence!](https://www.youtube.com/watch?v=h-79QmIV824)的演讲。演讲幻灯片在[这里](media/Pact%20-%20Deploy%20with%20Confidence!.pdf)。

Atlassion在其2016 summit上的演讲：[Verifying Microservice Integrations with Contract Testing](https://www.youtube.com/watch?v=-6x6XBDf9sQ&feature=youtu.be)，对消费者驱动的契约（和Pact）作出了很好的解释。

### 各种语言的实现
- [Ruby Pact](https://github.com/realestate-com-au/pact)
- [JVM Pact](https://github.com/DiUS/pact-jvm) 和 [Scala-Pact](https://github.com/ITV/scala-pact)
- [.NET Pact](https://github.com/SEEK-Jobs/pact-net)
- [JS Pact](https://github.com/DiUS/pact-consumer-js-dsl)
- [Go Pact](https://github.com/pact-foundation/pact-go) (还有一个v1.1版本的原生[Pact Go](https://github.com/SEEK-Jobs/pact-go))
- [Swift / Objective-C Pact](https://github.com/DiUS/pact-consumer-swift)


### 介绍性文章

* [Getting started with Pact](http://dius.com.au/2016/02/03/microservices-pact/)
* [Simplifying micro-service testing with Pact](http://dius.com.au/2014/05/19/simplifying-micro-service-testing-with-pacts/)
* [更多文章！](https://github.com/realestate-com-au/pact/wiki/News,-blogs,-videos-and-articles)


### 获得帮助
你可以从如下渠道获得关于Pact的相关帮助：

* **Google用户群组：** https://groups.google.com/forum/#!forum/pact-support
* **Gitter:** 加入https://gitter.im/realestate-com-au/pact和https://gitter.im/DiUS/pact-jvm的讨论
* **Twitter:** @pact_up
