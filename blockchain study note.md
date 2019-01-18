车轮互联 张翔
# 交易与协议

## 一个黑暗森林的模拟
有两个不认识，也许怀着恶意的A 和B，相隔很远，但是可以通讯  
他们需要一个快递员C，帮他们传递物品。而这里的快递员，都怀有无限恶意  
1. 第一天，A要给B 1元钱，所以A叫来了C，把1元钱给C
	- 结果B没收到钱
2. 第二天，A把钱锁在箱子里，让C给B，同时把钥匙让D给B
	- C和D串通，B还是没有收到钱
3. 第三天，A和B沟通，每人自己做了N把锁(Alock,Block)，对应一把钥匙(Akey,Bkey)
	- A把他的锁送了很多给B，B也把他的锁给了A
	- A拿一个箱子，用B的锁锁住了1元钱，叫C交给B
	- B收到箱子，用自己的钥匙打开取到了1元钱
4. 第四天，A继续让C来寄钱，但是B一直没收到
	- 钱被C扔到了海里，威胁A说如果不给小费，就继续扔，看谁耗得起
5. 第五天，A的钱用完了，他和B商量，以后不用钱了，直接记账吧
	- A和B联名发出了一封公开信，双方平账，互不亏欠
6. 第六天，A用B的锁写了转账纸条，A付给B-1元钱，从此A账户为-1，B为1
	- A把转账纸条发送给黑暗森林里的所有人，帮忙传递给B
	- B收到了很多张，只保留了金额正确的一张(重复支付时，账户余额会不匹配)
7. 第7天，A和B快乐的做着交易
	- C很不开心，他找到了D，一个阴谋正在萌芽
	- 他们偷了B的锁，写了很多假交易， 花着A的钱，B也无所谓，A一无所知
8. 第8天，AB重新约定，转账记录上，要有对方签字才算数
	- 这样相安无事了很久，直到C,D学会了伪造签名...

后来，来了一位数学家，发明了无法被伪造的签名，和不会被破解的锁

