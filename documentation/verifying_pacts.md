# 验证契约

“验证契约”是Pact测试过程中的第二步。契约文件中的每个请求会在提供者上进行重放，所返回的响应将会被用于与契约文件中的期望响应进行对比，如果两者匹配，我们就可以确信消费者和提供者能够保持兼容。

要验证契约，应该这样做：

1. 配置待验证契约的位置。可以是一个HTTP URL，也可以是一个本地文件系统路径。
2. 在[提供者状态](./provider_states.md)中预置数据。
3. （可选项）对将被用于播放请求的服务提供者应用进行配置。

关于如何在代码中玩转这些，请参考[Ruby例程](ruby/verifying_pacts.md)。

## 使用非Pact原生的语言？

如果对你使用的语言还未提供原生的验证支持，你仍然可以验证提供者API！参见命令行工具 [提供者验证器](https://github.com/DiUS/pact-provider-verifier-docker)。

### 使用Docker？

参见pact提供者验证器的[Docker镜像](https://hub.docker.com/r/dius/pact-provider-verifier-docker/)。