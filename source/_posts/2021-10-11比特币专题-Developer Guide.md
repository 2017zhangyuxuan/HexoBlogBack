---
title: 【比特币专题】Developer Guide导读
date: 2021-10-11 13:49:25
index_img: https://cdn.jsdelivr.net/gh/2017zhangyuxuan/picture_backend@master//hexo_img/bitcoin5.jpeg
categories: 
- [计算机知识,比特币]
tags: 
- Bitcoin
---


> 原文阅读地址：[Bitcoin Developer Guide](https://developer.bitcoin.org/devguide/transactions.html) 
>
> 预计通读耗时：12小时（个人数据，仅供参考）
>
> 中文翻译：
>
> [比特币开发者指南-巴比特图书](https://www.8btc.com/books/834/bitcoin-developer-guide/_book/5/5.html)  
>
> [简介 - 《比特币开发者指南 | Bitcoin Developer Guide》 - 书栈网 · BookStack](https://www.bookstack.cn/read/bitcoin_developer_guide/README.md)



# 章节概述

此次阅读的内容是比特币Developer Guide，详细介绍了比特币中各种设计和技术细节。

- Block Chain：介绍比特币的结构基础区块链，包括工作量证明，块高与分叉，交易数据，共识规则改变引起分叉等内容。
- Transaction：详细介绍了比特币中交易的相关内容，重点说明了交易的组成，交易中的UTXO的类型（使用何种支付方式P2PKH或者P2SH）。
- Contracts：这里的合约跟以太坊的智能合约不太一样，感觉更像是来说明交易双方（针对人而不是交易）该如何进行交易，应该遵守哪些规则。主要介绍了托管和仲裁、微支付通道、Coinjoin（混币）
- Wallet：钱包用来管理和花费UTXO，因为UTXO实际对应的是一个hash地址，因此钱包实际就是在管理公私钥，这部分内容也就更多在介绍公私钥的管理问题。
- Payment Processing：这部分重点介绍如何使用比特币进行支付的相关问题。
- Operating Modes：介绍比特币中全节点和SPV客户端。
- P2P Network：这部分就是详细介绍了比特币运行在P2P网络中相关细节，包括对等节点的发现与链接，全节点的初始化，区块传播等。
- Mining：介绍了单独挖矿和矿池挖矿两种挖矿（添加新的区块）方式。

# 重点内容

> 大部分是阅读各个模块时记录下的重点内容。

## Block Chain 区块链

- Block chain里UTXO的作用，只能使用一次，这也解决了双花问题
- Proof of work里也是充分利用了加密hash算法里随机的天然特性，hash number不可预知
  - 每2016块调整难度值，根据生成这2016块block的时间来调整，理想时间是1,209,600秒(two weeks).
- Transaction data：block里的第一个交易是一个coinbase交易（由生成的区块奖励和输入输出费用差组成），通常只有在100块之后，才能被花费使用，这么做的目的是因为分叉比较常见，防止因为分叉的原因导致原先的区块失效
  - 有个问题是，如何保证必须要在100块之后才能进行花费，是在创建block的时候，判断输入的UTXO所在的区块的深度超过100吗？
- 共识规则改变：共识规则改变，会有两种不同的情形，分别导致硬分叉和软分叉，不是很能理解其中的含义，查阅相关资料有了进一步认识。（现在比特币的更新，基本就是软分叉）

> **常见的理解是“硬分叉和老版本软件不兼容、软分叉和老版本兼容”**，这个定义是**不准确**的，但是很多地方已经在用了。。
> **硬分叉的定义是扩宽共识规则**，允许做之前禁止的事情，以前无效的交易/区块在硬分叉后会变成有效的；**软分叉是收紧共识规则**，禁止之前允许做的事情，以前有效的交易在软分叉后就无效了。
> **软/硬分叉是共识规则的改变，和链分叉/链重组完全是两码事；这两对概念的关系类似于“红烧/清蒸”与“烧糊/夹生”。**不当部署的软/硬分叉都有可能导致链分叉/链重组。
> 所以，可以想见：
> 硬分叉之后，几乎一定会产生让老节点拒绝接受的区块，所以，硬分叉会破坏前向兼容性；
> 软分叉之后，产生的新区块肯定是老节点也愿意接受的，前向兼容性得以保留。

- Detecting forks 发现分叉：通过监控区块链工作量证明而发现硬分叉的代码；监控最近区块的版本号



## Transaction交易

- **P2PKH** 即 pay to public key hash，向公钥哈希地址支付
  - 交易的输入使用一个TXID和output index number（对于output来说处在交易的哪个位置）来标识，然后还有一个签名脚本，验证该input对应的output包含的公钥脚本，用来证明这笔输入的钱的确是本人持有
  - 交易的输出包含一个隐含的index number，以聪为单位的余额量，一个公钥脚本
  - Bob要使用Alice的支付给他的UTXO，Alice创建转账的Tx交易时需要用到Bob的公钥，因此Bob的公私钥是提前生成好的，使用ECDSA椭圆签名曲线生成；然后Bob将公钥hash，使用Base58编码地址版本编号、公钥哈希、错误校验码，然后传给Alice，Alice可以进行解码得到公钥hash，并添加到交易中，作为output中的pubkey script，来标识输出到哪个地址；
  - 而当Bob需要使用这份UTXO，发布交易时，需要构造input，引用Alice创造的交易TX（即TXID和对应的output index number），还需要包含一个签名脚本(signature script），这个签名脚本用来验证这笔输入的确是Bob所有。这个签名脚本里包含了Bob的公钥和签名，而签名的数据包含了Alice创造的交易TXID和index number，输出给其他用户的pubkey script，输出的余额。而输出output，正如上述所说，包含一个隐含的index number，以聪为单位的余额量，一个公钥脚本。

![](https://cdn.jsdelivr.net/gh/2017zhangyuxuan/picture_backend@master//img/202110111350637.(null))

- **P2SH**：提出P2SH的目的主要是因为在之前的交易中，都是由发送者负责指定赎出币的条件。这样的话，如果赎出币的过程比较复杂，譬如要使用MULTISIG，那么对付钱的用户，也就是买家，就不够友好。使用P2SH的方式，可以由币的接收方设计好执行的脚本，然后不论脚本多么复杂，发送方只需要将币发送到一个20字节的哈希地址就行。
  - 有关资料参考：[P2SH机制](https://zhuanlan.zhihu.com/p/46072343)
- 标准的公钥脚本类型：P2PKH、P2SH、Multisig、Pubkey、Null Data
- 交易中Locktime和Sequence numbers的说明
  - Locktime, 也被称为nLockTime, 它定义了个最早时间，只有过了这个最早时间，这个transaction可以被发送到比特币网络。通常被设置为0，表示transaction一创建好就马上发送到比特币网络
  - Sequence numbers用来使签名者更新交易，如果sequence numbers设置成0xffffffff时，表明更新完成，就可以立即生效加入到比特币区块中，这个生效是不管Locktime是否过期。
  - 另一种理解：
    - LockTime ：绝对时间，用的是整个区块链的长度，或者时间戳来表达的。

- Sequence Number : 相对时间，当前交易所引用的UTXO所在的块（也就是输入所在的块），后面追加了多少个块。
- 参考：[深入浅出微支付通道](https://zhuanlan.zhihu.com/p/43171481)



## contract合约

- 结合之前的P2SH方式中，对multisig有了更进一步的理解。A向B买东西，A使用P2SH，把钱支付到一个脚本地址并使用2-of-3签名（此时比特币只属于这个脚本地址），当B发货了，A收到确认没问题了，那么使用A和B的签名就可以把脚本地址的UTXO转给B的地址。如何A反悔了，不肯提供签名，那么B可以找仲裁机构C，使用B和C的签名也能将脚本地址的UTXO转给B；同理B如果没有发货，A也能找C把钱转回给A。也就是说，相比P2PKH直接转账方式，P2SH相当于多了一步验证等待，验证成功了再转账到用户地址。
- Micropayment Channel 微支付通道

![](https://cdn.jsdelivr.net/gh/2017zhangyuxuan/picture_backend@master//img/202110111350785.(null))

- 其工作原理大体上可以这样描述：A给B打工，B首先使用A和B的签名，使用P2SH的方式，发送一定金额到脚本地址，并将该交易立即传播到比特币网络上；然后B创造第二个交易，并用到刚刚A的签名，其输入是第一个交易的脚本地址，输出是B的地址（相当于全额返回给B），然后给这个交易加上Locktime，比如说一天后才能广播这个交易。然后A给B工作一部分内容后，A要求B先支付这份工作量的薪水，那么B就创建一份新的交易，从原来全额给B变成分出一部分金额给A，这个新的交易拷贝给A，这样A就可以广播这个新的交易从而获得薪水。（实际上A只需要在locktime过期前，广播最后版本的交易即可）
- 为什么采用这样的方式呢，因为A诉求是及时支付薪水，但是因为量小，B不能每次都立即创造一个交易即刻支付，这样的交易费的成本太高了。所以利用这样的方式，既确保了A的薪水是及时得到确认的，又可以使得只需一个交易就一次性支付薪水
- 实际上，更改交易金额的输出，这个权利是在B的，因为B有A的签名但A没有B的签名，A拿的是经过B签名后的交易副本（这个交易被B签名过了，所以是有效的）。所以A能实时确保自己对应工作量的薪水能及时支付，就算中途B跑路了，也只是损失一小部分工作量的薪水，之前的薪水都可以得到支付。而对B来说，如果A没有工作，那B也能在locktime 过期后拿回自己的钱（不过这样就需要等待一个locktime的时间）

- Coinjoin混币交易，增加隐匿性，保护隐私，当和其他输入输出混杂在一起时，别人就难以追踪输出记录了

![](https://cdn.jsdelivr.net/gh/2017zhangyuxuan/picture_backend@master//img/202110111350075.(null))



## Wallet钱包

- 钱包程序主要可分为三个子程序，一个程序来发布公钥用于接收比特币，而另一个程序对交易做签名来花费比特币，还有一个网络程序用来跟P2P网络交互。更具体地说，一个完整的钱包程序应包括这些功能：生成私钥，生成对应的公钥，按照需要对公钥进行发布，监听以这些公钥作为输出的交易，创建交易并对其进行签名，广播已签名的交易。
- Wallet file 是对私钥进行管理；描述了私钥和公钥的格式
- 参考：[数字货币钱包 - 助记词 及 HD 钱包密钥原理_omnispace的博客-CSDN博客](https://blog.csdn.net/omnispace/article/details/79816141)



## Payment Processing 支付处理

- 介绍四种支付比特币的方式和相关的具体细节，包括明文，bitcoin:URI，QR码，新的支付协议X.509
- 其中提到一种“Merge Avoidance合并规避”的方法来保护用户隐私，该方法大体的意思就是让减少各个账户连接在一起的次数（或者说减少输入的个数）。因为当你使用UTXO作为输入进行花费时，UTXO原先的owner就可以追溯这次交易的信息，输入的UTXO越多那么能追溯到这笔交易的人就越多，隐私就会受到影响。用官方的例子来说，你有100，200，500，900的UTXO，然后你现在需要支付300BTC，那么你该选用500的UTXO来进行支付（而不是选择100，200作为输入，体现了合并，规避风险）。



## Operating Modes运行模式

- 比特币主要有两种运行模式：一个是全节点客户端（包含所有区块和交易信息），一个是SPV客户端（只保存区块头信息）。SPV客户端可以通过请求全节点拿到相应的区块信息，进行验证。然而SPV客户端有两个缺点，一个是可能会被全节点欺骗，解决办法是连接多个全节点，保证不要和诚实节点断开链接了；另一个是容易受到拒绝服务攻击，解决办法是布隆过滤器
- 参考：[布隆过滤器(Bloom Filter)、SPV和比特币 - shuwoom的博客](https://shuwoom.com/?p=857)



## P2P网络

- 因为共识规则不包括网络，所以有可选的网络和协议。这里用Bitcoin Core作为全节点代表，BitcoinJ作为SPV客户端代表。

### Peer Discovery

- 首先通过询问DNS seeds来获取对等网络其他有效运行的节点IP，和对等节点建立连接后，可以获取得到更多的网络节点IP。此外，在程序中会有一些固定的静态IP可以尝试连接，或者使用命令行工具尝试与指定IP连接。

### Connecting to Peers

- 节点通过发送version消息连接到一个对等节点。消息version 包含了节点的版本信息、块信息和距离远程节点的时间。一旦这个消息被对等节点收到，它必须回复一个verack。如果它愿意建立对等关系，它将发送自己的version消息。
- 一旦建立对等关系，节点可以向远程节点发送getaddr和addr消息来获得其它的对等节点信息。为了维持与对等节点的连接，节点默认情况下每30分钟内会给对等节点至少发送一次信息。如果超过90分钟没有收到回复，节点会认为连接已经断开

### Initial Block Download

- 一个全节点在正式工作或者提供服务前，需要进行初始化，把除了硬编码生成的第一个区块外的所有区块下载下来，这个过程就是IBD。
- Block-First 是其中一种下载方式，向一个对等网络节点进行询问，直接下载区块，其缺点也很明显
  - 下载速度的限制：只从一个同步节点下载，受限该节点的带宽
  - 重新下载：同步节点可能会发送不是最长链上的区块，就会导致快结束时才发现需要重新下载
  - 磁盘空间占用：和“重新下载”相关，下载时可能会将无用的区块保存到磁盘，占用空间
  - 大量内存使用：因为同步节点发送过来的区块可能是无序的，所以需要先保存到内存中，直到接收到父块才能进行验证
- Header-First 的下载方式，解决了Block-First中四个缺陷，它的工作方式是：先向同步节点下载block headers，当部分地验证headers有效性后，IBD节点就可以并行地做两天事——一个是继续向同步节点发送请求下载headers，另一个是向其他对等节点发送请求下载block



## Mining 挖矿

- 现在有两种挖矿方式：单独挖矿和矿池挖矿
  - 单独挖矿：bitcoind来获取P2P网络上的交易，挖矿软件通过RPC方法来获取列表，并构造一个Block模板，然后将对应的block header发送给ASIC进行运算。挖矿软件会将一个nonce值填入币基交易的的字段中，获得新的Merkle root的hash值，然后将新的Block header发送个ASIC。如果ASIC计算生成的block header hash小于预定的阈值，则表明添加的nonce值符合条件，将block header返回给挖矿软件，挖矿软件根据返回的block header更新block，最后将完整的block 返回给bitcoind，bitcoind再向网络传播区块

![](https://cdn.jsdelivr.net/gh/2017zhangyuxuan/picture_backend@master//img/202110111350617.(null))

- 矿池挖矿：工作流跟单独挖矿类似。不同的是，矿池设定的阈值要比网络上设定的阈值小很多（降低了难度），因此各个矿工通过挖矿软件返回给矿池的block hearder中，有很多是满足矿池的阈值但不满足网络的阈值，这些返回的block header相当于是份额，证明了矿工的工作量；同时总会有几率产生同时满足两个阈值的block header，矿池将满足网络阈值条件的block发送给bitcoind，从而获得奖励。然后矿池根据矿工贡献的份额，平均分发奖励。举个具体例子就是，矿工们总共返回了100个满足矿池条件的block header（相当于有100份额），只有1个满足网络阈值，那么每份额的奖励就是总奖励的1/100，矿工根据自己的份额获得相应奖励。

![](https://cdn.jsdelivr.net/gh/2017zhangyuxuan/picture_backend@master//img/202110111350905.(null))

- 还介绍了三种挖矿软件获取Block的RPC
  - getwork RPC：当前Bitcoin Core已经废弃，这个方法直接为矿工构造好block header，因此矿工可能需要调用成百上千次RPC
  - getblocktemplate RPC：获取以下内容；然后挖矿软件就能自己改变nonce值，自己生成block header
    - 构造币基交易的信息
    - bitcoind发送给矿池的交易列表和具体交易信息，这使得挖矿软件可以查看交易，并有选择性地添加或删除交易
    - 构造block header的其他信息
    - 矿池提供的难度值或者网络设定的难度值
  - Stratum：跟getblocktemplate很类似，但是不同的是，挖矿软件获取到的是重新构造Merkle树的必要信息，而不是具体的交易列表和交易内容。因此，挖矿软件不能添加或者删除交易，不过此时挖矿软件和矿池建立双向的TCP连接；在getblocktemplate 里，挖矿软件用的是HTTP longpoll长轮询，来获取最近的更新。

# 存在疑问

阅读完所有内容后，还是有很多问题和细节没有弄明白。

1. 原文：Since it is impractical to have separate transactions with identical txids, this does not impose a burden on honest software, but must be checked if the invalid status of a block is to be cached; otherwise, a valid block with the duplicates eliminated could have the same merkle root and block hash, but be rejected by the cached invalid outcome, resulting in security bugs such as [CVE-2012-2459](https://en.bitcoin.it/wiki/CVEs#CVE-2012-2459).

> 对于上述内容不是很能理解，不同的transactions怎么会有相同的TXID，可能是由于hash冲突导致的？虽然概率很小。然后“对于缓存一个无效区块的状态”，这该如何理解？什么时候用到了缓存，缓存什么内容（区块的状态？），以及如何判断区块是有效还是无效？如果因为有TXID冲突判定区块无效的话，可以去重TXID，达到有效？

1. 标准交易中，Null Data的的pubkey scripts类型用来干什么的？
2. P2PK 被 P2PKH所取代了，支付到公钥哈希的地址，可以使得公钥直到UTXO被使用时才会发布，延迟公钥发布的原因是什么呢？是为了避免攻击者利用公钥进行某些攻击吗？
3. 多种Signature hash types的用途是什么呢？为什么要有选择性的进行签名？
4. 交易被打包到区块，交易的费用是根据交易的签名字节长度计算出来，那么这笔费是由买家（支付者）来付吗？在对应的一个交易中，是会增加一个output来指向矿工吗？
5. 对于HD钱包，即分层确定性钱包中的工作原理还是不太理解，特别是extened keys到harderned keys的转变，为什么要这么做？这么做如何解决问题的？
6. SPV客户端具体是怎么使用布隆过滤器的？
7. 矿工发布区块时，具体是怎么给矿工发放奖励的？是在创建区块的时候，直接生成一个币基交易（把钱转账给该矿工），然后矿工打包所有交易后，开始找满足条件的nonce，找到后广播区块，这样如果区块得到其他节点认可上链了，那么矿工就切实得到了收益。（这样的话，每个节点在生成区块）



# 所思所感

阅读完这篇Guide概览，让我认识了解比特币中许多技术和实现细节，但是通读完一遍后，发现自己好像懂了，又好像没懂，或者说从整体上对比特币的整体架构有了一定的认识，具体有哪些部分组成，涉及哪些关键技术，但是对个各个模块进一步的细节还是似懂非懂，并且还是难以串联起来，各个部分有明显的联系（比如交易与钱包与支付处理），但是感觉自己还是很难将这三者的关系表示清楚，或者说，当把整个比特币看做一个整体时，各个部分是怎么样有机独立又相互配合的。

还有一个简单的思考，就是对于合约部分中多重签名的使用，例如托管和仲裁，A和B之间的交易还是要依赖于仲裁第三方C，这是否与比特币去中心化的思想相矛盾了呢？后来进一步思考和查阅资料，从另一个视角去看，去中心化，不是不要中心，而是由节点来自由选择中心、自由决定中心。简单地说，中心化的意思，是中心决定节点。节点必须依赖中心，节点离开了中心就无法生存（类似于没了支付宝就不能用淘宝？）。而在去中心化系统中，任何人都是一个节点，任何人也都可以成为一个中心。任何中心都不是永久的，而是阶段性的，任何中心对节点都不具有强制性。
