# 提供者状态

关于这一高级话题，请先阅读[提供者状态](../provider_states.md)一节的介绍。

提供者状态中的文本对于读者来说应该具有足够清楚的意义，如下（这是自动生成的文档为读者呈现出的）：


Given **an alligator with the name Mary exists** \*

Upon receiving **a request to retrieve an alligator by name** \*\* from Some Consumer

With {"method" : "get", "path" : "/alligators/Mary" }

Some Provider will respond with { "status" : 200, ...}

\* 这是提供者状态
\*\* 这是请求描述

### 消费者代码库

举个例子，消费者项目中用于创建一段契约的代码可能看起来像这样：

```ruby
describe MyServiceProviderClient do

  subject { MyServiceProviderClient.new }

  describe "get_something" do
    context "when a thing exists" do
      before do
        my_service.given("a thing exists").
          upon_receiving("a request for a thing").with(method: 'get', path: '/thing').
          will_respond_with(status: 200,
            headers: { 'Content-Type' => 'application/json' },
            body: { name: 'A small something'} )
      end

      it "returns a thing" do
        expect(subject.get_something).to eq(SomethingModel.new('A small something'))
      end
    end

    context "when a thing does not exist" do
      before do
        my_service.given("a thing does not exist").
          upon_receiving("a request for a thing").with(method: 'get', path: '/thing').
          will_respond_with(status: 404)
      end

      it "returns nil" do
        expect(subject.get_something).to be_nil
      end
    end
  end
end
```

### 提供者代码库

根据上述的提供者状态，要相应地定义能够创建适当数据的服务提供者状态，可以在服务提供者项目中这样来写。（此处的消费者名称必须与消费者项目中配置的消费者名称相符，才能正确地找到这些提供者状态。）

```ruby
# In /spec/service_consumers/provider_states_for_my_service_consumer.rb

Pact.provider_states_for 'My Service Consumer' do

  provider_state "a thing exists" do
    set_up do
      # Create a thing here using your framework of choice
      # eg. Sequel.sqlite[:somethings].insert(name: "A small something")
    end

    tear_down do
      # Any tear down steps to clean up your code
    end
  end

  provider_state "a thing does not exist" do
    no_op # If there's nothing to do because the state name is more for documentation purposes,
          # you can use no_op to imply this.
  end

end
```
在pact_helper.rb中引入上面所写的提供者状态文件

```ruby
# In /spec/service_consumers/pact_helper.rb

require './spec/service_consumers/provider_states_for_my_service_consumer.rb'
```

### 基状态

要定义一些能够对于给定消费者的每次交互前/后都会运行的代码，而不管是否指定了提供者状态，只需将set_up/tear_down代码块不被provider_state所包住即可。

```ruby
Pact.provider_states_for 'My Service Consumer' do

  set_up do
    # This will run before the set_up for provider state specified for the interaction.
    # eg. create API user, set the expected basic auth details
  end

  tear_down do
    # ...
    # This will run after the tear_down for the specified provider state.
  end
end
```

### 全局状态

全局状态将会在消费者特定的基状态被设置前而设置。避免使用全局设置来创建数据，因为当存在多个消费者时这会使测试变得脆弱。

```ruby
Pact.set_up do
  # eg. start database cleaner transaction
end

Pact.tear_down do
  # eg. clean database
end
```

### 测试错误响应

测试客户端如何处理错误响应是非常重要的。

```ruby
# Consumer codebase

describe MyServiceProviderClient do

  subject { MyServiceProviderClient.new }

  describe "get_something" do

    context "when an error occurs retrieving a thing" do
      before do
        my_service.given("an error occurs while retrieving a thing").
          upon_receiving("a request for a thing").with(method: 'get', path: '/thing').
          will_respond_with(
            status: 500,
            headers: { 'Content-Type' => 'application/json' },
            body: { message: "An error occurred!" } )
      end

      it "raises an error" do
        expect{ subject.get_something }.to raise_error /An error occurred!/
      end

    end
  end
end
```

```ruby
# Provider codebase

Pact.provider_states_for 'My Service Consumer' do
  provider_state "an error occurs while retrieving a thing" do
    set_up do
      # Stubbing is ususally the easiest way to generate an error with predictable error text.
      allow(ThingRepository).to receive(:find).and_raise("An error occurred!")
    end
  end
end
```

### 包含模块以便在set_up和tear_down中使用

按照这种方法包含进来的任意模块在set_up与tear_down代码块中都是可用的。 这样做的一个常见用途是将用于设置数据的工厂方法包含进来，于是提供者状态文件将不会过于臃肿。

```ruby
Pact.configure do | config |
  config.include MyTestHelperMethods
end
```
