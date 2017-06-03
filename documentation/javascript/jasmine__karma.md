# Jasmine / Karma

### Jasmine
如果你新建了一个工程，运行`npm init`去初始化`package.json`的安装。
在终端中，`package.json`的同一目录下，运行`npm install --save-dev --save-exact jasmine karma karma-jasmine karma-chrome-launcher karma-cli`去安装Jasmine和Karma。

一旦Jasmine安装完成，运行`npm install --save-dev --save-exact pact-consumer-js-dsl`安装DSL。

#### 设置 Karma
运行`karma init`。选择**jasmine** 作为*测试框架*并且**拒绝** *使用require.js*。

在`karma.conf.js`中告诉Karma关于`pact-consumer-js-dsl.js`。在`files: []`部分为`node_modules/pact-consumer-js-dsl/dist/pact-consumer-js-dsl.js`添加一个新的入口。

允许测试去加载`Pact Mock Server`。实现这个的一种方式是在`karma.conf.js`中将`browsers: ['Chrome']`修改为，

 ```javascript
browsers: ['Chrome_without_security'],
customLaunchers: {
  Chrome_without_security: {
      base: 'Chrome',
      flags: ['--disable-web-security']
  }
}
```
或者将`browsers: ['PhantomJS']` 改为：

```javascript
browsers: ['PhantomJS_without_security'],
customLaunchers: {
  PhantomJS_without_security: {
    base: 'PhantomJS',
    flags: ['--web-security=false']
  }
}
```

确认`karma.conf.js`的文件数组中中包含了源代码、测试文件。

#### 测试
像下面这样实现你的Jasmine测试：

```javascript
describe("Client", function() {
  var client, helloProvider;

  beforeEach(function() {
    //ProviderClient is the class you have written to make the HTTP calls to the provider
    client = new ProviderClient('http://localhost:1234');
    
    // setup your mock service
    // your client above should be routed through to this guy
    // during testing so expectactions can be recorded
    helloProvider = Pact.mockService({
      consumer: 'Hello Consumer',
      provider: 'Hello Provider',
      port: 1234,
      done: function (error) {
        expect(error).toBe(null);
      }
    });
  });

  it("should say hello", function(done) {
    var requestHeaders  = { "Accept": "application/json" };
    var responseHeaders = { "Content-Type": "application/json" };
    var responseBody    = { "name": "Mary" };

    helloProvider
      .given("an alligator with the name Mary exists")
      .uponReceiving("a request for an alligator")
      .withRequest("GET", "/alligators/Mary", requestHeaders)
      .willRespondWith(200, responseHeaders, responseBody);

    helloProvider.run(done, function(runComplete) {
      expect(client.getAlligatorByName("Mary")).toEqual(new Alligator("Mary"));
      runComplete();
    });
  });
});
```

#### 运行测试

在运行测试前你需要首先启动Pact Mock服务。运行下面的命令

```bash
bundle exec pact-mock-service -p 1234 --pact-specification-version 2.0.0 -l logs/pact.logs --pact-dir tmp/pacts
```

这个命令会：
* 生成一个新的 `logs` 文件夹，在这个目录你可以以日志的形式查看Mock服务里收到的所有交互内容
* 生成一个新的 `tmp` 文件夹，这个目录将会保存测试成功验证的`Pact`文件

在终端中输入`karma start`去执行测试。成功后会生成`tmp/pacts/hello_consumer-hello_provider.json`这样的pact文件，内容像下面一样：

```json
{
  "consumer": {
    "name": "Hello Consumer"
  },
  "provider": {
    "name": "Hello Provider"
  },
  "interactions": [
    {
      "description": "a request for an alligator",
      "provider_state": "an alligator with the name Mary exists",
      "request": {
        "method": "get",
        "path": "/alligators/Mary",
        "headers": {
          "Accept": "application/json"
        }
      },
      "response": {
        "status": 200,
        "headers": {
          "Content-Type": "application/json"
        },
        "body": {
          "name": "Mary"
        }
      }
    }
  ],
  "metadata": {
    "pactSpecificationVersion": "2.0.0"
  }
}
```
