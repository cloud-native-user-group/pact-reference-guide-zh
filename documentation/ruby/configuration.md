# 配置

## 菜单

#### 消费者与提供者配置选项
* [diffformatter](#diff_formatter)
* [logdir](#log_dir)
* [logger](#logger)
* [logger.level](#loggerlevel)

#### 仅适用于消费者的配置选项
* [pactdir](#pact_dir)
* [docdir](#doc_dir)
* [docgenerator](#doc_generator)
* [pactfilewritemode](#pactfile_write_mode)

#### 仅适用于提供者的配置选项
* [include](#include)

## 消费者与提供者

### log_dir（日志目录）

```ruby
Pact.configure do | config |
  config.log_dir = './log'
end
```

默认值为： `./log`

### logger（日志记录器）

```ruby
Pact.configure do | config |
  config.logger = Logger.new
end
```

默认值为：配置了log_dir为目录的日志文件记录器。

### logger.level（日志等级）

```ruby
Pact.configure do | config |
  config.logger.level = Logger::INFO
end
```

默认值为： `Logger::DEBUG`

### diff_formatter（差异展示格式）

```ruby
Pact.configure do | config |
  config.diff_formatter = :list
end
```

默认值为： [:list](#list)

可选项有：[:unix](#unix), [:list](#list), [:embedded](#embedded), [Custom Diff Formatter](#custom-diff-formatter)


#### :unix
<img src="https://github.com/realestate-com-au/pact/raw/master/documentation/diff_formatter_unix.png" width="700">

#### :list

<img src="https://github.com/realestate-com-au/pact/raw/master/documentation/diff_formatter_list.png" width="700">

#### :embedded

<img src="https://github.com/realestate-com-au/pact/raw/master/documentation/diff_formatter_embedded.png" width="700">


#### 定制差异展示格式

使用可接受`diff`作为参数，并提供`call`方法调用作为响应的任何对象。

```ruby
class MyCustomDiffFormatter

  def self.call diff
    ### Do stuff here
  end

end

Pact.configure do | config |
  config.diff_formatter = MyCustomDiffFormatter
end
```


## 消费者

### pact_dir（契约目录）

```ruby
Pact.configure do | config |
  config.pact_dir = `./spec/pacts`
end
```

默认值为： `./spec/pacts`

### doc_generator（文档生成器）

```ruby
Pact.configure do | config |
  config.doc_generator = :markdown
end
```

默认值为： none

可选项有： [:markdown](#markdown), [Custom Doc Generator](#custom-doc-generator)

#### :markdown

基于消费者创建的契约文件的内容生成Markdown文档。文档文件生成在：`${Pact.configuration.doc_dir}/markdown`路径。

#### 自定义文档生成器

使用可接受`pact_dir`和`doc_dir`作为参数，并提供`call`方法调用作为响应的任何对象。

```ruby
Pact.configure do | config |
  config.doc_generator = lambda{ | pact_dir, doc_dir | generate_some_docs(pact_dir, doc_dir) }
end
```

#### doc_dir（文档目录）

```ruby
Pact.configure do | config |
  config.doc_dir = './doc'
end
```

默认值为： `./doc`


### pactfile_write_mode（契约文件写入模式）

默认值为： `:overwrite`
可选项有： `:overwrite`, `:update`, `:smart`, `:none`

默认情况下，当使用pacts运行任何测试用例时，契约文件每次都会被彻底覆盖（从头开始）。这意味着如果某些交互没有在最近一次rspec运行时被执行的话，实际上会被从契约文件里删除掉。如果你有运行时间很长的pact用例（比如，通过Capybara使用浏览器生成的）而且正在并行开发消费者与提供者，或者试着修复挂掉的测试，一次运行所有的测试将会是很繁重的。这种场景下，你可以将pactfile_write_mode值设置为:update。这时将会保留所有已存在的交互，而且只更新那些发生变化了的交互，以描述和提供者状态字段作为识别标志。这样做不好的一面是如果这些字段任何一个发生了变化，那么旧有的交互将不会被从契约文件中移除。作为一个两全之策，捏可以将pactfile_write_mode值设置为:smart。这样当运行rake（系统通过使用'ps'命令调用来进行判断）的时候将会使用:overwrite模式，而运行单个用例的时候将会使用:update模式。该值设置为:none时将不会生成任何契约文件（在pact-mock_service版本大于等于0.8.1时）。

## 提供者

Pact使用RSpec和Rack::Test在契约文件基础上生成动态用例。如果没有合适的Pact配置特性的话，RSpec相关配置也可用于修改测试行为。如果你希望使用和单元测试一样的spec_helper.rb文件，就在pact_helper.rb中引入它，但是要注意，你为单元测试配置的RSpec选项有可能是，但也有可能不是你的pact验证测试中所需要的。

### include

```ruby
Pact.configure do | config |
  config.include MyTestHelperMethods
end
```

要想在提供者状态的set_up和tear_down代码块中使用某些模块，就需要在配置中将其包含进来，如下所示。这样做的一个常见用途是将用于设置数据的工厂方法包含进来，于是提供者状态文件将不会过于臃肿。