# 共享契约

在你的Pact之旅上，一定有些时候会生成许多`Pact`文件供`提供者`进行验证，而且要确保`提供者`访问的总是最新版本的契约文件。这当然很棒，但是很快就会繁琐到难于管理。

## Pact Broker

这时就该[Pact Broker](https://github.com/bethesque/pact_broker)登场了。它能够让你在项目之间共享`契约`，还可以让这些契约为人所用。它是正式使用Pact开发时的推荐方式，具有如下特性：

* 自动生成接口文档
* 动态生成调用关系网络图
* 对`Pact`打标签的能力 (譬如 "prod")，这样`提供者`就可以针对契约的某个固定版本进行验证，以保持向后兼容性。
* `Pact`发布时可用于触发提供者端构建的Webhooks。
* 提供对你的`消费者`和`提供者`发布周期之间的解耦能力
* 提供交叉测试head/prod版本的能力

感谢我们的赞助者[DiUS](https://www.dius.com.au/)提供了一个[免费托管的broker](https://pact.dius.com.au/)供你快速上手。

### 支持的语言

**Ruby** 通过`pact_broker-client` gem支持：

* [发布契约](https://github.com/bethesque/pact_broker-client#consumer)
* [提供者端验证](https://github.com/bethesque/pact_broker-client#provider)

** Pact JVM - Gradle **

* [发布契约](https://github.com/DiUS/pact-jvm/tree/master/pact-jvm-provider-gradle#publishing-pact-files-to-a-pact-broker-version-227)
* [提供者端验证](https://github.com/DiUS/pact-jvm/tree/master/pact-jvm-provider-gradle#verifying-pact-files-from-a-pact-broker-version-311231)

** Pact JVM - JUnit **

* [提供者端验证](https://github.com/DiUS/pact-jvm/tree/master/pact-jvm-provider-junit#download-pacts-from-a-pact-broker)

** Pact JVM - SBT **

* [提供者端验证](https://github.com/DiUS/pact-jvm/tree/master/pact-jvm-provider-sbt#verifying-pact-files-from-a-pact-broker)

** Pact JVM - Maven **

* [发布契约](https://github.com/DiUS/pact-jvm/tree/master/pact-jvm-provider-maven#publishing-pact-files-to-a-pact-broker-version-320)
* [提供者端验证](https://github.com/DiUS/pact-jvm/tree/master/pact-jvm-provider-maven#verifying-pact-files-from-a-pact-broker-version-311231)

** Golang **

* [发布契约](https://github.com/pact-foundation/pact-go/#publishing-pacts-to-a-broker-and-tagging-pacts)
* [提供者端验证](https://github.com/pact-foundation/pact-go/#provider)

** Pact JS **

* [发布契约](https://github.com/pact-foundation/pact-js/#publishing-pacts-to-a-broker)
* [提供者端验证](https://github.com/pact-foundation/pact-js/#provider-api-testing)

** Pact .NET **

* [发布契约](https://docs.pact.io/documentation/sharings_pacts.html)
* [提供者端验证](https://github.com/SEEK-Jobs/pact-net#service-provider)

** 通过HTTP手动操作 **

* 参见[API定义](https://github.com/bethesque/pact_broker/wiki/Publishing-and-retrieving-pacts)中的`cURL`例子

更多信息，请前往Pact Broker[站点](https://github.com/bethesque/pact_broker)。

## 另外的备选方法

##### 1. `消费者`的CI构建时将契约提交到`提供者`的代码库中

无须赘述。

##### 2. 将契约发布为CI构建产物

找出最近一次成功构建所创建契约的URL地址，在pact:verify任务中将配置指向该URL。

##### 3. 使用Github/Bitbucket URL

这只适用于不需要授权认证的仓库。在对pact测试做了任何修改并提交之前，确保你总会重新生成一份契约文件，并且提交之前测试总是通过的，因为你可不想在失败的构建基础上验证契约。

##### 4. 将契约推送到Amazon S3

[Pact::Retreaty](https://github.com/fairfaxmedia/pact-retreaty)工具提供了一套极轻量级的机制，可以将pact契约推送到S3和从S3上拉取契约。
