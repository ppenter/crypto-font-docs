---
layout: default
title: cFont NFT
nav_order: 1
parent: Token Information
---

# cFont NFT

**Token name:** Crypto Font

**Symbol:** cFont

**Price:** 100 eBTC

**totalSupply:** No

# Main function

```
    function createRandomcFont() external;
    function reverseSwap(uint256 id) external;
    function getcFonts() external view returns(cFont[] memory);
    function countAllRarity() external view returns(uint256);
    function countAllReward() external view returns(uint256);
    function countAllEthers() external view returns(uint256);
    function remainingEthers() external view returns(uint256);
    function remainingeBTC() external view returns(uint256);
    function DistributeReward() external;
    function showEthers(address account) external view returns(uint256);
    function getEthers(uint id) external;
    function getAllEthers() external;
    function showReward(address account) external view returns(uint256);
    function getReward(uint id) external;
    function getAllReward() external;
    function setName(string memory name, uint256 id) external;
    function getTimeRemain() external view returns(uint256);
    function AddeBTCFee(uint256 _amount) external;
    function getIdsOfOwner(address _owner) external view returns (uint[] memory);
```


# Contract address

**BSC Mainnet:** 

**BSC Testnet:**
