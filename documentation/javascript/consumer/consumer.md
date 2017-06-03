# Consumer

这个代码库提供了一个用于创建pacts的JavaScript DSL。下面的指南将帮助你选择你最喜欢的测试框架来开始使用Pact：

* [Mocha / Node](documentation/javascript/mocha__node.md)
* [Jasmine / Karma](documentation/javascript/jasmine__karma.md)
* [Flexible Matching](documentation/javascript/flexible_matching.md)

## Pact的Mock服务
In order to write Pact tests on the `Consumer`, you first need to install the Pact Mock Service.
为了在`消费者`端实现Pact测试，你首先需要安装Pact Mock服务。

### 安装Ruby
如果你使用Windows，请参考[pact-mock-service Windows安装指南](https://github.com/bethesque/pact-mock_service/wiki/Installing-the-pact-mock_service-gem-on-Windows).

为了安装Pact Mock服务，你首先得安装Ruby。在你的工程中创建一个`Gemfile`并添加下面的内容：

```ruby
source 'https://rubygems.org'
gem 'pact-mock_service', '~> 0.7.0'
```

然后，在终端中运行`gem install bundler && bundle install`。你应当可以看到和下面相似的输出：

```bash
Fetching gem metadata from https://rubygems.org/.........
Fetching version metadata from https://rubygems.org/..
...
Installing pact-support 0.5.3
Installing pact-mock_service 0.7.0
Bundle complete! 1 Gemfile dependency, 19 gems now installed.
```

_**注意:** Windows用户必须按照Wiki的指导说明运行安装命令_


### 使用Docker 

更多细节请参考[Docker仓库](https://github.com/madkom/docker/tree/master/pact-mock-service).

**运行**:

```bash
docker run -p 1234:1234 -v /tmp/log:/var/log/pacto -v /tmp/contracts:/opt/contracts madkom/pact-mock-service
```
这会创建一个运行在`1234`端口的pact server，同时日志(logs)挂载在卷`/tmp/log`以及pact文件挂载在`/tmp/contacts`。