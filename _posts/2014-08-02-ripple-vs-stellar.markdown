---
layout: post
title: "Stellard compared to Rippled"
date: 2014-08-02
comments: true
categories: [Ripple,Stellar]
---

So there’s a new fork in town going by the name of [Stellar](http://stellar.org)!

**Brief history**

Jed McCaleb had an idea for a new cryptocurrency which did not depend on mining and hired a small team of developers ([David Schwartz](https://github.com/joelkatz), [Stefan Thomas](https://github.com/justmoon) and [Arthur Britto](https://github.com/ahbritto)). This idea grew into one which borrowed from [Ryan Fugger](http://ryanfugger.com/)'s [original concept](https://classic.ripplepay.com/) of community credit and was designed to provide a scalable solution for global payments with liquidity provided by anyone who wanted to make an offer or supply credit to satisfy that payment. An elegant concept was the basis for the formation of OpenCoin, later to become Ripple Labs. Jed hired Chris Larsen, and a subsequent, well-documented fallout occurs over the allocation of 20% of the XRP to three individuals and the fair distribution of the remainder. Jed leaves Ripple Labs and announces a “Secret Bitcoin Project”, which it turns out is a fork of the rippled codebase with some minor modifications and a new user interface. The release is partnered with a clearly expressed set of rules governing the distribution of the XRP equivalent known as STR.

**Modifications**

So what are these code modifications and do they make a big difference to how likely Stellar is to succeed? Let’s have a look at the significant commits which have occurred since the fork attempt began on April 24th 2014\. We’ll disregard all of the obvious “rename ripple=>stellar” alterations.

*   [Account ids begin with a g](https://github.com/stellar/stellard/commit/bf9eeaf1edfe282fe1c262478d96b01e5a65f9d6) is a fairly straightforward change to the base58 alphabet for encoding account ids and [other Stellar types](https://github.com/stellar/stellard/commit/0e424457fc591f437ebe9b1b14141c402b51dcf4). The main result is that all account ids begin the letter “g”, rather than an “r”. Why “g”? Who knows.

*   [Add InflationDest field](https://github.com/stellar/stellard/commit/1b669a8a6def4a3e29f3401be723c069b2c3c4fe) is perhaps the most revolutionary change. The idea is that each account gets to nominate another account which, each week, receives a share of 0.019% of all the STR in existence, perhaps as a result of continued good stewardship of the network and supporting codebase. The field is [optional](https://github.com/stellar/stellard/commit/0d6dc2e70039ab40ea01b99a9c8440e849d1160b). The formula is [here](https://github.com/stellar/stellard/blob/b17a8174d9f3dba46a602b3bc7f87e09fc03effe/src/ripple_app/transactors/InflationTransactor.cpp) and then revised [here](https://github.com/stellar/stellard/commit/89506ac40b3477367de73519a4353323e24ed14a) and [here](https://github.com/stellar/stellard/commit/e63cf791736975ae2b35ae2a639c7edec7788c89) and [here](https://github.com/stellar/stellard/commit/a8bb6eb6b87ba03c64bd18b2441e3475abdd33ad). Two new fields FeePool and InflationSeq are [added](https://github.com/stellar/stellard/commit/52fda30ff80e24b25e05024bf17a8066cb615de7) to the LedgerHeader.

*   [Accounts](https://github.com/stellar/stellard/commit/0419b148839094d90ab3df2cbc8d1afb4dbf9989) [can be deleted](https://github.com/stellar/stellard/commit/399fd830d76c04b85ea94e8e0a57ac34c3452ccc) means that a user can consolidate his/her STR back into a single account from multiple accounts. Trustlines must first be removed.

*   Clean up old unimplemented data structures, such as [Nickname](https://github.com/stellar/stellard/commit/b17a8174d9f3dba46a602b3bc7f87e09fc03effe) and [GeneratorMap](https://github.com/stellar/stellard/commit/c6dc9cebf3f3bb1395e679974e233a659f0a4d5f).

*   Bootstrapping from a centralised peer provider is [removed](https://github.com/stellar/stellard/commit/984a28c8112b74438ea954f45288df55a79d18a5).

*   A [switch](https://github.com/stellar/stellard/commit/1ecc992b87c3732dd45e69ae3b02b8072c536015) to ed25519 from P256 for creating and verifying signatures is implemented.

*   [Expose wallet_public](https://github.com/stellar/stellard/commit/10abf4f3cc6feb31b1503c9c101658d1bf0b1520) to anyone and [rename to create_keys](https://github.com/stellar/stellard/commit/054c0a848af89f1e76c64f0aac424eadf7bca007). This is a security risk as someone could fake a response on a server and be in possession of your secret.

*   [Rename](https://github.com/stellar/stellard/commit/99708e74b5da0129e19f015551fecee769926e7d) some API calls and [change the default min_ledger](https://github.com/stellar/stellard/commit/ece899c21613a3a097fb81431aabeec05923fea0) for [some calls](https://github.com/stellar/stellard/commit/b8f3b24304861bc2b56d105233a5efe2d0ed77e4).

*   Remove [EmailHash, WalletLocator, WalletSize, MessageKey and Domain](https://github.com/stellar/stellard/commit/050bdcc89393d8563d8110561c4f485e1ce4b0fc) fields from AccountRoot serialization format and the flag [PasswordSpent](https://github.com/stellar/stellard/commit/054c0a848af89f1e76c64f0aac424eadf7bca007).

*   A [painful merge](https://github.com/stellar/stellard/commit/740bd9cb2f278ee17c55fbc0e40ff60d80297d2d) of the main rippled codebase.

*   Some surprising [lack of familiarity](https://github.com/stellar/stellard/commit/36f92c02289f7792e2b92b56a526b7b96f2b545e) with a key data structure in the codebase.

So, does the above represent any serious deviation or innovation on the rippled implementation? The inflation is an interesting idea, but it reminds me of the old bankers’ adage:

> "There are two types of people in the world. Those that understand compound interest and those that pay it".

The switch to ed25519 may one day permit performance gains, but not before any nodestore speed issues have been solved. The ability to delete an Account is useful. Everything else is mostly cosmetic and housekeeping. 

What is obvious from the team of three developers working on the C++ codebase is that there is not a deep understanding of what is going on in the internals, at least not yet. There is no published roadmap of future changes. Worst of all is that recent security fixes on the main rippled codebase have not been integrated into the stellard codebase and a new security flaw has been wilfully introduced.

The switch to an [open and thoroughly explained plan](https://www.stellar.org/about/mandate/) for STR distribution is a welcome one, but a web page with words on it is just that. Time will tell.