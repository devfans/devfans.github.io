---
layout: post
title:  "Bitcoin Mining with Stratum Miners"
author: devfans
categories: [ Bitcoin, mining, stratum, crypto ]
image: /images.pexels.com/photos/730567/pexels-photo-730567.jpeg?auto=compress&cs=tinysrgb&dpr=2&h=750&w=1260
tags: [sticky]
---

As many people ever heard about the blockchain stuff, probably bitcoin the most, and yes there's the community for the blockchain miners. So, maybe many guys are confused with it, what are they doing exactly? Right, this is the topic I am gonna write about in this post, but mostly about the tech facts.

## What is a blockchain miner?
What a typical miner would do is go to the miner manufactory's portal or contact their sales to buy the iron boxes, which is pretty loud and noisy when it's running, then host them in a mining farm or a rackplace, so the farm keeper is responsible for the normal operations including setup and cabling for electricity and network, also cooling down is necessary to keep the mining machines running smoothly. Normally, the miners are all connected to mining pools to achieve a much more stable income rate, and then the pool runner will be responsible for generating the jobs for the miners and check out their outcome. When the miners give a good outcome which could meet the requirement of constructing the next block on the chain, the pool will get the block's reward which is automatically done by a transaction coded in the block, then the pool will split rewards to the miners and keep a  bit fees for it. In addition, the farm owner will also charge the miner the fees for operations, rack spaces, and the electricity costs. After the miners receive the coins from the pool, they pay every cost they have to and with the left as the profit. If you are not interested in the tech details, you are done here.

## Tech details about PoW mining.
Here to explain the steps in one typical bitcoin mining loop.

+ Pool calls the bitcoin node with the method `getblocktemplate`, which will give the template of the next block on the chain as the response.

```
Sample block template:
version: 536870912,
previousblockhash: 00000000000000000007d0a09fd100e79d95e5af3310bd4398d5e75cba5967b2,
transactions: [...],
bits: 1729fb45,
height: 577539,
coinbasevalue: 1335905154

```


+ Pool calculate the hashes of the transactions in the template to get the merkle branch hash

+ Pool constructs the coinbase transaction with payout address as the `vout`, and a `vin` with some random bytes including the block height, timestamp, some custom strings, and a placeholder for the miners to fill in.

+ Pool constructs the stratum jobs with the informations above and assigns them to the connected stratum miners and tell the miner the minimum required target(leading 0s of the block hash) of the share to submit back.

+ Miners receive the job and iterate on the hashing with the merkle tree which is constructed by the coinbase transaction and block transactions merkle branch. If the output hash meets the desired target then submit the work back to pool stratum server as a share.

+ Pool collects the shares and checks for each. If one share meets the target of the next valid block, then construct the block and submmit to chain to broadcast the block. The target of the next block is calculated by the `bits` field in the block template and can be converted to the desired hash target.

+ Pool successfully broadcasted the block than any other pool via p2p communication with other nodes. The pool also received the coins sent to the defined wallet address in coinbase transaction. Then split the reward and pay back to the miners according how many shares did they ever commited to the pool. The argorithm could be `PPS`, `PPLNS`, other defined logics to make miners think it's profitable enough if join this pool. 

+ Loop ends for this block, and the next loop will start with this valid block as the previous one to make the template of the next block.

So this is the idea behind the mining on blockchain. Didn't cover the detailed fields in the process for this short post, but hope it helps you to understand what it is in the background.

In addition, I am not a native English speaker, just trying to get used with writing posts in it.
