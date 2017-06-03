# 提供者状态

pact中的每个交互都应当是独立验证，不需要前面的交互动作中保存状态。所以如何测试一个需要在提供者端保留数据的请求？通过提供者端状态可以在Pact中实现这点。

提供者端状态也允许消费者端发起相同的请求但是期望的响应不同（比如，不同的响应码，或者相同资源的不同数据子集）。

当你发出一个具有相应的请求/响应对的dsl.Given()子句，消费者端的状态就配置好了。

提供者端也需要一些配置，目前需要两个允许的API端点在验证的过程中去获取以及配置可用的状态。在dsl.VerifyRequest中必须提供的两个选项是：

```
ProviderStatesURL: 		    GET URL to fetch all available states (see types.ProviderStates)
ProviderStatesSetupURL: 	POST URL to set the provider state (see types.ProviderState)
```

使用标准的Go http包的路由例子看起来可能像这样，注意`/states`端点返回了每个已知消费者端的可用状态的列表：

```go
// Return known provider states to the verifier (ProviderStatesURL):
mux.HandleFunc("/states", func(w http.ResponseWriter, req *http.Request) {
	states :=
	`{
		"My Front end consumer": [
			"User A exists",
			"User A does not exist"
		],
		"My api friend": [
			"User A exists",
			"User A does not exist"
		]
	}`
		fmt.Fprintf(w, states)
		w.Header().Add("Content-Type", "application/json")
})

// Handle a request from the verifier to configure a provider state (ProviderStatesSetupURL)
mux.HandleFunc("/setup", func(w http.ResponseWriter, req *http.Request) {
	w.Header().Add("Content-Type", "application/json")

	// Retrieve the Provider State
	var state types.ProviderState

	body, _ := ioutil.ReadAll(req.Body)
	req.Body.Close()
	json.Unmarshal(body, &state)

	// Setup database for different states
	if state.State == "User A exists" {
		svc.userDatabase = aExists
	} else if state.State == "User A is unauthorized" {
		svc.userDatabase = aUnauthorized
	} else {
		svc.userDatabase = aDoesNotExist
	}
})
```

查看例子或者从 http://docs.pact.io/documentation/provider_states.html 阅读更多内容。
