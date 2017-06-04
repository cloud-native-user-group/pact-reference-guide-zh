# Pact和其它语言一起使用

当使用缺乏原生Pact支持的语言来写服务提供者时，你仍然可以使用通用的[Pact提供者端验证工具](https://github.com/pact-foundation/pact-provider-verifier)来验证是否满足契约。

## 通用Pact提供者验证

下面的设置简化了任何语言的Pact提供者端的验证过程。

**特性**:

* 验证发布到[Pact Broker](https://github.com/bethesque/pact_broker)的Pact文件
* 在开发环境验证供测试用的本地Pact`*.json`文件
* 安装有Ruby环境以及[sane](https://www.ruby-toolbox.com/projects/sane)的预先配置的Docker镜像，缺省为`src / Rakefile`，避免重复
* 应当会用到的[提供者端状态](https://github.com/realestate-com-au/pact/wiki/Provider-states)

以下两个解决方案使用[Docker](https://github.com/DiUS/pact-provider-verifier-docker)镜像和[Pact Provider Verifier](https://github.com/pact-foundation/pact-provider-verifier) 的Gem包。对于更高级的用法，你可直接使用[Pact Provider Proxy](https://github.com/bethesque/pact-provider-proxy) Gem，然而在大多数情况下，Pact Provider Verifier应该能够满足你的需求。

### 它是如何工作的

*步骤*:

1. 创建一个API和对应的Docker镜像
2. 将pact文件发布至Pact broker（或者创建本地文件）
3. 启动你的API
4. 运行Pact Provider Verifier
5. 停止你的API

验证工具之后会针对你的运行API重放所有的Pact文件，如果无法满足则会失败(`exit 1`)。
因为没有可用的测试DSL，所以在CI/CD流水线中运行时，你需要注意处理进程的退出代码。

如果你在使用Docker时和Docker compose一起使用，它会帮你自动搞定上面的步骤3-5。
### Docker的例子

以下使用Docker镜像的例子来自[Pact Provider Verifier](https://github.com/DiUS/pact-provider-verifier-docker)项目。
*步骤*:

1. 创建一个API和对应的Docker镜像
2. 将pact文件发布至Pact broker（或者创建本地文件）
3. 创建一个`docker-compose.yml`文件，连接你的API和Pact Verifier
4. 设置下面所需的环境变量：
   * `pact_urls` - 逗号分隔的pac文件URL列表
   * `provider_base_url` - pact提供者(比如你的API)的基本url
5. 运行`docker-compose build`以及`docker-compose up`

##### 一个运行在`4000`端口的Node API的docker-compose.yml例子文件：

```
api:
  build: .
  command: npm start
  expose:
  - "4000:4000"

pactverifier:
  image: dius/pact-provider-verifier-docker
  links:
  - api:api
  volumes:
  - ./pact/pacts:/tmp/pacts                 # If you have local Pacts
  environment:
  - pact_urls=http://pact-host:9292/pacts/provider/MyAPI/consumer/MyConsumer/latest
  #- pact_urls=/tmp/pacts/foo-consumer.json # If you have local Pacts
  - provider_base_url=http://api:4000
```

#### 含有提供者状态的API验证

通过实现以下的内容来对提供者端执行Pact验证：

* 消费者端的一个基于get请求的http接口，它返回契约中的提供者端状态

		{
			"myConsumer": [
				"customer is logged in",
				"customer has a million dollars"
			]
		}

* 一个基于post的http接口，用来设置活跃的pact消费者和提供者状态

		consumer=web&state=customer%20is%20logged%20in

需要以下环境变量：

* `pact_urls` —— 一组由逗号分隔的契约文件URL的列表
* `provider_base_url` - 服务 `提供者` 的基URL
* `provider_states_url` - 由消费者提供的能够返回 `提供者状态` 的完整URL
* `provider_states_active_url` - 用来设置契约相关的 `消费者` and `提供者` 状态的URL

*已更新的docker-compose.yml示例文件：*

	api:
		build: .
		command: npm start
		expose:
		- "4000"

	pactverifier:
		image: dius/pact-provider-verifier-docker
		links:
		- api
		environment:
		- pact_urls=http://pact-host:9292/pacts/provider/MyProvider/consumer/myConsumer/latest
		- provider_base_url=http://api:4000
		- provider_states_url=http://api:4000/provider-states
		- provider_states_active_url=http://api:4000/provider-states/active


### Ruby示例

如果你没有使用Docker，那么就需要：

* 安装Ruby运行时环境
* fork或者clone[代码库](https://github.com/DiUS/pact-provider-verifier-docker)或者将脚本拷贝至你的工程中
* 运行下面的命令：

```
bundle install
bundle exec rake verify_pacts
```
