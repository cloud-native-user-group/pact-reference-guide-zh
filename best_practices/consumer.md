# 消费者(Consumer)使用Pact的最佳实践

### 将`Pact`用于消费者与提供者的契约测试，而不是对提供者的功能测试

* 功能测试是确保提供者在某个请求下执行正确的动作。这些测试代码属于提供者团队，不应该由消费者团队完成。
* 而契约测试的目的是确保消费者团队和提供者团队对请求和响应达成共识。
* Pact测试应该关注于：
  * 检查消费者如何构建请求以及处理响应时所暴露出的bug
  * 检查提供者的行为，消除理解上的偏差
* Pact测试不应该关注于：
  * 提供者内部所暴露的bug（尽管有可能作为副产品产生）

对于提供者的功能测试和契约测试之间的区别，更多讨论见[后文](contract_tests_not_functional_tests.html)。

**关于Pact测试中测试范畴的取舍，经验法则是：如果在当前场景下不包括这部分测试内容，会引起消费者的什么bug，或者关于提供者的行为会出现怎样的理解偏差。如果答案是不会有问题，那么就不需要囊括到Pact测试的范畴中来。**

### 使用`Pact`进行隔离的（单元）测试

* mock与stub是有区别的：mock会在测试中对调用它的请求进行验证，而stub对调用它的请求不作验证。使用`Pact`作为Stub，获取期望的响应。
> as a mock (calls to mocks are verified after a test) not a stub (calls to stubs are not verified). Using `Pact` as a stub defeats the purpose of using `Pacts`.

* 所谓隔离的测试（如单元测试），是指只对负责从*消费者*发送HTTP请求的类进行测试，而不是对整个消费者代码库进行完整的集成测试。



* 对于消费者代码库中任何类型的功能测试或集成测试，应该谨慎对待。
**为什么？**

* 如果以传统的集成测试思维使用`Pact`，会将自己陷入泥潭。你的`消费者`测试将非常脆弱，因为需要使用`Pact`检查每个响应路径，JSON节点，查询参数和请求头。同时，你还会发现，`提供者`端的待验证用例数量将会呈笛卡儿乘积式地爆炸增长。这将大大增加`提供者`运行验证的时间，却难以有效提升测试覆盖率。
### 仔细考虑如何使用它进行非隔离测试（功能测试、集成测试）
> Think carefully about how you use it for non-isolated tests (functional, integration tests)

* 保持相互隔离的、精确匹配的验证。这将确保将域对象中的数据映射到请求中。
> Keep your isolated, exact match tests. These will make sure that you’re mapping the right data from your domain objects into your requests.

* 对于集成测试，为了避免用例的脆弱性，尽量使用松散的、基于类型匹配(而非基于数值)的用例。同时，将其放置在多个测试用例间进行共享，可以极大的减少用例的次数。
  (这将有助于构建“契约测试”中的交互验证集合，避免重复项)。
> For the integration tests, use loose, type based matching for the requests to avoid brittleness, and pull out the setup into a method that can be shared between tests so that you do not end up with a million interactions to verify (this will help because the interactions collection in the `Pact` acts like a set, and discards exact duplicates).

* 如果您不关心验证消费者与提供者之间的交互，那可以使用类似Webmock的工具实现集成测试，并使用共享的测试夹具，在集成测试和Pact测试间构建请求/响应。
> If you don’t care about verifying your interactions, you could use something like Webmock for your integrated tests, and use shared fixtures for requests/responses between these tests and the `Pact` tests to ensure that you have some level of verification happening.


### 通过URL提供最新的pact契约访问地址

* 关于如何实现的具体方法，请参考[消费者与提供者之间的契约共享](https://github.com/realestate-com-au/pact/wiki/Sharing-pacts-between-consumer-and-provider)。

### 确保所有对`提供者`的调用都是经过`Pact`测试的类所发送的

* 不要在`消费者`应用中直接创建HTTP请求。应该通过客户端类（一个负责处理与“提供者”进行交互的单一职责的类）进行测试，这样会让你更有把握，你的`消费者`应用是按照所设想的方式来创建HTTP请求的。
###确保可以根据期望的响应，创建测试中使用的模型
> Ensure the models you use in other tests could actually be created from the responses you expect

* 你已经确信客户端(客户端)能够将HTTP响应反序列化，但需要确保使用有效的属性创建测试的模型时。例如，模型的last_login_time或者Time、DateTime等）。一种可选的方法是使用工厂或测试套具创建所有模型。请参阅相关信息获得更详细的解释。
> Sure, you’ve checked that your client deserialises the HTTP response into the Alligator you expect, but then you need to make sure when you create an Alligator another test, that you create it with valid attributes (eg. is the Alligator’s last_login_time a Time or a DateTime?). One way to do this is to use factories or fixtures to create the models for all your tests. See this gist for a more detailed explanation.

### 当心无效的输入、输出与PUT/POST/PATCH
> Beware of Garbage In, Garbage Out with PUT/POST/PATCH

* 每次交互都被隔离地测试，这意味着你无法先使用PUT/POST/PATCH更新资源，再使用GET获取资源。例如，如果有一个可选的`surname`字段，但你发送的`lastname`，那么提供者很可能会忽略这个错误的字段，并返回一个200，但没有任何信号，
> Each interaction is tested in isolation, meaning you can’t do a PUT/POST/PATCH, and then follow it with a GET to ensure that the values you sent were actually read successfully by the `Provider`. For example, if you have an optional `surname` field, and you send `lastname` instead, a `Provider` will most likely ignore the misnamed field, and return a 200, failing to alert you to the fact that your `lastname` has gone to the big `/dev/null` in the sky.


* 为了避免这种无效的输入、输出的情况，可以在期望的响应体中包含新更新后的资源。
> To ensure you don’t have a Garbage In Garbage Out situation, expect the response body to contain the newly updated values of the resource, and all will be well.

* 但有时候出于性能考虑，不希望在响应中包含更新的资源，另一种避免GIGO的方法是在GET响应体和PUT/POST请求体之间使用共享的测试套件。这样，你就明确输出的字段是将要访问的字段。
> If, for performance reasons, you don’t want to include the updated resource in the response, another way to avoid GIGO is to use a shared fixture between a GET response body, and a PUT/POST request body. That way, you know that the fields you are PUTing or POSTing are the same fields that you will be GETing.
