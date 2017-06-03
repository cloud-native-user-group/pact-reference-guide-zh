# 灵活匹配

确认你已经阅读过[匹配](../matching.md)这一节。

#### 通过正则表达式匹配

Remember that the mock service is written in Ruby, so the regular expression must be in a Ruby format, not a JavaScript format. Make sure to start the mock service with the argument `--pact-specification-version 2.0.0`.

mock服务是用Ruby编写的，所以正则表达式必须符合Ruby格式，而不是JavaScript的格式。确保使用参数`--pact-specification-version 2.0.0`启动mock服务。

```javascript
provider
  .given('there is a product')
  .uponReceiving("request for products")
  .withRequest({
    method: "GET",
    path: "/products",
    query: {
      category: Pact.Match.term({matcher: "\\w+", generate: 'pizza'}),
    }
  })
  .willRespondWith(
    200,
    {},
    {
      "collection": [
        { guid: Pact.Match.term({matcher: "\\d{16}", generate: "1111222233334444"}) }
      ]
    }
  );
```

#### 基于类型匹配

```javascript

provider
  .given('there is a product')
  .uponReceiving("request for products")
  .withRequest({
    method: "get",
    path: "/products",
    query: {
      category: Pact.Match.somethingLike("pizza")
    }
  })
  .willRespondWith(
    200,
    {},
    {
      "collection": [
        { guid: Pact.Match.somethingLike(1111222233334444) }
      ]
    }
  );
```

#### 基于数组匹配

匹配提供了指定变长数组的能力。 例如：

```javascript
Pact.Match.eachLike(obj, { min: 3 })
```

`obj`可以任何的Javascript`对象`、值或者`Pact.Match`。

下面是一个使用了所有的Pact Matchers的例子。它接受一个可选的参数(`{ min: 3 }`)，这个参数`min`大于0并且缺省为1。

```javascript
var somethingLike = Pact.Match.somethingLike;
var term          = Pact.Match.term;
var eachLike      = Pact.Match.eachLike;

provider
  .given('there is a product')
  .uponReceiving("request for products")
  .withRequest({
    method: "get",
    path: "/products",
    query: { category: "clothing" }
  })
  .willRespondWith({
    status: 200,
    headers: { 'Content-Type': 'application/json' },
    body: {
        "items": eachLike({
            size: somethingLike(10),
            colour: term("red|green|blue", {generates: "blue"}),
            tag: eachLike(somethingLike("jumper"))
        }, {min: 2})
    }
  });
```
