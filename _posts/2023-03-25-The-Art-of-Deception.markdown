---
layout: post
title:  "[CTF]  The Art of Deception - Cyber Apocalypse 2023"
date:   2023-03-25 13:21:16 -0300
categories: jekyll update
---

Today gonna solve the last blockchain challenge from Cyber Apocalypse 2023 Hackthon

Summary : This challenge to be done you need to create a smart contract to interact with the target using a the external function


Challenge : 
![image](https://user-images.githubusercontent.com/37570367/227741382-ed69a59d-9e23-45c0-a905-68aa39cd5abf.png)

Ok lets download the Files and get the information from the docker :
![image](https://user-images.githubusercontent.com/37570367/227741532-d63043c6-b1cb-41c6-aca1-a73f7fd944d8.png)


There are two files :

# Setup.sol

{% highlight ruby %}
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.18;

import {HighSecurityGate} from "./FortifiedPerimeter.sol";

contract Setup {
    HighSecurityGate public immutable TARGET;

    constructor() {
        TARGET = new HighSecurityGate();
    }

    function isSolved() public view returns (bool) {
        return TARGET.strcmp(TARGET.lastEntrant(), "Pandora");
    }
}

{% endhighlight %}


# FortifiedPerimeter.sol
{% highlight ruby %}

pragma solidity ^0.8.18;


interface Entrant {
    function name() external returns (string memory);
}

contract HighSecurityGate {
    
    string[] private authorized = ["Orion", "Nova", "Eclipse"];
    string public lastEntrant;

    function enter() external {
        Entrant _entrant = Entrant(msg.sender);

        require(_isAuthorized(_entrant.name()), "Intruder detected");
        lastEntrant = _entrant.name();
    }

    function _isAuthorized(string memory _user) private view returns (bool){
        for (uint i; i < authorized.length; i++){
            if (strcmp(_user, authorized[i])){
                return true;
            }
        }
        return false;
    }

    function strcmp(string memory _str1, string memory _str2) public pure returns (bool){
        return keccak256(abi.encodePacked(_str1)) == keccak256(abi.encodePacked(_str2)); 
    }
}


{% endhighlight %}

Lets go to Remix IDE and paste the Target Smart Contract to see how works and what need to do yo bypass the security

First at all we need to understad what the Setup.sol needs to solve the challenge :

![image](https://user-images.githubusercontent.com/37570367/227742077-908aa260-1fc3-468e-b6a5-e87a5c92ac1c.png)


Ok lets see now the Target smart contract in this case `FortifiedPerimeter.sol` :

![image](https://user-images.githubusercontent.com/37570367/227742394-79cb7818-b3a1-4344-ab4a-79575299b256.png)

Ok so to solve this we need to bypass the enter function and register us like pandora, but how we can do that ?

To do that you need to understand the Interface and the external function that they have

An external function can only be called for external interaction, so this mean if we compile this smart contract and try to run wihout do nothing the enter function is gonna revert with an error , this is because the `name` function declared in the `Interface` has no returned value

So to solve this is gonna a be a little bit more difficult than the other challenges, because we need to create a smart contract to interact and modify the `name function` to bypass the `enter funcion` flow

Let me show you my attack smart contract and explain it for you, here a simple screenshot of the smart contract attack :

# Attack.sol
![image](https://user-images.githubusercontent.com/37570367/227742624-a0c9d12a-ccdd-489c-9756-2e9d461da3bd.png)


Ok let me explain to you how its work, is so simple :
![image](https://user-images.githubusercontent.com/37570367/227743338-661cad10-3d05-4af1-837e-4e4c531f5531.png)

Lets go more in deep of the flow : 
![image](https://user-images.githubusercontent.com/37570367/227743652-9678376b-f31c-4603-9ebb-616ec32e864c.png)

So when the flow is gonna be called in the Target Smart Contract, in this case "FortifiedPerimeter" we can bypass the whitelist and we can run the enter function, let me draw it :

![image](https://user-images.githubusercontent.com/37570367/227743966-468c2091-3e10-43f6-9b61-18d4620be91c.png)


And thats it, so lets deploy the smart contracts and see what happend after the attack : 


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
