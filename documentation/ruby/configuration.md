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

默认情况下，当使用pacts运行任何测试用例时，契约文件每次都会被彻底覆盖（从头开始）。这意味着如果某些交互没有在最近一次rspec运行时被执行的话，实际上会被从契约文件里删除掉。

By default, the pact file will be overwritten (started from scratch) every time any rspec runs any spec using pacts. This means that if there are interactions that haven't been executed in the most recent rspec run, they are effectively removed from the pact file. If you have long running pact specs (e.g. they are generated using the browser with Capybara) and you are developing both consumer and provider in parallel, or trying to fix a broken interaction, it can be tedious to run all the specs at once. In this scenario, you can set the pactfile_write_mode to :update. This will keep all existing interactions, and update only the changed ones, identified by description and provider state. The down side of this is that if either of those fields change, the old interactions will not be removed from the pact file. As a middle path, you can set pactfile_write_mode to :smart. This will use :overwrite mode when running rake (as determined by a call to system using 'ps') and :update when running an individual spec. :none will not generate any pact files (with pact-mock_service >= 0.8.1).

## Provider

Pact uses RSpec and Rack::Test to create dynamic specs based on the pact files. RSpec configuration can be used to modify test behaviour if there is not an appropriate Pact feature. If you wish to use the same spec_helper.rb file as your unit tests, require it in the pact_helper.rb, but remember that the RSpec configurations for your unit tests may or may not be what you want for your pact verification tests.

### include

```ruby
Pact.configure do | config |
  config.include MyTestHelperMethods
end
```

To make modules available in the provider state set_up and tear_down blocks, include them in the configuration as shown below. One common use of this to include factory methods for setting up data so that the provider states file doesn't get too bloated.
