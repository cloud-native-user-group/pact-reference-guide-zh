# 路线图

Pact是一个开源项目，需要依靠众多开发者的个人贡献。这也意味着以固定日期去交付特性几乎是不可能的。

不过话说回来，虽然每种语言都有各自的发展路线， 但是，作为一个社区我们在朝着同一方向一致努力。

## 规范 [状态: v3]
为保证不同语言间的互操作性，[Pact规范](https://github.com/pact-foundation/pact-specification/)规定了每个发布的主要版本的特性。当前最新被认可的版本是v3。

## 参考库 [状态: Alpha]

Pact的重要优势之一是具有各种语言的原生DSL，从而实现工具链之间的无缝集成。然而这也是其最大的挑战——规范一旦有任何变化，每种语言就得为其消费者端和提供者端的DSL实现复杂的匹配与验证逻辑。

Pact社区已经召集到一起来解决这个问题。我们的计划是用Rust创建一个原生的[嵌入式库](https://github.com/pact-foundation/pact-reference/)，具有良好定义的原生接口，每种语言可以吸收进来，从而完成公共的功能，例如：

* 运行模拟服务
* 执行匹配逻辑
* 验证契约

该项目正处于确认方法可行性的概念验证阶段，目前进展顺利，而且基本完成与v1版本的匹配。

## Node/JS [状态: Beta]

<p style="text-align: center;">
<iframe src="https://docs.google.com/presentation/d/1cbf1mJ1cpvi_xAn83NdX9eXfrcTtXLoZD__Qr_hCyjc/embed?start=false&loop=false&delayms=3000" frameborder="0" width="700" height="422" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>
</p>

## Golang [状态: Beta]
Golang的状况与JavaScript社区相同，都在底层用到了Ruby的“二进制文件”。我们正在致力于一个新版本的发布，移除这个依赖，以便支持上面提到的参考库。然而现在功能上仅支持到v2版本的规范，API还需要修改。

<p style="text-align: center;">
<iframe src="https://docs.google.com/presentation/d/1MIhu9dwVtWsOp-QfWSxU7uNzyDtDaHhVDxWQKzleEyQ/embed?start=false&loop=false&delayms=3000" frameborder="0" width="700" height="422" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>
</p>
