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

`matcher`用于确保实际请求是正确的格式，在`generate`字段中所指定的值将会作为一个向提供者进行重放时所使用的真实值的例子。这意味着提供者状态中仍然可以使用已知的值来创建数据，但是消费者测试中可以不受影响地生成数据。

你可以将`Pact::Term`用于请求和响应头信息、请求查询以及请求和响应体内部。注意，正则表达式只能用于字符串。更进一步地，当请求查询被指定为字符串时，它只是被当做普通字符串来做匹配的——即不会考虑参数顺序的灵活匹配。要想实现顺序灵活匹配，需指定为哈希，哈希中轮流包含`Pact::Terms`。

### 类型匹配

通常，你并不关注特定路径下的某个具体值是多少，你所关注的只是有这个值并且该值是所期望的类型。这种场景下，可以使用`Pact::SomethingLike`。

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

消费者测试中的模拟服务将会返回`{"name": "Mary", "age": 73}` ，但是当在提供者端执行`pact:verify`时，只会检查`name`的值是否是字符串，以及`age`的值是否是整数。如果你想精确匹配“Mary”，但是又允许age为任何值，应该在 `Pact::SomethingLike`中只包含进`73`。

对于请求匹配，模拟服务允许在消费者测试中使用任何类型的请求值，只要是同一类型即可，但是在`pact:verify`中将使用给定值进行重放。

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

### 查询参数

查询参数可以指定为字符串或者哈希。

当指定为字符串时，将会执行精确匹配。你可以使用Pact::Term，但是只能将查询字符串作为一个整体进行匹配。注意，查询参数在期望中应该是已被编码过的URL。（这一功能在v2版本的匹配中将会被修改。）

```ruby

animal_service.given("some alligators exist").
  upon_receiving("a request to search for alligators").
  with(
    method: "get",
    path: "/alligators",
    query: "name=Mary+Jane&age=8")
  ...

```

或者，如果查询参数的顺序并不重要，也可以将查询指定为哈希。你可以将Pact::Terms或Pact::SomethingLike嵌入到哈希内部去。请记住所有查询参数将会被解析为字符串，所以不要用SomethingLike去匹配数字类型。

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

### 灵活匹配
**注意：** *仅对于Pact规范2.0及以上适用*

长度灵活的数组：

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

当提供者端的校验运行时，将会确保`children`数组中的每个元素都有一个字符串类型的name、一个整数类型的age，以及数组中至少包含一个元素。

要打开v2版本Pact的序列化功能，你需要1.9.0版本的pact gem包，并且在消费者端的模拟服务配置中对`pact_specification_version`进行配置。提供者端将会自动获取到它。

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

#### 在Pact模拟服务中使用v2.0的匹配

要以v2匹配模式启动独立的pact模拟服务，只需在启动项中指定`--pact-specification-version 2.0.0`。