## 比特币的交易流程
比特币本身是没有实体存在的，只是交易的记录  
![bitcoin transaction](https://raw.githubusercontent.com/linkinbird/blockchain_book/master/pic/bitcoin_transaction_chain.png)

1. 初始比特币来源于矿工每次认证新block的激励（会定期衰减，直到结束，总额为2100万枚，靠交易手续费维持网络服务激励）
2. 发信人用自己的私钥（确认是自己发起） 加上公钥（确认花的是自己的币）进行签名，发给收信人的公钥地址  
![unlocking and locking scripts](https://raw.githubusercontent.com/linkinbird/blockchain_book/master/pic/msbt_0501.png)
3. 对堆栈进行scriptPubKey脚本操作，这里包含了收币人（可多人，币金额（可部分），以及更多开放的操作  
![P2PKH](https://raw.githubusercontent.com/linkinbird/blockchain_book/master/pic/msbt_0504.png)
4. 确认input来源于不重复的上一个输出，并且input和output数量一致
5. 交易的Hash值作为transaction identier (TXID)广播到整个网络
6. 收信人以TXID加上网络最长链的延时认证，但是这里有个bug，所以最保险的是UTXO (unspent transaction output) 认证。

如果网络出现分叉，需要proof of burn流程来确认一个货币被取消和转移。在soft fork产生的side chain里，也可以发送到指定的output，以便允许回滚。

## 公钥、私钥和签名
公钥密码学的实现方案有很多种， 常见的有RSA、ElGamal、迪菲－赫尔曼密钥交换协议中的公钥加密算法、椭圆曲线加密算法 ECC (Elliptic curve cryptography)，以及量子时代的OTS单次抗量子签名。
* 网银系统中主要使用的是RSA方案
	- 选取两个大质数乘积的[欧拉函数](http://www.ruanyifeng.com/blog/2013/06/rsa_algorithm_part_one.html)和定理
* 比特币秘钥系统则使用的是ECC方案
	- 交易签名使用椭圆曲线数字签名[ECDSA](http://www.instructables.com/id/Understanding-how-ECDSA-protects-your-data/)

### 比特币和椭圆曲线
为了完全理解这种加密方法，需要很深的数据功底。当然我们也可以通过一个小例子，让大家感受到数学的力量，同时尽量隐藏其中的推导细节

#### 史上最贱的数学题
![difficult math](https://raw.githubusercontent.com/linkinbird/blockchain_book/master/pic/fun_math_problem.png)

用a,b,c替换，去分母后：  
$$
a^3 + b^3 + c^3 - 3(a^2b + ab^2 + a^2c + ac^2 + b^2c + bc^2)- 5abc = 0
$$

多项式方程很容易找到某个特解，比如说， a=−1 ，b=1, c=0  
这意味着我们的立体方程（3维）实际上是个椭圆曲线  
但这里不是大家熟悉的圆锥曲线中的椭圆，而是域上亏格为1的光滑射影曲线。  
对于特征不等于2的域，它的仿射方程可以写成：   
$$
y^2=x^3+ax^2+bx+c
$$

复数域上的椭圆曲线为亏格为1的黎曼面。Mordell证明了整体域上的椭圆曲线是有限生成交换群（有限域），这是著名的BSD猜想（世界7大难题）的前提条件  
现在我们需要把椭圆曲线化成魏尔斯特拉斯（注：Weierstrass，提起他最著名的成就就是严密化微积分的ε-δ语言）形式。这是一个长得像这样的等式：  
$$
y^2=x^3+ax+b
$$

即使你不知道如何完成变换，验证它也是很容易的，或者说至少是机械的。对于我们而言，需要的变换由令人生畏的公式导出。  
$$
x=\frac{-28(a+b+2c)}{6a+6b-c}
$$
$$
y=\frac{364(a-b)}{6a+6b-c}
$$

得到的这个方程尽管看起来和原方程长得不怎么像，但确是如假包换的可靠模型  
$$
y^2=x^3+109x^2+224x
$$

从一个很好的有理数点x=−100, y=260（不是正整数解）开始，做椭圆曲线加法：  
相同点做切线，不同点做连线，取交点的垂足镜像，形成“有限加法循环群”  
![oval plus](https://raw.githubusercontent.com/linkinbird/blockchain_book/master/pic/Elliptic_function.png)

直到我们计算到9P  
```
a=154476802108746166441951315019919837485664325669565431700026634898253202035277999,  
b=36875131794129999827197811565225474825492979968971970996283137471637224634055579,  
c=4373612677928697257861252602371390152816537558161613618621437993378423467772036
```
这就是这道题的答案

#### secp256k1曲线 和 有限域
ECC方案中只使用了两类有限域：
* 质数有限域Fp，其中 q = p, p 为一个质数 (比特币系统使用)
* 基于特征值2的有限域F2^m，其中q = 2^m , m > 1

基于有限域Fp的椭圆曲线域E(Fp)
椭圆曲线函数：y^2 ≡ x^3 + ax + b (mod p)  
当：a, b ,x , y∈ Fp 且满足  4a^3+27b^2 ≠ 0 (mod p)时  
这条曲线上的点的集合P=(x,y)就构成了一个基于有限域Fp的椭圆曲线域E(Fp)

为描述特定的椭圆曲线域，需明确六个参数：T = (p, a, b, G, n, h)
* p: 代表有限域Fp的那个质数
* a,b：椭圆方程的参数
* G: 椭圆曲线上的一个基点G = (xG, yG)
* n：G在Fp中规定的序号，一个质数
* h：余因数（cofactor），控制选取点的密度。h = #E(Fp) / n

比特币系统选用的secp256k1中，参数为
* p = FFFFFFFF FFFFFFFF FFFFFFFF FFFFFFFF FFFFFFFF FFFFFFFF FFFFFFFE FFFFFC2F
	- = 2^256 − 2^32 − 2^9 − 2^8 − 2^7 − 2^6 − 2^4 − 1
* a = 0， b = 7
* G =04 79BE667E F9DCBBAC 55A06295 CE870B07 029BFCDB 2DCE28D9 59F2815B 16F81798 483ADA77 26A3C465 5DA4FBFC 0E1108A8 FD17B448 A6855419 9C47D08F FB10D4B8
* n = FFFFFFFF FFFFFFFF FFFFFFFF FFFFFFFE BAAEDCE6 AF48A03B BFD25E8C D0364141
* h = 01

#### 比特币的地址
256bit的ECC系统选择secp256k1曲线生成  
随机从[1,n-1]中选取一个数d, 计算Q = dG
* 其中，d就是私钥，而Q即为公钥。
* dG是一个标量乘法，可以转化为加法运算
	- 有限域中的加法和乘法是有特殊规定的，参考最贱数学题
	- 当加法取mod 循环n次时，对应的减法逆推可能要2^n

公钥是33字节的大数，私钥是32字节的大数  
公钥经过 Base58下列字段的组合，最后得到PubKHash：
* 00版本号
* HASH160: (RIPEMD-160(SHA-256))
* 上一步结果两次SHA-256的前8位

PubKHash经过Hase58Check 得到公钥地址，去除了容易混淆的0、o、l、I、+、/等6个字符

#### 椭圆曲线数字签名算法(ECDSA)
用户的密钥对:（d, Q）；(d为私钥，Q为公钥)  
待签名的信息：M；  
签名：Signature(M) = ( r, s)

签名过程（用到私钥d）：  
1. 根据ECC算法随机生成一个临时密钥对(k, R), R=(xR, yR)
2. 令 r = xR mod n，如果r = 0，则返回步骤1
3. 计算 H = Hash(M)
4. 按照数据类型转换规则，将H转化为一个big endian的整数e
5. s = k^-1 (e + rd) mod n，若s = 0, 则返回步骤1
6. 输出的S =(r,s)即为签名。

验证过程（用到公钥Q）：  
1.  计算 H = Hash(M)
2. 按照数据类型转换规则，将H转化为一个big endian的整数e
3. 计算 u1 = es^-1 mod n, u2 = rs^-1 mod n
4. 计算 R = (xR, yR) = u1G + u2Q, 如果R = 零点，则验证该签名无效
5. 令 v = xR mod n
6. 若 v == r，则签名有效，若 v ≠ r, 则签名无效。

### 一次性签名和 Quantum Resistant
除了公钥私钥这种非对称的加密和签名算法外，还有基于单向哈希的签名方法。
1. 私钥 -> 公钥 一般通过Hash
2. 私钥 + 原文 -> 签名 需要泄露一定私钥相关信息
3. 签名 + 原文 -> 公钥 验证方获得了一定等量的私钥信息

实用场景下的特点可以归纳为：
* 优点是 [Quantum Resistant](https://en.wikipedia.org/wiki/Post-quantum_cryptography)，速度也比椭圆曲线的ECC快
* 缺点是 基本的[winternitz OTS](https://cryptoservices.github.io/quantum/2015/12/04/one-time-signatures.html) (one-time signature)只能使用验证一次
  * 支付地址每次使用都会泄露部分信息，建议部分支付的时候，余额也同时转移到另外一个地址
  * 收款可以重复使用相同地址
* 基于哈希的[多次签名](https://cryptoservices.github.io/quantum/2015/12/07/few-times-signatures.html)也有人提出，但目前实践验证少。其中包括多个地址池挂在Merkle Trees的[方案](https://cryptoservices.github.io/quantum/2015/12/07/many-times-signatures.html)

目前使用这种加密算法的数字币主要有：
* QRL的[XMSS](https://cryptoservices.github.io/quantum/2015/12/08/XMSS-and-SPHINCS.html) (eXtended Merkle Signature Scheme)
  * Merkle Trees方案的进一步加强版
* IOTA的Kerl (Winternetz OTS)
  * Curl的升级版，因为被[DCI质疑](https://www.finder.com.au/reports-of-iota-cryptographic-vulnerabilities-debunked-in-email-leak)过存在多对一重复输出
  * 签约和验证过程如下
    1. IOTA的签名宽度（Winternetz里的哈希次数）是26(13 +- decimal from tryte of Normalized BundleHash)
    2. 公钥通过固定的哈希次数得到，并发布到网络
    3. 签名时按照原文内容计算出每个原文段的哈希次数，次数值小于26，而且不能为0(tryte不能出现M)
    4. 验证时按照原文内容计算出剩余哈希次数，哈希完成后和公钥结果一致即验证成功
    5. 每次签名会泄露50%的私钥（SEED不会泄露）
      * 因为每次签名会泄露当前位次的哈希结果，比如9次哈希的结果
      * 这时1~8的哈希结果无法反推，但是10,11...26的结果轻易得到
      * 多次泄露之后，攻击者就可以尝试伪造交易，只要交易内容对应的哈希位次都大于等于之前泄露过得位次，攻击者就可以伪造签名使交易成功
  * [milestone的签名](https://www.reddit.com/r/Iota/comments/7gsd3t/why_are_coordinator_signatures_still_secure/dqo9b9r/)使用了Merkle Trees的方案来保证官方节点地址不变，又可以反复使用，以引导初期的网络验证
    * milestone公布Merkle Root，并且始终不变
    * 随机选择Merkle Trees的一个叶子节点对应的私钥来生成公钥和交易签名
    * 签名存在交易文本里
    * 公钥不直接提供，只提供对应的authentication path
    * 区块对签名进行验证，做完补充哈希后的结果通过Tree路径，最后获得Merkle Root则验证成功
  * 一次性签名地址使用的复杂性，带来了[CarrIOTA Romeo](https://github.com/SemkoDev/romeo.html)这种轻钱包项目
    * 地址饱和后新建SEED
    * 泄露的address进行预警

## 匿名强化
### Layer1 匿名协议
单笔交易本身只记录了address，已经是高度匿名的了，但是通过地址的追踪，还是可以观测到每个币的流向。这个时候有两种隐私需要保护：第一是谁付的钱，第二是付了多少钱

#### 支付金额匿名
在协议层可以通过[zero-knowledge proof](https://en.wikipedia.org/wiki/Zero-knowledge_proof)的进化版 [Non-interactive zero-knowledge proof](https://en.wikipedia.org/wiki/Non-interactive_zero-knowledge_proof) 将交易验证过程匿名化，区块链里的应用叫做[zkSNARKs](https://blog.ethereum.org/2016/12/05/zksnarks-in-a-nutshell/)。基础原理是多项式同态（[中文原理](https://www.jianshu.com/p/b6a14c472cc1)）：

* 加法同态，乘法同态，全同态
  - 对于E(x)函数，难以求得反函数，及无法从E(x)反推x
  - 单调函数，及x≠y,，则E(x)≠E(y)
  - 通过E(x)和E(y)，可以计算E(x+y) 加法同态，计算E(xy) 乘法同态...
* 加法盲验证——用于交易和账本验证
  1. A计算E(x)，E(y)，并发送给B
  1. 因为函数E(x)满足加法同态，B可以通过E(x)，E(y)计算E(x+y)
  1. B独立计算E(7)，并验证E(x+y)=E(7)
     - 但也无法保障x,y没有从原来的[1,6] 篡改为[2,5]
* 多项式盲验证——已知输入，未知EVM代码，验证结果准确性  
$$
P(X) = a_0 + a_1 \cdot X + a_2 \cdot X^2 + ... + a_d \cdot X^d
$$

  - A知道需要验证的多项式P
    - 其中a为计算参数，可以代表某种计算过程
  - B想要验证对应某个不公开s的E(P(s))，及验证某个输入的计算结果是否准确
    1. A和B可以共同选择基于椭圆函数的α对 x⋅g作为E(x)
    1. B计算g, s⋅g , … , sd⋅g和α⋅g , αs⋅g , … , αsd⋅g并发送给A，相当于之前的E(x),E(y)
    1. A同态性计算a=P(s)⋅g，b=αP(s)⋅g并回传
    1. B验证a,b为α对，则A真的知道P，且a值即为B所需校验的E(P(s))结果
      - 存在一种可能，A使用另外的一个P'多项式，也可以找到符合α的a,b对，这种情况叫做“d-power knowledge of exponent assumption”
* 任意计算验证
  - 把任意计算转换为门电路向量
  - 把向量转化为多项式
    - QAP (Quadratic Arithmetic Programs)

区块链里的应用很多：
* 2016年10月发布的[ZCash](https://z.cash)也是使用类似技术，优化了交易速度，弥补了部分这个技术的短板
  * Sapling的升级版本用了特指的elliptic curve椭圆函数：[Jubjub](https://z.cash/technology/jubjub.html)，据[测试](https://blog.z.cash/cultivating-sapling-faster-zksnarks/)节省了80%的运行时间
* 2017年9月ETH 的Byzantium版本[支持](https://www.reddit.com/r/ethereum/comments/712idt/ethereum_testnet_just_verified_a_zcash_transaction/)了zk-snark proof
  * 主网有人通过智能合约实现了一个应用层的版本[AZTEC.sol](https://github.com/AztecProtocol/AZTEC)，通过独有的AZTEC Note作为中间体对交易金额进行join-split匿名化。后续甚至可以支持广义的匿名合约计算。
* 2018年2月底从比特币[分叉](https://www.reddit.com/r/BitcoinPrivate/comments/7todw0/historical_bitcoin_private_hard_fork_snapshot/)出了[Bitcoin Private](https://btcprivate.org/) (BTCP) 就是merge了ZClassicCoin (ZCL)和BTC主链的一个分叉，采用的也是zkSNARKs
  * 但年底爆出[丑闻](https://coinmetrics.io/bitcoin-private/)，通过shield pool藏匿了分叉时额外导出的UTXO output，总量达2,040,000 BTCP。相当于这些影藏账户预挖了这么多币，和白皮书描述的情况不符
* JPMorgan的[Quorum](https://www.jpmorgan.com/country/US/EN/Quorum)链是基于ETH的金融应用改造，也加入了[ZSL](https://github.com/jpmorganchase/quorum/wiki/ZSL) (zero-knowledge security layer) 和对特定监管节点透明的子版本
  * 子版本作者Simon Liu也是ZCash的贡献者，git上叫[bitcartel](https://github.com/bitcartel)，同时也是MultiChain背后公司[Coin Sciences](https://www.multichain.com/about-coin-sciences-ltd/)的区块链顾问，个人[博客](https://makebitcoingreatagain.wordpress.com/about/)非常低调
* ING也有相同道路的项目[zkrangeproof](https://github.com/ing-bank/zkrangeproof)，但看起来有些凉了

考虑到“可信启动”，“椭圆曲线”这些重度假设，最近两年有了新版本的尝试ZK-STARKs。详情可以看V神博客的翻译版本（[Part1](https://ethfans.org/posts/starks_part_1)，[Part2](https://ethfans.org/posts/starks_part_2))。当然，这也是有代价的：一个证明的大小将从 288 字节（b）上升到几百 千字节（kb）。

#### 支付关系匿名
有管理员的版本是[群签名](https://en.wikipedia.org/wiki/Group_signature)，由群管理者生成群公钥（Group Public Key）、私钥（Group Private Key）。加入该群的成员获得群管理者颁发的群证书（Group Certificate），就可以生成群签名。利用群公钥可以做验证，但是无法定位到具体的签名者。在争议爆发的时候，就需要群私钥解开签名提取真正的签名者。

加强版的[环签名](https://en.wikipedia.org/wiki/Ring_signature)则没有管理者，签名者利用自己的私钥和集合中其他成员的公钥就能独立完成签名，不需要其他人的帮助，集合中的其他成员可能都不知道自己被包含在其中：
* 签名者用自己的私钥和任意n个环成员的公钥为消息m生成签名a
* 验证者根据环签名和消息m，验证签名是否是环中成员所签

最初设计者是用来保护wikileaks这类泄密者的，比如多名白宫官员通过环签名，公布棱镜门的秘密后，政府无法罪责到具体个人。现在[Monero](https://getmonero.org)使用该机制来保护数字货币的交易者隐私，加上key image对币的染色，矿工可以验证该数字币的唯一性（类似序列号，检查是否被注册过）并防止double spending。再加上RingCT对金额的加密，已经齐全了。加拿大滑铁卢大学团队搞的[Iotex](https://iotex.io)也是同时使用了零知识证明和环签名，知乎[贾超](https://zhuanlan.zhihu.com/p/37041797)也在帮他们招聘。

#### 内容计算匿名
支付本身的匿名只能用在交易上，但智能合约承载的是合约内容，如何将合约或者内容文字匿名本身并不难，但要实现去中心化的验证、转移和加密环境下的搜索计算，是一个难题。有几个技术还在探索中，没有定型：
* 代理重加密([Proxy re-encryption](https://en.wikipedia.org/wiki/Proxy_re-encryption))，通过授权一个不知道密文代理执行转移操作，将基于A秘钥的密文，转移到B的秘钥版本
  * 最早在数字版权管理(DRM, digital rights management)领域，由DRM服务器来管理音乐的授权，[重加密方案](http://html.rhhz.net/buptjournal/html/20130602.htm)通过一个半可信的代理者实现密文转移
  * 云平台有基于同态加密的[版本](http://jst.tsinghuajournals.com/CN/rhhtml/20180205.htm)
  * 还有一份[专利](https://patents.google.com/patent/CN104639319A/zh)，写的比较清楚，最早是在Blaze, Bleumer Strauss的书《[Divertible protocols and atomic proxy cryptography](https://archive.is/20130212115133/http://link.springer.de/link/service/series/0558/bibs/1403/14030127.htm#selection-367.0-367.50)》里研究的
* Searchable Encryption Database (SEDB) ，网上有一些PHP和SQL的[实现](https://paragonie.com/blog/2017/05/building-searchable-encrypted-databases-with-php-and-sql)，但相关内容并不多
  * 基于blind indexing实现全匹配：将搜索条件变换后进行匹配，但加密key和nonce与encryption key区分开
  * 把原文进行多种预变化（大小写，去掉空格，同音字等等）生成多列blind indexing就可以实现弱的模糊匹配
  * 在lifeCODE.ai的基因组区块链里提到是通过tag标注来实现可搜索的，综合形成了一个Genomic Ordered Relational (GOR) 的数据管理系统。
* 匿名智能合约[Origo](https://origo.network)，通过编译层的优化，利用零知识证明来加密计算并保持效率
  * 对交易双方，交易金额和合约输入、输出都做了匿名处理
  * 计算过程通过off-chain的privacy-preserving application platform(PPAP)来保证效率和安全
  * 将智能合约转化为Arithmetic circuit 的算数电路结构在运营零知识证明时有额外优势
  * 匿名石头剪刀布

### Layer2 匿名通道
另一种简单的layer2的方法是一种mixing service中间商服务：把多个付款人，和多个收款人打乱，付款先付到资金池，然后随机轮转之后打乱支付给给收款方。
* Dash的[PrivateSend](https://dashpay.atlassian.net/wiki/spaces/DOC/pages/1146924/PrivateSend)就是这种形式，其隐私性促使其成为暗网的流行交易代币。
* 支付宝在银行的中间账户也可以扮演类似角色，对于央行来说直接隔离了资金流向的明细，所以现在央行要推网联，直接监管每一笔交易。

同样在Layer2的状态通道也可以在交易双方透明的情况下，保持对外部的匿名。但这里会存在double spending的风险，需要在绝对匿名和中心化第三方之间[寻找平衡](https://blockapps.net/multichains-and-privacy/)
* Hyperledger Fabric的private channel就是类似原理

## 跨币种的交易
比特币的交易从来都是双向的，一边是币权转移，另一边伴随着物权 或者另一种主权币的转移。当另一方非数字化时，就需要物理的中间商介入，当双方都是数字币时，则存在去中心化的方案。

Atomic Swap: 跨币种的交易可由[Lightning Network](https://lightning.network)或者[Altcoin Exchange](https://www.altcoin.io/)完成，其实也是中间网络，执行了交易合约。

* lightning Network
	- 基于并联加密，创建一个 two-party ledger entry，双方提取定量的币值合并在一起
	- 双方都可以快速的从这个fund交易（要符合blockchain可解析的脚本协议），交易本身不对主链广播，只是不断刷新自己，任何一方都可以关闭这个fund
	- 在跨币种网络建立lightning ledger entry 完成跨币种交易
	- 最后双方授权注销这个ledger entry

## 协议脚本的扩展
交易过程确认后执行的脚本，不只是记账而已，还可以做很多事情，比如访问某个外部接口，甚至注册一个域名，扩展到极致就是图灵完备的一个虚拟机。这一层被认为是**Layer1**核心扩展

以太坊[Ethereum](https://www.ethereum.org/)就是这个理念的产物，他有一本[黄皮书](https://github.com/ethereum/yellowpaper)专门描述这个EVM虚拟机的运作。整个系统看做一个状态机，每次交易就是严格按照挂在交易脚本下的二进制文件去执行状态变化，像个忠实的投币机器人。Qtum在此基础上做了X86指令集，优化了编程通用性。

基于这个EVM，又有很多上层扩展：
* [Hero Node](https://heronode.io) BAAS PLATFORM （预计2018-4-31上线）
	- block thain as a service，基于EVM的APP平台
	- 跨链开发，快速部署，兼容ETH, QTUM, IPFS.. 并提供统一API接口
	- 使用Javascript开发全端DApp，类似React Native的思路
* [DanKu](https://blog.algorithmia.com/trustless-machine-learning-contracts-danku/) from Algorithmia 机器学习竞标平台
	- Machine Learning Contracts on the Ethereum Blockchain
	- 由需求方发起合约，提供训练数据
	- 开发方下载数据，设计算法，再上传到EVM测试执行
	- 满足设计目标，完成支付

Hyperledger的[chaincode](http://hyperledger-fabric.readthedocs.io/en/release-1.0/chaincode.html)也是类似的概念，但是聚焦在智能账本合约的发起和执行，运行在docker上的三合一结构：APP通过Peers节点唤起chaincode执行下单，并更新ledger。他还平行的做了成员管理的协议，以支持交易撮合过程中的消息和状态管理

### 虚拟机的匿名性
交易的匿名扩展只实现了匿名验证，但在虚拟机环境里，计算的复杂度更高，简称为[secure MPC](https://en.wikipedia.org/wiki/Secure_multi-party_computation)的问题(Multiparty Computation)。通用的解决方案包括了保密数据分享已经零知识证明等等。但即使看似简单的计算，也需要定制化的协议设计：
* Two-party computation (2PC) 
  * 证明a >b 又称Millionaires' problem
    * The protocol of Hsiao-Ying Lin and Wen-Guey Tzeng
    * The protocol of Ioannidis & Ananth
* Multiparty computation (MPC)
  * 证明数据拥有 [Verifiable secret sharing](https://en.wikipedia.org/wiki/Verifiable_secret_sharing) (VSS)
    * Shamir's Secret Sharing
    * Additive Secret Sharing

通用协议在不同安全等级下还有不同的容错效果：
* 通讯是不安全的
* 中间人是恶意的
  * 恶意的程度 t < n/2
  * 非计划的恶意 Semi-Honest (Passive) Security
  * 计划的恶意 Semi-Honest (Passive) Security
  * 顾忌惩罚的恶意 Covert security
* 直接参与方是恶意的

### 状态机的验证

以太里的EVM虽然是状态机，但是区块记录的实际上是状态转移log，和记录交易日志一样。那UTXO的账户余额模式，也可以移植到状态机里，[Nervos](https://www.nervos.org/)项目就是基于状态结果而非日志事件的以太升级项目。其创始人Jan的[访谈](http://www.8btc.com/conversation-nervos)里可以看出，他们虽然从以太坊开发者社区来，但是对比特币的POW基础和能力推崇至极。项目底层Commom Knowledge Base (CKB) 的目标是分级客户端，可以运行在轻设备甚至light client运行在移动设备上。基于Cell（类似UTXO的不可修改状态，更新需要先复制再修改）, Validator, Generator等分工来优化并行，遵循新的混合共识机制。基于这样的事实，计算出答案和验证答案的复杂度差异很大。比如排序复杂度在$$O(NlogN)$$，而验证结果的复杂度只有$$O(N)$$。当计算和验证分工解耦之后，其对应的计算关系需要事先建立。对于任意计算的操作，需要自动生成验证计算。  
![state-focus](https://raw.githubusercontent.com/linkinbird/blockchain_book/master/pic/state-focus-design.png)

## 状态通道 State Channel
除了链上的交易协议外，为了提升效率，还有一种离线的状态通道来支持快速的点对点交易和智能合约。跨币种交易使用的lightning network也是一种state channel。这一层被认为是**Layer2**的上层扩展

[L4 Research](https://l4.ventures)(No.1 grant of Ethereum Foundation)在设计一种通用的[Generalized State Channels](https://medium.com/l4-media/generalized-state-channels-on-ethereum-de0357f5fb44)，根据Li Xuanji在2018年2月ETHDenver会议上的分享：
* Counterfactual Instantiation
	* 联合签名Multisig钱包，不区分是否是state channel
	* 使用Registry接口注册counterfactual address
	* 协议认证签名，本地执行代码部署（地址与上面关联）
* 实际执行只在参与方的机器上执行
* 注销协议需要双方签名，或者一方发起另一方超时
	* 支持跨平台的payment channel

## 消息通道
一般交易协议里都可以可以发消息的，但是每次发送交易后，用于验证的pubKey可能被恶意利用，所以建议更换地址，这样难以在同一个地址下连续广播签名消息。

hyperledger fabric 就是使用相对传统的身份注册的消息通道，主要用于交易撮合ordering service(or atomic-broadcast channel)

* 有一个membership services provider (MSP)授予身份
* 通过gossip protocol在组员间传递消息

IOTA 的MAM机制 Masked Authenticated Message
* 基于Verifiable Claims的身份认证（也可以直接通过智能合约实现）
  * 把verifiable claim公布给使用方
  * 把Claim Hash后的 attesthash以限制模式储存在tangle网络
  * 通过扫描二维码，结合两者进行验证
* 多种形式的权限管道
  * 公开通道 Public channel
    * root自己就是加解码地址，公开的
  * 私人通道 Private channel
    * address=hash(root)，只有自己知道root能解读
  * 受限模式 Restricted
    * 也是用address=hash(root)发送，但是需要sideKey解码
* Message Chain的订阅机制
  * 新的消息接在老的消息后面，老消息解码以后，可以看到新消息的nextRoot
  * 所以无法回溯老消息，但是一旦开始监听，就可以一直听下去
  * 受限模式下，nextRoot也可以找到address，需要sideKey配合root一起解码

# 网络与协作
因为交易本身是完全匿名的，所以网络也是设计成unstructured overlay 通过addr消息部分的广播其友邻网络，通过DNS bootstrapping 来寻找友邻。当然也有人把区块链1.0看成是寻找广播的最优路径，但是在不信任状况下的广播，必然涉及容错和共识，所以关于广播结构放在网络共识章节介绍。

对比分布式数据系统用的是ZooKeeper的Zookeeper Atomic Broadcast（ZAB）原子广播协议。在ZooKeeper群集中，其中一个节点具有领导角色，其余的则具有追随者角色。领导者负责接受来自客户端的所有进入状态更改，并将其复制到自己和跟随者。读请求在所有跟随者和领导者之间进行负载平衡。而区块链的网络层去掉了这种领导节点的角色。

## P2P
比特币用的是Gossip网络，大家每个周期内随机找个其他节点互相同步彼此不知道的状态，从概率上最终一定会收敛成所有节点都拿到最新的状态，只不过节点越多收敛越慢，肯定是最终一致性。2014年某次实验37天发现了872,648 个IP地址，但是高度不稳定，在线的一般几千个 [比特币实时网络节点](https://bitnodes.earn.com/nodes/live-map/)。如何在节点间达成共识，比特币用的是定义难，按时验证容易的一个证明题POW。

### 比特币的三步广播

为了高效的利用网络带宽，在交易广播时分为三步：
* Inventory messages (inv=h(tx))
* Get data messages (getdata) 最多等待2分钟
* Transaction messages (tx)

每个节点会维护自己的UTXO未消费账户池，同时彼此更新的交易先放进Mempool进行核对。有时交易有前后次序，万一上家的钱还没打给我，我打给下家的交易已经广播出去了，这就会出现一个无法关联历史账本的孤儿交易。这种交易存在MapOrphanTransactions的缓存里，一旦上家的交易来了，就会完成串联，完成前后交易的验证。

通过同时利用重复消费不广播的特性，和孤儿交易池会忽略inv消息请求的特点，我们可以嗅探节点间的网络链路，这就是[TxProbe](https://arxiv.org/abs/1812.00942)技术：  
![edge_inferring](https://raw.githubusercontent.com/linkinbird/blockchain_book/master/pic/Basic_edge_inferring_technique.png)

1. 创建数组相互冲突的重复消费交易parents和 flood交易
2. 在parents交易基础上创建二次交易markers
   1. 如果提前收到了flood交易，就会排除冲突交易parents
   2. 当再次收到后续交易markers的时候，因为缺少了parents交易，所以markers被计入孤儿交易缓存
   3. 多次构造markers以选取比较低的交易哈希，因为Mempool在满载后会随机剔除超过某个randomhash的交易
3. 将要探测的网络分为source和sink组
   1. source的规模要小一些，是孤儿交易广播的出发点
4. 广播构造的交易
   1. parents和markers进入source组
   2. flood进入sink组
   3. 为了阻止在开始探测前两个组提前交互，需要使用invblock，利用getdata的2分钟间隔，广播我们不希望他们知道的inv=h(tx)
   4. 只有markers交易时我们允许在两个组之间流转的，也是探测其关联的依据

#### block relay 的带宽和压缩优化
Bitcoin Relay Network 为了应对TCP协议丢包的情况，使用UDP-based transmission在每个包里额外增加一些信息，用来在少量丢包时可以用额外信息进行补全，而无需重复传递。这种方法叫Forward Error Correction, or FEC。实践证明额外传递信息的成本要低于丢包了重新连接和传递的成本，该方法在实时性要求很高的视频会议系统里已经有应用。

[FIBRE](http://bitcoinfibre.org)项目最近更新的[统计数据](http://bitcoinfibre.org/stats.html)显示，应用最新的网络和压缩技术，块广播的传递延迟可以在毫秒级，大大加快了交易处理能力。但其中关于cut-through routing的设计是否参考了先前的BloXroute和更早的[Falcon](https://www.falcon-net.org)一直存在[争议](https://twitter.com/el33th4xor/status/1080700582590115841)。有人乐观的认为这会带来更高的TPS，当然还要依赖其他多方面技术的更新。

[BloXroute](https://bloxroute.com)除了优化传输协议，还做了实时转发，可以一边接收一边广播，但交易验证是个问题。2019年初他们和Rawpool矿池开始[合作](https://news.bitcoin.com/block-propagation-startup-bloxroute-partners-with-mining-operation-rawpool/)测试他们优化的BDN分发网络（Blockchain Distribution Network）。他们称relay networks有两大问题：
* 能够监控个别节点，并产生差异服务，不够去中心化
* 依赖志愿者节点，不够可靠
  * 之前的 Fast Relay Network (FRN) 就是Matt Corallo提供的支援服务，但背后也有矿工社区的利益。后来被FIBRE取代了

### 以太坊的K桶寻址
以太坊和IPFS一样用的是[Kademila](https://www.jianshu.com/p/f2c31e632f1d)基于分布式哈希地址的索引。节点按照有顺序的异或距离（就是[汉明距离](https://blog.csdn.net/akadiao/article/details/79767113)，在二进制里与欧式距离等续）组成二叉树，在树的每一层级记录1个或几个友邻节点。这样查询任何一个目标ID的时候，都可以快速从高层级逐步下降，落到具体的叶子节点。这个结构可以在n位ID的二叉树下，每个节点只要记录n的近邻，在最多n次查询就能找到任意节点，整个树的容量有2^n。这种索引的几何意义就是按方向查找，从0~90度均匀铺设链接，角度小的邻居少，角度大的邻居多。通过有限近邻，及有限传递（2进制高维空间的紧凑性）就可以到达任意点。

## 数据结构
在P2P协议上，传递和计算的数据结构是另外一个独立的设计元素。他们可以是链表，可以是二叉树，可以是有向无环图。

### DAG
IOTA使用的有向无环图 [Directed acyclic graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph) DAG作为新的数据结构，也是一种新的传播网络。将生产者和消费者的角色融合，形成不停增长的账簿。其中的共识机制基于随机游走，虽然单点效率上不如近邻广播，但是集群总体有极强的扩展性。已经有不少[项目](https://coinpickings.com/dag-alternative-ledger-system-cryptocurrencies/)在运用这种协议了。

### DMT
Dynamic Multi-tree（DMT）也是一种偏中心化的广播网络模式。虽然信息传播很高效，但是共识达成和DPOS类似，树的根节点需要分权轮替。

## 分权与防刷
按照常规理解，这部分所讨论的POW和POS是共识协议的范畴，但是严格意义上，按照Emin Gün Sirer教授的[解释](https://twitter.com/el33th4xor/status/1006931729679044608)，他们只是防止女巫攻击的分权机制，其中关于如何投票和达成共识并不涉及。所以这里我们也将他们分开导论，但要知道分权和投票永远是组合在一起的，权力的分配和行使原则要放在一起考虑。所以大众统称他们为共识协议也可以理解。只是当这些机制交叉组合的时候会感觉有些奇怪，这个我们稍后举例。‏

### Proof of work
网络节点以算力为基础，接受新的交易以后，寻找满足某个Hash条件（以N个0开头）的nonce，找到以后对交易验证并叠加最长的链条上。整个网络都可以验证这个nonce的有效性，并且在最长链条上继续叠加。

新的block大概每10分钟生成一个，需要在产生新块之前，完成对旧块交易的确认。但是算力太高，生成太快也不行，因为网络可能还没来得及达成共识。所以每2016个块以后，都要调整认证算法的难度，极少数情况会降低，但是随着网络的扩大，大部分情况都是在上调难度  
![Confirmation Time and difficulty](https://raw.githubusercontent.com/linkinbird/blockchain_book/master/pic/bitcoin-difficulty.png)

#### 算法扩展
* litecoin(LTC 莱特币）：除了比特币默认的SHA256挖矿算法以外，使用了另外一种对CPU和内存资源要求更平衡的scrypt算法，力求避免GPU大矿池的垄断。同时也把块认证时间提升到了2.5分钟。目前有人统计，LTC已经成为暗网的[主流代币](https://www.recordedfuture.com/dark-web-currency/)。
* 还有人为了让计算资源有意义，开发了Primecoin ([XPM](http://primecoin.io))。这里认证算法是要求找出连续的质数链Cunningham chains
* Proof of Stake：一种基于coin age的激励算法，使用类似签到抽奖的机制，给予货币持有者一定的激励机会，在领取激励以后，这个机会coin age清空重新积累。Peercoin ([PPC](http://peercoin.net))就是这种币，网络节点无需再以无效的算力去竞争激励，而改为轮流抽奖。但也有作弊空间，就是偶尔联网收取激励，但大部分时间下线，这样coin age同样会增长。所以有proof of stake velocity使coin age非线性的增长，最后增加率降为0。
* Proof of Activity：除了有coin age积累以外，改变自主领奖为被动领奖。要求在发奖的时候点名，有N个人可以被点到，如果不在线就无法签名接受。这个点名过程叫做"follow-the-satoshi"

#### 硬件扩展
因为Proof of work是高度依赖CPU资源的计算，所以比特币矿工都已经单独定制ASIC(application-specific integrated circuit)芯片来挖矿。虽然代码是开源的，但是速度优化的代码面临专利诉讼风险。
* AsicBoost在2018年发起了最具争议的[专利行动](https://bitcoinmagazine.com/articles/there-bitcoin-patent-war-going-initiative-could-end-it/)
  - 他们有提高哈希效率30%的专利，几乎所有矿工都会使用
  - 开发者社区也提议绕过他们专利的挖矿算法和新协议
* 中国矿机企业Bitmain也会面临这样的局面
  - 一方面他可能已经在其销售的芯片里用了这个技术，但拒绝承认
  - 另一方面前任设计总监Yang Zuoxing自己创立了Bitewei，Bitmain起诉其窃取了技术
* Coinbase采取了防御性的DPL (Defensive Patent License)
  - William Ting和知识产权律师发起了升级版Blockchain Defensive Patent License ([BDPL](https://blockchaindpl.org))
  - AsicBoost 在2018-03-01宣布加入BDPL，改变了专利争议走向

2018年4月Bitmain发布针对[ETH的ASIC矿机](https://blockonomi.com/bitmain-asic-ethereum-miner/)，引发了ETH社区的[激烈讨论](https://www.reddit.com/r/ethereum/comments/8bkkv1/asic_resistant_hard_fork_discussion_overview/)，甚至已经发起了Hard Fork的提议。其中主要分为两派：
* Pro ASIC Resistance (PAR) 
  * 直接升级抗ASIC的POW算法，但是技术细节存疑，而且会面临攻守双方的技术追逐
* Pro Doing Nothing (PDN)
  * 重心放在加速Casper POS协议的实施，节奏比较慢，但是相对彻底的解决问题

#### 浏览器扩展
完整的挖矿需要保存和同步全节点，对硬件要求比较高。为了进一步整合空闲计算资源，还有很多轻矿机，不进行交易验证，只计算POW。其中浏览器插件是门槛最低的装备，比如[nimiq](https://nimiq.com)，现在应用在[天池](https://nimiq.skypool.org)这个项目里，开发者包括了上海交大软院的熊伟伦[azard](https://github.com/azard)。

当然门槛低带来的结果也就是收益低，但不排除之后IOTA这类免手续费“完全互助”协议的流行，会带来超低成本的普及型区块链，这时浏览器扩展就会发挥其人海战术的优势了。

浏览器的潜力还很大，耶鲁归来的张旋创立[Cloudacc](http://www.cloudacc-inc.com/)，使用web P2P技术来加速流媒体分发。通过flash/html5即可运行，无需任何客户端或插件，是睡莲网络的理想载体。他的另一个新项目[闪电盒子](http://www.jobui.com/company/14000595/)把战场转移到移动端，通过应用层虚拟化MAV（Mobile App Application-level Virtualization）无需下载，在远端直接运行原生的Android应用程序。通过借鉴趣头条的激励模式，融合积分墙打入应用分发和流量入口的领域。

### Proof of Stake
POS 机制是按照财富进行了权利分配，当然在联机时长等细节做了优化。

新秀[Qtum](https://qtum.org)就是结合了比特币和以太坊的优点，形成Decentralized Governance Protocol ([DGP](https://qtum.org/en/blog/qtum-s-decentralized-governance-protocol))管理的平台，通过自认知smart contract定制block size, Gas Price, Gas Limit 的POS共识体系

与POW时间的本质不同，POS更多是在使用经济规则放分身刷单

- 当POW加密币价格上涨的时候，矿机增加，算力增加，难度也增加，维持一个投入产出的平衡。币价下跌矿机撤出，算力减弱，但矿机收益反而增长，也会回到一个平衡
- 同样POS币价下跌，stake持有者退场时，剩余玩家人均收入分配就会增加。币价上涨时则相反
  - 但和POW矿机的单一功能不同，矿机算力难以快速迁移，所以矿力是长期稳定的
  - 但Stake本身具有高流动性，参与节点验证相当于买了国债，有固定收益率，但不一定是市场最高。所以当有更好的投资机会时，POS参与方会更加倾向于转移阵地。不一定是卖币，可能是使用货币交易实体商品和服务。当外部市场机会随着资金增加导致收益率收窄时，国债收益率上升，最终两者还是会平衡，维持一定数量的Stake保证网络验证有效性。
  - 法币的利率是和通胀率挂钩的，所以在总体通胀情况下对利率会非常敏感。但数字币本身是通胀稳定的，所以加入Stake就是最好也是最简单的抗通胀资产。

## 网络共识
consensus是经常提到的词，因为除了证明你的工作能力以外，还需要团队协作以达成一致的决策和目标。这里还要面对恶意节点的攻击，所以抽象化为BFT consensus (Byzantine fault tolerance)。在去中心化的世界里，解决的方案也逃不出人类的历史，分为民主派和集权派：

比特币共识网络是最典型的全民主型，要求50%以上的节点重复认证同一个交易，这里有太多的冗余，速度受到慢节点的瓶颈影响，难以扩展。将共识机制和防刷机制交叉组合，可以产生非常多的变种：

* 长链Longest chain共识和POW是天然的搭配
  * 因为长链共识要求最长链尾部的增长速度超过分叉增长的速度，这个POW最能保证，而POS则无能为力
  * 所以我们不会提到长链共识，只会说POW机制，因为这两个是强绑定的
* 基于资源的POS分权可以搭配的共识或者投票机制很多，这些机制反过来其实也可以搭配POW使用
  * PBFT
  * Ben-Or
  * Tendermint
  * Avalanche

还有一些分权和共识机制的组合具有代表性或代表某种设计思维，我们单独介绍

### DPOS委任共识-普选
[EOS](https://eos.io)声称解决多核并行，并结合了空间扩展和 DPOS (delegated proof-of-stake )机制，目标做成分布式的操作系统(EOSIO)。其中全体持币者票选委任节点(supernodes) 既保证了去中心化，又保证了效率和灵活性。从参数选择到分叉都有所设计。但普选的过程是否存在作恶和贿赂的现象，现在还难以断定。
  * 从最初的ERC-20标准衍生而来，可以说是ETH的继承者
      * 2018年6月2日开始从ETH剥离，转移到supernodes网络
  * 由持币方投票产生委任节点supernodes 来负责挖矿和区块验证
    * 每隔21个区块以后，要重新选举，以避免作恶
  * 但是他背后的[block.one](https://block.one)公司可以直接拿走融资当做盈利，又摆脱了后续维护的工作，为这么项目的前景蒙上了一层阴影
    * 2018年代码发布以后，亲生母亲就基本失去了控制权
  * 为期1年的ICO过程伴随着发币争夺和节点争夺
    * ICO到2018年6月26号截止，每天固定发币量，按照充值金额等比分配，价格波动比较剧烈。
      * 初期争夺激烈的话，人均获币量下降，币值飙升
      * 中期热度下降，人均获币量上升，货币贬值，初期玩家被套牢，只能持续跟进
      * 后期超级节点争夺，大量套利空间
    * 6月2号当天需要21个supernodes来确保过渡过程的顺利
      * 现在有[50多个](https://bitcoinmagazine.com/articles/eos-hype-builds-over-50-candidates-vie-21-supernodes/)公司参与supernodes选举，主要是区域的矿工集团和专业矿池，他们竞争的相当于特许经营权。但是这个经营权要根据表现来进行续约。
      * 选举设计1 EOS对应30票，意味着不会有激烈的抢票现象，但是一个漫长的智斗过程，人性面前，并不一定是最适合的人获胜。
      * 存在50%**贿选攻击**漏洞：
        * 只要投票者是逐利的，我就可以承诺100%的利益返还来获得网络控制权，这样的成本远低于POW的算力资源。
        * 防御方需要承诺超过100%的利益返还来争夺控制权，但这样亏本的，所以没有动机去防守
        * 攻击方甚至可以两面派，以防御的口号实施攻击的意图，唱个双簧
        * 实际上贿选攻击者是在以投票者所持EOS为代价进行杠杆攻击，而且对防御者有直接伤害，所以系统抵抗力远低于标准POS，和POW
    * 直到ICO结束也就是主网络supernodes确定以后，token才真正意义上流通 

DPOS是一种综合机制，其实现的方法也有很多数据结构，阿里移动总架构师创业的快付FPay项目，使用DMT (Dynamic Multi Tree:动态多叉树) 来通过树状广播和投票执行DPOS的过程。最终的root节点相当于supernode，但是信息广播有层级。看似解决了不可能三角，但是当面临攻击的时候，仍然需要多轮的共识。层级结构本身就蕴含了信息，或者信用，而这种结构的优点是在稳定后可以获得很高的效率，但在建设期或者被攻击重构的时候，需要很长的时间。相比而言DAG的重构会更加迅速，但是结构不够稳定，所以终态的确定会延迟。这里不可能三角仍然存在。

### dBFT议会制共识-分权
[NEO](https://neo.org)更加彻底的财富与权力分离：区块链里面，NEO代表投票权，GAS代表流通币。采用dBFT（一种改进的拜占庭容错算法）[迭代共识](http://docs.neo.org/zh-cn/node/consensus/consensus.html)方法更快达成共识。诚实节点越多，共识越快，而给EVM更多算力和激励。节点分为三总：

* 议长 负责向系统发送一个新的区块的提案
* 议员 负责对议长的提案进行投票，大于等于2/3的议员投票时，提案通过

共识网络的执行情况如下：
* 初期挖矿集中在NEO，因为付给EVM的执行GAS会按照NEO份额重新分配。
  - 而且bookkeeping nodes集中在创始的[7台机器](https://www.reddit.com/r/NEO/comments/7h06en/what_only_7_bookkeeping_nodes)上(2018年会[扩展](https://thebitcoin.pub/t/neo-centralized-nodes-will-go-decentralized-in-2018/12848)，但总量还是受控的），可以说中心化程度很高了
  - 投票节点间的delegation process也约束了节点数量，在2018年3月3日的宕机也引来了[对dBFT的不少质疑](https://ethereumworldnews.com/neos-consensus-failure-spreading-fud/)
* 中期GAS作为交易货币随着交易量上升持续升值，共识网络激励是直接支付GAS的。NEO因为供需关系的原因，不一定会等比升值，反而可能形成trade NEO-> for GAS逆流
* 长期NEO每22年生出一个GAS，最终NEO和GAS都是1亿枚。交易支付GAS，运行计算支付GAS然后再按NEO重分配，是可以平衡的

### VBFT匿名委任共识
源自于图灵奖得主Silvio Micali在2017年发表的一篇论文 “[Algorand: Scaling Byzantine Agreements for Cryptocurrencies](https://eprint.iacr.org/2017/454)”。提出了基于可验证随机数Verifiable random function [VRFs](https://en.wikipedia.org/wiki/Verifiable_random_function)的代理人选举方式，当然VRF的概念他们早在1999年就提出了。区块链委任共识里，按照VRF选出的区块见证者，是匿名确认自己当选的，而且单次有效，所以无法形成多人的串通或者保留权力。[Algorand](https://www.algorand.com)的本质，就是通过密码学的方法，来做一个随机数发生器来随机决定下一个区块的生成者是谁。如果这个随机数发生器是完全随机的，不需要广播，不需要见证，仅凭验证函数就能达成共识。那也就是说，任何人（包括上个区块的发布者）都无法预测下一个的发布者是谁，那么其实这就是一个可用的共识算法了。这个现在也是一个币来着。

模仿这个设计，2018年4月Ontology就发布了他们的[VBFT共识](https://medium.com/ontologynetwork/ontology-launches-vbft-a-next-generation-consensus-mechanism-becoming-one-of-the-first-vrf-based-91f782308db4)项目，是POS和VRF的合体。在VRF确保了随机性、匿名性、和瞬时性的基础上，可以大大加快共识过程就保证安全性不受损害。Cardano和Dfinity也都使用这个思想，可以说更加接近机器控制的共识，每个节点参与的操控力被大大限制了。

### 个体自治 共享共识
更彻底的个体自治是单向网络图模式，主要的推进者是[IOTA](https://iota.org)，他们的动机是在物联网领域，需要提升网络效率。但这个模式还需要验证。单向图(DAG)基于当前未确认的交易节点进行后续链接，取消了Peer discovery，全局信息弱，这里很多问题值得讨论：
* 混合了POW和POS
  - Minimum Weight Magnitude来决定POW的nonce大小
  - 权重由关联节点累加而来POS
  - 从tips出发的蒙特卡洛随机游走来选择需要验证的交易bundle
    - 50%以上tips间接引用，表示大概率会被认证
    - 99%的间接引用可以说confirm了，因为接下来的游走几乎必定会走到
* 当交易量很低的时候，网络的链接会非常脆弱，可以观察这个实时的[IOTA节点网络图](http://tangle.glumb.de)。
  - 交易最终确认需要被所有tips节点间接链接，分叉的可能性很大，需要reattach或者promote
  - 策略上应该选择权重大，且depth比较小的接入，这样自己被验证的几率也大
  - iota 通过fundation控制的Coordinator full node插入milestones来在初期保护网络免受恶意攻击。这些Coordinator也同样会被全网检验，所以可以保证去中心化
* 当交易量很高的时候，因为选择tips接入节点的策略是开放的，有可能出现负载不均衡，一些冷门支链的交易确认时间过长。
  - 2018-03-05 Coordinator把milestone发放速度从2分钟加快到了1分钟，这样交易可以更快的被milestone引用，意味着更快的交易确认
* 在进行网络攻击的时候，攻防的胜负是概率事件，防御理论尚未完善。

### 其他组合共识的变种

PoW+DAG+modified heaviest chain. That'd be GHOST.  这个套路的希伯来大学开发团队Yonatan Sompolinsky, Yoad Lewenberg, and Aviv Zohar同时也是IOTA的创始人，所以两者的机制听起来很像。这套机制还有一个很长的名字 “Serialization of Proof-of-work Events: Confirming Transactions via Recursive Elections” (SPECTRE)，但其实背后都是相同的人。之前还有个竞争对手[ByteBall](https://byteball.org)，不通过随机游走，而是在DAG里选择一条main chain来确定交易顺序，但是现在转型重点做钱包了。

PoS+DAG+Avalanche, as in Ava coin. 是[Emin Gün Sirer](https://twitter.com/el33th4xor)教授一直在推荐的[Avalanche consensus](https://hackernoon.com/protocol-spotlight-avalanche-3f5dfd366a26)，和微软研究院早年推出的[Avalanche P2P](https://www.microsoft.com/en-us/research/project/avalanche-file-swarming-with-network-coding/)网络不知道是否有关系。而且和交大谷老师他们的选型也有点像。类似的也可以将 PoS + multisets + Avalanche组合，或者PoS+forests or chain+Snowball，都是类似的概念。当然性能优化的效果会有不同。

Sharding在这里也可以算是数据结构和共识的一种新组合，类似政治里的大区自制。按照某种高效而随机的方式把节点分为不同的自治区，每个区处理分派给自己的任务。但这项技术还在很早期的阶段，ETH在努力的推进，预计2019年能够成熟

# 性能扩展
网络层，分身防刷，是底层基础，有一定扩展空间。  
共识投票机制决定了投票权与分配，也有少量扩展空间。  
运行在次之上的算力和应用，还有其独自可以扩展的地方。  
这些全部组合在一起，就是区块链性能扩展的3个层次。

## 空间扩展
从协议层到平行的储存策略，也有一个发展历程。
### 分叉树结构 Merkel Tree
交易挂在Merkle Tree 只把Merkle Root加到block Header。
历史交易验证的时候，从叶节点往上追溯，只要追到认证的Merkle Root即可  
![merkle root](https://raw.githubusercontent.com/linkinbird/blockchain_book/master/pic/Bitcoinpaymentverification_merkeltree.png)

轻节点利用这种储存结构可以基于较小的文件来进行交易判断。Cardano进一步进化提出了Non-Interactive Proofs-Of-Work ([NIPoPoWs](https://nipopows.com))，NIPoPoW钱包只需下载差不多250个区块头的采样就可以运行，比常规SPV轻钱包加载所有Merkle root要少很多。原理就是利用了nonce产生额外头0的概率分布，原本只需要10个0开头的哈希，有均匀的概率获得11，12个0，这些额外的区块组成了超级区块。加上区块间的interlinking，通过velvet fork生效。

### 账户快照 Snapshot
BTC和ETH目前都是保留全纪录，但是iota的snapshot会清理空账户，只保留当前的账户余额。主要也是因为iota的一次性签名不能重复使用，会导致大量的弃用账户

### 去中心化的分布式文件系统
除了扩展账本空间，当储存的不单是账本，而是大文件或者数据资源时，全节点将不可能储存所有的资料。引申出两个方向的解决方案：
* [filecoin](https://filecoin.io)的共享储存空间，2017-8的ICO 现在平台还没正式上线
  - P2P文件系统IPFS 发明者Juan Benet 的项目，解决IPFS没有储存激励，导致文件有效时间短的问题
    - 当然也有贡献自己存储空间，换来[永久储存](https://medium.com/@merunasgrincalaitis/how-to-host-your-ipfs-files-online-forever-f0c56b9b5398)的方法。
  - 基于Proof-of-Replication (PoRep)的共识机制（Proof-of-Storage的升级）
    - 使用MerkleCRH生成树的zk-SNARKs验证文件完整
    - 维持PFT: Power Fault Tolerance以保证文件不丢失
  - 储存和读取分为两个交易市场
* [sia](https://sia.tech)是比filecoin早落地的去中心化存储项目
  * 隐私加密，开源
  * [Reed–Solomon codes](https://en.wikipedia.org/wiki/Reed–Solomon_error_correction)保证冗余分片和复原保障
  * 有storage proof的环节，但主要支持冷备份，热存储的提取保证还有待加强
  * 利用存储“市场”来交易储存空间，其激励原理更加简洁一些
  * 陈钢测试发现，上传文件前要先同步好多G的区块账本...
* [SWARM](http://swarm-gateways.net/bzz:/theswarm.eth/) 是基于以太的分布式存储和内容分发服务
  * 其中包括[wolk](https://www.wolk.com)开发的NoSQL数据库[SWARMDB](https://github.com/wolkdb/swarmdb)
  * 存储原理和Kademlia类似，加上带宽和存储的双重激励机制
  * 有自己的swarm accounting protocol (SWAP)
* carro的[blockchain on hadoop](https://github.com/linkinbird/CarChain/wiki)，2018年2月启动的项目
  - 利用hadoop的分布式技术，上下夹了区块链的加密和共识，形成三明治的架构
  - 区块链保证了数据所有权，HDFS保证了数据安全和高效存储
  - EVM计划用来做数据代码执行的沙箱

新的存储协议也带来了新的挖矿机会，但是类似filcoin(IPFS)这类的矿机，对硬盘和带宽要求的特别高的，核心的要素是单TB成本低，且IO和带宽稳定。矿工可以选择[自建](https://www.zhihu.com/question/264885361)，基于NAS类的硬盘管理做二次开发。厂商在主链上线前卖的矿机基本都是基于预测的，风险不小：
* 做存储起家的StorSwift推[CX-MINNER](http://www.storswift.com/product/swiftminer.html)方案
* 玛雅矿机[MayaMiner](https://www.maya.top) H1和H2
* [星际大陆](http://www.ipfsmain.cn)拿自己和玛雅对标，号称性价比更高一些
* 三角矿机[Acute Angle PC](http://www.acuteangle.com/home.html)有自己的主链网络Acute Angle Cloud(AAC)，他的逼格设备，更像是睡莲网络的节点
* 朋友圈 杨明做的星空矿机，目前还没有上市

## 综合算力扩展
### 比特币架构扩展
算力和共识机制是无法完全割裂的，因为如果是在proof of work框架下，算力扩展是很困难的，这就是在2017年比特币面临的两次分叉：
* 第一次分叉是Core的[lightning network](https://lightning.network) 在layer2创建一个共享账簿，撮合交易以后要双方签字广播到区块链里做认证
	- 多次交易，一次认证。
	- 支持atomic swaps
	- 需要[Segwit](https://en.bitcoin.it/wiki/Segregated_Witness)启动
* 比特大陆发起第二轮分叉，SegWit2x基于"[New York Agreement](https://medium.com/@DCGco/bitcoin-scaling-agreement-at-consensus-2017-133521fe9a77)"
	- 主要目的还是将区块大小扩容到 2M，这样全网账簿的体积也会扩大
	- 太多的政治思维，忽视中小玩家导致了最终分叉失败，但也再次证明了比特币去中心化不可篡改的能力
* ![image](https://raw.githubusercontent.com/linkinbird/blockchain_book/master/pic/bitcoin_charts.png)

虽然比特币的价值得到了认可和肯定，但是波动的价格和随之而来高额的交易手续费，使其在日常交易或者暗网交易里都变得不实际。最终比特币作为虚拟货币的入口和稳定等价物（黄金）存在着，人们用其储存价值，但是会先兑换更加便利的LTC和Dash来进行日常(暗网)交易。

### 以太坊架构扩展
ETH从2016年开始也计划着类似的分叉（升级）EIPs (Ethereum Improvement Proposal) 代号"Metropolis"，2017年完成了第一步[Byzantium](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-609.md)，2018计划完成第二步Constantinople聚焦在两个方向，一起解决共识机制和算力的扩展问题：
* [carsper](https://github.com/ethereum/casper) POS
  - FFG (Friendly Finality Gadget) partial consensus mechanism combining proof-of-stake algorithm by Vitalik
    - Hybrid POW/POS, every 50th block is going to be a POS
    - Bet on block discover, failed(malicious) ones will get their stake slashed off
  - CBC (Correct By Construction) partially specified protocol by Vlad
    - 部分定义的动态协议，通过"ideal adversay"来最终达到平衡的完整协议
    - 还是一种理想化的概念设计，有点像GAN神经网络的训练思路
* [sharding](https://ethresear.ch/c/sharding) 分布式扩展，来源于数据库技术
  - 但最近的core team会议里（2018-02），parallelization安全问题太多，可能不纳入到Constantinople分叉
  - Prysmatic团队提出了一个两步走的[sharding方案](https://medium.com/@rauljordan/ethereum-sharding-update-prysmatic-labs-implementation-roadmap-c625cd013aeb)（2018-03-07）
  - 新加坡的[Zilliqa](https://www.zilliqa.com)团队基于新加坡国立研究的[Secure Sharding protocol](https://dl.acm.org/citation.cfm?id=2978389)，扩容的同时只可以承受25%的恶意节点，和Vitalik也在合作
  - 分布式依赖的随机生成RNG会用到[RANDAO](https://github.com/randao/randao)方案，经过一些预防串通和预防懒节点的优化，似乎可以应用了
  - Telegram的区块项目TON里明确了sharding的计划

此外L4 Research提出的[Layer 2](https://medium.com/l4-media/making-sense-of-ethereums-layer-2-scaling-solutions-state-channels-plasma-and-truebit-22cb40dcc2f4) 解决方案也很有价值：
* State Channel 状态通道
	* 在交易协议扩展里已经介绍过，通过联合签名把高频交易转移到链下
	* channel注册和注销的过程如果特别集中，会有并发问题
* Plasma 裂变
  * 可以从主链生出子链，子链还可以再生，子链以主链为root做中转
  * 子链使用自己的高效共识模式，比如Proof of Authority(PoA)
  * 资产必须先从主链生成再传递给子链
  * 如果发生恶意子节点，有两种防御：
    * 任何人都可以先发起fraud proof，证明false block的存在，最终子链回滚
    * 如果子链信息不全，无法证明fraud，那就直接发起Proof of Funds，把资产提回主链
* Truebit 离线算力
  * 当计算特别消耗资源的时候，邀请solver离线计算
  * 利用verification game邀请全链验证结果，不是全部的结果，而是有争议的部分

### 智能算力扩展
在《[重构区块链](http://bcrb.io)》一书中提及的应用反向选择算力，市场上已经有项目在实践：
* [Golem](https://golem.network) 闲置算力的共享市场，基于以太的激励系统，今年4月上线。后续陆续支持GPU
  * 还有[Joanna Rutkowska](https://blog.invisiblethings.org/about/)配合集成安全计算服务[SGX](https://blog.golemproject.net/introducing-graphene-ng-running-arbitrary-payloads-in-sgx-enclaves-a03f219447a5)。她是安全操作系统Qubes OS的创立者，目前还有[Graphene LibOS](https://github.com/oscarlab/graphene/wiki)支持这一实现，他们Fork的新版本会在2018年暑假发布。
  * 对比之下 Intel以前的Trusted Execution Technology (TXT)是从业务侧反过来检查计算核心做[自我保护](https://blog.invisiblethings.org/2011/12/13/trusted-execution-in-untrusted-cloud.html)的，但是无法实现对环境的完全监控，可以被镜像攻击。
* [SingularityNET](https://singularitynet.io)  是专做人工智能算力的

  * 用区块链实现AI智能民主化，实现AI agent之间的自主合作，还有数据和模型共享
  * 但他们似乎没有很好的区分训练和应用，如果只是应用的话，并不需要这么复杂的体系
  * 最终还是会走向综合算力
* 深脑链[DeepbrainChain](https://www.deepbrainchain.org) 也是国内的人工智能算力平台

  * 由之前的深度学习云平台演化而来，基于以太发工资
  * 2018年6月宣布和SingularityNET建立了[合作](http://www.cointime.com/blockchain/10744.html)，使奇点的AI agent可以选择调用深脑链的算力。
* 迅雷的玩客云OneCloud号称类似方向，但实际只做了存储

  * 基于共享硬件，500块卖设备，其实是类似共享NAS的概念
* 2018年新成立的项目[OasisLabs](https://www.oasislabs.com)，主打安全计算
  * University of California, Berkeley and 
    MIT的明星团队
  * 上来就是\$45M的A轮，后续不知道是否会ICO
* [Cortex AI](http://www.cortexlabs.ai)是清华系做的项目，自己定义了一种链
  * 上面运行的叫Cortex Virtual Machine，支持深度模型的应用
  * 代码合约叫Smart AI Contract，有独立的类似Gas的结算机制
* 上海交大团队的TEEX 可信计算区块链
* Ray [RLlib](https://ray.readthedocs.io/en/latest/rllib.html): Scalable Reinforcement Learning
  * 非区块链的分布式AI架构，类似的就罗列了
* [PlatON](https://www.platon.network/)不认为TEE是足够安全的（确实爆出[漏洞](https://www.bleepingcomputer.com/news/security/sgxspectre-attack-can-extract-data-from-intel-sgx-enclaves/)，但也及时修复了），所以他们聚焦在mpc算法做优化
  * 经过两年研发，他们基于逻辑门电路的编译层加密来隐私和算力
  * 很像做智能合约隐私的Origo项目

还有不少像[difinity](https://dfinity.org)和Cardano这样的layer1 扩展项目，以及Danku这样layer2的扩展项目，静待发展吧

### 虚拟机和多链扩展
现在几乎所有的智能合约平台都无法实现EVM虚拟机的并行化，开放的节点政策和恶意节点的存在，使这个问题异常复杂。目前有一些宣称往这个方向的项目，预言像cosmos和irisnet这种多链的扩展，对细分行业应用更加够好，能够更快的推广。各种不同共识机制的创新和匿名保护以及智能算力的应用都可以放在这个多链框架内，有点一统江湖的感觉，即使普遍都没有成型：

* [COSMOS](https://cosmos.network)是基于多链并行项目
  * 第一条链是Hub，作为中心链和管理员，类似master节点，但本身有去中心化保证
  * 其余的Zone是子链，重要性和Hub是不平等的
  * IBC是Zone和Hub间交互的协议，Hub不限制Zone的实现，只要是通过IBC协议和Hub交互就是一个合法的Zone
  * 在其之上[irisnet](https://www.irisnetwork.cn)整合了包括ipfs在内的实用工具，为特定的应用场景做优化和服务接口，都为Go的开发环境
* [Lisk](https://lisk.io) 定制侧链平台
  * 基于模板的定制，包括了发币和功能为一体
  * 对JavaScript，NodeJS和web apps的重点支持
  * [Ark](https://ark.io)是Lisk的DPOS克隆，而Lisk起源于[Crypti](https://github.com/borispovod/crypti)项目，是这一技术线的脉络
* ELA ([Elastos](https://www.elastos.org)) 联合NEO和Bitmain的[G3](https://neonewstoday.com/events/g3-summit-event-report/)联盟
  * NEO的POS协议逐渐成熟了，但是Elastos的所谓区块链操作系统还不见踪影
  * Elastos声称将主链（用于交易确认）和侧链（用于执行智能合约）分离，而且Runtime支持OS, EM 和 SDK
    * 实际上侧链就是一个state channel
* Telegram的TON也是多链和sharding公用的方案

# 弱点与攻防
## 操纵交易
* Double spending: 占有超过1/2的算力，就可以同时发布两个冲突的交易，先让欺骗性的那个领先，然后再让对自己有利的隐藏块赶超取而代之
* transaction malleability attack (double withdraw): 取币时，如果发币方以TXID为标识，收币方可以在实际收到币以后，在保持签名不变的情况下，篡改TXID (Protocol bug: TXID hash include everything but signature in a script dosen't include redeem script which is replaced by a dummy scriptcan -recursive. So real script can be altered after the transaction with out changing the signature but change the hash.) 因为签名不变，所以被篡改的TXID也可以被执行和记录，和原来hash的TXID要竞争算力看哪个被认可。
* Sybil attack:
  女巫攻击，创建多个分身，获取超过1/2的算力，可以配合double spending,也可以进行“自私挖矿” - 挖到新的块以后先私下保存，监控其他节点的情况，浪费对方算力，并在最后关头广播新块。
* eclipse attacks: 将某个节点的外围网络进行控制，可以将该节点孤立并且保持算力压制

### 智能合约的交易漏洞
基本交易流程已经反复验证，但是基于交易的智能合约在各种程序判断的条件约束下，隐藏着新的漏洞。合约的每个交易，严格说来，首先不一定成交，然后交易类型分好几种，最后还有很多的条件判断，其中任何一个环节出错都有被攻击利用的可能。

* Fomo3D死亡3分钟
  * 首先fomo3D融合了倒计时拍卖和庞氏分层激励。游戏开始倒计时，清0无人下单后，最后一个下单的人获得奖金池。简单的倒计时，玩多了很多老手已经不信了，虽然区块链解决了庄家机器人作弊的问题，但任然无法解决其中的逻辑陷阱。在倒计时还有1秒钟的时候，玩家有两个选择：不下单的话收益为0；下单的话有一定概率获得奖金池，加上下单成本不高，所以ROI看上去会非常高，总有人选择下单。但实际上，下单的获奖概率是所有人在你之后都不跟单的概率。当人数达到1000人时，即使每个人都很忙，人均跟单概率只有1%，但是所有1000人都不跟单的概率为$$(1-1\%)^{1000} = 0.004\%$$ 不到万分之一。
  * 为了改造这个问题，吸引老玩家上钩，fomo3D把发币那一套引入进来。赌徒下单买的是不一次性门票，而是具有分红效应的钥匙。之后所有的钥匙销售额，都将48%上缴给累计的钥匙持有者，2%交给社区，剩余的50%才存入基金池。但这个分红是一个递减数列，假设钥匙价格为1，你是第1000个买入者，那你的分红就是0.48/1000, 0.48/1001...0.48/2000...这个衰减是如此之快，到了第10000个人入坑的时候，你的收入才是1.1元，利润10%。如果不巧你是第10000个买入者，那要等到10万人下单后你也才赚10%。可见不论何时加入，即使等到规模扩大了10倍，也没赚到多少钱。
  * 所以上述两个规则下，这个游戏是不赚钱的，只有最早的投入者，和最后拿到奖金池的人才有收益。最早投入者显然是开发者，而最后拿到奖金池，则要求你把所有跟投者挡在门外。在2018年8月22日下午游戏结束的最后3分钟里，居然只有一笔订单成交，所有其他的跟投者都被阻挡在外。这个唯一的订单地址拿走了奖池里的10469个ETH，近300万美金。其实从6191898区块开始，就出现了黑客的账户0x18e1b664c6a2e88b93c1b71f61cbf76a726b7801，他的每一笔交易需要耗费360万或420万的gas，两笔交易加起来是780万gas，而ETH一个区块目前能容纳的gas总量是800万左右。因为0x18e总gas费高，会被优先选择打包，又因为它占的gas量大，所以两笔交易就能填满整个区块，容不下其他的交易。于是在交易消失的死亡3分钟里，在区块6191896与区块6191909之间，所有的节点都在打包与0x18e相关的交易，即使这些交易因gas超量失败无需付费，但成功阻挡了其他跟投者的交易。那么在此之前，他所掩护的，就是真正的获奖账户0xa169。他们其实已经演练了好几次，在最后那次确保0xa169拿下最后订单时，0x18e全力阻挡其他交易，掩护0xa169上岸。
* 问答游戏的交易隐藏
  * 和交易阻塞与掩护这种主动攻击型不同，问答游戏QUESTION的合约是一个[钓鱼诱饵](https://wallstreetcn.com/articles/3385163)。
  * 合约创建者会设置一个问题，然后明文设置答案的交易看上好像是一个漏洞，正确做法是提供答案的哈希
  * 其实合约作者利用一个中间合约将当前游戏的题目给改掉了，这个类型的交易在etherscan中会被隐藏，但可以在交易的Trace中查看到。所以只看etherscan的开发者很容易上这个钓鱼诱饵的当。
* EOS DAPP[漏洞频发](https://bcsec.org/index/detail/id/273)，已成黑客取款机
  * 经典的重放漏洞
    * 有人在微信上丢骰子就玩过这个花招：两个人同时丢骰子，其中一方马上断网，并且偷偷丢出另外两个骰子。老实人的骰子得出了结果，却看到对方的骰子一直在转。其实作弊一方已从自己多个筛子里选出最大的那个，把其他取消掉，再重新联网。这样作弊一方几乎总是可以丢出6，虽然会稍微慢半拍。
    * 类似的重放漏洞使EOS Happy Slot 中招：黑客首先赌较小的金额，并在 transfer 方法中加入**钩子开关**。transfer 过程可以被黑客预留的钩子中断，所以 reveal 被卡死，当 reveal 被卡死时候，并且这时候 bet 是仍然可以被调用的，重放攻击就成立了。黑客选择默认关闭开关，当检测到得到五倍以上的开奖金额之后，就再发一笔数额较大的金额，并开启开关，从而每一次都可以获得数倍的回报。
    * FairDice的攻击类似，由于 FairDice 的随机数算法和时间相关，因此多次同一笔下注在不同时间开奖可以获得不同的结果，黑客正是利用了这一点，拒绝了所有失败的开奖结果，从而可以让自己的下注总是获胜。
  * 币型判断失误，纯粹的人为失误
    * EOSBet被攻击，是因为代码中并没有检查收到的EOS是不是eosio.token产生的EOS，黑客通过自己创建的名字同为EOS的“假币”，套取了真的EOS。
    * 其他拷贝代码的同类项目，EOS.win/dice，去中心化交易所- newdex都一起中招。

## 操纵价格
比特币因为去中心化，所以他的价格是完全市场作用的，这就给大资本操纵比特币市场提供了工具，比如Pump and Dump。为了应对这样情况，出现了几个变种：
* [Tether](https://tether.to/) - 通过基于美元的中央储备reserves，来控制数字货币价格，成本很高，但是储备深度并不对外公布
* bitUSD - 同样使用中央储备bitShares，但是把他放到了一个分布式区块链上。当bitUSD低于$1时，用bitShares购买bitUSD以[维护其价格](https://coinmarketcap.com/currencies/bitusd/)
* [NuBits](https://nubits.com) - 就牛逼了，直接通过调控货币供应来控制升值，通过“加息”来控制抛售跌价，甚至也有NuShares对冲货币。
  实践下来，NuBits在2016-05因为遭遇巨大抛售，价格暴跌，最后在6月跌至50美分而失败。但bitUSD和Tether目前都成功维持住了价格
* [Dai](https://makerdao.com) 是更加市场化的债务抵押机制，通过抵押市场和货币市场的双向对接来平衡Dai对美元的币值
  * 最早基于以太坊平台，现在可以跨平台Multi-Collateral Dai
  * 和现货市场利用期货市场来对冲的概念一样，两个市场来进行双向负反馈。
    * 以ETH为抵押物，以美元为挂钩目标举例：
    * 当我预计ETH会涨价，我就抛售Dai交易ETH，Dai降价。ETH完成升值带来抵押市场套利空间，更多ETH抵押物来发行Dai，双向加速Dai的稀释。最终Dai的稀释和ETH的增值达到平衡，维持Dai对美元的稳定
    * 当我预计ETH会降价，我就购入Dai，Dai升值。抵押率降低，赎回ETH降低Dai的发行，加速Dai的稀缺。最终达到平衡
  * 和期货市场假设一样，有爆仓和期货违约的[风险](https://prestonbyrne.com/2018/01/11/epicaricacy/)
    * 抵押物市场和现货市场的流动是不平衡的，抵押物以ETH为保证，现货市场以美元为保证。如果ETH市场崩盘，ETH对美元跌停，转而购买Dai抵押物。看似Dai升值了，但是也带了Dai的违约风险，ETH不值钱了，以他为抵押的Dai存在系统性风险，Dai在美元市场遭遇非理性抛售。
    * 如果ETH异常波动，抵押物投资方判断错误，刚完成抵押ETH就升值了，这时只有持有Dai等待。而买ETH升值的人现在应该直接跳过Dai，兑换美元套现。Dai跌停，抵押方资产缩水，面临爆仓的风险。
  * 综合下来，Dai是在ETH以及美元中间的一个流动缓冲池
    * 他的稳定性以两方主力货币为基础，在技术上利用债务投资损益来对波动进行了缓解。
    * Dai作为中间缓冲池，在出现异常逆流的情况下，有可能流干失去调节力。
    * 在缓冲池强制平仓的时候，一定是有人赚钱有人赔
### 货币流动与金融博弈
价格博弈是始终存在的，因为存在货币流通。如果一个国家实行外汇管制，那么汇率是很容易稳定的。一旦开发，就是一个自由博弈的过程。货币价格背后反映的是货币供应，货币供应影响实体经济，通过货币流动攻击可以攫取一个国家的资源，重创经济。

假设大国A和小国B，货币攻击暂定为大量购买敌国货币，导致敌国通货膨胀，抑制出口，在经济脆弱时，抛售货币，粉碎经济。
* 当A大国攻击B小国时，只有B国所有人自觉抵抗，才能稳定币值。但从个体角度看，当汉奸的收益是最大的，这就变成了囚徒困境。所以小国必须外汇管制
* 当B小国攻击A大国时，只要大部分人抵抗，就能获胜，汉奸收益不大，且有道德风险，所以A国愿意开发外汇市场

所以在跨国金融博弈里，要么国家总体实力强，要么个体间协同约束强，才能开放市场，否则实施中央外汇管制才能获得博弈平衡

# 应用实例
区块链的价值已经获得普遍认可，中国也在2018年3月由工信部[筹建全国首个区块链和分布式记账标准化技术委员会](https://www.leiphone.com/news/201803/hLtR0J38yfA9TduJ.html)

## 交易结算
正因为比特币不是实体，而是交易的记录，所以比特币网络更像是一个脱离于实体金融的匿名交易中间商。这里分为庄家和交易方：
1. A是买家，花了2000$ 从庄家购买一枚比特币
2. A将比特币支付给B，B给A某种匿名的服务或商品
3. B将比特币卖给庄家，获得2000$

上述流程可以看到，实际上对于现实的买卖双方，不会长期持有比特币，所以需要币值保持稳定，双方就是等价交易的。  
而对于庄家而言，如果长期持有同时收支平衡的话，持币量和实体资金量是不变的。但是现在比特币增值明显，大家买的其实是未来作为庄家的权利。当大家越来越认可这个交易平台的作用，在代币总量有限的情况下，交易需求越大，币值就越大，庄家的影响力就越大。这就是比特币升值的内在动力。

NANO借助[Twitch](https://www.theusacommerce.com/nano-nano-this-cryptocurrency-is-going-mainstream-starting-with-twitch/)的支付渠道，也一跃冲击主流了

### 账本合约
[Hyperledger](https://www.hyperledger.org)是商用分布式账本，结合基础智能合约的功能。旗下有很多细分功能的子项目：
* BlockChain Explorer 展 和查询区块链块、事务和相关数据的Web应用
* Fabric 区块链技术的实现
* STL - Sawtooth Lake  度模块化的分布式账本平台
* Iroha 轻量级的分布式账本, 侧重于移动
* Cello BaaS的 具集,帮助创建、管理、终止区块链

其中[Fabric](https://github.com/hyperledger/fabric)使用复杂的多元体分布式架构：
* 身份管理，因为商用账本要兼容现有的商户IT，所以这里增加了一层“虚拟账户”
  * public key infrastructure (PKI) 管理秘钥和数字认证
  * Membership Service Provider (MSP) 管理子网路的参与方，认证方和消息订阅方的关系
* Peer节点，储存了核心账本和合约，可以1对多的关系
  * Ledger账本信息
  * Chaincode合约代码
  * Peer通过身份管理可以归属到特定组织Organization
* 链 = Peers + Ledger + Ordering channel
  * 多链中，链将参与者和数据(chaincode等等)隔离
  * 一个Peer可以参与多个链
* State database 保存全局交易信息
  * LevelDB and CouchDB这类Key/Value数据库

### 跨境结算
[Ripple](http://ripple.com)作为获得了Google支持的B轮5500万美元投资的明星企业，正在支付结算上快速渗透，尤其是跨境支付，提供更高效，更低成本的方案。本质上是使用一种半中心化的可控网络，作为国际支付中间商，使用[XRP](https://xrpcharts.ripple.com/)代币，快速进行分布式结算，绕开了传统跨境结算中心。

对于结算交易来说XRP的价格变动其实影响不大，但是XRP作为庄家控制权的代表，一开始的初始1000亿XRP币，200亿会给予投资人和创始人，500亿会被免费派发（即一段时间内开设账户免费，不需要挖矿，网络成本靠交易手续费覆盖），另外300亿将由OpenCoin持有，不定时抛售以获得利润。OpenCoin也坦承，持有及抛售XRP币是其赢利的唯一途径

### 交易所
作为交易中间体，需要将数字币在发币前频繁的转换。作为矿工的收益，也是矿工保证去中心化的基础，需要有保值升值的功能，还要保障流动性。所有这些都构成了数字货币交易所，可以说交易所是区块链初期的核心支柱，因为在token被广泛接受用于支付之前，token间和法币间的交易，是频率最高的交易。

但是偏偏，目前主流的大型交易所都是中心化的，用户开设的账号和余额只是交易所内部系统存放的一个数字记录。用户的存款是统一存放在交易所自己的大账号里，这导致交易所一旦被黑客攻破，就有可能损失大量的存款。而用户在交易所内的资产，并没有关联自己的私钥，并不真正由自己可控。而且项目上交易所的过程非常不透明，内审制导致腐败，而投票制带来公开的贿选，所以去中心化的交易所有不少尝试：
* bitshare等基于智能合约的资产交易平台（智能资产），自然也实现了跨币和法币的交易
  * openledge也是一个东西
* Cosmos Hub的IBC通信可以直接实现其子链间的跨链交易，无需交易所

fcoin将激励机制放到了交易所里，分享总计51%的FT币，每月按照交易量分发，6月首月据说分发量达到千万美元。后续平台的81%收入也会分红，所以带来了前期大量的初始玩家。但前期走势发现，FT持有者的分红情况并不能弥补FT的价格下跌。其主要原因是，由于存在邀请返佣的机制，使得大量矿工在进行高频交易，而且会将第一天返还的FT卖出换为BTC、ETH之后继续刷量。因此存在大量的FT抛压，按照每天接近1.5%-2%的释放量，根本不可能接的过来。所以后来6月29日公告暂停返佣活动，但刷单一旦撤离，平台收入锐减，币值的跌势无休止。交易者用BTC,ETH的交易手续费去换fcoin，需要币值支撑。一旦跌破临界点，堆砌无意义的交易，支付手续费来换fcoin也就变得无利可图。fcoin8月已经凉透了，但怀揣技术理性的韭菜一旦被收割，区块链的技术社区支柱将遭受打击。

钱包因为掌握了大量用户数据，实际上也在成为一种交易所，只不过有可能你的钱还是存在一个区块链地址，但是私钥会由钱包托管，在服务端或者客户端。最安全的方式显然是保存离线私钥，但这也最麻烦，所以交易所，热钱包，冷密码最终形成了从安全到便捷的3个层级。

### 金融衍生品
智能合约最明显的用途是在金融领域：
* CFD (Contract for Difference) 差价合约
* ETF (Exchange trade fund) 交易所开放指数基金，交易所交易基金
  - Bitcoin ETFs 被SEC严格监管，基本没有放行的，包括Winklevoss Bitcoin Trust ETF
  - Blockchain ETFs 更容易批准，但太分散，不如比特币来的集中
  - 尽管监管限制，但是JPMoragan在2018年2月已经改变立场，开始[推荐比特币ETF](http://www.businessinsider.de/jpmorgan-explains-why-a-bitcoin-etf-is-a-holy-grail-2018-2?r=US&IR=T)
* 存款合约

芝加哥的Chicago Board Options Exchange ([Cboe](http://www.cboe.com/))有着最早的比特币期货交易所，但随之而来的也是交易监管。2017年12月起，Coinbase、Bitstamp、itBit和Kraken等交易平台需要将与期货合约有关的交易数据分享给芝商所，但目前已经提供的都不满足要求，导致美国商品期货交易委员会(CFTC)介入并传唤这些交易所。

国内火币网以及关联的金色财经，从年初的[杜均黑幕](http://www.pingwest.com/dujun-the-banker/)炒作到6月11日大跌下的砸盘，充分暴露了这一法外市场被恶意操纵的必然。

2018年7月coinbase终于获得了美国证券监管的[认可](https://techcrunch.com/2018/07/16/coinbase-reportedly-gets-approval-from-u-s-regulators-to-start-listing-tokenized-securities/)，将数字货币以有价证券(deemed securities)形式交易。但哪些数字币可以定义为证券，需要参考豪威测试(Howey test)。后续的监管细则还没有出。

## 资产登记
除了以代币形式记录交易外，还可以把物权的转移记录在区块链上：
* 固定资产登记
  - 美国佛蒙特州Vermont在2018年3月完成了第一个[区块链房产交易](https://www.zerohedge.com/news/2018-03-08/first-us-real-estate-transaction-blockchain-completed-whats-next)，平台是ETH
* 车辆登记
  - 有德国大学项目，利用3个节点的私有链，帮政府做[车辆交易登记](https://github.com/dfherr/carchain)。一秒钟一个block，兼顾了数据安全和性能，也是在ETH平台上
* 生产供应链
  - 以整车生产为例，从供货商进的货，到装配在成品车上，目前是用VIN码做关联
  - 以后市场为例，二手配件和维保记录，都可以作为资产登记的一种形式记录到区块链上
* 数字资产，类似虚拟物品
  - 常用的资产标记是ERC721标准的资产币ID
  - 也有基于monero的侧链项目[tari](https://www.tari.com)来做更复杂的数字资产

## 币权融资 ICO
ICO不是必须的，比特币没有ICO过，IOTA、CarChain这类物联网项目也不需要ICO。  
ICO的目的是以项目代币为权益单位的融资行为，IPO的对标版本。当区块链项目开发成本高，周期长，可以通过ICO来给开发团队融资激励，同时吸引更多的产业合作伙伴。好比你开了一个游乐场（赌场），进场全程用代币，那么喜欢来玩的人多，门票就贵，黄牛就多。  
![top 10 icos](https://raw.githubusercontent.com/linkinbird/blockchain_book/master/pic/ico_ranking.png)  
这个做法存在很多问题，Facebook和google都屏蔽了ICO的广告投放：
* 估值全靠吹
	- 还没有形成专业的估值体系，比风险投资更危险，大量拿了建筑图纸，就来融资了
	- 也可能被价值低估，因为专业的项目，识货的人不多
* 执行难监管
  - 用代币换来了法币，那意味着还是要遵循公司法和经济规律，比如一级市场
  - 但对代币无约束，提前抛售，割韭菜，项目容易虎头蛇尾，好比二级市场

但也存在把ICO价值和实体经济价值挂钩的方式，下述两种都有注册证券交易所的可能，也在某种程度上稀释了法币交易所的权益范围：
* 收入转移协议
	- 比如公司业务收入，会按比例充值到指定区块链里
	- 或者公司某类经营流程必须通过区块链，并产生手续费
* 股权转移协议
	- 公司经营活动的投票权和代币持有挂钩
	- 基本上就是股权的代币资产化

另外还有用代币做营销推广的玩法：airdrop，撒币。但总归一句话，你游乐园造的好玩，自然有人来。建筑质量要一批批游客验证以后才有价值。现在很多游乐场有一个游乐项目就敢开门营业了，之后肯定会逐渐被综合性游乐场（成熟技术整合的综合链）淘汰。

## 智能合约
如果说比特币是一种价值网络，加上他本身也是一个信息网络，所以把信息扩展为算力，就可以形成完全独立的算力和价值兑换网络，也就是智能合约 Smart contract。通过价值币，来购买网络里的计算能力，这里的算力是指商业化的算力，而不是维持proof of work的基础算力。未来如果物流，人流也完全信息化控制以后，都可以纳入智能合约的结算网络内。

以太坊[Ethereum](https://www.ethereum.org/)就是基于区块链技术的智能合约货币，曾经经历一次分裂，因为核心开发团队的货币被盗，所以从原来ETC切换到了现在的ETH。
但其实ETH框架的潜力更大，可以做计算和储存，是“类图灵完备”的，好比一个分布式世界共享计算机Ethereum Virtual Machine (EVM)。其野心就是彻底把整个互联网的中间人（服务器）除掉。

但目前ETH的blockchain运行程序的能力只相当于老式手机，无法允许用户运行哪怕稍微多一点的代码，还要记录数据，成本很高，速度很慢，唯一优势就是去中心化的运行保证。所以目前只有一堆小应用[Dapps](https://dapps.ethercasts.com/)。通过GAS来限制每次合约的计算量上限（避免死循环耗尽算力资源），再用Gas price为算力出价。

ETH也是可以挖矿的，因为[Ethash](https://github.com/ethereum/wiki/wiki/Ethash)算法对CPU和内存都有要求，所以暂时无法设计专门的矿机，用个人电脑是比较合适的，这样也更加公平。
### 预测市场
目前Dapp里最多的就是彩票机，比较公平，但更有想象力的是预测市场，就是变相赌博，美其名曰群体智慧：
* [Augur](https://app.augur.net/) 最早的预测市场，还在Beta版本，区块货币为REP目前交易还比较少
* [GNOSIS](https://gnosis.pm/)是以Augur为对标的项目，预测市场专家Martin Koppelmann和Stefan George带领，顾问团队更是星光璀璨，包括以太坊创始人Vitalik Buterin和ConsenSys创始人Joe Lubin。也是以太坊平台上第一个众筹的应用，在2015年一共众筹到500万美元

## 数据市场
本质上区块链是分布式技术的一种特例，数据也是一种独特的资产。目前分布式数据技术已经证明创造了非常多的价值，而区块链交易和智能合约做一定改造，就有可能创造更加去中心化，又高效的大数据平台。这里说的数据库和Hyperledger Fabirc里在交易端点间储存交易状态的[CouchDB](http://hyperledger-fabric.readthedocs.io/en/release-1.1/couchdb_as_state_database.html)不同，是更通用的和区块链协议平行的数据架构，比如“空间扩展”里讨论的filecoin或者blockchain on hadoop这样的夹心结构。这样之前大数据的应用几乎都可以移植到区块链里，存储效率不变，但是所有权去中心化和数据产权保护可以做的更好。

数据市场按照数据来源和应用领域可以分为很多行业，但问题最后抽象下来，都是一个数据安全流通和交易使用的问题。其中包含了区块链智能合约的资产认证，IPFS的分布式存储，还有安全多方计算。技术方向上是相通的，在不同应用领域开花：
* 车联网还是以[carro.io](http://carro.io)项目为代表，完成了设备端、钱包端的加密控制和智能合约的资产登记。下一阶段是落实IPFS的存储
* 基因数据领域的[药明明码](https://wuxinextcode.cn)去年融资2亿多美元，扶持了一个区块链[基因数据](http://lifecode.org.cn/)平台。其中不以算力挖矿，是以数据换数据，有点像“文库”。
* 2018年新成立的项目[covalent.ai](http://covalent.ai/)，海外留学生做的，技术包括可信执行环境（TEE），完全同态加密(FHE)和零知识证明(ZKP 具体是zk-STARK版本)。类比IPFS定义了完全分布式存储，他们也想定义去中心化数据交易的标准
  * 6月获千万美元投资，投资者包括FBG Capital、真格基金、大都会资本、IOST、节点资本等。
  * 链上第一个应用预计是交易医疗数据的medata
  * 在智能算力（包括Caffe RNN这样的深度学习算力）也有涉及，但缺少详情

### IoT 物联网
一般物联网数据比较琐碎，单位价值不高，所以基于POW的区块链技术带来额外的成本是难以接受的。目前主要应用在IoT领域的项目还是IOTA和加拿大滑铁卢大学团队搞的[Iotex](https://iotex.io)。他们通过DAG网络的资源共享，达成零手续费的数据交易通道。将成千上万的城市传感器链接在一起。

入门开发者可以选择[ARDUINO](https://www.arduino.cc)这种开发版子，有[教程](https://www.youtube.com/watch?v=T2b5WZDfg3c)叫你如何将ARDUINO UNO的温度数据上传到Tangle。

### IoV 车联网
作为物联网的一个细分领域，汽车在成为第二大移动终端（第一大是手机），而汽车深厚的产业链集群，使这些数据的价值远超一般物联网领域。所以大数据区块链在IoV这个领域起步很早，已经有几家2017年起步的项目。同时2018年的博世开发者大会上，包括保时捷，大众在内的主流厂家都纷纷加入。
* [CyberCar](http://www.cybercar.io/)，原来做通讯业务的团队创新项目
* 阿尔法车链 是整合了很多技术的综合体，但也没有主页，据说要上韩国交易市场
  * 基于Hyperledger Fabric的Alphaledger交易链
  * 车主通过购买汽车俱乐部上的后市场服务来获得奖励的积分
  * 通过以太坊发行 ERC20 代币 ACAR（Alpha Car）Toke来和积分兑换
* 据说有SIGMA团队的无名车联网项目，但是主页都还没有


## 自由之城
在比特币规模不大的时候，作为交易记录和中间件是比较轻的存在，并不影响现在社会的秩序。但是极限情况，如果比特币的总规模可以达到一座城市甚至一个国家的GDP，那么完全有可能成立以比特币为法币的国家主权。一旦主权化，“比特国”内部的比特币交易就不受通货规模，不受政府、央行的影响，成为稳定的代币。而所谓比特币的价格也只是本国货币对外币的汇率。

那第一个问题是什么样条件使得上述情况成立？这里要关注3个指标：
1. 持币人数——这里对标的是城市人口，因为只能统计秘钥地址数量而无法具体到自然人，所以有的说有200万人，有的说有1000万，无法确定。但可以说达到形成小社会的基础了
2. 流通货币数量——最终会有2100万比特币产生，因为一个比特币还可以拆成很多聪，所以流通作用也没有问题。毕竟在“比特国”内部比特币是和物体交换，所以物价是和流通量相互平衡的
3. 国际流通标准下的市值——如果“比特国”自给自足，人口固定，那么这个市值没有意义。但全球化的今天，“比特国”同样需要国际采购以及支持资源、人口流动。那按照市值阶梯：
	- 1 Billion\$ =完整功能的小企业
	- 10 Billion\$ =大城市一个区，一个多元化企业
	- 100 Billion\$ =一座城市，一个国际企业
	- 210 Billion\$ =比特币单价1万美元的极限市值
	- 300 Billion\$ =[Marginal Revolution](http://marginalrevolution.com/marginalrevolution/2017/11/bitcoin-just-bubble.html)预估的2017年底比特币市值

这就产生了第二个问题，如何一步步达成上述目标：
1. 发布国家宣言，预备公民招募——就是比特币白皮书，以及购买比特币认为是一种预备公民
2. 国库充值——靠大资本购买国债（外来投资）和民众信仰充值（税收变为持股）来支持。虽然看上去很民主，但是早有大资本布局。
3. 外汇管制——国库充值完成以后要快速锁定，因为预备国民是非常容易流动的，到这个阶段如果建国失败，或者被别国竞争压制，那会面临崩盘的损失。所以为了能够实施下面一步的最终身份确认，需要一定时间的外汇管制，以切割法币和外币的界限。如果没有这个界限，国家的根基会非常薄弱，而且容易被外汇市场攻击。
4. 身份认证（管制）——身份代表了交易信任，本国居民间的交易可以有稳定通货来保障，而不受外汇市场的影响。持币但是没有确认身份的当做外围民，受到外汇管制限制。只有在身份界限清晰之后，才可以精确的计算国库货币量以及为居民身份定价。
5. 货币闯关——当国民身份确定，国家货币量确定以后就可以计算人均身价。国或者单个人的最终价值衡量标准都是生产力。如果价格高于生产力，那法币就会被外围民迅速抛弃，外汇和人口在这个时期都有可能剧烈波动，所以管制是维稳的必须条件。实体国家都有军队和政权来管制，但是“比特国”要如何管理，是一个问题
6. 国际建交——在一个没有政府的“比特国”里，仍然需要国际化合作来提高生活水平。稳定了人口和货币以后，可以按照现在的国际惯例和各国建交，设立外汇市场。
7. 国家机器——到底是否需要国家机器？在暂时稳定以后，如果完全开放人口和货币流动，会不会在多个平行比特国之前，产生激烈的竞争？毕竟一个国的市值，是他国民竞争力的代表，那人分九等，人的流动可能直接导致国的崩塌

虽然不知道有没有可能比特币走完上述的过程，但是直觉感到，即使建国成功了，国家机器最终还是会变成和现在类似。毕竟排除了物理资源以后，只有人是这个国的唯一能动力和资源禀赋。而人分九等，人始终是社会人。

目前的比特币是在交易中间件和价值储存两个功能中间震荡。以官方[交易统计](https://charts.bitcoin.com/chart/daily-transactions)为基础：目前币值10k\$，市值100B\$以上，每笔交易一直维持在5个bitcoin，单日交易30万笔*每笔50k\$ = 15B\$ 占外汇市场日5000B\$量级的1%不到，不知是否满足黑市交易，没有官方数据，很难说。

### 加密经济
星云链[Nebulas](https://nebulas.io)推出一种经济指标，并以此来标定不同账户对经济总量的贡献度。他们的星云指数公式是账户资产余额中值和账户交易出入度的乘积。经济体要交易，也要稳定。回顾常见的Coin age加权，在一定时间内将交易次数乘以距离上次交易的时间再累加，会导致一定时间内交易10次和交易1次的效用无差异。这种coin age的经济指标只和资产总量相关，并没有鼓励交易。但同样过快的交易也有刷单嫌疑，所以交易的综合效用函数应该是两头低，中间高的。这种效果正是通过余额和交易的混合激励来实现：账户指数 = 余额中值 x 交易总金额。账户余额中值即统计时间内持有时间过半以上的余额，以此防止同一笔钱在两个账户互倒获得翻倍效果。结合交易的流出和流入额，在持有和交易间让用户自己平衡。以合计为10的余额为例，相乘结果最大的情况是对半 $$5*5$$，就是保留一半，交易一半，也就实现了中间高，两头低的效果。

经典货币理论有这样的数量公式：$$M \times V = P \times Y$$。这里M为货币总量，V为流通速度，P是物价水平，Y是经济产量。Chris Burniske提出了类似的理论[公式](https://medium.com/@cburniske/cryptoasset-valuations-ac83479ffca7)：$$M \times V = P \times Q​$$。
* M = size of the asset base
* V = velocity of the asset
* P = price of the digital resource being provisioned
* Q = quantity of the digital resource being provisioned

其他还需要更多的经济指标来配合，不论作为调控政策的依据，还是币值定价的参考。从欧元区学来一些除了利率，汇率之外的候选指标和概念：
* 消费端
  * 消费者信心指数
  * HICP 消费者价格调和指数
  * 国际油价和粮价
  * 贸易顺差和逆差
* 生产端
  * 生产者物价指数PPI
  * 制造业采购经理人指数
  * 商业信心指数
  * 就业率
* 投资与货币
  * M3货币供应
    * M3=M2+其他金融机构的定期存款和储蓄存款
    * M2是钞票，硬币，活期存款
  * ZEW经济景气指数
  * 其他投资指数

# 参考资料
## books
* Bitcoin: A Peer-to-Peer Electronic Cash System - Satoshi Nakamoto
* Bitcoin and Beyond:
A Technical Survey on Decentralized Digital Currencies - Florian Tschorsch & Björn Scheuermann
* [Mastering Bitcoin](http://chimera.labs.oreilly.com/books/1234000001802) - Andreas M. Antonopoulos
## websites
* [比特币官网](https://bitcoin.org/en/) 包含比特币钱包的推荐
* [比特币区块和矿池信息](https://btc.com/)
* [比特币实时网络节点](https://bitnodes.21.co/nodes/live-map/)
* [Gnosis ICO 项目分析](https://zhuanlan.zhihu.com/p/26520429)