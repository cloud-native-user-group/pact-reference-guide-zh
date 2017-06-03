# 将Pact文件发布到Pact Broker

更多关于Broker的细节可以查看[Pact Broker](http://docs.pact.io/documentation/sharings_pacts.html)文档以及这篇如何使用的[文章](http://rea.tech/enter-the-pact-matrix-or-how-to-decouple-the-release-cycles-of-your-microservices/)。

## 用Go代码发布

```go
pact.PublishPacts(&types.PublishRequest{
	PactBroker:             "http://pactbroker:8000",
	PactURLs:               []string{"./pacts/my_consumer-my_provider.json"},
	ConsumerVersion:        "1.0.0",
	Tags:                   []string{"latest", "dev"},
})
```

## 用CLI发布

使用下面的curl命令发起PUT请求将pact文件放到正确的位置，指定你的消费者名字、提供者名字和消费者的版本。

```
curl -v -XPUT \-H "Content-Type: application/json" \
-d@spec/pacts/a_consumer-a_provider.json \
http://your-pact-broker/pacts/provider/A%20Provider/consumer/A%20Consumer/version/1.0.0
```

### 使用Pact Broker进行基本身份验证

The following flags are required to use basic authentication when
publishing or retrieving Pact files to/from a Pact Broker:

从Pact Broker发布或获取Pact文件的基本身份验证需要以下标志位：

* `BrokerUsername` - Pact Broker基本身份验证的用户名
* `BrokerPassword` - Pact Broker基本身份验证的密码
