# Randomness

来源: https://blog.priewienv.me/post/randomness-blockchain-1/

## 真/伪 物理/软件 随机数

性质:

+ 均匀性
+ (每一位前后之间)独立性
+ 不可预测性
    * 比如不要直接使用 自然对数

例子:

+ 混沌效应
    * 大气噪声
+ 量子随机过程
    * 输入值相同 输出值也可能不同

伪利用真作为seed扩展出随机数序列

## 输入
+ oracle
    * 中心化
+ hash
    * 可能被挖矿方控制

## 随机数交互生成协议

### 直接输入？相互独立，均匀分布。
存在问题：猜拳后手

### 承诺
存在问题：多方时，前几个人已经 reveal 了 commitment，后面的人一看不利，拒绝reveal，趋利避害，终止协议或要求重新执行

### 惩罚机制
需要具体设计惩罚金额

### 门限机制
多方时，就算后面的人拒绝执行，够人数就执行

+ 单纯取 前 t 个？
    * 可能被 女巫攻击，分身占满 前 n 个
+ 无分发者的 secret sharing
    * stateful
        - 节点之间相互交互次数可能较多
        - 必须知道 总人数，才能计算出 门限
    * 如果参与者作弊，故意 broadcast 自己不正确的share？ 改进： Publicly Verifiable Secret Sharing, PVSS, 多分发一个 证明，用来验证每个人收到的 share 和别人一致
+ Distributed Key Generation, 分布式密钥碎片 + 文件签名
    * 除了密钥分发的阶段，之后的阶段只需要 n 个人中 t 个人提交有效的签名即可得出结果。
    * 哪 t 个人提交，最后对同样消息生成的签名都是相同的

### examples

来源:  https://my.oschina.net/u/3919161/blog/2906935

#### Algorand
>Algorand 的共识过程利用了随机抽签，它的随机抽签所依赖的种子本质是通过取 __前t个输入__ 来生成的。Algorand 的共识过程要求节点先在本地抽签，通过一个 __Verifabale Random Function__ 算出来一个可验证的确定的随机数，这样的随机数是根据上一轮的 __公共信息加上每个节点自己的私钥__ 计算出来的，是 __唯一确定的，并且可被其他人验证。__ 随机数被用于每个节点的本地抽签。本地抽签后，每个人知道自己是否被选中。之后，被选中的人 __广播抽签结果、证明和候选区块__ 到全网节点，根据区块的 quality 大小，选出来候选区块。而确定哪个区块的 quality 更大是需要做 BA 共识的，这个时候，就需要再进行一轮本地抽签，所有的节点会自己知道是否被选中去做 BA*（一个拜占庭类型的共识），去投票选自己认为的 quality 最高的候选区块，投票会进行很多轮，每一轮都要重新进行一次本地抽签。可以看出，__Algorand共识的本质就是我们每个人都生成一个随机数。但是我们最终只想要一个随机数，这样我们才能根据最后确定的随机数去决定哪个块会被全网接受。__这个时候的方案就是根据某种确定的规则从众多备选结果中取一个，方法是通过拜占庭共识达成一致，这就是 Algorand 的核心思想。

#### Dfinity 
>它的机制和 algorand 很像，也需要一个 __委员会__ ，委员会会运行一个分布式的生成随机种子的协议。一样，我们先假设已经有了这样的随机数种子了，这样，随机数种子可以被利用算出每一个节点的排名。同时节点可以注册进入不同的组，不同的组中可以有相同的节点。委员会就是在这些组中随机挑选的一个，随机种子是刚刚提到的那个种子。此时，大家都可以提交候选区块，广播给所有的节点，但是在所有人做区块认证的时候，诚实节点总是会选择 __排名最高__ 的块签名，签好后，广播给所有的节点，直到节点收到或拥有某个获得一半以上签名的区块。一轮的公共种子实际上是根据上一轮的种子以及开始设置的密钥对唯一生成的。其实就是 __无分发者的秘密分享加上门限签名__ 的方案。首先，在最开始，会有一次分布式密钥生成来进行初始化配置。此时，组内的 n 个人必须要全部参与到这个协议中。通过这样一种方式，每个人获得一把私钥的一个碎片，以及用来验证这些碎片正确性的证明，这个碎片是用来做门限签名的。私钥的碎片，用来对上一轮生成的随机数和轮数拼在一起的字符串进行签名，然后广播之。这样的 n 个签名中只需要任意t个就可以拼出完整的可被公钥验证的签名，这个签名就是当前轮的随机数。整个过程，算法是确定的，对于一个节点来讲，唯一不确定的是其他人的签名结果。值得注意的地方就是 Dfinity 此处采用基于 BLS 的门限签名方案，这个方案的第一个好处就是仅需要进行一次全局设置。之前提到的秘密分享，每次都需要分享一次给每一个人。但是使用门限签名，除了密钥分发的阶段，之后的阶段只需要 n 个人中 t 个人提交有效的签名即可得出结果。第二个好处就是唯一性，无论哪 t 个人提交，最后对同样消息生成的签名都是相同的，这保证了最后生成的随机数是无法被偏移的。

## VDF
see: https://blog.priewienv.me/post/verifiable-delay-function-1/

### RSA Groups assumption
Everyone seems to love VDFs, but the complexity theory around them is a bit underwhelming — why do they only work against adversaries with a polynomial compute advantage?

[A Note on Low Order Assumptions in RSA groups](https://eprint.iacr.org/2020/402)


## [Mining for Privacy: How to Bootstrap a Snarky Blockchain](https://eprint.iacr.org/2020/401.pdf)

Achilles heel: If the randomness used for the generation is known, the soundness of the proof system can be broken with devastating consequences for the underlying blockchain system that utilises them

chicken-and-egg


