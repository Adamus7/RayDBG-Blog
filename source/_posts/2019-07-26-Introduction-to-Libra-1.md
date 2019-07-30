---
title: Libra技术原理浅析（一）
date: 2019-07-26 11:36:13
tags:
- Libra
- Blockchain
---
Facebook在2019年揭示了其在加密货币层面点野心。19年6月，Facebook突然宣布了自己的发币计划，Libra。其目的是想要打造一个属于“the internet of money”的时代。Facebook希望借助Libra去服务全球万千用户，让所有人享受电子支付带来的便利。关于Libra的争议和各种政策层面分析层出不穷，但是从技术层面该如何理解Libra，似乎并没有非常多的讨论。媒体和分析师更喜欢用夺人眼球的角度去看待Libra，得出了很多“怪异”的预测和结论。抛开这些繁杂的概念和争议，本文希望从技术角度去理解Libra所构建的区块链世界。全部资料来源于Libra项目白皮书，另外目前Libra还在开发过程中，讨论仅限于白皮书中已经披露的技术细节。
<!--more-->

# Libra Blockchain 初窥
Libra要构建一个“全新的”加密货币系统，其目标是要做到：
* 实现一个能够承载数十亿的账户的区块链系统，这个系统需要由高吞吐量，低延迟的交易能力以及一个高效的大容量存储系统；
* 极高的安全性，足以保障资金和金融数据的安全；
* 足够的灵活性以便支撑Libra生态的管理和金融创新。

在设计上，Libra借鉴了当前市场上已有的区块链系统，选择了三个重要的技术方向：
* 设计了一套新的智能合约编程语言，Move；
* 选择Byzantine Fault Tolerant作为共识算法；
* 遵循已有的区块链数据存储模型。

## Libra Reserve
有新意的地方是Libra Reserve的发币机制。在传统的数字加密货币系统中，如比特币，数字货币往往是被“凭空”挖掘出来的。这种加密货币的“发行”行为是一种纯粹的“体内循环”的激励机制，其发币的动机是为了维持该加密货币系统的运转（激励矿工为用户的交易做验证、打包和执行等工作）。这些加密货币的背后并没有和实体资产有任何的锚定关系，也没有组织和机构宣布过对这些加密货币的刚性对付。而Libra Reserve则描绘了一个不一样“发币”机制：首先，Libra的投资者和用户需要使用法币从Libra协会手中“购买”Libra代币，当有1块钱的法币被Libra协会“收储”之后，才会有等价1块钱的Libra代币被“发行”出来（原谅我使用“收储”这个词，实在没有想到更好的动词去描述这个动作）。Libra协会会去集中管理这些“收储”的法币，并使用这些法币去做高安全、低收益的投资，例如投资各种主权基金。这些投资的收益大部分会用来支持Libra生态的运转。看起来这似乎更像是一个“世界银行”，用户可以把各种法币“储蓄”到Libra上，然后在Libra的的支付网络上使用Libra代币去做交易，或者通过Libra授权经销商把代币兑换成法币。个人认为这种发币机制更多的是金融层面的概念，本文不再做更多讨论，其带来的影响和利弊还是留给专业人士去探讨。而从技术角度去看，这里实在没有什么好解释的，既没有比特币“挖矿”过程的艰辛，也不涉及复杂的执行过程。

# Libra 的设计
Libra在设计上把自己的主要结构定义成一个“可信”数据库（a cryptographically authenticated database，不知道该如何翻译），然后通过Libra协议在这个数据库上维护了一个全局状态统一的总账本。Cryptographically Authenticated的意思是说这个数据库中保存的数据都是经过密码学验证过的，可以保证数据的真实可靠。而在网络结构上，Libra有两类节点：用户节点（Client）和验证节点（Validator）。如下图，client可以提交或查询交易，Validator则负责根据Libra协议去处理这些交易并维护账本的更新。
{% asset_img 1.png %}

## Transactions and States
State（状态）指账本中某个数据的状态（值），在不同的时间节点上，数据可能有不同的状态。Trasaction（交易）指一个去改变某些数据状态的指令。比如：
> Alice当前账户余额是100，这是一个状态；
> Alice要买一个包，向商户作出了一个付款的行为，这个付款指令是一个交易；
>付款之后，Alice的账户余额变成了50，这是一个新的状态。

{% asset_img 2.png %}
另外，Libra网络还维护了一个Ledger State，这里面存储了账户地址和账户数据之间的映射。
## Transaction 模型
一个Transaction由下列内容组成：
* Sender Address，交易发起者的地址（不是物理意义的地址，可以理解成发起者的“银行账号”）；
* Sender's publick key，交易发起者的公钥；
* Program，交易指令；
* Gas Price，Gas价格。Gas是衡量计算量一个度量，每执行一定的交易代码就会产生一定Gas，客户要为这些Gas买单；
* Maximum gas amount，客户愿意支付的Gas上限；
* Sequence number，序列号；
* Expiration time，失效时间（如果一个交易在失效时间内没有被执行，则交易作废）；
* Signature，数字签名。

乍一看和很多传统区块链系统（以太坊）的Transaction模型类似，比较独特的地方有两处：Program和Sequence number。
* Program部分包含三个部分：
   1. Move语言字节码组成的交易脚本
   2. （可选项）交易脚本的输入内容，在转账交易中，脚本的输入是转账的金额和接收方地址。
   3. （可选项）Move语言字节码模块，可以理解成一个智能合约的合约内容。
