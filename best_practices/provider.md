＃ 提供者使用Pact的最佳实践


### 确保最新的pact契约得到验证

* 使用最新的pact契约访问地址。
> Use a URL that you know the latest `Pact` will be made available at.

* 不要依赖手动干预（例如，将文件复制到“提供者”项目中）。因为这个步骤将分解或中断验证过程，导致某些验证任务错误。
> Do not rely on manual intervention (eg. someone copying a file across to the `Provider` project) because this process will inevitably break down, and your verification task will give you a false positive.

* 不要试图通过手动的方式更新pact契约，这个过程无法“保护”验证的完整性。
> Do not try to "protect" your build from being broken by instigating a manual pact update process.

* `pact verify`是集成验证的金丝雀方式 - 而手动更新则像给金丝雀戴上防毒面具，会另它失效 :)
> `pact:verify` is the canary of your integration - manual updates would be like giving your canary a gas mask.

### 确保Pact测试作为CI构建的一部分
> Ensure that Pact verify runs as part of your CI build

它应该与所有其他测试一起运行
> It should run with all your other tests.

### 只stub那些请求内容已经被验证过的
> Only stub layers beneath where contents of the request body are extracted

运行`pact：verify`，如果没有在`提供者`中stub任何数据，那就证明不需要。
> If you don't have to stub anything in the `Provider` when running `pact:verify`, then don't.


如果您需要stub某些内容（例如为下游系统），请确保只有在提取和验证请求体的内容后，在stub相关内容。否则，可能会在“POST”或“PUT”中发送旧的内容，但不会另测试失败。
> If you do need to stub something (eg. a downstream system), make sure that you only stub the code that gets executed after the contents of the request body have been extracted and validated. Otherwise, you could send any old garbage in a `POST` or `PUT` body, and no test would fail.

### Stub调用下游系统

考虑在下游系统中使用单独的“Pact”，并使用共享测试套件。
> Consider making a separate `Pact` with the downstream system and using shared fixtures.
