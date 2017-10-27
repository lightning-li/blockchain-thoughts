### libsnark 库使用示例介绍

#### 1. libsnark 库简介

#### 2. zkSNARK-toy 代码详解

gadget 是构建 R1CS 实例的基础组件，gadget 是个基类，任何针对具体功能的 gadget 都继承自 gadget，比如说针对 sha256 功能的 sha256_compression_function_gadget、针对 Merkle 树的 merkle_tree_check_read_gadget 等。组合使用这些基本的 gadget，我们构建出复杂的 R1CS 实例。

##### 代码结构

- src/gadget.hpp : 提供了 toy_gadget 类，使用其它基础 gadget 构建 zkSNARK-toy 相关的 constraint 与 witness

- src/snark.hpp : 提供对密钥生成、零知识证明生成以及验证零知识证明的简单封装，在 main.cpp 中使用

##### 使用到的关键 gadget

1. sha256_compression_function_gadget : 进行 sha256 哈希运算时用到的 gadget

2. merkle_authentication_path_variable : 保存了 merkle 分支

3. merkle_tree_check_read_gadget : 进行 merkle 分支验证的 gadget
