## 使用——一个场景示例

我们打算用Pact测试在消费者Zoo App和提供者Animal Service之间写一个集成测试。在消费者项目中，我们打算使用一个模型（Alligator类）来表示从Animal Service返回的数据，以及一个客户端（AnimalServiceClient）来负责向Animal Service发起HTTP调用。

![Example](../media/zoo_app-animal_service.png)

### 在Zoo App（消费者）项目中

#### 1. 从模型类开始

设想有一个类似这样的模型类。Alligator的属性依赖于远程服务，需要通过向Animal Service发HTTP调用才能够取到。

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
#### 3. 配置模拟的Animal Service

以下代码将在`localhost:1234`端口上创建一个模拟的服务，将用于对应用的HTTP查询请求作出响应，就像是真实的“Animal Service”一样。这里还创建了一个模拟的提供者对象，将用于设置期望值。用于访问模拟的服务提供者的名字可以是你在服务参数中所给出的任何名字——在本例中为“animal_service”。

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

Running the AnimalServiceClient spec will generate a pact file in the configured pact dir (`spec/pacts` by default).
Logs will be output to the configured log dir (`log` by default) that can be useful when diagnosing problems.

Of course, the above specs will fail because the Animal Service client method is not implemented, so next, implement your provider client methods.

#### 6. Implement the Animal Service client consumer methods

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

#### 7. Run the specs again.

Green! You now have a pact file that can be used to verify your expectations of the Animal Service provider project.

Now, rinse and repeat for other likely status codes that may be returned. For example, consider how you want your client to respond to a:
* 404 (return null, or raise an error?)
* 500 (specifying that the response body should contain an error message, and ensuring that your client logs that error message will make your life much easier when things go wrong)
* 401/403 if there is authorisation.

### In the Animal Service (provider) project

#### 1. Create the skeleton API classes

Create your API class using the framework of your choice (the Pact authors have a preference for [Webmachine][webmachine] and [Roar][roar]) - leave the methods unimplemented, we're doing Test First Develoment, remember?

#### 2. Tell your provider that it needs to honour the pact file you made earlier

Require "pact/tasks" in your Rakefile.

```ruby
# In Rakefile
require 'pact/tasks'
```

Create a `pact_helper.rb` in your service provider project. The recommended place is `spec/service_consumers/pact_helper.rb`.

See [Verifying Pacts](https://github.com/realestate-com-au/pact/wiki/Verifying-pacts) and the [Provider](documentation/configuration.md#provider) section of the Configuration documentation for more information.

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

#### 3. Run your failing specs

    $ rake pact:verify

Congratulations! You now have a failing spec to develop against.

At this stage, you'll want to be able to run your specs one at a time while you implement each feature. At the bottom of the failed pact:verify output you will see the commands to rerun each failed interaction individually. A command to run just one interaction will look like this:

    $ rake pact:verify PACT_DESCRIPTION="a request for an alligator" PACT_PROVIDER_STATE="an alligator exists"

#### 4. Implement enough to make your first interaction spec pass

Rinse and repeat.

#### 5. Keep going til you're green

Yay! Your Animal Service provider now honours the pact it has with your Zoo App consumer. You can now have confidence that your consumer and provider will play nicely together.
