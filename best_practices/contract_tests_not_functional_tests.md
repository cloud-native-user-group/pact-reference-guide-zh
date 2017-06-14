# 契约测试与功能测试之间的区别（又名怎样做契约测试才不会陷入令人抓狂的误区）


对于刚开始正式接触契约测试的团队，对于契约测试和功能测试之间的区别经常会存在争论。难就难在其实并不存在什么非黑即白的答案，更为重要的应该是逐渐加深对契约测试的理解和使用。

使用契约测试常见的场景，是验证规则或者处理错误请求。例如，我们有一个*用户服务*，允许消费者使用POST请求注册新用户，并在HTTP Body中包含所创建用户的详细信息。

一个简单的消费者与提供者交互的常规场景如下所示：
```
Given "there is no user called Mary"
When "creating a user with username Mary"
  POST /users { "username": "mary", email: "...", ... }
Then
  Expected Response is 200 OK
```

如果只验证常规场景，则会遗漏不同响应的其他场景，并可能使消费者误解提供者的行为。所以，我们还需要验证失败的场景：

```
Given "there is already a user called Mary"
When "creating a user with username Mary"
  POST /users { "username": "mary", email: "...", ... }
Then
  Expected Response is 409 Conflict
```

可以看出，通过使用不同的响应代码，我们覆盖了一个新的错误场景验证。

接下来，*用户服务*的团队告诉我们，用户名只允许包含字母，其最大长度不能超过20个字符，空白的用户名是无效的。那我们是不是应该在契约测试中加点东西呢？

这会成为连锁反应的开始，很可能会向契约测试中添加3个场景，类似于这样：
```
When "creating a user with a blank username"
  POST /users { "username": "", email: "...", ... }
Then
  Expected Response is 400 Bad Request
  Expected Response body is { "error": "username cannot be blank" }
```

```
When "creating a user with a username with 21 characters"
  POST /users { "username": "thisisalooongusername", email: "...", ... }
Then
  Expected Response is 400 Bad Request
  Expected Response body is { "error": "username cannot be more than 20 characters" }
```

```
When "creating a user with a username containing numbers"
  POST /users { "username": "us3rn4me", email: "...", ... }
Then
  Expected Response is 400 Bad Request
  Expected Response body is { "error": "username can only contain letters" }
```

到这一步为止我们其实已经滥用了契约测试，我们实际上是在测试*用户服务*是否正确实现了校验规则：而这实际上是功能测试，应该在*用户服务*自己的代码库中所覆盖。

可是这有什么坏处呢……多测试些东西，不是挺好么？问题在于这些场景被过度设计，从而产生了不必要的紧耦合契约。如果*用户服务*团队更改主意，觉得20个字符对用户名来说过于严格，并将这个限制增加到50个字符，该如何处理？如果用户名允许使用数字了，又该如何处理？理论上任何消费者都不应受到任何这类变更的影响，但不幸的是，仅仅放宽了一点验证规则，*用户服务*就会破坏Pact契约。这其实并不是什么破坏性的修改，但是由于我们对场景的过度设计，阻碍了*用户服务*团队对新需求的实现。

所以，回到之前的场景，我们换成只是验证*用户服务*对输入错误的响应：
```
Given that username "bad_username" is invalid
When "creating a user with an invalid username"
  POST /users { "username": "bad_username", ... }
Then
  Response is 400 Bad Request
  Response body is { "error": "<any string>" }
```

只做了一点修改，但看起来更灵活了！现在*用户服务*的团队可以改变他们的验证规则，而不会破坏我们的契约……

我们其实并不关心用户名最大长度是多少，或允许使用什么类型的字符，我们只关心如果消费者发送的请求不合法时，*用户服务*是怎样响应的。

在编写契约测试的交互时，应该明确测试想要覆盖的内容。契约能够帮助我们捕捉到以下这些问题：
- 消费者中的缺陷
- 从消费者角度对于接口使用的理解是否存在偏差
- 提供者对接口的破坏性变更
简而言之，契约场景不应该深入到提供者的业务逻辑细节，而应该专注于验证消费者和提供者是否对请求和响应达成共识。在我们的示例中，所写的场景更关注于验证是*怎样*失败的，而不是验证*为何*失败。