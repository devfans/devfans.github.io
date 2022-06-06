---
layout: post
title:  "Secret Channel with Hackers"
author: devfans
categories: [ Crypto ]
image: /static.livefeed.cn/static/blog/secret-channel.jpeg
tags: [ hacker ]
---

Hackers proved various vunerabilities among DEFI society, after the hacks, commonly the only clue is the public address of the hacker, leaving the possible communication channel for the project owners to establish with the hacker, is send a simple transaction on chain with message attached as extra bytes. The message could be transparent or encrypted when there're some secret negotiations are underneath. 

So here's to describe the encryption logic.

If Alice wants to send a secret message to Bob, Alice needs to encrypt the message with Bob's public key, then only Bob can decrypt the message with the private key, the same way around.


Sample code piece:

```javascript
const c = require('eth-crypto');

const encrypt = (pk, msg) => c.encryptWithPublicKey(Uint8Array.from(Buffer.from(pk, 'hex')), msg).then(c.cipher.stringify).then(console.log);

const decrypt = (pri, msg) => {
  const em = c.cipher.parse(msg)
  return c.decryptWithPrivateKey(pri, em).then(console.log)
}

const decryptEM = (pri, em) => {
  c.decryptWithPrivateKey(pri, em).then(console.log)
}

```

