# 匹配 (消费者端测试)

除逐字值匹配之外，`dsl`包中还有3个有用的匹配函数，可以提高表达力并减少脆性测试用例。

* `dsl.Term(example, matcher)` - 告诉Pact应该使用正则表达式去陪陪值，在mock的响应中使用`example`。`example`必须是一个字符串。
* `dsl.Like(content)` - 告诉Pact只要元素_类型_ (合法的JSON数字、字符串、对象等)匹配，值本身并不重要。
* `dsl.EachLike(content, min)` - 告诉Pact值应当是数组类型，并且由传入的元素构成。`min`必须 >= 1。`content`可能是一个可用的JSON值：比如，字符串，数字和对象。

*例子:*

下面是一个使用所有3个术语的复杂的例子：

```go
jumper := Like(`"jumper"`)
shirt := Like(`"shirt"`)
tag := EachLike(fmt.Sprintf(`[%s, %s]`, jumper, shirt), 2)
size := Like(10)
colour := Term("red", "red|green|blue")

match := formatJSON(
	EachLike(
		EachLike(
			fmt.Sprintf(
				`{
					"size": %s,
					"colour": %s,
					"tag": %s
				}`, size, colour, tag),
			1),
		1))
```

这个例子将会从mock服务器生成一个像下面这样的响应体：
```json
[
  [
    {
      "size": 10,
      "colour": "red",
      "tag": [
        [
          "jumper",
          "shirt"
        ],
        [
          "jumper",
          "shirt"
        ]
      ]
    }
  ]
]
```

更多匹配的例子可以查看[匹配测试](https://github.com/pact-foundation/pact-go/blob/master/dsl/matcher_test.go)。

*注意*: 需要注意的一点是，你必须要使用有效的Ruby[正则表达式](http://ruby-doc.org/core-2.1.5/Regexp.html)以及双转义反斜杠。

阅读更多关于[灵活匹配](https://github.com/realestate-com-au/pact/wiki/Regular-expressions-and-type-matching-with-Pact)的内容。
