---
layout: post
title:  "Hierarchical Deterministic Wallet Concepts"
author: devfans
categories: [ Crypto, Wallet ]
image: /static.livefeed.cn/static/blog/hd-wallet.webp
tags: [sticky]
---

Crypto wallet account works as the main and only(mostly) access identity for users of blockchains, that means each user has to maintain a crypto wallet, normally holding a private key in it, or multiple sometimes. Private keys are normally random bytes with a strict size. So how to let users keep them easily and safely? And for Bitcoin, it's preferred to generate an account and throw it away after it's spent, next time to generate a new one. For centralized exchanges, the enterprise needs to keep a unique receiver address for each user & token. To cope with these troubles, some proposals came out with good ideas.

Early days of Bitcoin, some wallet will generate a batch of standalone private keys and keep them in the wallet, it's the simplest way, but not proper to manage when scaling up. Called as random or non-deterministic wallet.

Some wallet adopted to generate a seed, and compute the seed with account index to generate the private key, sounds good, called as sequential deterministic wallet.

Then, more ideas came out to form the HD(Hierarhical Deterministic) wallet concepts, which becomes kind of a wallet standard for Bitcoin and other chains like Ethereum. 

![Child Key Derivation](https://static.livefeed.cn/static/blog/derivation.png)



Despite of the time of proposals, here's to describe the main idea from mnemonic words to leaf accounts.


In [BIP39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) the idea behind generating a wallet:
 
- Generate a random bytes: Entropy

- With strict ENT(Entropy size) and CS(Check Sum), result in the length of MS(Mnemonic Sentence)

- With an ideal word list mapping bytes to words, then we have the mnemonic sentence.

- So with such a sentence and optional passphrase, do HMAC-SHA512 computations will generate a seed in 512 bits.


```
CS = ENT / 32
MS = (ENT + CS) / 11

|  ENT  | CS | ENT+CS |  MS  |
+-------+----+--------+------+
|  128  |  4 |   132  |  12  |
|  160  |  5 |   165  |  15  |
|  192  |  6 |   198  |  18  |
|  224  |  7 |   231  |  21  |
|  256  |  8 |   264  |  24  |
```

In [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) the idea for key derivation from the seed.

- Do HMAC-SHA512 with "Bitcoin seed" as key, seed bytes as data, generate a 32 bytes sequence, first half as master private key, second half as master chain code.

- With child account index `i`, define if `i >= 2^31`, use hardened derivation, otherwise, use normal derivation

- k(p), c(p) -> k(i), c(i): For hardened derivation, compuate HMAC-SHA512 with master private key, master chain code and child index, we get 512 bits, first half do addition with master private key to get the child private key, second half as child chain code.

- k(p), c(p) -> k(i), c(i): For normal child, compute HMAC-SHA512 with master public key instead.

- K(p), c(p) -> K(i), c(i): For normal child public keys, as last one but first half added to master public key to get the child public key, and second half as child chain code.

- k(p), c(p) -> K(i), c(i): in hardened way, a bit different, with master private key involved to generate the child public key.

- Use same steps to generate further children.

From the derviation logic, we call know some properties like:

- Child private key was an addition to parent private key, so if using non-hardened derviation, and parent public key gets exposed along with parent chain code (that means the extended parent public key gets leaked), it wont be hard to find the parent private key.

- Also, when extended parent public key gets exposed, it'll be easy to calculate the child public keys, and the ones of further grand children, making behavior tracking possible.


In [BIP44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki), the schema is more comprehensive.

```m / purpose' / coin_type' / account' / change / address_index
```

Here:

- Apostrophe means hardened derivation.

- `purpose` used to indicate the schema purpose, proposed in BIP43.

- `change` normally only useful to indicate bitcoin change addresses(internal) or a payout address


To sum things up, HD wallet concepts formed a better way for accounts management, which powers user to derive tons of public keys without the need to maintain private keys for each. With a hierarchical schema, the wallet becomes versatile and with hardened derviation to prohibit some possible behavior tracking. 



