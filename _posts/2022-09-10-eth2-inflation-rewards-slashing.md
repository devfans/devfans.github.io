---
layout: post
title: "Ethereum 2.0 Inflation/Rewards/Slashing"
author: devfans
categories: [ Blockchain, Ethereum ]
image: /static.livefeed.cn/static/blog/secret-channel.jpeg
tags: [ eth2 ]
---

Did a code reading about Ethereum2.0, here's part of inflation, rewarding and slashing.



# Ethereum 2.0 - Inflation/Rewards/Slashing







PREPARED BY
Stefan Liu





## Inflation
https://ycharts.com/indicators/ethereum_supply
https://ethereum.org/en/upgrades/merge/issuance/

￼

#### ETH issuance
​		Mining rewards ~13,000 ETH/day pre-merge
​		Staking rewards ~1,600 ETH/day pre-merge
​		After The Merge, only the ~1,600 ETH per day will remain, dropping total new ETH issuance by ~90%
​		The burn: At an average gas price of at least 16 gwei, at least 1,600 ETH is burned every day, which effectively brings net ETH inflation to zero or less post-merge.

##### Pre-merge issuance breakdown
Total ETH supply: ~119,300,000 ETH (as of Q2 2022)
Execution layer issuance:
		Estimating at 2.08 ETH per 13.3 seconds*: ~4,930,000 ETH issued in a year
		Currently inflating at ~4.13% (4.93M per year / 119.3M total)
		*This includes the 2 ETH per canonical block, plus an average of 0.08 ETH over time from ommer blocks. Also uses 13.3 seconds, the baseline block time target without any influence from a difficulty bomb. (See source)
Consensus layer issuance:
		Using 13,000,000 total ETH staked, the rate of ETH issuance is ~1600 ETH/day (See source)
		Results in ~584,000 ETH issued in a year
		Currently inflating at ~0.49% (584K per year / 119.3M total)

Total annual issuance rate: ~4.62% (4.13% + 0.49%)
~89.4% of the issuance is going to miners on the execution layer (4.13 / 4.62 * 100)
~10.6% is being issued to stakers on the consensus layer (0.49 / 4.62 * 100)

##### Post-merge inflation breakdown
​		Total ETH supply: ~119,300,000 ETH (as of Q2 2022)
​		Execution layer issuance: 0
​		Consensus layer issuance: Same as above, ~0.49% annual issuance rate (with 13 million ETH staked)
​		Total annual issuance rate: ~0.49%
Total annual issuance rate: ~0.49%
Net reduction in annual ETH issuance: ~89.4% (0.49% / 4.62% * 100)



## Rewards

### Attestation Rewards
#### Conditions:
- PackedSlot > AttSlot +  minAttestationInclusionDelay(1)
- PackedSlot < AttSlot + SlotsPerEpoch(32)
- ValidSignature

#### Rewards:
baseReward =AttesterEffectiveBalance * BaseRewardFactor(64)/ sqrt(totalBalance)

numerator = baseRewards * (timelySourceWeight(14) + timelyTargetWeight(26) + timelyHeadWeight(14))

denominator = (weightDenominator(64) - proposerWeight(8)) * weightDenominator(64) / properserWeight(8)

- IncreaseBalance: proposerRewards 
= numerator / denominator(448)
= AttesterEffectiveBalance * 7.714285714285714 / sqrt(totalBalance)
~2111gwei (* 420000/32 = 0.02818eth)

https://github.com/prysmaticlabs/prysm/blob/d077483577bc8fdc658940adf71d8d11dfa5949b/beacon-chain/core/altair/attestation.go#L48



## Slashing

### Proposer Slashing 
#### Conditions
- Same proposer
- Same slot
- Diff header
- Valid signature
https://github.com/prysmaticlabs/prysm/blob/84916672c6ad0856db7b22861a7c4903326b0121/beacon-chain/core/blocks/proposer_slashing.go#L97
#### Slash
- ExitEpoch = slotEpoch + 1 + maxSeedLookahead(4)
- WithdrawableEpoch = max(
	.	 exitEpoch + minValidatorWithdrawabilityDelay(256),
	.	slotEpoch + epochsPerSlashingVector(8192)
)
https://github.com/prysmaticlabs/prysm/blob/5d4078305acf9b0779a2416e60885ee4d3ccc2ff/beacon-chain/core/validators/validator.go#L44
- DecreaseBalance on slashed proposer:  effectiveBalance/penaltyQuotient(32)
~ 1 eth
- IncreaseBalance  
  proposerReward  = effectiveBalance/whistleBlowerRewardQuotient(512) / properoserRewardQuotient(8)
  ~0.0078125 eth
- IncreaseBalance 
  whistleBlowerReward = effectiveBalance/whistleBlowerRewardQuotient(512) - proposerReward
  ~ 0.0546875
- MarkAsSlashed
- InitExit
- SlashBeforeWithdraw:  WithdrawableEpoch - 4096
- DecreaseBalance = effectiveBalance * min(activeBalance, totalSlashedBalance * 3)/activeBalance
	~ 32 ETH

