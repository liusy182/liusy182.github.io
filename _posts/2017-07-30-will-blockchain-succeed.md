---
layout: post
title:  Will blockchain succeed?
date:   2017-07-30 10:00:20 +0800
categories:
tags:
- blockchain
- bitcoin
- ethereum
- hyperledger
---

Bitcoin and the whole idea of cryptocurrency is definitely a hot topic now, or 
at least it is "hot" in the market. The screenshot taken from [coinbase] shows
that the price of bitcoin has gone up several folds over the last year! It is 
probably still too early to judge whether it is a bubble.

<img src='{{ site.url }}/assets/bitcoin/coinbase.png' />

Blockchain, the underlying technology of bitcoion, is basically a distributed 
database connected in a p2p fashion. In recent years, people have started to 
realize its potential application on different problem domains. Big companies like IBM (with its [hyperledger project]) is pushing heavily on blockchain's
application in private sectors. Tons of blockchain startups are also catching up
with waves of innovations.

To join the wave, I have given [ethereum] a try. The first impression was disappointing. 
As one of the "matured" blockchain implementation at the moment, ethereum was hardly useful. 
The Proof-of-Work (PoW) consensus simply took too long to get through a transaction, and 
it was absolutely resource intensive. My powerful PC hang up the moment I started GPU mining. 
I cannot imagine how many tons of greenhouse gas are produced by the mining 
industry with thousands or even millions of GPUs running 24/7. Although there are plans to replace the heavy consensus algorithm with a lightweight 
Proof-of-Stake (PoS) consensus, the tradeoff between security and speed needs to be 
carefully designed and evaluated. 

Blockchain requires each node (peer) 
connected to the network to carry a copy of the database with ALL transaction 
histories. This means more than 20GB of disk space on today's ethereum network. The worst thing
is, the size will go much bigger as time goes. Yes, storage is cheap, but how can ethereum take
cheap storage as granted?

[coinbase]: https://www.coinbase.com/charts?locale=en
[hyperledger project]: https://www.hyperledger.org/
[hyperledger fabric]: https://www.hyperledger.org/projects/fabric
[Ethereum project]: https://www.ethereum.org/
[ethereum]: https://www.ethereum.org/
[go-ethereum]: https://github.com/ethereum/go-ethereum