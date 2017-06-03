# Mocha / Node

### Mocha
如果你新建了一个工程，运行`npm init`去初始化`package.json`的安装。

在终端中，`package.json`的同一目录下，运行`npm install --save-dev --save-exact mocha`去安装[Mocha](https://mochajs.org/)。同时运行`npm install --save-dev --save-exact chai`去安装[Chai.js assertion library](http://chaijs.com/)。
一旦Jasmine安装完成，运行`npm install --save-dev --save-exact pact-consumer-js-dsl`安装DSL。

#### 测试
像下面这样用`ES2015`语法去实现Mocha测试。

```javascript
import { expect } from 'chai'
import Pact from 'pact-consumer-js-dsl'

describe("Client", () => {
  // ProviderClient is the class you have written to make the HTTP calls to the provider
  const client = new ProviderClient('http://localhost:1234');

  var helloProvider;

  beforeEach(() => {
    // setup your mock service
    // your client above should be routed through to this guy
    // during testing so expectactions can be recorded
    helloProvider = Pact.mockService({
      consumer: 'Hello Consumer',
      provider: 'Hello Provider',
      port: 1234,
      done: (error) => {
        expect(error).to.be.null;
      }
    });
  });

  it("should say hello", (done) => {
    const requestHeaders  = { "Accept": "application/json" };
    const responseHeaders = { "Content-Type": "application/json" };
    const responseBody    = { "name": "Mary" };
    
    // setting expectation
    helloProvider
      .given("an alligator with the name Mary exists")
      .uponReceiving("a request for an alligator")
      .withRequest("GET", "/alligators/Mary", requestHeaders)
      .willRespondWith(200, responseHeaders, responseBody);

    // verifying expectation
    helloProvider.run(done, (runComplete) => {
      expect(client.getAlligatorByName("Mary")).to.eql(new Alligator("Mary"));
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

在终端中输入`mocha`去执行测试。成功后会生成`tmp/pacts/hello_consumer-hello_provider.json`这样的pact文件，内容像下面一样：

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
