### zk-SNARK 算法简介

zk-SNARK 是 zero-knowledge succint non-interactive arguments of knowledge 的简称，其中：

- Succint (简洁性) : 与实际计算的长度相比，生成的零知识证据消息很小。

- Non-interactive (非交互性) : 几乎没有任何交互。对于 zk-SNARK 算法来说，通常有一个构建阶段，构建阶段完成之后，证明者 (prover) 只需向验证者 (verifier) 发送一个消息即可。而且，SNARK 通常还有一个被称作是 "公开验证者" 的特性，意味着任何人无需任何交互即可验证零知识证据，这对区块链是至关重要的。

- Arguments (争议性) : 验证者只能抵抗计算能力有限的证明者的攻击。具有足够计算能力的证明者可以创建伪造的零知识证据以欺骗验证者 (**注意**刚提到的足够的计算能力，它足可以打破公钥加密，所以现阶段可不必担心)。这也通常被称为 "计算完好性 (computational soundness)"，而不是 "完美完好性 (perfect soundness) "。

- of Knowledge : 对于一个证明者来说，在不知晓特定证明 (witness) 的前提下，构建一个有效的零知识证据是不可能的。

zero-knowledge 前缀说明了在验证过程中，验证者除了知道证明者的陈述是正确有效的，不能学习到任何关于该论述的内容，举例来说，在 zcash 中，矿工知道一笔交易是有效的，但是却不知道这笔交易的发起者，接收者以及转账金额等关键性隐私信息。

### 一、了解 zk-SNARK 算法
我们首先来了解一下将 zk-SNARK 算法应用到实际场景中需要经过哪些步骤 (参考 vitalik 在 medium 发布的关于 zk-SNARK 算法的一系列文章 https://medium.com/@VitalikButerin/quadratic-arithmetic-programs-from-zero-to-hero-f6d558cea649 , https://medium.com/@VitalikButerin/exploring-elliptic-curve-pairings-c73c1864e627 , https://medium.com/@VitalikButerin/zk-snarks-under-the-hood-b33151a013f6 有兴趣和能力的朋友应该去读一读，深入浅出的文章，我正是看了这些文章才入的门)：

- **验证规则到 R1CS 形式的转换：** 实际场景中，大部分问题但是 NP 问题，即可以在多项式时间内验证结果的正确性，zk-SNARK 算法适用于所有的 NP 问题，目前还不清楚，对于 NP 之外的问题，zk-SNARK 算法是否适用。我们要做的就是确定问题的验证规则，例如在比特币中，一笔交易是否有效的问题就是 NP 问题，验证规则主要是，输入的金额是否大于等于输出的金额，这笔交易是否有合适的签名，输入是否属于 UTXO等。所谓的 R1CS (rank-1 constraint system) 就是一系列三元组向量 (a, b, c)，它是验证规则的数学形式体现，如何将验证规则转换为 zk-SNARK 友好的 R1CS 形式，可查看 vitalik 的文章，这里就不再赘述。

- **R1CS 到 QAP 的转换：** QAP 是 Quadratic Arithmetic Programs 的简称。通过拉格朗日插值法或者快速傅里叶变化，我们可以将 R1CS 转换一系列成多项式的形式，使得 A(x) * B(x) - C(x) = H(x) * Z(x) 成立，零知识证据的证明主要围绕着 QAP 展开。

其实 zk-SNARK 算法背后还涉及了有限域、椭圆曲线等知识，最重要的椭圆曲线结对的使用，大家有兴趣可取寻 vitalik 的文章查看。

### 二、如何使用 zk-SNARK 算法

zk-SNARK 算法的主要作者们写了一个 [libsnark](https://github.com/scipr-lab/libsnark) 库，提供了一些基础组件，zcash 用的零知识证明库就是 libsnark 的一个 fork 版本。我本人刚开始看到这个库时也是一筹莫展，即使在我了解了这么多关于 zk-SNARK 背后的技术知识后。而后，寻得一个作者编写的简单[例子](https://github.com/ebfull/lightning_circuit)，才基本入了门。最后，自己也加了一些复杂的功能，放在了我的 [github](https://github.com/lightning-li/zkSNARK-toy) 上。
