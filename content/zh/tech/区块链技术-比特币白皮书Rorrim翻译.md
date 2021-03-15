+++
title = "区块链-比特币白皮书(Rorrim翻译中...)"
date = "2021-03-09T10:56:00+08:00"
tags = ["BITCOIN","BTC", "BLOCKCHAIN"]
slug = "bitcoin-white-paper"
indent = true

+++

# Bitcoin:A peer-to-peer of electronic cash system

> Author:[Satoshi Nakamoto](https://zh.wikipedia.org/wiki/%E4%B8%AD%E6%9C%AC%E8%81%AA) satoshin@gmx.com www.bitcoin.org 2008.10.31

> 中文翻译: Rorrim worldofrorrim@gmail.com 2021.03.09

# 比特币:一种点对点的电子现金系统


> 作者:[中本聪](https://zh.wikipedia.org/wiki/%E4%B8%AD%E6%9C%AC%E8%81%AA) [Satoshi Nakamoto](https://zh.wikipedia.org/wiki/%E4%B8%AD%E6%9C%AC%E8%81%AA) satoshin@gmx.com www.bitcoin.org 2008.10.31

> 中文翻译: Rorrim worldofrorrim@gmail.com 2021.03.09

> **Abstract.** A purely peer-to-peer version of electronic cash would allow online payments to be sent directly from one party to another without going through a financial institution.Digital signatures provide part of the solution，but the main benefits are lost if a trusted third party is still required to prevent double-spending.We propose a solution to the double-spending problem using a peer-to-peer network.The network timestamps transactions by hashing them into an ongoing chain of hash-based proof-of-work，forming a record that cannot be changed without redoing the proof-of-work.The longest chain not only serves as proof of the sequence of events witnessed,but proof that it came from the largest pool of CPU power. As long as a majority of CPU power is controlled by nodes that are not cooperating to attack the network,they'll generate the longest chain and outpace attackers. The network itself requires minimal structure. Messages are broadcast on a best effort basis,and nodes can leave and rejoin the network at will,accepting the longest proof-of-work chain as proof of what happened while they were gone. 
>
> **概要：** 一个纯粹的电子现金的点对点版本,将允许在线支付从一方到另一方直接发送,而不需要通过任何的金融机构。数字签名提供了解决方案的一部分，但是，如果仍然需要一个受信任的第三方来防止双重支出的话，那么这个系统的主要优势就丧失了。我们提出了一个使用点对点网络时的双重支出问题的解决方案。网络通过将交易散列到一个正在进行的基于哈希的工作量证明的链条中来对交易进行时间戳记，从而形成如果不重做这个工作证明就无法改变的记录。最长的链不仅可以作为已被见证事件的顺序的证明，而且可以作为它是来自最大的CPU算力池的证明。只要CPU算力的绝大多数被不与攻击网络者合作的节点(良性节点)控制着,那么,良性节点将会超过攻击者,从而生成最长链.这个网络本身需要最小的结构。消息在尽最大的努力的基础上广播，节点可以随意的离开和重新加入，节点加入时，接受最长的工作证明链作为未参与期间发生的一切的证明。

---

## 1. 简介（Introduction）

---

Commerce on the Internet has come to rely almost exclusively on financial institutions serving as trusted third parties to process electronic payments.While the system works well enough for most transactions,it still suffers from the inherent weaknesses of the trust based model.Completely non-reversible transactions are not really possible,since financial institutions cannot avoid mediating disputes.The cost of mediation increases transaction costs，limiting the minimum practical transaction size and cutting off the possibility for small casual transactions,and there is a broader cost in the loss of ability to make non-reversible payments for non-reversible services.With the possibility of reversal, the need for trust spreads. Merchants must be wary of their customers, hassling them for more information than they would otherwise need. A certain percentage of fraud is accepted as unavoidable. These costs and payment uncertainties can be avoided in person by using physical currency, but no mechanism exists to make payments over a communications channel without a trusted party.

互联网上的商业几乎完全依赖作为可信第三方的金融机构处理电子支付。虽然对于绝大多数交易来说，这个系统运行良好。但是，它仍旧存在基于信任的模型的固有缺点。完全不可逆的交易实际上是不可能的，因为金融机构无法避免仲裁争议。仲裁成本增加了交易的成本，限制了最低实际交易规模，切断了小额临时交易的可能性，并且失去为不可逆服务支付不可逆款项的能力，这带来了更大的成本。随着逆转的可能性，对信任的需求蔓延开来。商人必须提防着他们的顾客，向他们索取比他们原本需要的更多的信息。一定比例的欺诈，被认为是不可避免的。这些成本和支付的不确定性，虽然在人与人之间直接使用物理货币支付的时候是可以避免的；但没有任何一个机制能在没有可信方的情况下通过通信渠道进行支付。

What is needed is an electronic payment system based on cryptographic proof instead of trust, allowing any two willing parties to transact directly with each other without the need for a trusted third party. Transactions that are computationally impractical to reverse would protect sellers from fraud, and routine escrow mechanisms could easily be implemented to protect buyers. In this paper, we propose a solution to the double-spending problem using a peer-to-peer distributed timestamp server to generate computational proof of the chronological order of transactions. The system is secure as long as honest nodes collectively control more CPU power than any cooperating group of attacker nodes.

我们需要的是一个基于密码证明而不是信任的电子支付系统，允许任何愿意的双方直接相互交易，而不需要可信的第三方。计算上不切实际的逆转交易将保护卖方免受欺诈，常规托管机制可以很容易地实施以保护买方。在本论文中，我们提出了一种使用对等分布式时间戳服务器生成交易时间顺序的计算证明的双重支出问题的解决方案，只要诚实节点集体控制比任何合作的攻击者节点组掌握更多的 CPU 算力，此系统就是安全的，

## 2. 交易（Transactions）

---

We define an electronic coin as a chain of digital signatures. Each owner transfers the coin to the next by digitally signing a hash of the previous transaction and the public key of the next owner and adding these to the end of the coin. A payee can verify the signatures to verify the chain of ownership.

![](https://github.com/worldofrorrim/worldofrorrim.github.io/blob/master/static/images/transaction-en.jpg?raw=true)

我们将电子硬币定义为数字签名链。每个所有者通过对上一次交易的哈希和下一个所有者的公钥进行数字签名，并将这些添加到硬币的末尾, 将硬币转移到下一个所有者手中。收款人可以验证签名以验证所有权链。

![](https://github.com/worldofrorrim/worldofrorrim.github.io/blob/master/static/images/transactions.svg?raw=true)

The problem of course is the payee can't verify that one of the owners did not double-spend the coin. A common solution is to introduce a trusted central authority, or mint, that checks every transaction for double spending. After each transaction, the coin must be returned to the mint to issue a new coin, and only coins issued directly from the mint are trusted not to be double-spent. The problem with this solution is that the fate of the entire money system depends on the company running the mint, with every transaction having to go through them, just like a bank.

问题当然是收款人无法核实其中一个所有者没有重复消费过这个电子货币。常见的解决方案是引入一个可信的中心化权威方，或称“铸币厂”，检查每笔交易的双重支出。每一次交易发生之后，硬币必须返回到铸币厂才能发行新的硬币, 只有从铸币厂直接发行的硬币才能被信任不会重复支出。这种解决方案的问题在于，整个货币体系的命运取决于经营铸币厂的公司，每笔交易都必须通过铸币厂,像银行一样。

We need a way for the payee to know that the previous owners did not sign any earlier transactions. For our purposes, the earliest transaction is the one that counts, so we don't care about later attempts to double-spend. The only way to confirm the absence of a transaction is to be aware of all transactions. In the mint based model, the mint was aware of all transactions and decided which arrived first. To accomplish this without a trusted party, transactions must be publicly announced[^1], and we need a system for participants to agree on a single history of the order in which they were received. The payee needs proof that at the time of each transaction, the majority of nodes agreed it was the first received.

我们需要一种方法让收款人知道以前的所有者没有签署任何早期交易。就我们的目的而言，只有最早的交易是算数的，所以，我们并不关心其后的双重支付企图。确认一笔交易不存在的唯一方法是知道所有的交易。在铸币厂模型之中，铸币厂已然知道了所有的交易，并且能够确定这些交易的顺序,哪个先到。为了在没有受信任方的情况下完成此操作，必须公开宣布交易[^1]，并且我们需要一种让参与者就接收订单的单一历史达成共识的系统。收款人需要证明在每笔交易发生之时，大多数节点能够认同它是第一个被接收的。

## 3. 时间戳服务器（Timestamp Servrer）

---
The solution we propose begins with a timestamp server. A timestamp server works by taking a hash of a block of items to be timestamped and widely publishing the hash, such as in a newspaper or Usenet post[^2] [^3] [^4] [^5]. The timestamp proves that the data must have existed at the time, obviously, in order to get into the hash. Each timestamp includes the previous timestamp in its hash, forming a chain, with each additional timestamp reinforcing the ones before it.

![](https://github.com/worldofrorrim/worldofrorrim.github.io/blob/master/static/images/timestamp-server-en.jpg?raw=true)

我们提出的解决方案开始于时间戳服务器。时间戳服务器的工作原理是对要打时间戳的记录块进行散列，并广泛发布(就像在报纸或新闻组帖子一样[^2] [^3] [^4] [^5])散列。显然，时间戳能够证明那数据在那个时间点已然存在，否则那哈希也就无法生成。每个时间戳在其哈希中包含着前一个的时间戳，从而形成了一个链条，每一个新加入的时间戳都会增强之前的时间戳。

![](https://github.com/worldofrorrim/worldofrorrim.github.io/blob/master/static/images/timestamp-server.svg?raw=true)


## 4.工作证明（Proof-of-Work）

---



## 5.网络（Network）

---



## 6.奖励（Incentive）

---



## 7.回收硬盘空间（Reclaiming Disk Space）

---



## 8.简化版支付确认（Simplified Payment Verification）

---



## 9.价值的组合与分割（Combining and Splitting Value）

---



## 10.隐私（Privacy）

---



## 11.计算（Calculations）

---



## 12.结论（Conclusion）

---



---

## 参考文献（References）

---

1.W Dai（戴伟）,a scheme for a group of untraceable digital pseudonyms to pay each other with money and to enforce contracts amongst themselves without outside help（一种能够借助电子假名在群体内部相互支付并迫使个体遵守规则且不需要外界协助的电子现金机制）, “B-money”, http://www.weidai.com/bmoney.txt, 1998↵

2.H. Massias, X.S. Avila, and J.-J. Quisquater, "Design of a secure timestamping service with minimal trust requirements,"（在最小化信任的基础上设计一种时间戳服务器） In 20th Symposium on Information Theory in the Benelux, May 1999.↵

3.S. Haber, W.S. Stornetta, "How to time-stamp a digital document," （怎样为电子文件添加时间戳）In Journal of Cryptology, vol 3, No.2, pages 99-111, 1991.↵

4.D. Bayer, S. Haber, W.S. Stornetta, "Improving the efficiency and reliability of digital time-stamping,"（提升电子时间戳的效率和可靠性） In Sequences II: Methods in Communication, Security and Computer Science, pages 329-334, 1993.↵

5.S. Haber, W.S. Stornetta, "Secure names for bit-strings,"（比特字串的安全命名） In Proceedings of the 4th ACM Conference on Computer and Communications Security, pages 28-35, April 1997. on Computer and Communications Security, pages 28-35, April 1997.↵

6.A. Back, "Hashcash - a denial of service counter-measure,"（哈希现金——拒绝服务式攻击的克制方法）http://www.hashcash.org/papers/hashcash.pdf, 2002. ↵

7.R.C. Merkle, "Protocols for public key cryptosystems," （公钥密码系统的协议）In Proc. 1980 Symposium on Security and Privacy, IEEE Computer Society, pages 122-133, April 1980. S. Haber, W.S. Stornetta, "Secure names for bit-strings,"（比特字串安全命名） In Proceedings of the 4th ACM Conference on Computer and Communications Security, pages 28-35, April 1997. on Computer and Communications Security, pages 28-35, April 1997. H. Massias, X.S. Avila, and J.-J. Quisquater, "Design of a secure timestamping service with minimal trust requirements,"（在最小化信任的条件下设计一种时间戳服务器） In 20th Symposium on Information Theory in the Benelux, May 1999. ↵

8.W. Feller, "An introduction to probability theory and its applications," （概率学理论与应用导论）1957 ↵