* Sequence number是当前账户所发起的交易序列号，每个账户的交易序列号从0开始，每完成一笔交易则序列号自增1。

## Account 模型
Libra中的账户模型和以太坊类似，从逻辑上看，账户是一个拥有两类资源的一个集合：Move Mouldes（程序代码）和Move Resources（数据）。Mouldes存储的是Move语言字节码，即智能合约的代码，这些代码可以去访问或更新Resouces中的数据。Resouces存储是是数据部分，账户拥有的Libra Coin也是存储在Resource中。
账户的地址Address是一个256bit的值。前面提到，Libra网络通过一个`k-v map`的形式在Ledger State中维护了Address到Account（Mouldes和Resoucrces）的映射。
要创建一个Libra账户，首先要构造一对公私钥`(vk, sk)`，该账户的地址等于公钥`vk`的`hash`值，`address=hash(vk)`。注意，这个时候账户并没有在Ledger State中出现，只有当一个已经存在的账户向该地址发起一笔交易的时候，Libra网络才会在Ledger State中创建有关该账户的映射结构。一个用户可以有无数个账户，一个账户下面可以有无数的Mouldes和Resource。

## Versioned Database
在讨论这个部分之前，先说一点题外话。虽然Libra把自己称之为Libra Blockchian，然而Libra似乎并没有引入区块的链式（Block chain）结构，仅在共识协议的实现上引入了“区块”的概念作为共识算法的“优化”手段。这并不奇怪，实际上业界已经意识到Blockchian这个词并不能代表这一类系统的核心特性，很多学者和分析师更愿意用分布式账本技术（Distributed Ledger Technology，DLT ）来称呼这一类技术。这类系统的核心是在对等的分布式环境下维护一个全网状态统一的账本，至于是不是基于区块链机制实现的，并不重要。
回到Versioned Database，Libra定义了一个基于version的三元组存储在这个数据库中：对于每一个Version i，数据库中存储了这样的一个三元组`<Ti, Oi, Si>`，分别指交易`Ti`，交易`Ti`执行时产生的输出`Oi`，交易`Ti`执行之后的账本状态`Si`（交易`Ti`是在状态`S_(i-1)`的基础上执行的）。简单讲，Libra这个数据库在**逻辑上**是一个线性的结构，交易记录在数据库中只能单向增长，像一部定格电影，在每个时间点上都有一个对应的世界状态的快照。

# 一笔交易的执行过程
当一笔交易请求提交到Validator上之后，会触发一系列的处理过程。本文以“**Alice向Bob转账10Libra**”为例来详解这个过程。
## 构造交易
首先，Alice需要在本地构造这个交易，就像填一张“支票”一样，如下图。
{% asset_img 3.png %} 

## Validator 的工作
Libra在Validator上划分出了多个逻辑组件，不同的组件负责不同的操作，这些组件包括：
* Admission control（AC）
* Virtual Machine（VM）
* Mempool
* Consensus
* Execution
* Storage

后面会结合下图去解释一个交易的执行过程
{% asset_img 4.png %}

### 接受交易
1. Validator通过AC获取交易。
2. AC通过VM执行交易检查，包括：使用交易中的公钥（地址）验证交易签名（基于密码学的数字签名原理：有且只有通过Alice的公钥解开Alice的签名可以获得和原文内容一致的文本；由此可以确认交易的发起者一定是Alice本人，且交易的内容真实可信），检查Alice余额是否足够，交易序列号是否正常等。
3. 当交易通过检查，AC会把这个交易放到Mempool中。

### 在 Validator 之间共享交易信息
4. Mempool中可能已经有很多笔交易了。
5. Mempool会通过shared-mempool协议和其他Validator节点共享各自所有的已接受的交易信息。

### 打包提议
6. 假设当前Validator是共识过程中的proposer/leader（不在此详细讨论Libra共识算法的具体过程），该节点会从Mempool中拿出一部分交易，打成一个区块（Block）。Consensus模块负责同步这个块到其他Validator节点上。
7. Consensus模块接下来负责协调各个Validator对该区块内当交易内容达成共识，包括交易记录的顺序。

### 执行区块中的交易
8. 当Validator们达成共识之后，这个区块（一个排序好的交易集合）会被送到Execution模块。
9. Execution模块通过VM去按序执行区块中的交易。对于Alice的交易来说，执行过程在逻辑上需要把Alice的账户余额减少，把Bob的账户余额增加；物理上需要对Resource部分的数据进行修改。
10. 执行完成后，Execution会把这些交易按序添加到一个临时的Merkel树结构中。
11. Leader节点的共识模块再次协调所有Validator节点对执行结果进行确认并达成共识。

### Commit 区块
12. 当一个区块当执行结果被绝大多数当Validator认可之后，Execution模块就会从刚刚的cache中读取之前的执行结果，然后把所有当交易提交到存储模块做持久化保存。
13. 至此，Alice的转账交易完成，Alice的账户余额减少了`(10 + gas) ` Libra，Bob的账户增加`10`，Alice的`sequence number`从`5`变成`6`。

# 小结
从基本的设计结构和流程上，Libra似乎没有给世界带来什么特别大的惊喜。Libra Reserve的发币机制是一个特色，但更多的是金融层面的创新。
在后续的文章中，我们会再看一下Libra的Move语言、性能和共识算法的等相关内容。

参考资料：
https://libra.org/en-US/white-paper/
https://developers.libra.org/docs/assets/papers/the-libra-blockchain.pdf
https://www.raydbg.com/2018/Understand-Blockchain-and-Bitcoin/



