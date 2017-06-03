# 用其它语言使用Pact

对于缺乏`提供者`原生Pact支持的语言，你仍然可以使用通用的[Pact 提供者端验证工具](https://github.com/pact-foundation/pact-provider-verifier)来验证是否满足它们的Pact。

## 通用的Pact提供者验证

下面的设置简化了任何语言的Pact提供者端的验证过程。

**特性**:

* 验证发布到[Pact Broker](https://github.com/bethesque/pact_broker)的Pact文件
* 在开发环境验证供测试用的本地Pact`*.json`文件
* Pre-configured Docker image with Ruby installed and a sane, default `src/Rakefile` keeping things DRY
* 安装有Ruby环境以及[sane](https://www.ruby-toolbox.com/projects/sane)的预先配置的Docker镜像，缺省为`src / Rakefile`避免重复
* 应当需要使用[提供者端状态](https://github.com/realestate-com-au/pact/wiki/Provider-states)

下面的两个解决方案使用[Docker](https://github.com/DiUS/pact-provider-verifier-docker)镜像和[Pact Provider Verifier](https://github.com/pact-foundation/pact-provider-verifier) Gem。更高级的用法，你可以直接使用[Pact Provider Proxy](https://github.com/bethesque/pact-provider-proxy) Gem，然而，在大多数情况下，Pact Provider Verifier应该满足你的需求。

### 如何运行

*步骤*:

1. 创建一个API和对应的Docker镜像
2. 将pact文件发布到Pact broker（或者创建本地文件）
3. 启动API
4. 运行Pact Provider Verifier
5. 停止API

验证工具之后会针对你的运行API重放所有的Pact文件，如果无法满足则会失败(`exit 1`)。
因为没有可用的测试DSL，所以在CI/CD流水线中运行时，你需要注意处理进程退出代码。

如果你在使用Docker和Docker compose，会帮你自动搞定上面的步骤3-5。
### Docker的例子

下面的使用Docker镜像的例子来自[Pact Provider Verifier](https://github.com/DiUS/pact-provider-verifier-docker)项目。
*步骤*:

1. 创建一个API和对应的Docker镜像
2. 将pact文件发布到Pact broker（或者创建本地文件）
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

#### 提供者状态的API

通过实现以下的内容来对提供者端执行Pact提供者验证：

* 一个消费者返回pact 提供者端状态的http get的端点

		{
			"myConsumer": [
				"customer is logged in",
				"customer has a million dollars"
			]
		}

* 一个设置活跃的pact消费者和提供者状态的http post端点

		consumer=web&state=customer%20is%20logged%20in

需要下面的环境变量：

* `pact_urls` - a comma delimited list of pact file URL
* `provider_base_url` - the base URL of the pact `Provider`
* `provider_states_url` - the full URL of the endpoint which returns `Provider States` by consumer
* `provider_states_active_url` - the full URL of the endpoint which sets the active pact `Consumer` and `Provider` state`

*更新例子docker-compose.yml文件：*

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


### Ruby的例子

如果你没有使用Docker，那么就需要：

* 安装Ruby运行时环境
* fork或者clone[代码库](https://github.com/DiUS/pact-provider-verifier-docker)或者将脚本拷贝到你的工程中
* 运行下面的命令：

```
bundle install
bundle exec rake verify_pacts
```
