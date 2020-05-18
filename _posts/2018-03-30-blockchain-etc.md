---
layout: post
title:  Notes from Blockchain and Cryptocurrency Technologies
date:   2018-03-30 08:00:00 +0800
categories:
tags:
---

[Blockchain and Cryptocurrency Technologies] is an excellent course that demystified
internal working of the blockchain. Here are some fragments of notes I have taken so far.

---
<p></p>

The fundamentals  of blockchain is similar to digital signature. Only the owner
can sign using private key, but everyone can verify using public key. In practice,
hash of the public keys are used as "addresses" in the block chain.

Storing bitcoin is equivalent to storing your private keys.

---
<p></p>

**Proof-of-XXX** is basically an algorithm of selecting a random node in proportion
to a resource that no one can monopolize.

- Proof-of-work: resource is computing power

- Proof-of-stake: resource is ownership

Incentives of participating in POW:

1. block reward. For bitcoin, block creator collects reward only if the block
created end up in the longest chain.

2. transaction fee. Creator of the transaction can choose to pay a transaction fee.
Whoever makes the transaction into the block gets the transaction fee.

---
<p></p>

#### Some numbers in bitcoin:

Average time of finding the next block is 10 mins.

```
Prob(person A wins next block) == fraction of global hash power he controls.
```

Hence for person A to find the next block,

```
mean time of find next block == 10 minutes / fraction of global hash power he controls
```

And


```
mining reward (block reward + transaction fees) - electricity cost = profit
```

Hard coded values in bitcoin:

- 10 min ave. block creation time

- 1M bytes / block (10 mins)

  e.g. 250 bytes per transaction ==> 7 transactions / sec (Very low throughput!!)

- 20k signature operations / block

- 100M satoshis / block

- 21M total bitcoin

- 50, 25, 12.5 ... mining reward.

- only 1 hash algorithm (ECDSA/P256). This may break in future especially when
computing power increases.

---
<p></p>

#### How a bitcoin chain look like:

```
   prev Hash       prev Hash       prev Hash
               <--             <--
... tx hash         tx hash         tx hash
                      |
                (merkle tree)
                     H()
                    /   \
                  H()   H()
                 / \     / \
               H() H()  H() H()
               |    |    |   |
               tx  tx   tx  tx
```

---
<p></p>

#### Soft fork

Add features that "limit" the set of valid transactions

e.g.

- pay-to-script hash - pay to a hash of a script that contains another tx

- extra per block metadata

#### Hard fork

e.g.

- new op codes

- mining rate, size limit change


---
<p></p>


**Wallet software** Help user to keep track of coins. A very nice trick is to user a separate
key(private key)/address(public key) for each coin. This provides some anonymity
to end user.

**Multi-sig** example: 3 out of the 4 keys must sign to release a coin. Useful when working in a
company etc. Internally it uses quadratic function for secret sharing. Support K-out-of-N splitting.







[Blockchain and Cryptocurrency Technologies]: https://www.coursera.org/learn/cryptocurrency/