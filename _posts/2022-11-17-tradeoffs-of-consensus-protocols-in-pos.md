---
layout: post
title:  "Trade-Offs of Consensus Protocols in PoS"
author: devfans
categories: [ Blockchain ]
image: /static.livefeed.cn/static/blog/consensus.png
tags: [ sticky, Consenus, Blockchain, PoS, BFT ]
---


In the early age of blockchains, most of the chains adopted PoW as the consensus model, using math puzzles of certain difficulties to reach a consensus for new blocks to extend the chain. The main drawback is the cost of electric power which is harmful to our ecosystem. I believe PoS is a replacement solution that's proposed to fix this drawback, tho PoS has other drawbacks than PoW. PoS is a stake-secured model, the main idea is as:

- All consensus nodes should have enough stake locked to join the consensus
- Honest nodes should follow the protocol to reach a consensus
- Faulty nodes should be slashed for their misbehaviors.
- Honest nodes are normally rewarded as an incentive for their effort to a consensus.
- Faulty nodes should be limited to a certain portion of the total nodes for a sustainable chain
- Use a consensus protocol to provide safety and liveness

Safety ensures finalized blocks can not be reverted, and liveness ensures transaction will be committed and the chain will be extended. Unlike PoW which serves as a complete solution for the consensus of a chain, PoS needs to combine with a consensus protocol, so consensus nodes (with stake locked) follow the protocol to reach a consensus when there's a need.


I read some papers on these kinds of protocols adopted for PoS chains, here's to talk about the tradeoffs of each.


### PBFT (assume up to f faulty nodes, 3 messages delay)

PBFT has three phases: pre-prepare, prepare and commit. The first two are used to order requests in the same view even when the proposer is faulty. For blockchains, a request means a new block with a height n. The last two are used to ensure the requests that commit are ordered across views.

![pbft-diagram](/static.livefeed.cn/static/blog/pbft.png)

#### Safety:

