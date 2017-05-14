# 开始

### Pact是怎样工作的？

1. 在提供者所面向的消费者项目代码中编写测试，期望响应被设置在模拟的服务提供者上。
2. 在测试运行时，模拟的服务将返回所期望的响应。请求和所期望的响应将会被写入到一个“pact”文件中。
3. pact文件中的请求随后在提供者上进行重放，并检查实际响应以确保其与所期望响应相匹配。

![](https://raw.githubusercontent.com/pact-foundation/pact-foundation.github.io/test/media/pact.png)
