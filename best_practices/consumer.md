# Consumer Best Practices
# 消费者端使用Pact的最佳实践

### Use Pact for contract testing, not functional testing of the provider
### 使用Pact作为契约测试，而非提供者端的功能测试

* Functional testing is about ensuring the provider does the right thing with a request. These tests belong in the provider codebase, and it's not the job of the consumer team to be writing them.
* 功能测试是确保提供者端按照消费者需求提供正确的响应。这些测试属于提供者代码库，消费者团队的工作不在于编写它们。

* Contract testing is about making sure your consumer team and provider team have a shared understanding of what the requests and responses will be.
Pact tests should focus on.

* 契约测试能帮助消费者端的团队和提供者端的团队对请求和响应的有共同的了解。

* exposing bugs in how the consumer creates the requests or handles responses
exposing misunderstandings about how the provider behaves
Pact tests should not focus on.



* exposing bugs in the provider (though this might come up as a by product)
You can read more about the difference between contract and functional tests here.

* The rule of thumb for working out what to test or not test is - if I don't include this scenario, what bug in the consumer or what misunderstanding about how the provider behaves might be missed. If the answer is none, don't include it.



### Use Pact for isolated (unit) tests
### 使用`Pact`进行隔离的（单元）测试

* as a mock (calls to mocks are verified after a test) not a stub (calls to stubs are not verified). Using `Pact` as a stub defeats the purpose of using `Pacts`.
* 作为Stub（调用stubs不会被验证），而不是Mock（调用mock后，在测试中会对行为验证），消费者端将使用`Pact`作为Stub，帮助其获取提供者端的响应。

* for *isolated tests* (ie. unit tests) of the class(es) that will be responsible for making the HTTP calls from your `Consumer` application to your `Provider` application, not for integrated tests of your entire consumer codebase.

* 所谓隔离的单元测试，Pact将使用单元测试的方式将*消费者端*的HTTP请求发送到*提供者端*，而不需要启动整个消费者与提供者，进行集成测试

* *carefully*, for any sort of functional or integrated tests within your consumer codebase.

* 对于消费者代码库中的集成测试，应该谨慎对待。

**为什么？**

如果您以传统的集成测试思维使用*Pact*。你会发现消费者端的测试非常脆弱，因为`Pact`可以检查每个响应路径，JSON节点，查询参数和或者HTTP头。
同时，你还需要在*提供者端*进行大量笛卡儿结果的验证。这将大大增加*提供者端*验的的时间，却对测试覆盖率没有效率的提升。


### Think carefully about how you use it for non-isolated tests (functional, integration tests)

* Keep your isolated, exact match tests. These will make sure that you’re mapping the right data from your domain objects into your requests.
* For the integration tests, use loose, type based matching for the requests to avoid brittleness, and pull out the setup into a method that can be shared between tests so that you do not end up with a million interactions to verify (this will help because the interactions collection in the `Pact` acts like a set, and discards exact duplicates).

If you don’t care about verifying your interactions, you could use something like Webmock for your integrated tests, and use shared fixtures for requests/responses between these tests and the `Pact` tests to ensure that you have some level of verification happening.

###仔细考虑如何进行非隔离测试（功能测试、集成测试等）

* 保持独立的、完全匹配的测试。这些将确保将域对象中的数据映射到请求中。

* 对于集成测试，为了避免其脆弱性，应该对请求使用松散的基于类型的匹配，并将公共的配置部分抽象出来，以便在测试用例之间共享。这样，也会避免大量的交互测试（这将有助于*Pact*中不需要收集）。

如果你不关心交互行为的验证，可以使用像Webmock这样的工具进行集成测试，并使用共享的测试套件进行测试和“Pact”测试之间的请求/响应，以确保您有一定程度的验证发生。

### Make the latest pact available to the `Provider` via a URL

See [Sharing pacts between `Consumer` and `Provider`](https://github.com/realestate-com-au/pact/wiki/Sharing-pacts-between-consumer-and-provider) for options to implement this.

### Ensure all calls to the `Provider` go through classes that have been tested with `Pact`

Do not hand create any HTTP requests directly in your `Consumer` app. Testing through a client class (a class with the sole responsibility of handling the HTTP interactions with the `Provider`) gives you much more assurance that your `Consumer` app will be creating the HTTP requests that you think it should.

### Ensure the models you use in other tests could actually be created from the responses you expect

Sure, you’ve checked that your client deserialises the HTTP response into the Alligator you expect, but then you need to make sure when you create an Alligator another test, that you create it with valid attributes (eg. is the Alligator’s last_login_time a Time or a DateTime?). One way to do this is to use factories or fixtures to create the models for all your tests. See this gist for a more detailed explanation.

### Beware of Garbage In, Garbage Out with PUT/POST/PATCH

Each interaction is tested in isolation, meaning you can’t do a PUT/POST/PATCH, and then follow it with a GET to ensure that the values you sent were actually read successfully by the `Provider`. For example, if you have an optional `surname` field, and you send `lastname` instead, a `Provider` will most likely ignore the misnamed field, and return a 200, failing to alert you to the fact that your `lastname` has gone to the big `/dev/null` in the sky.

To ensure you don’t have a Garbage In Garbage Out situation, expect the response body to contain the newly updated values of the resource, and all will be well.

If, for performance reasons, you don’t want to include the updated resource in the response, another way to avoid GIGO is to use a shared fixture between a GET response body, and a PUT/POST request body. That way, you know that the fields you are PUTing or POSTing are the same fields that you will be GETing.