If prepared(m, v, n, i) hold true for one non-faulty replica, then prepared(m', v, n, i) is false for any non-faulty replicas.

When a block is decided by 2f + 1 replicas, at least f + 1 non-faulty replicas hold prepared(m, v, n, i) as true, and this indicates prepared(m, v + 1, n, i) will hold since there's at least one of them propagated it in view-change(v + 1, n, c, p, i)(included in p set), and re-broadcast with new-view(v+1, vc, o)(included in vc and o).

#### Liveness:

To avoid starting a new view too soon, a non-faulty replica needs to wait for 2f + 1 view change messages after it multicasts it and then start a timer. If the timer expires before a new-view message or before executing a request in the new view. It starts a new view, and the timer is doubled.

To avoid change-view too late, a non-faulty replica sends view-change when f + 1 view-change messages are received.

Faulty nodes are unable to force a view change if it's not the primary.

#### Drawbacks:

All replicas need to broadcast messages, for a normal case, the burden is O(n^2), and during a view-change, it's O(n^3) for a replica, O(n^4) for the primary.


### Tendermint (assume less than 1/3 faulty nodes, 3 messages + fixed duration delay)


Assumes the network is partially synchronous, which has some unknown upper bound on the time of the messages to be delivered. A block is said to be committed by the network when a 2/3 majority of validators had signed and broadcast commits for that block.

![tendermint-diagram](/static.livefeed.cn/static/blog/tendermint.png)

#### Safety:

If a non-faulty validator decides on block B at Round R, then at least 1/3 of validators hold proof-of-lock for B, this makes unlocking it impossible to decide on another block B'.

#### Liveness:

With less than 1/3 faulty validators it won't deadlock. When a deadlock happens without an active global adversary, older lock R will be unlocked by R'.

Regardless of the current round or step, a non-faulty validator immediately enters the Commit step if not yet when receiving more than 2/3 commits for a particular block.

After committing a block, nodes stay until a fixed duration past CommitTime to allow including more than the minimum 2/3 of commits.

#### Drawbacks:

All replicas need to broadcast messages to the neighbors, for a normal case, the burden is O(n^2), and during a view-change, it's O(n^3) for a validator.


### Basic HotStuff (assume up to f faulty nodes, 8 messages delay)

Resolves around a three-phase core, introduce a second phase to allow replicas to change their mind after voting without a leader proof required. After GST, the correct leader sends only O(n) authenticators to drive a consensus decision(Linear View Change), O(n^2) at worst case. In addition, the leader needs to wait just for the first n - f responses to guarantee it can create a proposal that will make progress(Optimistic Responsiveness).

![basichotstuff-diagram](/static.livefeed.cn/static/blog/basic_hotstuff.png)

#### Safety:

For any QC(type, view, node), if with the same type and QC1.node conflicts with QC2.node then QC1.view != QC2.view.

Conflicts nodes can not be both committed. If node1 conflicts with node2 and both are committed, then we have v1 != v2. Let CommitQC(s) has the minimum view for a valid QC and has a node that conflicts with node1, then prepareQC(s).node is not extended from prepareQC1.node, since QC(s) has the minimum view, then the prepareQC(s).justify.view has to be equal to prepareQC1.view which is not, so verification of a safe node would fail.

#### Liveness:

If a correct replica is locked then at least f + 1 non-faulty nodes voted for a prepareQC matching the lockedQC. After GST, there exists a bounded time period Tf such that if all correct replicas remain in the same view, during Tf, and the leader is not faulty, then a decision will be made to reach a consensus.

However if combine prepare phase with precommit phase to a single QC, then there's the possibility of an infinite non-deciding scenario. Say Rv are locked on QC(b), and in a new view, a leader does not collect it from them, then the new QC(v+1).justify.view = v - 1, which would be rejected by Rv.

#### Drawbacks:

In HotStuff paper, it's using threshold signature which is unfair when compared with other protocols. So here we ignore it. For a normal case, the communication cost is O(1) or O(n) for a replica, O(n) or O(n^2) for the leader.

In addition, to reach a decision, there's an 8-messages delay, which is a lot more than other solutions.

Besides, a noticeable situation could lead to a fork of chains. During the DECIDE phase, if a faulty leader collected enough(2f +1) votes and hold that without broadcasting it, or a non-faulty leader collected the votes but can only successfully send the QC(nodeB) to some faulty nodes due to a transient network issue,  and failed to send the new view message to the next leader. There will be a new QC(nodeB') that can be decided on with a valid parent relation: nodeB'.parent = nodeB.


### Casper FFG (assume less than 1/3 faulty nodes, 2 checkpoints delay)

Assumes there is a fixed set of validators and a proposal mechanism, and a majority link is weighted with the sum of the deposit of the participants of the link. A justified checkpoint c hold, when c is the root or there's a majority link c' -> c where c' is justified. A check point c is finalized when there's a majority link c -> c' where h(c) = h(c') - 1, and c' is justified.

Casper's FFG has two fundamental properties: accountable safety and plausible liveness. The first ensures two conflicting checkpoints won't be both finalized unless there are 1/3 of the validators slashed. the latter ensures when at least 2/3 of validators follow the protocol, there's always possible to reach a consensus.

![casperffg-diagram](/static.livefeed.cn/static/blog/casperffg.png)

#### Safety:

If s1->t1 is distinct from s2->t2 then h(t1) != h(t2).

If s1->t1 and s2->t2 are distinct, then h(s1) < h(s2) < h(t2) < h(t1) wont hold.

There exists at most one majority link on a target height and at most one justified checkpoint on that height.

Two conflicting blocks won't be both finalized. If Bm(with direct child Bm+1) and Bn(with direct child Bn+1) are distinct and both finalized, then on the chain compatible with Bn, there exists a Bk that's the maximum checkpoint lower than Bm and Bh that's minimum checkpoint higher than Bm+1, this is impossible.

#### Liveness:

Majority links can always be added to produce new finalized checkpoints, provided there exist children extending the finalized chain. Say a justified checkpoint a with the greatest height, checkpoint b with higher height, and a' has h(a') = h(b) + 1, it's always possible for a' to have a direct child justified checkpoint to finalize a'.

#### Drawbacks:

For a normal case, the communication cost of a validator is always O(1), while the drawback is the delay is too much for finalization, which is at least 2 checkpoints(which is more than 10 minutes for Ethereum 2.0). Besides that, block reorganizations are always possible.

