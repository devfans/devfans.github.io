---
layout: post
title:  "Understanding signatures in Ethereum"
author: devfans
categories: [ Crypto ]
image: //static.livefeed.cn/static/blog/cryptographic.jpeg
tags: [ ecdsa, bls, schnorr, signature, ethereum ]
---


Understanding Signatures in Ethereum

After the Ethereum2.0 merge, the Ethereum chain switched it's consensus from PoW to PoS. One key part the Ethereum2.0 is adopting BLS signature for validator signing, with this, it can decrease a lot hardware pressure from  networking traffic, computation for verifications and the storage for signatures. All this results from some nice features of BLS signatures. ECDSA signatures are still be used for transactions, with verification supported in EVM. Currently BLS signature verification support is proposed in EIP-2537, however it's still not included in an upgrade yet. See dicussion here: https://github.com/ethereum/pm/issues/343

Here's to describe how these signatures were designed. 

 ## ECDSA Signatures

Elliptic Curve Digital Signature Algorithm or ECDSA is a cryptographic algorithm used by Bitcoin, Ethereum and some other Blockhains. It is dependent on the curve order and hash function used. Typically `Secp256k1` and `SHA256` respectively.  

![ecdsa-signature-design](//static.livefeed.cn/static/blog/ecdsa.png)

`secp256k1` define the curve *E*: *y2 = x3+ax+b* over F*p* as:

- *a* = 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000
- *b* = 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000007

The base point G in compressed form is:

- *G* = 02 79BE667E F9DCBBAC 55A06295 CE870B07 029BFCDB 2DCE28D9 59F2815B 16F81798

Hence the curve equation is *y2 = x3 + 7*.

### Signature Design

Define varialbe as:

- G, the curve base point
- n, the integer order of G so n x G equals the identity element
- d, the private key
- Q, the public key, so d x G = Q
- k, the random integer from [1, n-1]
- R, the point that k x G = R
- z, the message hash, so z = Hash(m)
- r, x of point R(x,y)
- s, the scalar that s = (z + r x d)/k  mod n

Signature equation:
z x G + r x Q = s x R
correctness denoted as:
R = (z x G + r x Q)/s = (z x G + r x d x G) x k / (z + r x d)
   = (z + r x d) x k / (z + r x d) x G
   = k x G
   = R
	

### Private key recovery

In the design, k should be randomly selected, and a collision will introduce a vulnerability to be used to recovery the private key d.

The recovery denoted as:
s - s' = (z + r x d - z' - r x d) / k
so
k = (z - z') / ( s- s')
d = (s x k - z) / r


### Public key recovery

Q = (-z / r) x G + (s / r) x R
  = (s x k - z) / r x G
  = (z + r x d -z) / r x G
  = d x G


## Schnorr Signature

Schnorr signature changes variable z, r and s as:
- z = H(Q, R, m),  m is the mesage to sign
- R = k x G
- s = k + z x d

![schnorr-signature-design](//static.livefeed.cn/static/blog/schnorr.png)


Signature design
s x G = R + z x Q
correctness denoted as:
Q = (s x G - R) / z
  = (k + z x d - k) / z * G
  = d * G
  = Q

Nice features the schnorr signature inluces:

#### Batch validation

Since (s1 + s2 ... + sn) x G = (R1 + R2 ... + Rn) + (z1 x Q1 + z2 x Q2 ... + zn x Qn), batch validations could decrease the computation pressure of signature varifications.

#### Signature aggregation

Signature aggregation is commonly used to decrease the signature list size. 
With Schnorr signature, we have:

R = R1 + R2
then
(s1 + s2) x G = R + z x Q1 + z x Q2

The drawback is the signers need to communicate with each other to share their R values.

## BLS Signature

BLS signaure was introduced in 《Short signatures from the Weil pairing》 by Boneh, Lynn and Shacham. It's completely deterministic signature algorithm without random number involved.

However, the signature requires to map the message to point on the curve, so we have:
- H(m) = Z
- S = d x Z
- Q = d x G


![bls-signature-design](//static.livefeed.cn/static/blog/bls.png)

### Signature design

With pairing property, we have:
e(Q, Z) = e(d x G, Z) = e(G, d x Z) = e(G, S)

Nice features of BLS signature include:

#### Batch validation with signature aggregation

Define S = S1 + S2 ... + Sn, hence:
e(G, S) = e(G, S1 + S2 ... + Sn) = e(Q, Z1) * e(Q, Z2) ... * e(Q, Zn)

#### Signature Aggregation

Define S = S1 + S2 ... + Sn, Q = Q1 + Q2 ... + Qn, hence:
e(G, S) = e(G, S1 + S2 ... + Sn)
  = e(G, (d1 + d2 ... + dn) x Z)
  = e(Q1 + Q2 ... + Qn, Z)
  = e(Q, Z) 

#### MulitiSignature(Threshold Signature) m-of-n

Setup:
ai = H(Qi, Q1, Q2 ... , Qn)
Q = a1 x Q1 + a2 x Q2 ... + an x Qn
MKi = (a1 x d1) x H(Q, i) + (a2 x d2) x H(Q, i) ... + (an x dn) x H(Q, i)

Define:
Z = H(Q, m)
Si = di x H(P, m) + MKi = di x Z + MKi
S = Sa + Sb ... + Sm
Qs = Qa + Qb .. + Qm

Verification will be:

e(G, S) = e(G, da x Z + MKa + db x Z + MKb ... + dm x Z + MKm)
  = e(Qs, Z) * e(G, MKa + MKb ... + MKm)
  = e(Qs, Z) * e(Q, H(Q, a) + H(Q, b) ... + H(Q, m))
  
  
  
## Comparison

EDSA is commonly used for transaction signatures, the verifications involve point addition and scalar multiplications, regarded as a bit heavy. With simple extra logic, it can do threshold signatures in smart contract. Schnorr signature has a nice feature for batch verifications, and it support signature verifications too whith pre-shared random point chosen. BLS signature is full deterministic and support easy signature aggregations and this could decrease much storage pressure and verification pressure when signer size becomes quite big. In addition, it support threshold signatures too.





