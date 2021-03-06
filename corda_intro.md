#### Corda

##### 1. Corda 简介

*Corda* 是一个去中心化的数据库平台，具有以下新奇特性：

- 可以使用 JVM 字节码定义新交易类型。

- 交易可以在不同的节点上并发执行，无需节点知晓发生在其它节点上发生的交易。

- 节点以认证的 p2p 网络形式组织。所有的交流通信是直向连接的。

- 没有区块链。使用可插拨的 *notaries* 来解决交易竞争引起的冲突。一个 Corda 网络中可能会包含多个 notaries，它们使用多种多样的不同算法来提供它们的保证。如此，Corda 没有被绑定到任何特定的共识算法。

- 数据基于 **need-to-know** 共享。当节点向其它节点发送一笔交易时，节点按需提供该笔交易的依赖图谱，但是没有交易全局广播的概念。

- **Bytecode-to-bytecode 译码**被用来允许复杂、多步的交易构建协议，称为 *flows*，建模为阻塞码。代码被转换为一个异步状态机，当消息发送和接收的时候，checkpoints 被写入到节点支撑的数据库中。一个节点一次可能会有成千上万个活跃的 flows，并且它们可能会持续数天，跨越节点重启，甚至升级等过程。Flows 可以将进展信息暴露给节点管理员、用户，并且可能会与人和其它节点进行交互。一个 Flow library 被提供给开发者，以复用公共的 Flow 类型，例如 notarisation，membership broadcast 等等。

- 数据模型允许允许任意的对象图谱存储在 ledger 中。这些图谱被称为 *states*，并且是数据的原子单位。

- 节点由关系型数据库支撑，ledger 中存放的数据可以使用 SQL 语句查询，以及与私有表数据连接，由于状态定义中的 slots，被保留用来 join keys。

- 平台提供了一个富类型系统来代表诸如日期、货币、合法实体以及金融实体，例如现金、保险、合同等等。

- States 能声明一个关系映射，能够使用 SQL 来进行查询。

- 与现有的系统集成从一开始就被考虑进来。Corda 网络支持从其它数据库系统快速导入数据，而且对网络不会造成负担。ledger 上的事件通过内嵌的 JMS 兼容消息代理来通知。

- States 可以声明调度的事件。例如，一个债券状态如果没有及时被支付，可以声明转移到一个默认的状态。

##### 2. Corda 总括

Corda 使用 UTXO 模型，不同于比特币的是，Corda 网络中的交易必须同时满足输入和输出中的程序脚本 (比特币中，只需 txin 中包含满足引用的先前交易的输出中scriptPubKey 的 scriptSig)，Corda 的 *states* (类似于 utxo) 可以包含任意的数据，而不仅仅是值字段。*Issuance transctions* 可以添加新的 state 到数据库中，却不用花费任何已经存在的 state。但是与比特币不同的是，这些交易没有什么特殊的地方，能被任何人在任何时间创建。

Corda 没有使用区块链来对交易进行排序。Corda 中每一个 state 指向一个 *notary*，notary 是一个服务，它会保证，如果该笔交易中所有的 input states 是未花费的，它就会对该笔交易签名。一笔交易不允许花费被多个 notaries 控制的 states，如此，在 notaries 之间没有任何两阶段提交。如果跨 notaries 的 states 结合出现了，那么首先一个特殊的交易类型被用来将它们移动到一个单 notary 中。

#### 3. p2p 网络

##### 3.1 网络总括

一个 Corda 网络包括以下组件：

- 节点使用 TLS 之上的 AMQP/1.0 来通信。节点使用关系型数据库来存储数据

- 一个权限服务，自动化执行提供 TLS 证书的过程。

- 一个网络映射服务，在网络中发布节点的信息。

- 一个或多个 notary 服务，一个 notary 本身可能是分布式的。

- 0 或 多个 oracle 服务，一个 oracle 是一个众所周知的服务，如果它陈述一个事实，并认为给事实是正确的，它就会对交易签名。它们也许会提供事实，这也是 ledger 与 真实世界连接的通道。

##### 3.2 身份与权限服务

##### 3.3 网络映射服务
