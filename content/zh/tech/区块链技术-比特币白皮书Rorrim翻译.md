+++
title = "区块链-比特币白皮书(Rorrim翻译)"
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

> Abstract.A purely peer-to-peer version of electronic cash would allow online payments to be sent directly from one party to another without going through a financial institution.Digital signatures provide part of the solution，but the main benefits are lost if a trusted third party is still required to prevent double-spending.We propose a solution to the double-spending problem using a peer-to-peer network.The network timestamps transactions by hashing them into an ongoing chain of hash-based proof-of-work，forming a record that cannot be changed without redoing the proof-of-work.The longest chain not only serves as proof of the sequence of events witnessed,but proof that it came from the largest pool of CPU power. 
>
> 概要： 一个纯粹的电子现金的点对点版本,将允许在线支付从一方到另一方直接发送,而不需要通过任何的金融机构。数字签名提供了解决方案的一部分，但是，如果仍然需要一个受信任的第三方来防止双重支出的话，那么这个系统的主要优势就丧失了。我们提出了一个使用点对点网络时的双重支出问题的解决方案。网络通过将交易散列到一个正在进行的基于哈希的工作量证明的链条中来对交易进行时间戳记，从而形成如果不重做这个工作证明就无法改变的记录。最长的链不仅可以作为已被见证事件的顺序的证明，而且可以作为它是来自最大的CPU算力池的证明。
>
>
>
>