### Attester Slashing
#### Conditions:
- Double votes or Surround votes
- ValidSignature
https://github.com/prysmaticlabs/prysm/blob/84916672c6ad0856db7b22861a7c4903326b0121/beacon-chain/core/blocks/attester_slashing.go#L145
#### Slash:
- ExitEpoch = slotEpoch + 1 + maxSeedLookahead(4)
- WithdrawableEpoch = max(
    	 exitEpoch + minValidatorWithdrawabilityDelay(256),
    	 slotEpoch + epochsPerSlashingVector(9182)
    )
https://github.com/prysmaticlabs/prysm/blob/5d4078305acf9b0779a2416e60885ee4d3ccc2ff/beacon-chain/core/validators/validator.go#L44
- DecreaseBalance on slashed proposer:  effectiveBalance/penaltyQuotient(32)
~ 1 eth
- IncreaseBalance  
  proposerReward  = effectiveBalance/whistleBlowerRewardQuotient(512) / properoserRewardQuotient(8)
  ~ 0.0078125 eth
- IncreaseBalance 
	whistleBlowerReward = effectiveBalance/whistleBlowerRewardQuotient(512) - proposerReward
~ 0.0546875 eth

- MarkAsSlashed
- InitExit
- SlashBeforeWithdraw:  WithdrawableEpoch - 4096
  DecreaseBalance = effectiveBalance * min(activeBalance, totalSlashedBalance * 3)/activeBalance
  ~ 32 ETH

### Sync Slashing
#### Conditions:
- Vote on last slot
- Same last block root or not
- Valid Signature
https://github.com/prysmaticlabs/prysm/blob/d077483577bc8fdc658940adf71d8d11dfa5949b/beacon-chain/core/altair/block.go#L129
#### Slash: 
- (didnt vote, or vote on diff root)

baseRewardIncrement = effectiveBalanceIncrement(1gwei) * baseRewardFactor(64) / sqrt(activeBalance)

totalBaseRewards
= baseRewardIncrement * activeBalance/effectiveBalanceIncrement(1gwei)
 = activeBalance * baseRewardFactor(64) / sqrt(activeBalance)

participantReward 
= totalBaseRewards * syncRewardsWeight(2) / weightDenominator(64) /  slotsPerEpoch(32) / syncCommitteeSize(512)
= 0.0001220703125 * activeBalance / sqrt(activeBalance)
~ 14270 gwei (*32 * 225  = 0.1027eth)

proposerRewards
= participantReward * proposerWeight(8) / (weightDenominator(64) - proposerWeight(8))
= 0.00001743861607142857 * activeBalance / sqrt(activeBalance)
~ 2038 gwei    ( * 512 = 0.00104eth)

IncreaseBalance: participantRewards
IncreaseBalance: proposerRewards
DecreaseBalance: participantRewards

### Inactivity Slashing on Epoch End
#### inactivityScore
- isPrevEpochTargetAttester: inactivityScore - 1
- !isPreveEpochTargetAttester: inactivityScore + 4
- !isInInactivityLeak: inactivity - 16
https://github.com/prysmaticlabs/prysm/blob/d077483577bc8fdc658940adf71d8d11dfa5949b/beacon-chain/core/altair/epoch_precompute.go#L74

#### Slash:
baseRewardMultiplier
= effectiveBalanceIncrement(1gwei) * baseRewardFactor(64)/sqrt(activeBalance) 

baseReward
= effectiveBalance * baseRewardMultiplier / effectiveIncrement(1gwei)
= effectiveBalance * baseRewardFactor(64)/sqrt(activeBalance) 

activeIncrement = activeBalance / effectiveIncrement(1gwei)
inactivityDenominator = inactivityScoreBias(4) * inactivityPenaltyQuotient(2**24)
https://github.com/prysmaticlabs/prysm/blob/d077483577bc8fdc658940adf71d8d11dfa5949b/beacon-chain/core/altair/epoch_precompute.go#L287 

quota = 
srcWeight(14) * prevEpochAttested + 
targetWeight(26) * prevEpochTargetAttested + 
headWeight(14) * prevEpochHeadAttested

validatorReward
= baseReward * quota / effectiveIncrement(1gwei) / (activeIncrement * weightDenominator(64)) 
~ 14781 (* 225 = 0.00332eth)

validatorPenalty
= baseReward / weightDenominator(64) * (srcWeight + targetWeight) + effectiveBalance * inactivityScore / inactiveDenominator
~ 10949 Gwei ( * 225 = 0.00246eth )







### Summary 2022 9.14

#### Issued ETH
120,000,000 ETH
#### ActiveBalance
13,607,322 ETH
#### Validators
427057
#### ETH burnt 
1423ETH/Day   ~ -0.43% inflation
#### Produced ETH Beacon Chain
1915ETH/256Epochs ~0.5% inflation
#### Proposer Reward per Block 
0.029 ETH
#### Validator Reward per Day
0.00332 ETH
#### Sync Committee Reward per 256 Epochs
0.1168ETH



Sync Committee / 256 epochs
	14270 * 32 * 256 * 512 = 59.85
Proposer / 256 epochs
	2111 * 427057/32  * 32 * 256 +  2038 * 512 * 32 * 256= 239.33
Attestation / 256 epochs
	14781 * 256 * 427057 = 1615.9







