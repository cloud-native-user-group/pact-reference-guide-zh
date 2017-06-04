# Ruby

这份上手指南能够帮助你通过Ruby开始使用Pact。更多详情或高级话题，可移步至[Ruby Pact GitHub仓库](https://github.com/realestate-com-au/pact)。

## 安装

在应用的Gemfile里面增加这一行：

    gem 'pact'

然后执行：

    $ bundle

或者自己这样安装：

    $ gem install pact

## 使用方法——一个示例场景

我们打算用Pact测试在`消费者`Zoo App和`提供者`Animal Service之间写一个集成测试。在消费者项目中，我们打算使用一个模型（Alligator类）来表示从Animal Service返回的数据，以及一个客户端（AnimalServiceClient）来负责向Animal Service发起HTTP请求。

![Example](../media/zoo_app-animal_service.png)
### 在Zoo App（消费者）项目中

#### 1. 从模型类开始

设想有一个类似这样的模型类。Alligator的属性依赖于远程服务，需要通过向Animal Service发送HTTP请求才能获取到。

```ruby
class Alligator
  attr_reader :name

  def initialize name
    @name = name
  end

  def == other
    other.is_a?(Alligator) && other.name == name
  end
end
```

#### 2. 创建Animal Service客户端类的骨架

设想有一个类似这样的Animal Service客户端类。

```ruby
require 'httparty'

class AnimalServiceClient
  include HTTParty
  base_uri 'http://animal-service.com'

  def get_alligator
    # Yet to be implemented because we're doing Test First Development...
  end
end
```

#### 3. 配置Animal Service的模拟服务

以下代码将在`localhost:1234`端口上创建一个模拟服务，用于对应用的HTTP查询请求作出响应，正如真实的“Animal Service”应用一样。这里还创建了一个模拟的服务提供者对象，用于设置期望值。用于访问模拟服务提供者的方法名你可以通过服务参数任意指定——在本例中为“animal_service”。`

```ruby
# In /spec/service_providers/pact_helper.rb

require 'pact/consumer/rspec'
# or require 'pact/consumer/minitest' if you are using Minitest

Pact.service_consumer "Zoo App" do
  has_pact_with "Animal Service" do
    mock_service :animal_service do
      port 1234
    end
  end
end
```

#### 4. 给Animal Service客户端写一个失败的测试用例

```ruby
# In /spec/service_providers/animal_service_client_spec.rb

# When using RSpec, use the metadata `:pact => true` to include all the pact functionality in your spec.
# When using Minitest, include Pact::Consumer::Minitest in your spec.

describe AnimalServiceClient, :pact => true do

  before do
    # Configure your client to point to the stub service on localhost using the port you have specified
    AnimalServiceClient.base_uri 'localhost:1234'
  end

  subject { AnimalServiceClient.new }

  describe "get_alligator" do

    before do
      animal_service.given("an alligator exists").
        upon_receiving("a request for an alligator").
        with(method: :get, path: '/alligator', query: '').
        will_respond_with(
          status: 200,
          headers: {'Content-Type' => 'application/json'},
          body: {name: 'Betty'} )
    end

    it "returns a alligator" do
      expect(subject.get_alligator).to eq(Alligator.new('Betty'))
    end
  end
end
```

#### 5. 执行测试用例

执行AnimalServiceClient用例，将会在配置好的pact文件夹（默认为`spec/pacts`）下生成一个pact文件。

日志将输出在一个配置好的日志文件夹（默认为`log`）下，用于诊断问题。

当然，以上用例将会失败，因为Animal Service的客户端方法还没有实现，那么下一步，来实现提供者的客户端代码吧。

#### 6. 实现Animal Service的消费者客户端方法

```ruby
class AnimalServiceClient
  include HTTParty
  base_uri 'http://animal-service.com'

  def get_alligator
    name = JSON.parse(self.class.get("/alligator").body)['name']
    Alligator.new(name)
  end
end
```

#### 7. 再次运行该用例

测试通过！现在你会得到一个pact文件，它可用于在Animal Service提供者的项目中验证期望。

现在，对于其他可能的返回状态码重复上述步骤。例如，考虑以下场景中你的客户端需要如何响应：

- 404（返回null，还是抛出错误？）
- 500（具体说明响应体中所包含的错误信息，并确保客户端日志记录下了这些错误信息，这样当出现问题时会省很多事）
- 校验授权时是否需要返回401/403。

### 在Animal Service（`提供者`）项目中

#### 1. 创建API相关类的骨架

使用你所选择的框架创建API类（Pact的作者更喜欢使用[Webmachine](https://github.com/webmachine/webmachine)和[Roar](https://github.com/trailblazer/roar)）——先不用实现方法，我们要先写测试，后写实现，还记得吧？

#### 2. 告诉提供者需要遵守之前的pact文件中的规约

在Rakefile中引入`pact/tasks`。

```ruby
# In Rakefile
require 'pact/tasks'
```

在服务提供者项目中创建名为`pact_helper.rb`的文件。推荐放在`spec/service_consumers/pact_helper.rb`下。

更多信息，查看配置文档中的 [Verifying Pacts](https://github.com/realestate-com-au/pact/wiki/Verifying-pacts)和[Provider](documentation/configuration.md#provider)章节。

```ruby
# In specs/service_consumers/pact_helper.rb

require 'pact/provider/rspec'

Pact.service_provider "Animal Service" do

  honours_pact_with 'Zoo App' do

    # This example points to a local file, however, on a real project with a continuous
    # integration box, you would use a [Pact Broker](https://github.com/bethesque/pact_broker) or publish your pacts as artifacts,
    # and point the pact_uri to the pact published by the last successful build.

    pact_uri '../zoo-app/specs/pacts/zoo_app-animal_service.json'
  end
end
```

#### 3. 运行失败的测试

    $ rake pact:verify
恭喜你！现在你有一个失败的测试需要通过了。

在这一阶段，你可能会想要在每实现一个特性之后能够只运行众多用例中的某一个。在pact:verify的失败输出结果的底部，你可以看到用于重新独立运行每条失败的请求的命令。只运行一条请求的命令类似这样：

    $ rake pact:verify PACT_DESCRIPTION="a request for an alligator" PACT_PROVIDER_STATE="an alligator exists"
#### 4. 实现代码让你的第一个测试用例通过

然后重复上述步骤。

#### 5. 继续直到所有测试通过

哇！你的Animal Service现在已经可以遵守与你的Zoo App消费者之间的契约了。现在你就有信心让你的消费者和提供者能够一起很好地工作了。

### 使用提供者状态

契约中的各个交互应该是在互相隔离状态下进行验证的，不得持有前一个交互中的上下文。那么怎样测试那些依赖于在提供者端已存在的数据的请求呢？请在[这里](https://github.com/realestate-com-au/pact/wiki/Provider-states)查看关于提供者状态的介绍。