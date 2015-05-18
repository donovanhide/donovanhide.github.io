---
layout: post
title: "Ripple Ledger Storage Inefficiencies"
date: 2014-07-25
comments: true
categories: [Ripple]
---

_I was [asked](https://ripple.com/forum/viewtopic.php?f=1&amp;t=7426&amp;start=20#p53514) on the Ripple forum about my experiences working with the Ripple ledger during development of the [RippleBot ledger browser](http://ripplebot.com), and how rippled could be made less storage-hungry. Here’s a repost of that answer._

I decided to dedicate a lot of time to the project because I really love the concept of a network graph superimposed on top of a cryptographically secured and open ledger of account balances. I love the idea of decentralisation also, but am sad that that reality is still far away. So time has been spent, funded by the XRP I received from the ComputingForGood giveaway, to make it happen and open up the ledger for inspection.

During this time I’ve read through a lot of the code in rippled and examined and processed the whole nodestore to try and understand it better. Through this analysis I’ve come up with ideas for better storing the three main entities of the ledger, which have different lifetimes, characteristics and storage requirements.

**First, let's review:**

*   AccountRoot stores the balance of XRP for an account and various configurable settings for that account. Only the most recent one is needed for validation.
*   RippleState stores the balance of IOU’s and a few settings for that balance and how it may be used. There may be zero or many per account. A Ripplestate has an interval-like lifetime in the sense that it can be Created-&gt;Modified-&gt;Deleted-&gt;Recreated… Only the most recent one per currency/issuer combination per account is needed for validation.
*   Offer is the basis of trading and sets a price at a point in time. It can follow the path Created-&gt;Deleted, Created-&gt;Modified…-&gt;Deleted. There can be zero to many per account. Only the most recent ones per account are required for validation.

So we have a graph composed of the above entities and they are created, modified and deleted by transactions. Then we have a ledger which is a point in time when the most recent transactions are ordered and each one is applied to the graph with the possibility of success, failure or postponement. The change in the three entity types is reflected in the account state tree which results in a single hash being generated which concisely summarises and proves the effect of these changes. The same is done for the non-postponed transactions which results in another tree and another hash.

**The problems**

So, why is validation such a disk-intensive process? Because the whole tree for the account state consists of hundreds of thousands of inner nodes which change when any of their descendants change. The amount of change depends on the depth of the affected leaf nodes. This tree is essential for the hashing process to be cryptographically secure, but for reasons discussed below, it doesn’t need to be persisted to disk, but it is in the rippled implementation.

Omitted from the above list of entities are Directories and LedgerHashes “skiplists”. These are secondary indexes for navigating the persisted account state trees for any ledger. Unfortunately they are embedded in the account state tree as well. This paradox is a fundamental weakness of the Ripple protocol and is perhaps the root cause of the disk usage issues and the inability to prune the ledger into a sensible size. If the three main entities were indexed externally on their above described lifetime characteristics, then the whole inner node tree (95% of the ledger disk usage) would not need to be persisted and could only be used in its in-memory representation to create the Merkle tree hashes. Maybe there is some cryptographic requirement for validating these indexes also, but that is a separate and interesting problem that could be solved.

Another interesting design decision in the Ripple protocol is that the outcomes of each transaction are stored in metadata attached to the transaction itself. At the end of the ledger processing the last of the outcomes per account is the one committed to the account state tree. What you have is two levels of granularity, per point in time during processing of each transaction in a ledger and the ledger close time itself. This has interesting consequences for both compression and replay-ability. It’s actually more useful and less duplicative to store the Ledger/TransactionIndex level of data than it is to store that and also the end of ledger data. With external secondary indexing at the more granular level you get more insight and no repetition of account state. However this scenario either makes it more complicated or impossible to have the full state of all accounts without having all the data from the Genesis ledger onwards.

**Coping stategies**

If there was a prospect of a Ripple V2, I’d strongly recommend some of these ideas to be considered. But that is unlikely and Directories and LedgerHashes are probably here to stay, embedded in the ledger and eating disk space along with the inner nodes.

So, what is the best way to store historical ledger data? In a compressed and heavily normalised database. My experience says that MariaDB using the TokuDB storage engine provides the best possible combination of query speed, disk use and indexability. With use of in-memory lookup tables it is possible to reduce all the longer and repeatedly used field types such as Account, Currency,Issuer and PublicKey to a 4 byte unsigned integer (maybe one day, if Ripple is successful it could be extended to 8 bytes!). If interesting slicing and dicing is required then judicious use of indexing can make for some very interesting analysis possibilities. The main source of entropy are the TxnSignature fields and there’s not too much that can be done to compress them significantly.

With only the ledger headers and transactions with metadata stored and not the account states, and a suitable library for re-encoding TransactionWithMetadata Types and reconstructing AccountState nodes from the metadata, it should even be theoretically possible to cursor through the ledgers and revalidate the whole sequence of ledgers using an in-memory version of the two trees. I am currently testing this theory. :-)