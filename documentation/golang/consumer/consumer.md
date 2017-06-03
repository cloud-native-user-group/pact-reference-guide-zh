# 消费者端

消费者端的Pact测试是一个隔离测试，可以确保给定的组件能和另一个(远程)组件协作。Pact会自动在后台启动一个Mock服务器，作为协作组件的测试替身。

这意味着Mock服务器任何期望的交互都会被验证，如果交互没有完成或者意外的交互出现，都会导致测试失败：

典型的消费者端测试如下所示：

```go
func TestLogin(t *testing.T) {

	// Create Pact, connecting to local Daemon
	// Ensure the port matches the daemon port!
	pact := &Pact{
		Port:     6666,
		Consumer: "My Consumer",
		Provider: "My Provider",
	}
	// Shuts down Mock Service when done
	defer pact.Teardown()

	// Pass in your test case as a function to Verify()
	var test = func() error {
		_, err := http.Get("http://localhost:8000/")
		return err
	}

	// Set up our interactions. Note we have multiple in this test case!
	pact.
		AddInteraction().
		Given("User Matt exists"). // Provider State
		UponReceiving("A request to login"). // Test Case Name
		WithRequest(&Request{
			Method: "GET",
			Path:   "/login",
		}).
		WillRespondWith(&Response{
			Status: 200,
		})

	// Run the test and verify the interactions.
	err := pact.Verify(test)
	if err != nil {
		t.Fatalf("Error on Verify: %v", err)
	}

	// Write pact to file
	pact.WritePact()
}
```

如果这个测试顺利完成，Pact文件内容会写入`./pacts/my_consumer-my_provider.json`，其中包含了消费者端和提供者端的所有交互。