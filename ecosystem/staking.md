---
layout: default
title: NFT Staking
nav_order: 1
parent: Ecosystem
---

# NFT Staking

### Minting cFont
**minting 1 cFont always use 100 eBTC**

You need to allow eBTC for minting first then you can mint cFont.

![image](https://user-images.githubusercontent.com/55227490/146706874-a7aa79da-66af-4106-9e6f-2d8fd71d77a9.png)
![image](https://user-images.githubusercontent.com/55227490/146706928-e1100ca9-2b91-4687-97d5-399af4e655fb.png)

### Reward calculation

![NFT Staking](../assets/nftStaking.PNG)

NFT Staking reward depends on NFT rarity. Every 7 days any user can call "distributeReward()" function to distributed reward to self and other staker. 
The reward will be calculated by the fomula below

```
Reward of NFT(id) = ((eBTC.balanceOf(cFontContract) / Allrarity) !> 10 * 10**18 ) + feePerRarity

In English
- distribute no more than 10 eBTC token per 1 NFT rarity and adding feePool the the reward late.
```

### Recieve reward

![image](https://user-images.githubusercontent.com/55227490/146707010-aa698c99-7ae4-4c27-a2c4-3365de017298.png)

User can recieve reward in 2 parts. First is eBTC reward and second is ETH reward(Native token change depend on chain).

After withdraw reward. Your wallet will topup automatically.
