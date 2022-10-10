---
layout: post
title:  "PoS Proof Relay Considerations"
author: devfans
categories: [ crosschain, PoS ]
image: https://sloanreview.mit.edu/wp-content/uploads/2018/11/FL-Ready-Trust-Leadership-2400-300x300.jpg
tags: [featured]
---

When PoS network gets big and more decentralized, the validator size will increase a lot. To maintain a light chain on side chain for proof relays, the cost will increase along. Here to describe some considerations for the moment.



### ECDSA Header Signature signatures

The majority validators sign the PoS block headers, and the signatures will be packed into the header. With these full signature list, side chain contract will be able to verify the header, then verify cross chain messages with valid headers.

Implementation requirements:

- During epoch change, new validators should be stored in the contract storage
- Need to send full header with full signature list to side chain for header verifications



### BLS (BLS12-381) signature aggregation

Replace header ECDSA signatures with BLS aggregated signature. This will great decrease the signature size when validator size becomes quite big.

Implementation requirements:

- Side chain contract need to support BLS(12-381) signature verification, EVM(EIP2537 Shanghai upgrade)

- Every validator needs to be bound with a BLS keypair.

  

### Selected signer group

A subgroup of validators to be selected as signers, then these signers need to sign well structured cross chain messages, emit signatures. Then side chain use these signatures to verify cross chain messages.

Implementation requirements:

- Signer group selection mechnism needs to be we designed, especially when epoch changes.
- Slashing/Rewarding needs to be implemented for performance ensurance.
- Signer group can choose to sign on cross chain messages, header or just state root, when sign only cross chain messages, the header chain wont be maintained on side chains, this may limit some capabilities.



### 400 Validators with Signer Group size 10

|                       | ECDSA Header Signature | BLS Aggregation | ECDSA Signer Group  |
| --------------------- | ---------------------- | --------------- | ------------------- |
| Validator List Size   | 12800 Bytes            | 19200 Bytes     | 12800 (+ 320) Bytes |
| Signature List Size   | 26000 Bytes            | 96 Bytes        | 650 Bytes           |
| Verification Gas Cost | 1200000                |                 | 30000               |



Since BLS signature verification is not well supported on most VMs, we here to attempt to propose the the third solution, using a selected signer group.

### Considerations

- Sign group size, it can not be too small to be less secure, or too big to cost too much. Here we say 8 - 20 for the moment?.

- Group selection. Group selection should be consistent with epoch change. If validators of epoch N, selected a signer group, epoch N + 1 should decide on a new signer group for security consistence. The signer group can be chosen from the top ranking of validator stake, this way implies chosen signer is forced to sign accordingly. Or go through the validator proposal process. A proposed signer group may last for a defined long period?

- Rewarding is needed to cover vote costs and better performance. Estimated rewarding could be > 100% costs and < 500% costs?

- Slashing should be implemented to ensure a stable performance. Here we propose the slashing amount to be same as rewarding.

- Signer group can vote on below targets:

  - Block header

    As extra header signatures, this way the header chain could be maintained on the side chains for general oracle purpose.

  - State root.

    Light header. Do we set an interval for votes? Like vote every 10 blocks?

  - Cross chain messages

    We can pack the pending cross chain message into plain list or a merkle tree same as Poly V1. Then signer group vote on the root. This way, the call data will be more light to be submitted on the side chain for verifications, while this will limit the general oracle purpose.











