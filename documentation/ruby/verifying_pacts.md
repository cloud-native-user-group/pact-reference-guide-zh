# 验证契约

关于本话题请先参考[验证契约](../verifying_pacts.md)一节的介绍。


## 使用rake pact:verify

使用`pact:verify`任务是最常见的一种验证契约的方式。这也是用于配置你的服务提供者应遵守的契约的默认集合之处。

在Rakefile中引入`'pact/tasks'`就可以使用了。

```ruby
# In Rakefile
require 'pact/tasks'

# Remember to add it to your default Rake task
task :default => 'pact:verify'

```

被`pact:verify`任务用于验证的契约是在提供者端代码库中的 `pact_helper.rb` 文件里进行配置的。

该文件必须命名为 `pact_helper.rb` ，尽管其存储位置可以允许一定程度上的灵活性，但是推荐放在 `spec/service_consumers/pact_helper.rb`下。
为了保证每次使用的是消费者契约的最新版本，推荐你要么使用[Pact Broker](https://github.com/bethesque/pact_broker)，要么在消费者端成功构建之后将契约发布成你的CI系统中的产物。

注意：Pact使用了Rack::Test，并且假定你的服务提供者是一个Rack应用。如果你的提供者不是Rack应用，参见如下的选项说明。

```ruby
# In specs/service_consumers/pact_helper.rb

require 'pact/provider/rspec'

# Require the provider states files for each service consumer
require 'service_consumers/provider_states_for_my_service_consumer'

Pact.service_provider "My Service Provider" do

  # Optional app configuration. Pact loads the app from config.ru by default
  # (it is recommended to let Pact use the config.ru if possible, so testing
  # conditions are closest to runtime conditions)
  app { MyApp.new }

  honours_pact_with 'My Service Consumer' do

    # This example points to a local file, however, on a real project with a continuous
    # integration box, you would publish your pacts as artifacts,
    # and point the pact_uri to the pact published by the last successful build.

    pact_uri '../path-to-your-consumer-project/specs/pacts/my_consumer-my_provider.json'
  end

  # This block is repeated for every pact that this provider should be verified against.
  honours_pact_with 'Some other Service Consumer' do
    ...
  end

end
```

### 使用基本的授权

要从需要某些基本的授权的URL上验证某个契约，只需添加用户名和密码选项：

```ruby
  pact_uri 'http://...', {username: '...', password: '...'}
```

## 使用rake pact:verify:at从任意的URL验证契约

你可以使用`pact:verify:at`任务对任意的本地或远程URL上的契约进行验证。

这在并行开展消费者和提供者的开发，希望验证刚从消费者代码库中生成的契约时十分有用。这里和`pact:verify`使用的pact_helper文件是相同的。

    $ rake pact:verify:at[../path-to-your-consumer-project/specs/pacts/my_consumer-my_provider.json]
    $ rake pact:verify:at[http://build-box/MyConsumerBuild/latestSuccessful/artifacts/my_consumer-my_provider.json]

如果需要基本的授权，请设置环境变量`PACT_BROKER_USERNAME`和`PACT_BROKER_PASSWORD`，或者使用基本授权URL格式，如`http://username:password@pactbroker.yourdomain/...`。

## 验证存储在Amazon S3上的契约

[Pact::Retreaty](https://github.com/fairfaxmedia/pact-retreaty)工具提供了一套极轻量级的机制，可以将pact契约推送到S3和从S3上拉取契约。

## 使用定制化的pact:verify任务

如果希望定制一个快捷任务，能够验证任意URL的契约，又不想作为普通的pact:verify任务所验证契约的一部分时，（例如，当你并行开展消费者和提供者的开发，并且希望得到比运行CI更短的反馈周期时）就可以将下列内容添加到Rakefile里面。pact.uri可以是一个本地文件系统路径或者一个远程URL。

```ruby
# In Rakefile or /tasks/pact.rake

# This creates a rake task that can be executed by running
# $ rake pact:verify:dev

Pact::VerificationTask.new(:dev) do | task |
  task.uri '../path-to-your-consumer-project/specs/pacts/my_consumer-my_provider.json'
end
```

如果需要基本的授权，请设置环境变量`PACT_BROKER_USERNAME`和`PACT_BROKER_PASSWORD`，或者使用基本授权URL格式，如`http://username:password@pactbroker.yourdomain/...`。

## 每次只验证一个交互

在某些阶段，当你在逐步实现每个特性时，会希望具有每次只运行一个测试的能力。在运行失败的pact:verify输出结果的底部你可以看到能够用于独立重复运行每个失败测试的命令。只运行某个测试的命令看起来类似这样：

    $ rake pact:verify PACT_DESCRIPTION="a request for something" PACT_PROVIDER_STATE="something exists"

## 为非Rack应用验证契约

### Ruby应用
如果你的应用是非Rack的Ruby应用，你也许能找到一个Rack适配器。如果找到了，然后将`Pact.service_provider`代码块里的`app`指向你的适配器的某个实例。否则的话，就使用[pact-provider-proxy](https://github.com/bethesque/pact-provider-proxy) gem吧。

## 配置RSpec

Pact使用动态创建出的RSpec用例来验证契约。如果你想修改底层RSpec执行过程的行为，你可以这样做：

1. 在pact_helper中使用普通的`RSpec.configure`代码来配置RSpec。
2. 或者在自定义的rake校验任务中设置`task.rspec_opts`，这与在普通的RSpec rake任务中进行声明可以起到同样的效果。

不过深入一步讲，尝试使用提供者状态中的set_up/tear_down代码块吧，因为我们有可能将来会换掉RSpec中的自定义校验代码。

## Pact Helper的位置

pact_helper的搜索路径如下：

```ruby
[
  "spec/**/*service*consumer*/pact_helper.rb",
  "spec/**/*consumer*/pact_helper.rb",
  "spec/**/pact_helper.rb",
  "**/pact_helper.rb"]
```
