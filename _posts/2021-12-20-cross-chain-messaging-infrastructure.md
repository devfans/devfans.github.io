---
layout: post
title:  "Cross Chain Messaging Infrastructure"
author: devfans
categories: [ CrossChain, BlockChain, Bridge ]
image: /static.livefeed.cn/static/blog/ccm.jpg
tags: [featured]
---

Cross chain asset tranfer functionality (aka bridging) becomes more and more important as blockchain's foastering more and more hot concepts like DEFI, NFT, DEX, etc. Advanced users are getting familar with cross chain bridges, comparing the efficiency and fee cost. With seeing more requirements, several great bridge/swap tools become popular and attract more users to lock assets on their contracts. 

However, cross chain functionality in blockchain area is not an easy thing to do, which involves contract security, proof relay across homogeneous and heterogeneous chains and consensus security risk on each chain. Each small defact could leave a possible vunerability that could be found by hackers. It's like every day, you can find news about a new bridge/swap project get hacked, the PolyNetwork one is the largest one among the hacks and possible the luckiest one since the Hacker returned almost all of the stolen assets.

Though cross chain token transfer is the main requirement currently for bridging, the fundermental cross chain messaging infrastructure is the core tech that powers most of the bridges. Some bridges may use simpler solutions for token bridging, like `Celer` uses hash locks and focus on asset transfer, but possible these solutions can be abstracted/extended further as a common messaging infrastructure. Here's digging a bit into the one built by PolyNetwork.

PolyNetwork built a typical fundermental cross chain message infrastructure with a solution compatible with homogeneous and heterogeneous chains as long as the chain provide the state proof of cross chain messaging. For Ethereum and similar chains, the state proof is provided and could be verified with account proof against headers, so the role for submitting the headers to the middle chain is important, also the hardfork attack should be taken care of in the middle chain handler.

A typical cross chain message could be identified by a following fields.

- Source Chain ID
- Source Chain Message ID (autoincremental)

The message body could include:

- Target Chain ID
- Target Contract
- Target Contract Method and Arguements

With these abstraction, the middle chain could verify the existence of the  message source with the proof and header chain:

```

func VerifyProof(address common.Address, proof *ceth.ETHProof, header *types.Header) (key, val []byte, err error) {
	nl := new(light.NodeList)
	for _, p := range proof.AccountProof {
		nl.Put(nil, common.FromHex(p))
	}
	if !bytes.Equal(address.Bytes(), common.HexToAddress(proof.Address).Bytes()) {
		err = fmt.Errorf("Not target address %s", proof.Address)
		return
	}
	v, err := trie.VerifyProof(header.Root, crypto.Keccak256(address.Bytes()), nl.NodeSet())
	if err != nil {
		err = fmt.Errorf("Verify account proof failed %w %+v", err, *proof)
		return
	}
	balance, _ := new(big.Int).SetString(strings.TrimPrefix(proof.Balance, "0x"), 16)
	nonce, _ := new(big.Int).SetString(strings.TrimPrefix(proof.Nonce, "0x"), 16)
	a := &eth.ProofAccount{
		Codehash: common.HexToHash(proof.CodeHash),
		Storage:  common.HexToHash(proof.StorageHash),
		Balance:  balance,
		Nounce:   nonce,
	}
	data, err := rlp.EncodeToBytes(a)
	if err != nil {
		return
	}
	if !bytes.Equal(v, data) {
		err = fmt.Errorf("Invalid account")
		return
	}
	if len(proof.StorageProofs) != 1 {
		err = fmt.Errorf("Invalid storage proof size")
		return
	}
	sp := proof.StorageProofs[0]
	nl = new(light.NodeList)
	for _, p := range sp.Proof {
		nl.Put(nil, common.FromHex(p))
	}
	key = common.HexToHash(sp.Key).Bytes()
	val, err = trie.VerifyProof(common.HexToHash(proof.StorageHash), crypto.Keccak256(key), nl.NodeSet())
	return
}

```

Here, the `value` returned is a `keccak256` hash of the message body used to verify against the message. With this verification, as long as the header chain submitted by the relayer is reliable, the cross chain messaging from source chain could be assumed as trustworthy. So the middle chain should implement a valid header chain verification to protect from hard-fork attack.

Another popularly adopted solution is using majority voting, so voters keep observing the source chain, and submit the valid cross chain messages and sign with private keys. When the majority votes arrive, the message will be marked as valid and ready to be confirmed and dispatched.

The proof relay to target chain only involves the middle chain proof to be verified on the target chain. For PolyNetwork, the consensus node(validator sets) need to be synchronized to the target chain(exactly where the hack was rooted). Then the target chain contract will be able to verify the cross chain message with the signatures against the validator set. 

Here's the code in target chain smart contract:

```

// Get stored consensus public key bytes of current poly chain epoch and deserialize Poly chain consensus public key bytes to address[]
address[] memory polyChainBKs = ECCUtils.deserializeKeepers(eccd.getCurEpochConPubKeyBytes());

uint256 curEpochStartHeight = eccd.getCurEpochStartHeight();

uint n = polyChainBKs.length;
if (header.height >= curEpochStartHeight) {
    // It's enough to verify rawHeader signature
    require(ECCUtils.verifySig(rawHeader, headerSig, polyChainBKs, n - ( n - 1) / 3), "Verify poly chain header signature failed!");
} else {
    // We need to verify the signature of curHeader 
    require(ECCUtils.verifySig(curRawHeader, headerSig, polyChainBKs, n - ( n - 1) / 3), "Verify poly chain current epoch header signature failed!");

    // Then use curHeader.StateRoot and headerProof to verify rawHeader.CrossStateRoot
    ECCUtils.Header memory curHeader = ECCUtils.deserializeHeader(curRawHeader);
    bytes memory proveValue = ECCUtils.merkleProve(headerProof, curHeader.blockRoot);
    require(ECCUtils.getHeaderHash(rawHeader) == Utils.bytesToBytes32(proveValue), "verify header proof failed!");
}

// Through rawHeader.CrossStatesRoot, the toMerkleValue or cross chain msg can be verified and parsed from proof
bytes memory toMerkleValueBs = ECCUtils.merkleProve(proof, header.crossStatesRoot);

```

Also the double spend/execution need to be taken care of, usualy with the solution of using a `DONE` check in the middle chain and target chains.


Cross chain messaging infractructure will play a more and more important role in DEFI area, with opening messaging tunnels that enable the inter-operations between chains. Various DEFI concepts may arise from time to time that use these tunnels to link the isolated chains to combine the strength of the each other for implementing awesome projects and carry the application of the blockchain technology a big step forward.





