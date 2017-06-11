# 契约测试与功能测试之间的区别

> The difference between contract tests and functional tests (or How To Do Pacts In a Way That Won't Make You Want To Stab Your Eyes Out)


对于刚开始接触契约测试的团队，经常会在契约测试和功能测试间的使用上存在分歧。它的挑战在于，它不是一种非黑即白的场景，而是应该在契约测试上，逐渐完成越来越多的验证。

> The difference between contract tests and functional tests is a debate that seems to surface quite often for teams who start investing seriously into contracts testing. The challenge is that it's not a black and white kind of situation, but more something that starts creeping up on the depth of contract testing.

使用契约测试常见的场景，是验证规则或者处理错误请求。
> One place where it can be common is around validation rules and rejected requests.

例如，我们有一个**用户服务**，允许消费者使用POST请求注册新用户，并在HTTP Body中包含创建用户的详细信息。
> For example, we might have a simple *User Service* that allows Consumers to register new users, typically with a POST request containing the details of the created user in the body.

消费者与提供者交互的常规场景如下所示：
> A simple happy-path scenario for that interaction might look like:

```
Given "there is no user called Mary"
When "creating a user with username Mary"
  POST /users { "username": "mary", email: "...", ... }
Then
  Expected Response is 200 OK
```

如果只验证常规场景，则会遗漏不同响应的其他场景，并可能使消费者误解提供者的行为。所以，我们还需要验证失败的场景：
> Sticking to happy-paths is a risk of missing different response codes and potentially having the consumer misunderstand the way the provider behaves. So, let's look at a failure scenario:


```
Given "there is already a user called Mary"
When "creating a user with username Mary"
  POST /users { "username": "mary", email: "...", ... }
Then
  Expected Response is 409 Conflict
```

可以看出，通过使用不同的响应代码，我们覆盖了一个新的错误场景验证。
> So far so good, we're covering a new behaviour, with a different response code.

接下来，**用户服务**的团队告诉我们，用户名只允许包含字母，其最大长度不能超过20个字符，空白的用户名是无效的。那我们如何添加这些功能对应的契约测试呢？

> Now we've been talking to the Team managing the *User Service* and they tell us that username have a maximum length of 20 characters, also they only allow letters in the username and a blank username is obviously not valid. Maybe that's something we should add in our contract?

如下所示，我们添加其他三个场景的验证：
> This is where we get on the slippery slope... it's very tempting to now add 3 scenarios to our contract, something like:

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

通过实现这几个场景的契约测试，我们验证**用户服务**是否实现了需求所描述的规则。但实际上，这是功能测试，它应该由**用户服务**在其代码库中覆盖。

> We've gone past the contract testing as this point, we're actually testing that the *User Service* implements the validation rules correctly: this is functional testing, and it should be covered by the *User Service* in its own codebase.

什么，通过实现更多的用例，保障行为交互的正确性，这样不对吗.....？
> What is the harm in this... more testing is good, right? 

问题是当前场景太特殊了，于是造成了不必要的高度耦合。如果**用户服务**的团队更改主意，20个字符对用户名来说不合理，并将其增加到50个字符，如何处理？如果用户名允许使用数字，如何处理？消费者不应受到这些变更的影响，但实际上，基于松耦合的验证规则，**用户服务**将破坏Pact契约。通过过度使用松耦合验证，我们阻碍了**用户服务**的新需求实现。
> The issue here is that these scenarios are going too far and create an unnecessarily tight contract - what if the *User Service* Team decides that actually 20 characters is too restrictive for username and increases it to 50 characters? What if now numbers are allowed in the username? Any Consumer should be unaffected by any of these changes, unfortunately the *Users Service* will break our Pact just by loosening the validation rules. These are not breaking changes, but by over-specifying our scenarios we are stopping the *User Service* Team from implementing them.

所以，回到之前的场景，只是验证**用户服务**对错误输入的响应：
> So let's go back to our scenarios and instead only test the way the *User Service* reacts to bad input:

```
Given that username "bad_username" is invalid
When "creating a user with an invalid username"
  POST /users { "username": "bad_username", ... }
Then
  Response is 400 Bad Request
  Response body is { "error": "<any string>" }
```

不错，看起来更灵活了！现在**用户服务**的团队可以改变他们的验证规则，而不会破坏我们的契约...
> Subtle, but so much more flexible! Now the *User Service* Team can change their validation rules without breaking the Pact we give them...

我们并不在乎用户名最大长度限制，或使用什么类型的字符。实际上，我们只关心，如果消费者发送的请求不合理，**用户服务**对请求的响应方式。
> we don't really care what's the maximum length of the username or what type of characters are allowed, we only care that if we send something wrong, then we understand the way the *User Service* responds to us.

在编写契约测试时，明确想要覆盖的内容是非常关键的。契约能够帮助我们捕获到这些变化：
> When writing a test for an interaction, ask yourself what you are trying to cover. Contracts should be about catching:

- 消费者中的缺陷
- 消费者对提供者响应的使用是否合理
- 提供者对契约的变更或者破坏
>- bugs in the consumer
>- misunderstanding from the consumer about end-points or payload
>- breaking changes by the provider on end-points or payload

简而言之，“契约”的场景不应该考虑到具体的业务逻辑，而应坚持验证消费者和提供者是否对请求和响应达成共识。
> In short, your Pact scenarios should not dig into the business logic of the Provider but should stick with verifying that Consumer and Provider have a shared understanding of what requests and responses will be. 

在我们的示例中，关注**验证失败的处理，而不是为什么**验证失败。
> In our example of validation, write scenarios about *how* the validation fails, not *why* the validation fails.
