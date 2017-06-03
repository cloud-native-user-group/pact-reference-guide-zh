# 消费者端(Consumer)使用Pact的最佳实践
> Consumer Best Practices

### Pact主要用于消费者与提供者的契约测试，而非对提供者(Provider)进行的功能测试

> Use Pact for contract testing, not functional testing of the > provider


* 功能测试是确保提供者按照消费者的需求提供期望的响应。这些测试代码属于提供者团队，不应该由消费者团队完成。

> Functional testing is about ensuring the provider does the right thing with a request. These tests belong in the provider codebase, and it's not the job of the consumer team to be writing them.


* 契约测试能帮助消费者团队和提供者团队对请求和响应达成共识。

> Contract testing is about making sure your consumer team and provider team have a shared understanding of what the requests and responses will be. Pact tests should focus on.


* 契约测试不必关注消费者如何构建请求或提供者如何处理响应的实现。
> exposing bugs in how the consumer creates the requests or handles responses exposing misunderstandings about how the provider behaves
Pact tests should not focus on.


* 对于提供者的功能测试和契约测试，二者的侧重点不相同，后面的部分我们会详细讨论。
> exposing bugs in the provider (though this might come up as a by product) You can read more about the difference between contract and functional tests here.


* 是否需要契约测试的经验是：如果没有契约测试，当消费者与提供者协作出现缺陷，会导致什么意外。如果意外可以接受，那就不需要契约测试 :)

> The rule of thumb for working out what to test or not test is - if I don't include this scenario, what bug in the consumer or what misunderstanding about how the provider behaves might be missed. If the answer is none, don't include it.

### Use Pact for isolated (unit) tests
### 使用`Pact`进行隔离的（单元）测试

* Stub对请求者提供响应，但不做行为的验证；Mock对请求者提供响应，但会对响应过程验证。对消费者而言，使用`Pact`作为Stub，获取期望的响应。

> as a mock (calls to mocks are verified after a test) not a stub (calls to stubs are not verified). Using `Pact` as a stub defeats the purpose of using `Pacts`.

* 所谓隔离的单元测试，是指使用单元测试的方式将*消费者*的HTTP请求发送到*提供者*，但并需要完全运行消费者与提供者

> for *isolated tests* (ie. unit tests) of the class(es) that will be responsible for making the HTTP calls from your `Consumer` application to your `Provider` application, not for integrated tests of your entire consumer codebase.


* 对于消费者代码库中的集成测试，应该谨慎对待。
> *carefully*, for any sort of functional or integrated tests within your consumer codebase.


**为什么？**

* 如果以传统的集成测试思维使用*Pact*。你会发现消费者端的测试非常繁琐，因为你会使用`Pact`检查每个响应路径，JSON节点，查询参数和或HTTP头。
同时，你还要在*提供者*端进行大量笛卡儿乘积结果的验证。这将大大增加*提供者*运行验证的时间，但却提升测试的效率及覆盖率。

> If you use Pact with exact matching for integrated tests, you will drive yourself nuts. You will have very brittle Consumer tests, as Pact checks every outgoing path, JSON node, query param and header. You will also end up with a cartesian explosion of interactions that need to be verified on the Provider side. This will increase the amount of time you spend getting your Provider tests to pass, without usefully increasing the amount of test coverage.




### 仔细考虑如何使用它进行非隔离测试（功能，集成测试）
> Think carefully about how you use it for non-isolated tests (functional, integration tests)


* 保持隔离的，精确匹配的验证。这将确保将域对象中的数据映射到请求中。

> Keep your isolated, exact match tests. These will make sure that you’re mapping the right data from your domain objects into your requests.


* 对于集成测试，为了避免用例的脆弱性，尽量使用松散的、基于类型匹配(而非基于数值)的用例。同时，将其放置在多个测试用例间进行共享，可以极大的减少用例的次数。
(这将有助于构建“契约测试”中的交互验证集合，避免重复项)。

> For the integration tests, use loose, type based matching for the requests to avoid brittleness, and pull out the setup into a method that can be shared between tests so that you do not end up with a million interactions to verify (this will help because the interactions collection in the `Pact` acts like a set, and discards exact duplicates).


* 如果您不关心验证消费者与提供者之间的交互，那可以使用Webmock类似的工具实现集成测试，并使用共享的测试套件，在集成测试和Pact测试间构建请求/响应。

> If you don’t care about verifying your interactions, you could use something like Webmock for your integrated tests, and use shared fixtures for requests/responses between these tests and the `Pact` tests to ensure that you have some level of verification happening.


### 通过URL，提供最新的pact契约
> Make the latest pact available to the `Provider` via a URL

参考[消费者与提供者]之间的共享协议（https://github.com/realestate-com-au/pact/wiki/Sharing-pacts-between-consumer-and-provider）。

> See [Sharing pacts between `Consumer` and `Provider`](https://github.com/realestate-com-au/pact/wiki/Sharing-pacts-between-consumer-and-provider) for options to implement this.


### 通过使用了“Pact”测试类，确保对提供者的调用
> Ensure all calls to the `Provider` go through classes that have been tested with `Pact`

* 不要在“消费者”的应用中直接创建任何HTTP请求。通过客户端类（一个负责处理与“提供者”交互的类）进行测试

> Do not hand create any HTTP requests directly in your `Consumer` app. Testing through a client class (a class with the sole responsibility of handling the HTTP interactions with the `Provider`) gives you much more assurance that your `Consumer` app will be creating the HTTP requests that you think it should.

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
