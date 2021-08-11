---
layout: post
title:  "Aug 10 PolyNetwork Hack Explained | $600M"
author: devfans
categories: [ hack, PolyNetwork ]
image: /static.livefeed.cn/static/blog/poly-hack.jpg
tags: [sticky]
---

It's a big shock that the PolyNetwork was attacked by a hacker with a loss about 600M dollars. I endorsed this project, and trust the fundamental cross chain solutions which was built with bunch of solidity smart contracts. I was spending some time to check the detailed procedure and trying to find any possible vulnerabilites, but no I didnt figure that out. That's why I trust the project so much. So it's been a bit ashamed it's hacked and this hack left a siginificant attack in DEFI society. Here's to explain the path the hack was using to steal the assets.

The vulnerable function is:
```
    function verifyHeaderAndExecuteTx(bytes memory proof, bytes memory rawHeader, bytes memory headerProof, bytes memory curRawHeader,bytes memory headerSig) whenNotPaused public returns (bool)
```
`https://github.com/polynetwork/eth-contracts/blob/master/contracts/core/cross_chain_manager/logic/EthCrossChainManager.sol`


The initial design of the contract metioned here is to serve as message bus to decouple the biz details from the cross chain infrastructure, which leads to the case the target contract address and method is not limited in the function body. However, the only desired usage is to call the proxy contract to unlock the wrapped assets. With this door, the hacker called the cross chain data contract:

```
    function putCurEpochConPubKeyBytes(bytes memory curEpochPkBytes) public whenNotPaused onlyOwner returns (bool) 

```
`https://github.com/polynetwork/eth-contracts/blob/master/contracts/core/cross_chain_manager/data/EthCrossChainData.sol#L45`

The poly project is using multi-sig validators to authorize valid transactions to be fullfilled, however this public method is open to the cross chain manager contract(the first one), so the hacker with this way changed the book keepers in the target chain storage, which leads to the proxy contract fully open to the hacker to unlock any assets. 



The lesson we can learn is about the free functions, DO LIMIT it to the known possible usages, do not leave it fully free without control.


This article is fully persional assumption, please check the official anouncement when it's out.

And it's rumor that if someone says a big backdoor was kept by the project.



- Go on, PolyNetwork!.



P.S.

There's indeed another protection for the UNLIMITED method call, but it has some defect which leads to a not hard hash collision attack.






