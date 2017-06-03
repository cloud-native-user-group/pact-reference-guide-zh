# 提供者端

提供者端的Pact测试，包括验证契约-Pact文件-可以由提供者端满足。

1. 启动你的提供者端API:

	```go
	mux := http.NewServeMux()
	mux.HandleFunc("/setup", func(w http.ResponseWriter, req *http.Request) {
		w.Header().Add("Content-Type", "application/json")
	})
	mux.HandleFunc("/states", func(w http.ResponseWriter, req *http.Request) {
		fmt.Fprintf(w, `{"My Consumer": ["Some state", "Some state2"]}`)
		w.Header().Add("Content-Type", "application/json")
	})
	mux.HandleFunc("/someapi", func(w http.ResponseWriter, req *http.Request) {
		w.Header().Add("Content-Type", "application/json")
		fmt.Fprintf(w, `
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
			]`)
	})
	go http.ListenAndServe(":8000"), mux)
	```

	注意服务器有两个端点：`/states`以及可以让验证在每个测试运行前去设置[提供者端状态](http://docs.pact.io/documentation/provider_states.html)的`/setup`。

2. 验证提供者端API

	你现在可以告诉Pact读取Pact文件并且验证你的API会满足已知的每个消费者的需求：

	```go
	err := pact.VerifyProvider(&types.VerifyRequest{
		ProviderBaseURL:        "http://localhost:8000",
		PactURLs:               []string{"./pacts/my_consumer-my_provider.json"},
		ProviderStatesURL:      "http://localhost:8000/states",
		ProviderStatesSetupURL: "http://localhost:8000/setup",
	})

	if err != nil {
		t.Fatal("error:", err)
	}
	```

  Pact(从远程或者本地资源中)读取特定的pact文件并且对运行的提供者端重放。如果所有的交互都满足，我们可以说契约双方都满足，测试通过。

	[Pact Broker](http://docs.pact.io/documentation/sharings_pacts.html)).
	注意`PactURLs`是本地pact文件的列表或者远程的基础url(比如来自一个[Pact Broker](http://docs.pact.io/documentation/sharings_pacts.html))。
	更复杂的E2E测试的例子请查看[集成测试](https://github.com/pact-foundation/pact-go/blob/master/dsl/pact_test.go)