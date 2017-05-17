# 匹配

本节描述在`消费者`端测试时可用的不同种类的请求/响应匹配技术。注意，以下演示的例子使用的是Ruby DSL，因为各种实现有所不同，请参考自己所使用的特定语言和框架。

###### 注意

*如果在`消费者`端编写测试时所使用的语言与`提供者`端不同，必须确保两者使用的是共同的Pact规范，否则就无法进行验证。*

*比如，如果你使用的是JS消费者端Pact DSL的v2版本，而提供者端是.NET，那就没法工作了，因为.NET的提供者只支持v1.1版本。这种情况下，你就只能在`消费者`端使用v1.1以保持兼容。*


### 正则表达式 

有时请求或响应中的某些键值是事先无法知道的——比如时间戳或者生成的ID。

你所需要的是能够以某种方式表达“我期望某些东西能够匹配这个正则表达式，但我不关心其实际值具体是多少”。

```ruby
animal_service.given("an alligator named Mary exists").
  upon_receiving("a request for an alligator").
  with(
    method: "get",
    path: "/alligators/Mary",
    headers: {"Accept" => "application/json"}).
  will_respond_with(
    status: 200,
    headers: {"Content-Type" => "application/json"},
    body: {
      name: "Mary",
      dateOfBirth: Pact.term(
        generate: "02/11/2013",
        matcher: /\d{2}\/\d{2}\/\d{4}/)
    })
```

注意`Pact::Term`的用法。当你跑消费者的测试的时候，模拟的服务将会返回你所指定“生成”的值，然后当你在提供者的代码库中校验契约的时候，它可以确保该键值与指定的正则表达式相匹配。

你也可以对请求匹配使用`Pact::Term`。

```ruby
animal_service.given("an alligator named Mary exists").
  upon_receiving("a request for an alligator").
  with(
    method: "get",
    path: "/alligators/Mary",
    query: Pact.term(
      generate: "transactionId=1234",
      matcher: /transactionId=\d{4}/),
  will_respond_with(
    status: 200, ...)
```

`matcher`用于确保实际请求是正确的格式，在`generate`字段中所指定的值将会作为一个向提供者进行重放时所使用的真实值的例子。这意味着提供者状态中仍然可以使用已知的值来创建数据，但是消费者测试中可以互不影响地生成数据。

You can use `Pact::Term` for request and response header values, the request query, and inside request and response bodies. Note that regular expressions can only be used on Strings. Furthermore, request queries, when specified as strings are just matched as normal String - no flexible ordering of parameters is catered for. For flexible ordering, specify it as a Hash, which in turn may include `Pact::Terms`

### Type matching

Often, you will not care what the exact value is at a particular path is, you just care that a value is present and that it is of the expected type. For this scenario, you can use `Pact::SomethingLike`.

```ruby
animal_service.given("an alligator named Mary exists").
  upon_receiving("a request for an alligator").
  with(
    method: "get",
    path: "/alligators/Mary",
    headers: {"Accept" => "application/json"}).
  will_respond_with(
    status: 200,
    headers: {"Content-Type" => "application/json"},
    body: {
      Pact.like(
        name: "Mary",
        age: 73
      )
    })
```

The mock server will return `{"name": "Mary", "age": 73}` in the consumer tests, but when `pact:verify` is run in the provider, it will just check that the type of the `name` value is a String, and that the type of the `age` value is a Fixnum. If you wanted an exact match on "Mary", but to allow any age, you would only wrap the `73` in the `Pact::SomethingLike`.

For request matching, the mock server will allow any values of the same type to be used in the consumer test, but will replay the given values in `pact:verify`.

```ruby
animal_service.given("an alligator named Mary exists").
  upon_receiving("a request to update an alligator").
  with(
    method: "put",
    path: "/alligators/Mary",
    headers: {"Accept" => "application/json"},
    body: {
      age: Pact.like(10)
    }).
  will_respond_with(
    status: 200,
    headers: {"Content-Type" => "application/json"},
    body: {
      age: 10
    })
```

### Query params

Query params can be specified as a string or a hash.
When specified as a string, an exact match will be performed. You may use a Pact::Term, but only over the query string as a whole. Note that the query params must already be URL encoded in the expectation. (This will change for v2 matching.)

```ruby

animal_service.given("some alligators exist").
  upon_receiving("a request to search for alligators").
  with(
    method: "get",
    path: "/alligators",
    query: "name=Mary+Jane&age=8")
  ...

```

Alternatively, if the order of the query parameters does not matter, you can specify the query as a hash. You can embed Pact::Terms or Pact::SomethingLike inside the hash. Remember that all query params will be parsed to strings, so don't use a SomethingLike with a number.

```ruby

animal_service.given("some alligators exist").
  upon_receiving("a request to search for alligators").
  with(
    method: "get",
    path: "/alligators",
    query: {
      # No need to encode params in the hash
      name: 'Mary Jane',
      age: '8',
      # Specify a param with multiple values using an
      # array - order will be enforced
      children: ['Betty', 'Sue']
    })
  ...

```

### Flexible matching
**NOTE:** *Only available in Pact Specification 2.0+*

Flexible length arrays:

```ruby
animal_service.given("an alligator named Mary exists").
  upon_receiving("a request for an alligator").
  with(
    method: "get",
    path: "/alligators/Mary",
    headers: {"Accept" => "application/json"}).
  will_respond_with(
    status: 200,
    headers: {"Content-Type" => "application/json"},
    body: {
      name: "Mary",
      children: each_like(name: "Fred", age: 2)
    })
```

When the provider verification is run, it will ensure that each of the elements in the `children` array has a String name, an Integer age, and that there is at least one element in the array.

To turn the v2 Pact serialisation on, you will need version 1.9.0 of the pact gem, and to configure the `pact_specification_version` in the mock service configuration in the consumer. The provider will pick it up automatically.

```ruby

require 'pact/consumer/rspec'

Pact.service_consumer "Zoo App" do
  has_pact_with "Animal Service" do
    mock_service :animal_service do
      port 1234
      pact_specification_version "2.0.0"
    end
  end
end
```

#### Using v2.0 matching with the Pact Mock Service

To start the standalone pact mock service in v2 matching mode, specify `--pact-specification-version 2.0.0` in the startup options.
