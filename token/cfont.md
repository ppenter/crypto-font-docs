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
contract cryptoFont is ERC721Enumerable,utils  {
    uint256 COUNTER = 0;
    address _owner;
    IERC20 public eBitcoin;
    IERC20 public lpToken;
    uint256 public vcost = 100 * 10**18;
    uint256 public vfee = vcost * 30 / 100;
    uint256 public vstakeFee = vcost * 20 / 100;
    uint256 public _feePool;
    uint256 public _stakePool;
    uint256 public _rewardPool;
    uint256 public _pastDistributedReward;

    // counters
    uint256 public lpCounter;
    uint256 public rarityCounter;

    struct staker{
        uint256 amount;
        uint256 index;
        bool exists;
    }

    mapping(uint256 => uint256) public tokenToReward;
    mapping(uint256 => uint256) public tokenToEthers;

    mapping(address => uint256) public stakerReward;
    mapping(address => staker) public lpStakers;
    
    address[] public stakerIndexes;

    function setLpAddress(address _address) public{
        require(msg.sender == _owner);
        lpToken = IERC20(_address);
    }

    function depositeLp(uint256 amount) public{
        require(address(lpToken) != address(0));
        require(lpToken.balanceOf(msg.sender) >= amount);
        if(lpStakers[msg.sender].exists){
            lpToken.transferFrom(msg.sender, address(this), amount);
            lpStakers[msg.sender].amount += amount;
        }
        else{
            stakerIndexes.push(msg.sender);
            lpStakers[msg.sender].amount = amount;
            lpStakers[msg.sender].index = stakerIndexes.length-1;
            lpStakers[msg.sender].exists = true;
        }
        lpCounter += amount;
    }

    function withdrawLp(uint256 amount) public{
        require(lpToken.balanceOf(address(this)) >= amount);
        require(lpStakers[msg.sender].amount >= amount);
        lpToken.approve(msg.sender, amount);
        lpToken.transfer(msg.sender, amount);
        lpStakers[msg.sender].amount -= amount;
        lpCounter -= amount;
    }

    struct cFont {
        string name;
        uint256 id;
        uint256 dna;
        uint8 rarity;
        uint256 stat;
        uint16 power;
        uint8 size;
        uint256 mintDate;
        bool burn;
    }

    cFont[] public cFonts;

    event NewcFont(address indexed owner, uint256 id, uint dna);

    function _createRandomNum(uint256 _mod, string memory str) internal view returns(uint256){
        uint256 randomNum = uint256(keccak256(abi.encodePacked(block.difficulty, block.timestamp, msg.sender, str)));
        return randomNum % _mod;
    }

    constructor(address eBitcoinAddress)
        ERC721("Crypto Font", "cFont")
    {
        eBitcoin = IERC20(eBitcoinAddress);
        _pastDistributedReward = block.timestamp;
        _owner = msg.sender;
    }
    
    function _createcFont(string memory _name) internal{
        uint8 randRarity = uint8(_createRandomNum(100, _name)) + 1;
        uint256 randDna = _createRandomNum(10**16, _name);
        uint256 randStat = _createRandomNum(10**16, _name);
        uint16 randPower = uint16(_createRandomNum(10000, _name) + 1);
        uint8 randSize = uint8(_createRandomNum(140, _name) + 10);
        uint8 rarity = 1;
        uint256 mint_date = block.timestamp;

        if(randRarity < 30){
            rarity = 2;
            if(randRarity < 6){
                rarity = 3;
                if(rarity <1){
                    rarity = 4;
                }
            }
        }

        rarityCounter+= rarity;
        
        cFont memory newcFont = cFont(_name, COUNTER, randDna, rarity, randStat, randPower,randSize,mint_date,false);
        cFonts.push(newcFont);
        _safeMint(msg.sender, COUNTER);
        emit NewcFont(msg.sender,COUNTER, randDna);
        COUNTER++;
    }

    function createRandomcFont() public{
        require(eBitcoin.balanceOf(msg.sender) >= vcost);
        require(eBitcoin.transferFrom(msg.sender,address(this), vcost));
        _rewardPool+= (vcost-vfee-vstakeFee);
        _feePool+=vfee;
        _stakePool+=vstakeFee;
        string memory name = concatenate("cFont #",uint2str(COUNTER));
        _createcFont(name);
    }

    function create10Font() public{
        for(uint256 i = 0;i < 10; i++){
            createRandomcFont();
        }
    }

    function reverseSwap(uint256 id) public{
        require(ownerOf(id) == msg.sender);
        _burn(id);
        rarityCounter -= cFonts[id].rarity;
        eBitcoin.transfer(msg.sender, (vcost - vfee - vstakeFee));
        cFonts[id].burn = true;
    }

    function getcFonts() public view returns(cFont[] memory) {
        return cFonts;
    }

    function _getRarity(uint256 id) internal view returns(uint256){
        uint256 rarity = 0;
        if(!cFonts[id].burn){
                rarity=cFonts[id].rarity;
            }
        return rarity;
    }

    function countAllReward() public view returns(uint256){
        uint256 reward = 0;
        uint256 i;
        for (i=0;i<COUNTER;i++){
            reward+=tokenToReward[i];
        }
        
        return reward;
    }

    function countAllEthers() public view returns(uint256){
        uint256 reward = 0;
        uint256 i;
        for (i=0;i<COUNTER;i++){
            reward+=tokenToEthers[i];
        }
        
        return reward;
    }


    function remainingEthers() public view returns(uint256){
        return address(this).balance - countAllEthers();
    }

    function remainingeBTC() public view returns(uint256){
        return eBitcoin.balanceOf(address(this)) - countAllReward() - _feePool - _rewardPool;
    }

    function DistributeReward() public{
        require(block.timestamp > _pastDistributedReward + 0 days);
        uint256 reward = _feePool;

        if(remainingeBTC() > _feePool){
            reward += _feePool;
        }
        else{
            reward += remainingeBTC();
        }

        uint256 rewardPerRare =  reward / rarityCounter;
        uint256 ethersPerRare = remainingEthers() / rarityCounter;
        uint256 i;
        for(i=0; i<COUNTER;i++){
            tokenToReward[i] += rewardPerRare * _getRarity(i);
            tokenToEthers[i] += ethersPerRare * _getRarity(i);
        }

        uint256 stakeReward = _stakePool;
        uint256 rewardPerLp = stakeReward / lpCounter;

        uint arrayLength = stakerIndexes.length;
        for (i=0; i<arrayLength; i++) {
            stakerReward[stakerIndexes[i]] += rewardPerLp * lpStakers[stakerIndexes[i]].amount;
        }

        _stakePool = 0;
        _feePool = 0;
        _pastDistributedReward = block.timestamp;
    }

    function showEthers(address account) public view returns(uint256){
        uint256 reward = 0;
        for(uint256 i =0; i< balanceOf(account); i++){
            reward += tokenToEthers[tokenOfOwnerByIndex(account, i)];
        }
        return reward;
    }

    function getAllEthers() public{
        uint256 reward = 0;
        for(uint256 i = 0;i< balanceOf(msg.sender); i++){
            reward += tokenToEthers[tokenOfOwnerByIndex(msg.sender, i)];
            tokenToEthers[tokenOfOwnerByIndex(msg.sender, i)] = 0;
        }
        payable(msg.sender).transfer(reward);
    }

    receive() external payable {}

    function showReward(address account) public view returns(uint256){
        uint256 reward = 0;
        for(uint256 i =0; i< balanceOf(account); i++){
            reward += tokenToReward[tokenOfOwnerByIndex(account, i)];
        }
        return reward;
    }

    function getAllReward() public{
        uint256 reward = 0;
        for(uint256 i = 0;i< balanceOf(msg.sender); i++){
            reward += tokenToReward[tokenOfOwnerByIndex(msg.sender, i)];
            tokenToReward[tokenOfOwnerByIndex(msg.sender, i)] = 0;
        }
        eBitcoin.transfer(msg.sender, reward);
    }

    function setName(string memory name, uint256 id) public {
        require(ownerOf(id) == msg.sender);
        require(bytes(name).length > 0);
        cFonts[id].name = name;
    }


    function AddeBTCFee(uint256 _amount) public{
        require(eBitcoin.balanceOf(msg.sender) >= _amount);
        require(eBitcoin.transferFrom(msg.sender,address(this), _amount));
        _feePool += _amount;
    }

    function getIdsOfOwner(address _address) public view returns (uint[] memory) {
        uint[] memory _tokensOfOwner = new uint[](ERC721.balanceOf(_address));
        uint i;

        for (i=0;i<ERC721.balanceOf(_address);i++){
            _tokensOfOwner[i] = ERC721Enumerable.tokenOfOwnerByIndex(_address, i);
        }
        return (_tokensOfOwner);
    }
}
```


# Contract address
