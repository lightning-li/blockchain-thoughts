#### 主流区块链简介

作者 ：李康

---

## 目录

[1. Bitcoin](#1)
  - [1.1 Blockchain](#11-blockchain)
  - [1.2 UTXO 模型](#12-utxo-模型)
  - [1.3 Script language based on stack](#13-script-language-based-on-stack)
  - [1.4 POW 共识算法](#14-pow-共识算法)
  - [1.5 Future](#15-future)

[2. Ethereum](#2)
  - [2.1 Account 模型](#21-account-模型)
  - [2.2 Transaction 模型](#22-transaction-模型)
  - [2.3 Block 模型](#23-block-模型)
  - [2.4 Smart Contract 与 EVM](#24-smart-contract-与-evm)
  - [2.5 POW 共识算法](#25-pow-共识算法)
  - [2.6 Future](#26-future)


[3. Hyperledger Fabric](#3)

[4. Zcash](#4)
  - [4.1 Zerocoin](#41-zerocoin)
  - [4.2 Zerocash](#42-zerocash)
  - [4.3 Zcash](#43-zcash)

[5. Corda](#5)

[6. Chain](#6)

---

<h1 id="1"> 1. Bitcoin </h1>

针对比特币网络中重要概念的讲解。

### 1.1 Blockchain

区块链一词起源于比特币网络。

![比特币区块链](/images/2016/12/bitcoin-blockchain.png)

由于网络传播因素存在，比特币区块链会分叉，但是矿工会根据最长链原则，在最长链之后进行挖矿，这样才有可能获得奖励。

### 1.2 UTXO 模型

UTXO : 未花费的输出。

![UTXO](/images/2016/12/spend-tx.png)

每一笔交易的输入都会引用先前有效交易的未花费输出，然后创建新的未花费输出。账户余额就是发往该账户地址的一笔笔未花费输出的累加。

### 1.3 Script language based on stack

- **P2PKH Script Validation**

![bitcoin-stack](/images/2016/12/bitcoin-stack.png)

 - **P2SH : Pay-to-Script-Hash **

    - P2SH 交易允许发送者创建一个包含另一脚本哈希值的 pubkey script，其中另一脚本被称为 redeem script ( 赎回脚本)。

    - p2sh 工作流如下 :
    ![bitcoin-p2sh](/images/2016/12/bitcoin-p2sh.png)

    Bob 按照需要创建一个赎回脚本，对该脚本进行哈希，将得到的哈希值传递给 Alice。Alice 创建一个 P2SH-style 的输出，包含 Bob 的赎回脚本哈希。

    当 Bob 花费该笔输出时，他提供 signature script，该解锁脚本中包含：signature、full serialized redeem script。比特币网络会保证解锁脚本中的赎回脚本的哈希值等于先前 Alice 放入输出中的赎回脚本哈希。验证通过后，比特币网络处理该赎回脚本，就像原先的 pubkey script 执行一样，Bob 能够花费该输出，如果赎回脚本没有返回错误。

- **MultiSig**

  - P2SH Multisig 一般用于多方签名交易。在一个 multisig pubkey script 中，称为 m-of-n，其中 n 是提供的公钥数量，m 是匹配公钥的签名最小数量。m 和 n 是操作码 OP_1 到 OP_16。

  - 由于比特币实现中的一个错误，OP_CHECKMULTISIG 会消费栈中 m + 1 个元素，因此在 signature script 中第一个操作码为 OP_0，注意 OP_0 被消费但是不会被使用。

  - signature script 必须按照 pubkey script 或者 redeem script 中的公钥顺序来提供签名，这是有操作码 OP_CHECKMULTISIG 决定的。

  - 实现多方签名的两种形式 :

  ![bitcoin-multi-pubkey](/images/2016/12/bitcoin-multi-pubkey.png)

  ![bitcoin-multi-p2sh](/images/2016/12/bitcoin-multisig-p2sh.png)

- **MicroPayment Channel**

  - Alice 为 Bob 审查稿件，每审查完一个稿件，Alice 希望立即得到报酬，Bob 却不想这样做，原因是大量的小额支付交易会花费他大量的手续费，因此 Alice 建议使用微支付通道。Bob 索要 Alice 的公钥，创建两笔交易，第一笔交易是将 100m BTC 发送到一个 2-of-2 的多方签名的赎回脚本中，该笔交易被称为 bond tx (纽带交易)。第二笔交易将这 100m BTC 扣除手续费过后如数返回给 Bob，但是这需要 Alice 的签名，且该笔交易需要 24 小时的延迟才能被确认，借助 locktime 字段 Alice 在 24 小时之前，可将最新版本的余额分配方案发送到比特币网络中，从而得到报酬。

  - 示意图如下 :

  ![bitcoin-micro](/images/2016/12/bitcoin-micro.png)

### 1.4 POW 共识算法

矿工监听来自比特币网络中的交易，将交易打包到区块中，此时会导致区块头中 merkle root hash 的改变，因此区块头的哈希也会发生改变。当然矿工也可以不必等待新交易的到来，因为区块头中还包含一个随机值 nonce，通过改变 nonce 值，区块头的哈希值也可以改变。矿工不断地尝试，直到得到的区块头的哈希值小于区块头中的 target threshold nBits。

理想地，大概每 10 分钟产出一个区块，由于矿工节点不断地加入与离开，难免会导致全网算力的增加或减少。因此需要进行难度调整。比特币的共识协议，将难度调整的周期定为 每 2016 个块，理想地，2016 个区块的产生需要两周时间，若过去 2016 个区块的时间大于两周，证明算力降低了，需要调低难度值，即增大 nBits；若时间小于两周，反之即可。

### 1.5 Future

- 区块扩容：由于比特币区块的大小有严格限制，导致每个区块容纳的交易笔数不高，使得比特币网络无法承载更多的交易需求。

- 闪电网络：链下建立可信的微支付通道，将最后的结算放入到比特币区块链中，需要比特币隔离见证的支持。

- 隔离见证：将比特币交易输入中的解锁脚本 signature script 拿出来，解决了交易延展性问题 (transaction-malleability)。同时可以减小交易的大小，使得在同样的区块大小限制中，允许更多的交易。

- 侧链：将比特币区块链与其它区块链链接起来，实现更大的价值转移。

<h1 id="2"> 2. Ethereum </h1>

### 2.1 Account 模型

以太坊采用 Account 模型，因此以太坊的 World State 可以体现在最新的区块中，而不用像比特币，需要回溯整个历史。

Account 有两种类型：externalised account 与 contract account。Account 由以下字段组成：(1) nonce : 在 externalised account 中，代表从该账户地址发送过的交易数量，该数量会出现在交易的字段中，起到防止双花的目的。在 contract account 中，代表由该账户创建过的合约数量；(2) balance : 该账户拥有的余额；(3) storageRoot : Merkle Patricia 树的 256 位根哈希，该树是一种 Merkle 与 Trie 树的混合体，是以太坊独有的概念。storageTree 包含了属于该账户的存储内容 (一个关于 256 位整数值之间的映射) 的编码；(4) codeHash : 属于该账户的 EVM code 的哈希值。当该账户地址接收到一个 message call 的时候，该 EVM code 会被 EVM 执行。不像账户的其它字段，codeHash 是不可变的，被用来从状态数据库中获得相应的 ECM code。

### 2.2 Transaction 模型

- **nonce :** 与该交易发送地址中的 nonce 相同，256 位；

- **gasPrice :** 交易执行过程需要消耗 gas，每一个单位 gas 的价值为 gasPrice Wei (1 Ether = 10 ^ 18 Wei)，256 位；

- **gasLimit :** 交易执行过程中能够消耗的最大 gas 的个数，为了防止垃圾交易的恶意攻击，gasPrice * gasLimit 数量的 Wei 需要提前在发送者的账户中扣除，256 位；

- **to :** 该 message call 的 160 位接收方地址；对于一笔 “合约创建” 交易，to 为空，160 位；

- **value :** 发送给接收者的 Wei 数量，对于 “合约创建” 交易，作为新创建的合约账户的余额，256 位；


- **v, r, s :** 与该交易签名有关的值，用来确定该交易的发送者，v 为 5 位，r、s 均为 256 位；

- **init :** 账户初始化过程中需要的无限制大小的字节数组，该字段为 “合约创建” 交易所特有。init 是一个 EVM code 片段，执行完成之后会返回另一个 EVM code body，每当接收到一个 message call (要么是一笔交易触发，要么是一个合约代码的内部执行所触发)，body 就会被执行。init 只会在合约账户创建过程中执行一次，随后该代码片段会被丢弃；

- **data :** message call 所需要的输入数据，无限制大小，为 message call 交易所特有。

### 2.3 Block 模型

**BlockHeader** 由以下结构组成：

- **parentHash :** 父区块头的 Keccak 256 位哈希值；

- **ommersHash :** 该区块的叔区块列表的 Keccak 256 位哈希值；

- **beneficiary :** 该区块所有交易手续费的接收者地址，即成功挖到该区块的矿工地址；

- **stateRoot :** 在区块中所有交易执行完成后，state trie 树的根哈希值，该树的叶子节点是 account，索引的 key 为 account 地址；

- **transactionsRoot :** 该区块中所有交易组成的 Merkle Patricia 树的根哈希值；

- **receiptsRoot :** 该区块中所有交易对应的所有的收据组成的 Merkle Patricia 树的根哈希值；

- **logsBloom :** 布隆过滤器，由 receipts 中每一个 log entry 中的 logger address 与 log topics 形成；

- **difficulty :** 该区块的难度值，由前一个区块的难度值以及时间戳计算得出；

- **number :** 该区块有多少个父区块，创世区块中该值为 0；

- **gasLimit :** 该区块中 gas 消耗的限制数量；

- **gasUsed :** 该区块中所有交易使用的 gas 数量；

- **timestamp :** 一个合理的该区块开始生成的 unix 时间戳；
extraData : 一个任意的与该区块相关联数据的字节数组，大小限制为 32 个字节；

- **mixHash :** 一个 256 位的哈希值，与 nonce 结合来证明在该区块上已消耗了足够多的计算能力；

- **nonce :** 一个 64 位的哈希值，与 mixHash 结合来证明在该区块上已消耗了足够多的计算能力；

**block 组成结构 :** (header, transaction list, ommer block headers list)

### 2.4 Smart Contract 与 EVM

**基于以太坊的域名注册系统**

![ethereum-name-register](/images/2016/12/ethereum-name-register.png)

**基于以太坊的代币系统**

![ethereum-token](/images/2016/12/ethereum-token.png)

**EVM**

EVM 是以太坊开发的基于栈的 Ethereum 虚拟机，智能合约的代码是跑在 EVM 上的。EVM 支持图灵完备，大大扩展了比特币的脚本语言功能，使得开发人员可以编写基于以太坊平台的任意功能的智能合约。来看**域名注册系统的 EVM 字节码分析 : **

```PUSH1 0 CALLDATALOAD SLOAD NOT PUSH1 9 JUMPI STOP JUMPDEST PUSH1 32 CALLDATALOAD PUSH1 0 CALLDATALOAD SSTORE```

该字节码程序实现了域名注册系统，任何人都可以发送一个包含 64 个字节 data 的消息，其中 32 个字节当作 key，即域名注册者；另外 32 个字节当作 value，即域名。合约检查 key 是否存在于合约账户的存储中，如若不存在，将其插入合约账户自身的存储中。

在执行过程中，一个无限可延伸的字节数组，称为 “memory”，程序计数器，指向当前要执行的指令，称为 “PC”，和一个基于 32 字节的栈被 EVM 所维护使用。开始执行时，PC = 0，内存与栈是空的。现在假设，一个消息被发送，消息包含 123 wei gas 费用与 64 字节的数据，头 32 个字节是数字 54 的编码，后 32 个字节是数字 2020202020 的编码。

因此，初始状态是 : {PC : 0, STACK : [], MEM : [], STORAGE : {}} ，位于 0 处的指令是 PUSH1，代表将一个字节的数据压入栈中，并且在 code 中跳跃 2 步，即现在的状态是 {PC : 2, STACK : [0], MEM : [], STORAGE : {}}。位于 2 处的指令是 CALLDATALOAD，代表从当前的栈中弹出一个值 index，装载位于消息 64 个字节数据中 index 位置处的 32 个字节，并且将该数据压入栈中，即现在的状态是 {PC : 3, STACK : [54], MEM : [], STORAGE : {}}。

位于 3 处的 SLOAD 指令，从栈中弹出一个值 index，将合约账户 storage 中索引 index 处的值压入栈中，由于该合约是第一次使用，因此值为0，即现在的状态为 {PC : 4, STACK : [0], MEM : [], STORAGE : {}}。位于 4 处的 NOT 指令从栈中弹出一个值，如果该值为 0 则将 1 压入栈中，反之将 0 压入栈中，即现在的状态为 {PC : 5, STACK : [1], MEM : [], STORAGE : {}}。位于 5 处的指令 PUSH1 执行完成之后，状态变为 {PC : 7, STACK : [1, 9], MEM : [], STORAGE : {}}。

位于 6 处的指令 JUMPI 从栈中弹出两个值，a1 : 9，a2 : 1，如果第二个值 a2 为非 0 值，则跳转到 a1 指定的值。如果合约账户的 storage 的索引 54 处的值不为 0，那么 a2 将会变为 0 (由于 NOT 指令)，那么我们就不会跳转到 a1 指定的值，那么接下来的指令就将会是 STOP，从而中止代码的执行。现在的状态是 {PC : 9, STACK : [], MEM : [], STORAGE : {}}。

位于 9 处的指令 JUMPDEST 不会对虚拟机状态造成任何影响，仅仅是标记一个有效的跳转地址，PC 变为 10，接下来位于 10 处的 PUSH1 执行完成之后，状态变为 {PC : 12, STACK : [32], MEM : [], STORAGE : {}}。位于 12 处的指令 CALLDATALOAD 执行完成之后，从栈中弹出一个值 index : 32，然后将消息中 64 字节数据中位于索引 32 处之后的 32 个字节压入栈中，现在状态变为 {PC : 13, STACK : [2020202020], MEM : [], STORAGE : {}}。接下来执行 13 处的 PUSH1，状态为 {PC : 15, STACK : [2020202020, 0], MEM : [], STORAGE : {}}。

位于 15 处的 CALLDATALOAD，从栈中弹出索引值 0，再次将消息数据中前 32 个字节数据压入栈中，此时状态变为 {PC : 16, STACK : [2020202020, 54], MEM : [], STORAGE : {}}。位于 16 处的 SSTORE 操作从栈中弹出两个值 a1 : 54，a2 : 2020202020，存入合约账户的存储中，即 {PC : 17, STACK : [], MEM : [], STORAGE : {54 : 2020202020}}，位于 17 处没有任何指令，因此 EVM 中止运行。

### 2.5 POW 共识算法

以太坊中使用 pow 算法是基于 Dagger-Hashimoto 的一个修改版本，称为 **Ethash**。与比特币pow 类似，是一个不断寻找 nonce 的过程，直到得到的结果小于 difficulty 阈值；与比特币 pow 不同的是，Ethash pow 是特消耗内存的，这样使得它可以抵抗 ASIC 矿机。这意味着计算 pow 需要依赖于 nonce 与 header 的一个固定资源，这个资源 (几个 G 大小) 称为 DAG。每 30000 个块 (大概 100 小时，称为一个 epoch) DAG 是完全不同的，需要消耗一定的时间来生成。既然 **DAG** 只依赖于区块高度，它可以预先被生成，如果不这样做的话，那么矿工就必须等待 DAG 生成之后才能继续挖矿。

**注意 :** 在验证 pow 是否有效的时候，不需要 DAG 的参与，因此允许低内存与低 CPU 的客户端对 pow 进行有效性的验证。

**Mining Rewards :** 成功挖到区块的矿工接收一下奖励 :

- 成功挖到区块的固定奖励 5 Ether；

- 执行交易过程中，消耗的 gas，从交易发起者账户中扣除，以太坊网络协议发送给矿工；

- 包含叔区块到区块中的额外奖励，以被包含的叔区块的 1/32 作为额外奖励。注意：最多往上追溯 6 代，有效的叔区块被奖励，以中和网络滞后带来的挖矿奖励分散的影响，7/8 的奖励会发送给叔区块的缔造者，即 4.375 个 Ether。另外，每个块至多允许包含 2 个叔区块，而且叔区块只能被包含一次。

### 2.6 Future

- 从 POW 过渡到 POS Casper 算法，将在 Ethereum Serenity 即 2.0 中实现。

- 实现 sharding 技术，提高区块链目前的 scalability，也将在 Ethereum Serenity 即 2.0 中实现。

<h1 id="3"> 3. Hyperledger Fabric </h1>

**[戳这里 !!!](https://fabric.iethpay.com)**

<h1 id="4"> 4. Zcash </h1>

### 4.1 Zerocoin

**Zerocoin** 可以被看做成一个去中心化的混合器，它应用了零知识证明来阻止交易图谱的分析。Zerocoin 证明 coins 是有效的，通过以零知识形式地证明 coins 属于一个公开的有效 coins 列表 (维护在区块链中)。但是 Zerocoin 不是一个成熟的匿名货币，它更像是一个**去中心化的混合器 (decentralised mix)**，用户可以借助 Zerocoin 协议周期性地清洗他们的比特币。日常的交易还是需要以比特币正常交易形式进行，由于以下原因：

- **性能。**赎回 Zerocoin 需要 double-discrete-logarithm 知识证明，该知识证明的大小超过 45KB，需要花费 450ms 时间来验证 (在 128 位的安全水平)。这些证据需要在整个网络中广播，被每一个节点验证，永久地存储在区块链中。所需的花费比比特币中正常交易花费要高出几个数量级，给正常运行的比特币网络带来严重的负担。

- **功能性。**首先，Zerocoin 使用固定面值的 coins：它既不支持精确价值的付款，也不提供一种方式找零。其次，Zerocoin 没有提供用户向另一用户直接以 "zerocoin" 形式付款的机制。最后，尽管 Zerocoin 通过断开一笔交易与它的原始地址的关联提供了匿名性，但是却没有隐藏交易的金额以及其它关于发生在网络中的交易的元数据。
Zerocoin 扩展了比特币协议，通过创建两种新交易类型：mint 与 spend。一笔 mint 交易允许用户交换一定数量的比特币来发行一个新的 zerocoin。每一个 zerocoin 由基于随机序列号 sn 的数字承诺 cm 组成。随后的某一个时刻，一个用户发行一笔 spend 交易，包含目的实体，序列号 sn，与一个针对 NP 陈述 {"我知道秘密 cm 与 r 满足 (i) cm 能由 sn 与 承诺随机数 r 得出，并且 (ii) cm 在过去某一个时刻被发行 (mint)"} 的非交互性零知识证明。至关重要地，零知识证明没有将 spend 交易与特定的 mint 交易 (目前为止的所有 mint 交易) 相关联。如果证明验证正确，并且 sn 先前没有被使用过，那么该协议就会将一定数量的比特币转到目的地址。Zerocoin 表现的就像是一个去中心化的混合器。

### 4.2 Zerocash

我们构建一种**去中心化匿名支付模式 (decentralized anonymous payment (DAP) scheme)**，是一种去中心化的电子货币模式，允许任意数量的直接匿名转账付款。接下来以 6 个功能逐渐递增的步骤来刻画 DAP 的构建过程 (基于任何基于 ledger 的货币，例如比特币)：

- **Step 1 :  user anonymity with fixed-value coins。**首先给出一个简化的构建，所有的 coins 有相同的价值，例如 1 BTC。这个构建模式与 Zerocoin 类似，展现如何隐藏一笔转账付款的原始地址。我们使用 **zk-SNARKs** 和 承诺 (commitment) 模式。令 COMM 为统计学上隐藏地非交互性承诺模式 (例，给定一个随机数 r 与 消息 m，承诺是 cm := COMMr(m)；随后，可以通过披露 r 与 m 来验证 COMMr(m) 是否等于 cm )。在该简化构建中，一个新币 c 按照如下方式发行：用户 u 随机选一个序列号 (serial number) sn 与 trapdoor r，计算出 coin commitment cm := COMMr(sn)，并且令 c := (r, sn, cm)。一笔相应的发行交易 txMint，包含 cm (却既不包含 sn 也不包含 r)，被发送到 ledger 中；只有在用户 u 已经付了 1 BTC 到一个支撑的保管池后，txMint 才能够被追加到 ledger中。txMint 就像是存款的凭证一样，可以从支撑池中获取它们的价值。

  随后，令 CMList 为 ledger 中所有 coin commitment 的列表，u 可以发送一笔花费交易 txSpend 来花费 c，txSpend 包含 (i) coin  的序列号 sn；(ii) 一个 zk-SNARKs 证明 π 针对于 NP 陈述 “ 我知道 r 使得 COMMr(sn) 出现在 CMList 中”。假设 sn 不曾在 ledger 中出现过，用户 u 就能够赎回 1 BTC，这 1 BTC 用户 u 可以随意支配，保留、转账、发行新币等。如果 sn 曾经出现过，则该笔交易会被认为是双花交易，裁定为无效并且被丢弃。

  该构建方法实现了用户匿名性，因为证明 π 是零知识的：尽管 sn 被披露，但是没有任何关于 r 的信息。并且寻找花费交易 txSpend 所对应的发行交易 txMint 相当于寻找 f(x) := COMMx(sn)，被认为是不可行的，因此转账付款的起始地址得到了保护。


- **Step 2 : 压缩 CMList。**step 1 提到的 CMList 极大地限制了证明确认算法的扩展性，由于时间和空间上的复杂度。由于 CMList 会随着新币的发行不断增长，并且已被花费的 coin  对应的 commitment 不能从 CMList 中删除，由于零知识提供的匿名性。
我们应用一种抵抗冲突的函数 CRH 来避免 CMList 的显示表达形式，即我们在 CMList 上维护一棵基于 CRH 的 Merkle 树，令 rt 代表 Tree(CMList)。众所周知，rt 可以在于树深度成比例的时间与空间复杂度规模内通过插入叶子节点即 coin commitment 来进行更新。因此时间与空间复杂度从原先的 CMList 的长度，降低到对数范围。利用该思想，我们修改上述的 NP 陈述为：”我知道 r 使得 COMMr(sn) 作为一个叶子出现在一棵根为 rt 基于 CRH 的 Merkle 树中”。这使得 zk-SNARKs 算法可以支撑更多的 coin commitments。

- **Step 3 : 扩展 coins 使得能够支持直接地匿名转账付款。**目前为止，coin commitment cm 是相对于币序列号 sn 的承诺。但是，却带来一个很严重的问题。假设用户 Alice 创建了 coin c，Alice 想将 c 转移给 Bob，Bob 能够花费该 coin 的前提是 Alice 将手中的 r 与 sn 传递给自己，这样一来 Bob 花费 c 的行为就不再匿名，因为 Alice 也知道 sn，Alice 可以监听网络中的花费交易来匹配 sn。除此之外，Alice 也能够花费该 c 除非 Bob 一拿到 Alice 的 r 与 sn 就花费掉 c。所以 Step 1 所提出的简单构建不能满足 DAP 的需要。

  我们通过修改 coin commitment 的获取方式解决了上述问题，使用伪随机函数来获取序列号与定位付款目标。我们使用了 3 个伪随机函数 (从单个伪随机函数派生出来)。对于一个种子 x，3 个伪随机函数的表现形式为 PRF[x][addr]、PRF[x][pk]、PRF[x][sn]。

  为了提供付款的目标，我们使用 address 的概念：每一位用户生成一个地址密钥对 (apk, ask)。用户的 coins 需要包含 apk，而且只有知道 ask 的情况下才能花费 coins。ask 是通过随机挑选生成的，令 apk := PRF[x][addr]\(0)，用户可以生成任意个地址密钥对。

  接下来，我们重新设计了发行交易来允许更大的功能性。为了发行一个想要的价值 v 的 coin，用户 u 首先挑选了一个随机数 ρ，ρ 是一个决定序列号 sn 的安全值，sn := PRF[x][sn]\(ρ)。接下来 u 使用两阶段提交 (apk, v, ρ)：(i) 用户 u 计算 k :=  COMMr(apk || ρ) 对于随机挑选的 r 值；(ii) 用户 u 计算 cm := COMMs(v || k) 对于随机挑选的 s。发行结果是 coin c := (apk, v, ρ, r, s, cm) 和一笔发行交易 txMint := (v, k, s, cm)。任何人都可以通过 COMMs(v || k) 来验证 cm 是否是一个价值为 v 的 coin 的承诺，但是却不能分辨出拥有者 apk 与 序列号 sn (从 ρ 得出)，因为这些都统统被隐藏在了 k 中。和以前一样，只有用户 u 花费了等量数值 v 的 BTC，txMin 才可以被 ledger 所接受。

  pour 交易花费 coins，使用想要花费的 coins 作为输入，生成新鲜的 output coins，output coins 的总价值要等于 input coins 的总价值。假设持有地址密钥对 (ask, apk) 的用户 u 想要花费他的 coins c_old = (apk_old, v_old, ρ_old, r_old, s_old, cm_old)，产生两个 new coins c1_new, c2_new，并且 v_old = v1_new + v2_new，相应的两个目标地址为 apk1_new、apk2_new (可以属于 u 或者任何其它用户)。对于 i ∈ {1, 2}，用户 u 执行下列操作：
  - (Ⅰ) u 随机挑选能够获得序列号的随机数 ρi_new；

  - (Ⅱ) u 计算 ki_new := COMMri_new(apki_new || ρi_new)，对于随机挑选的 ri_new；

  - (Ⅲ) u 计算 cmi_new := COMMsi_new(vi_new || ki_new)，对于随机挑选的 si_new。

  这就产生了新币 c1_new := (apk1_new, v1_new, ρ1_new, r1_new, s1_new, cm1_new) 与 新币 c2_new := (apk2_new, v2_new, ρ2_new, r2_new, s2_new, cm2_new)。接下来，用户 u 产生一个 zk-SNARK 证明 **πPOUR**，对应于下列我们成为 **POUR 的 NP 陈述**：
```
  给定 Merkle-Tree 根 rt，序列号 sn_old，与 coin commitments cm1_new，cm2_new，我知道 coins c_old，c1_new，c2_new 与地址密钥 ask_old 使得：

  1. coins 是正确构建的：对于 c_old，下列成立：k_old = COMMr_old(apk_old || ρ_old) 与 cm_old = COMMs_old(v_old || k_old)；c1_new 与 c2_new 也是类似；

  2. 地址私钥匹配地址公钥，即 apk_old = PRF[ask_old][addr]\(0)；

  3. 序列号被正确计算：sn_old = PRF[ask_old][sn_old]\(ρ_old)；

 4. coin commitment cm_old 出现在根为 rt 的 Merkle 树的叶子节点中；

  5. 价值转移相等：v_old = v1_new + v2_new。
```
  即 pour 交易的形式为 **txPour := (rt, sn_old, cm1_new, cm2_new, πPOUR)**，被添加到 ledger 中，和之前一样，如若 sn_old 在先前的一笔交易中出现，那么该交易就会认为是双花而被拒绝进入 ledger。

  如若用户 u 不知道与 apk1_new 相关联的 ask1_new，那么 u 是不能花费 c1_new，因为 u 无法在随后的 pour 操作的见证中提供 ask1_new，即无法生成合格有效的 πPOUR。还有就是当拥有 ask1_new 的用户花费 c1_new 的时候，用户 u 是无法追踪的，因为 u 不知道 c1_new 对应的序列号 sn1_new (sn1_new := PRF[ask1_new][sn]\(ρ1_new) )。

  注意，txPour 没有披露任何关于 c_old 的价值是如何分配到 ci_new 的信息，也没有披露 c_old 对应花费的是哪一笔 cm_old 的信息，还没有披露 ci_new 对应的目标地址公钥的信息，因此，转账是以完全匿名的方式进行的。

  更简单地且不是通用性地，我们设置 Nnew = Nold = 2。当 Nold < 2 的时候，用户可以发行一个价值为 0 的 coin，然后将该 coin 作为 null input；当 Nnew < 2 的时候，用户可以发行一个价值为 0 的 new coin。对于 Nold > 2 或者 Nnew > 2，用户可以构建 log Nold + log Nnew 个2-input/2-output pours。

- **Step 4 : 发送 coins。**假定 apk1_new 是用户 u1 的地址公钥。为了允许 u1 能够花费上述产生的 c1_new，用户 u 必须将 c1_new 中的某些安全值发送给用户 u1。一种方法是用户 u 给 u1 发送一条私密消息，但是依赖于一条可信的私密通道，无疑增加了使用的负担。我们避免了链下的私密通信，将这种能力嵌入到 zerocash 的构建过程。

  - 首先，我们修改了地址密钥对的结构。现在每一位用户拥有一个密钥对 (addr_pk, addr_sk)，其中 addr_pk := (apk, pk_enc)，addr_sk := (ask, sk_enc)，(apk, ask) 的生成办法同先前一样，(pk_enc, sk_enc) 是一个 “key-private encryption scheme” 密钥对。所谓的 key-private 意思是，密文不能被链接到加密它们的公钥，或与使用同一把公钥加密得到的其它密文联系起来。

  - 然后，用户 u 使用 pk_enc1_new 加密明文 (v1_new, ρ1_new, r1_new, s1_new) 得到密文 C1，并且将 C1 加进 txPour 中。用户 u1 然后可以找到并且使用 sk_enc1_new 解密 C1 从而还原 (v1_new, ρ1_new, r1_new, s1_new)，注意用户 u1 需扫描 ledger 中的 txPour 消息。再次注意，密文 C1 的出现没有泄露任何关于目的地址、转账金额的信息，由于加密方法的 key-private 特性。(用户 u 对 c2_new 进行同样的操作，将密文 C2 加入到 txPour。)

- **Step 5 : 公开的输出。**到目前为止，构建方法允许用户发行、合并、分裂 coins。但是一个用户如何赎回他 coins 的价值，例，将它转换为基础货币 (BTC)？为了达成这个目的，我们修改了 pour 操作以包含一个 public output。当花费一个 coin 的时候，用户 u 同时也可以设置一个非负值 v_pub 和一个交易描述 info ∈ {0, 1}*。NP 陈述 POUR 中的余额计算方式相应发生改变：v1_new + v2_new + v_pub = v_old。如此，输入 v_old 中的一部分 v_pub 被公开地声明，并且 v_pub 的目标接收地址也被披露，在 info 里。info 可以被用来具体定义赎回金额的目的地址 (在 Bitcoin 中，是 public key)。v_pub 与 info 被包含在 交易txPour 中。(v_pub 是可选的，用户可以将其设置为 0)

- **Step 6 : non-malleability。**为了防止对于交易 txPour 的延展性攻击 (例如，通过修改 info 来使 v_public 的输出地址发生改变)，我们修改该 NP 陈述 POUR，并且使用数字签名。具体地，在 pour 操作过程中，用户 u

  - (i) 对于一次性签名模式，随机生成一个密钥对 (pk_sig, sk_sig)；

  - (ii) 计算 hSig := CRH(pk_sig)；

  - (iii) 计算两个值 h1 := PRF[ask1_old][pk]\(hSig) 与 h2 := PRF[ask2_old][pk]\(hSig)，作用是作为消息验证码将 hSig 绑定到两个地址的私钥上；

  - (iv) 修改 POUR 来包含 hSig、h1、h2，并且证明后两者被正确地计算出来；

  - (v) 使用 sk_sig 来对相关的值进行签名，获得签名 σ，与 pk_sig 一起放入 txPour 交易中。既然 aski_old 是私密的，并且每一次 pour 交易，hSig 都会改变，因此 h1 与 h2 是不可预测的。

![zerocash_setup](/images/2016/12/zerocash_setup.png)

![zerocash_createaddress](/images/2016/12/zerocash_createaddress.png)

![zerocash_mint](/images/2016/12/zerocash_mint.png)

![zerocash_pour](/images/2016/12/zerocash_pour.png)

![zerocash_txpour](/images/2016/12/zerocash_txpour.png)

![zerocash_paipour](/images/2016/12/zerocash_paipour.png)

![zerocash_cmrt](/images/2016/12/zerocash_cmrt.png)

![zerocash_verifytx](/images/2016/12/zerocash_verifytx.png)

![zerocash_receive](/images/2016/12/zerocash_receive.png)

![zerocash_dap](/images/2016/12/zerocash-dap.png)

### 4.3 Zcash

**Zcash** 是 **Zerocash 协议**的一种具体实现，相比于原始论文 **Decentralized Anonymous Payment schem Zerocash**，Zcash 进行了一些安全问题上的修复，以及一些性能、功能性、术语方面的调整。

Zcash 与 Zerocash 不同之处 :

- 加密密钥对获取方式不同。Zcash 传输消息时的加密采用的是认证过的一次性对称加密，而 Zerocash 采用的加密是非对称加密方法，具体见 Zcash 协议 4.1.3 及 4.1.1 节；

- Zcash 在原有 Zerocash 的 output 的接收者增加了 Memo 字段，见 Zcash 协议 7.2 节；

- Zcash 将 Zerocash 中的 mint 与 pour 两种交易类型统一为一种交易类型，见 Zcash 协议 7.3 节；

- 修复了 ρ 的生成方法，防止了 Faerie Gold attack，具体见 Zcash 协议 7.4 节；

- 修复了内部使用的哈希函数碰撞攻击，见 Zcash 协议 7.5 节；

- 更多见 Zcash 协议 7.6、7.7、7.8 节。

<h1 id="5"> 5. Corda </h1>

**[戳这里 !!!](corda_intro.md)**

<h1 id="6"> 6. Chain </h1>

**[戳这里 !!!](https://chain.iethpay.com/)**
