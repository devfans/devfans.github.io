---
layout: post
title:  "Math Formulas in Bitcoin Mining"
author: devfans
categories: [ Bitcoin, mining, stratum, crypto, formula, hashrate ]
image: https://static.livefeed.cn/static/blog/bitcoin-blockchain-close-up-1097946.jpg
tags: [featured]
---

This post is to list common formulas in bitcoin mining.

## Mining Difficulty

A sha256 hash has 256 bits value as the target. The lower the value is, the more difficult the job is to be done to generate such a hash that can meet the target threshold.

To calculate a difficulty of a hash target:

``` difficulty = difficulty_1_target / current_target ```

Each bitcoin block has the required difficulty coded as bits in block header, which is a 32 bits value in compact `nBits` format to represent the block target. To convert the `nBits` to `target`:

``` sign = nbits & 0x00800000 ```

``` mantissa = (nbits & 0x7fffff) ```

``` exponent = nbits >> 24 ```

``` target = (-1^sign) * mantissa * 256^(exponent-3) ```

For bitcoin, the commonly used `difficulty_1_target` is `0x1d00ffff` in nbits. So the below target has the difficulty as 1.

``` 0x00000000FFFF0000000000000000000000000000000000000000000000000000 ```

## Mining Hashrate

Hashrate or hashpower is used to measure a mining machine's hashing performance. Hashrate `1KH/s` means `1000` hashes per second, and there's more units commonly used as below:

``` 1 EH/s = 1000 PH/s = 1000, 000 TH/s = 1000, 000, 000 GH/s = 1000, 000, 000, 000 MH/s  = 1000, 000, 000, 000, 000 KH/s = 1000, 000, 000, 000, 000, 000 H/s ```

In a typical pooled mining, the pool would collect the shares submitted by the mining machines to calculate the hashrate of a mining worker, which would be like:

```  hashrate (TH/s) = (share_diff_sum_in_1h / 3600) * pow(2, 32) / pow(10, 12) ```


## Mining luck

Mining luck is used to measure the round luck, which means you are lucky if you find a block with less work done compared to the one calculated.

``` luck = 1/(share_diff_sum_round / netdiff) ```

So, for example the bitcoin chain's network difficulty is `9853622692186` (about 9.85T) when I am writing this post.
This would mean to find a block within one day with a normal luck 100%, it would need this much hashpower:

``` hashrate(PH/s) = (9.85T / (60*60*24)) * pow(2, 32) / pow(10, 15) = 489.8 PH/s ```

Why? It's a math logic as described below:

+ Convert diff to target

``` target = (0xffff * pow(2, 208)) / diff ```

+ Hashes required to meet the target

``` hashes = pow(2, 256) / target = diff * pow(2, 48) / 0xffff ```

To simplify a bit, use `pow(2**16)` to replace `0xffff`

``` hashes = diff * pow(2, 32) ```

+ To do such hashes within 24 hours would require hashrate as

``` hashrate(H/s) = hashes/ (60*60*24) = diff * pow(2, 32) / (60*60*24) ```


## Rejection Rate

For pooled mining, a share submitted by mining worker might be rejected for several reasons, like low difficulty, stale job, network latency, etc. 

``` rejectionrate = shares_rejected / shares_total ```

## PPS Reward

PPS is a commonly used mining reward calculation method to distribute mining rewards to miners. With this method, the mining pool's profit would depends on the pool's luck, while the miner's rewards would be stable.

``` reward = block_reward * hashrate / network_difficulty ```

## PPLNS Reward

PPLNS is another commonly used mining reward calculation method which is used to avoid pool hoppers. The N is normally bigger or equal than 5. With this method, miner's profit would depend on the pool's luck. The more shares the worker contribute to pool's last N round, the more reward will be paid out to this user from this round's block reward.

``` reward = block_reward * miner_shares_in_last_N_round / shares_total_in_last_N_round ```


In addition, I am not a native English speaker, just trying to get used with writing posts in it.

