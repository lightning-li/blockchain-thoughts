#### ciphertext indistinguishability

密码学中最常使用的不可分辨定义分为 3 种：
1. indistinguishability under chosen plaintext attack (IND-CPA)；

2. indistinguishability under (non-adaptive) chosen ciphertext attack (IND-CCA1)；

3. indistinguishability under adaptive chosen cipertext attack (IND-CCA2)。

后一种定义地安全性要强于且包含前一种定义。

#### IND-CPA
对于随机非对称密钥加密算法来说，IND-CPA 被定义为一种对手与挑战者之间的游戏。对于基于计算安全性的模式来说，对手被建模为随机多项式时间图灵机，意味着对手必须在多项式时间步数内完成游戏并且输出一个猜测。E(PK, M) 代表 PK 对 消息 M 的加密消息：
1. 挑战者基于某个安全参数 k (密钥位数) 生成密钥对 <PK, SK>。挑战者保留 SK；

2. 对手可能执行多项式地加密操作和其它操作；

3. 最终，挑战者提交两个不同的明文 M0、M1 给挑战者；

4. 挑战者均匀地随机选择一个 b ∈ {0, 1}，发送挑战密文 C=E(PK, Mb) 给对手；

5. 对手可以自由地执行任何数量的附加计算和加密操作。最终，它输出一个 b 值的猜测。

如果每一个随机多项式时间的对手相比于随机猜测仅仅有极小的优势时，那么一个加密系统是 IND-CPA 的。也即对手赢得该游戏的可能性为 1/2 + ε(k)，ε(k) 是一个相对于安全参数 k 来说的一个极小函数。

尽管对手知道 M0、M1、PK，加密函数的随机性意味着 Mb 的密文仅仅是众多有效密文中的一个。因此加密 M0、M1，然后将得到的密文与挑战密文比较，不会增长任何优势。

上述的定义是基于非对称密钥加密系统来说的。可以被调整为对称密钥加密系统中，通过将公钥加密函数替换为 “encryption oracle”，它保留安全加密密钥，对来自于对手的请求中的任意数据进行加密。

#### IND-CCA1, IND-CCA2
IND-CCA1、IND-CCA2 使用了和 IND-CPA 相似的定义。然而，除了公钥 (在对称密钥加密系统中，encryption oracle)，对手有权利访问 decryption oracle，它可以解密任何来自对手请求的数据。在 non-adaptive 的定义中，对手被允许查询 decryption oracle 直到它接收到挑战密文。在 adaptive 的定义中，对手可以继续查询 decryption oracle 即使在它收到了挑战密文之后。但是不允许将挑战密文发送给 decryption oracle 来解密。

1. 挑战者基于某个安全参数 k (密钥位数) 生成密钥对 <PK, SK>。挑战者保留 SK；

2. 对手可能执行任意数量的加密操作，基于任意的密文调用 decryption oracle，和其它的操作；

3. 最终，挑战者提交两个不同的明文 M0、M1 给挑战者；

4. 挑战者均匀地随机选择一个 b ∈ {0, 1}，发送挑战密文 C=E(PK, Mb) 给对手；

5. 对手可以自由地执行任何数量的附加计算和加密操作。
	- 在 non-adaptive (IND-CCA1) 中，对手不会再进行 decryption oracle 的调用；

	- 在 adaptive (IND_CCA2) 中，对手也许会再次进行 decryption oracle 的调用，但是不会提交挑战密文给 decryption oracle；

6. 最后，对手输出一个 b 值的猜测。

如果没有对手有一个明显的优势赢得该游戏，那么该模式就是 IND-CCA1 、IND-CCA2。
