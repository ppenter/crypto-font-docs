---
layout: default
title: NFT Staking
nav_order: 1
parent: Ecosystem
---

# NFT Staking

![NFT Staking](assets/nftStaking.PNG)

NFT Staking reward depends on NFT rarity. Every 7 days any user can call "distributeReward()" function to distributed reward to sulf and other staker. 
The reward wwill calculated by the fomula below

```
Reward of NFT(id) = (feePool/allRarity) * NFT.rarity

Reward of staker(address) = (stakingPool/allLpToken) * staker.amount

*note that*
feePool = 30% of overall minting distribute to NFT Token.
stakingPool = 20% of overall minting distribute to Token staking user.
```

