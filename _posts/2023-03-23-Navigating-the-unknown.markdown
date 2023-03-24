---
layout: post
title:  "[CTF]  Navigating the Unknown - Cyber Apocalypse 2023"
date:   2023-03-23 21:21:16 -0300
categories: jekyll update
---

Hello guys, today i gonna show you how i solve the CTF Navigating the Unknown Challenge of the Cyber Apocalypse 2023 from Hack the Box hackthon

In the challenge you have multiple ways to solved, but i decide to show you the most simple way to solve because is much easy to explain whats happend


First at all you need need yo read the challenge :

![image](https://user-images.githubusercontent.com/37570367/227410502-1f052dfc-a2c1-43aa-a821-da30b416be3f.png)

Ok so we start the docker and download the files from the challenge :

![image](https://user-images.githubusercontent.com/37570367/227410625-15bd0a98-abff-4265-9652-bd2819e3d406.png)

after extract the file downloaded i see this files :

- README.md
- Setup.sol
- Unknown.sol

So lets see first what says the README.md : 

![image](https://user-images.githubusercontent.com/37570367/227410796-5bec16b7-ffb3-44ca-bbac-12e43620e298.png)

Ok ok, but this is a lot a information, we need to understand it per parts, lets go for he ports section first

![image](https://user-images.githubusercontent.com/37570367/227411051-81f1bf9a-e267-419e-ba3d-23f8285e1269.png)

ok this means we have 2 ports for the connection :
- one is gonna be the information about conne
- the another gonna be for the RPC connection with the blockchain

So Lets keep reading : 

![image](https://user-images.githubusercontent.com/37570367/227411830-c41a0ea4-6b45-416f-9565-b6a50d0c4e9f.png)

ok we have 2 files :
- Setup.sol
- Challenge.sol ( this name could be whatever name, in this case called : Unknown.sol )

Lets read the last part

![image](https://user-images.githubusercontent.com/37570367/227412351-4a05a9de-a117-43f6-bbb1-3d102dd9b9eb.png)

Ok we gonna neeed :
- private key
- target smart contract
- rpc url

And if you remember in the first section says one of the ports has information about the connection, so lets check with netcat what server has a response, lets try connect to using `nc 165.22.116.7 31092`, we wait some seconds and ...

![image](https://user-images.githubusercontent.com/37570367/227413357-669083db-2339-4bcb-9901-22d1c0efb376.png)

nothing happends, so we can asume this is the RPC connection

lets try the anotherone ... 

![image](https://user-images.githubusercontent.com/37570367/227413543-398ca390-075c-4a92-a70e-147290158a76.png)

ok this connection is more interesting, lest check the connection information : 

![image](https://user-images.githubusercontent.com/37570367/227413653-ab75b8e3-7c7b-4aa3-b921-97467df61615.png)

Ok we now have the information about the connection with the rpc , but how exacly we can connect with him ? and what exactly we need to do with the msart contracts ?

Lets go for parts, first we gonna use a more easy method than the web3js or web3py, ethers or things like that, because is not necessary if the code is not gonna be automatizated for something, so we gonna use :
- Metamask ( Crypto wallet )
- Remix IDE ( IDE for solidity Smart contracts )
- Smart Contracts
And we gonna interact with the smart contracts mor easy and more simple to understand what happend really in a general level

ok so first at all we need to download metamask and do the steps for configuration, i gonna skip that part because is not necessary to show how to do that, if you are curious you can check directly with this link : https://metamask.io/

So lets keep going : we gonna use the information showed before to connect, we know the :
- RPC connection
- Private key
- Setup.sol ( code and address )
- Target.sol ( code and address )

So first we need to connect to the RPC, we gonna use the same metamask to do that ;) :
- in this case the RPC url is  : `165.22.116.7:31092`

so we only need to put the http:// before the ip and ports , so looks like this :
- `http://165.22.116.7:31092`

![image](https://user-images.githubusercontent.com/37570367/227415482-deab8eb5-a644-40f6-bf8e-8494aed009dc.png)
![image](https://user-images.githubusercontent.com/37570367/227415630-5b8b5b87-0ebf-4e40-afb1-ea788cc90625.png)

And here happends some intersting, you cant add a new network if you dont know the Chain ID : 

![image](https://user-images.githubusercontent.com/37570367/227415912-a8edfff8-bc20-4ede-9724-ed6fdb7af80c.png)


but let show you a trick, when you dont know what chain id it is just put whatever value , in this case i gonna put `1`, and then click outside the field to launch the form error :
![image](https://user-images.githubusercontent.com/37570367/227416156-ea7a6854-ddb6-4399-afa4-a2323e7bd0d3.png)

Ok we know what is the chain id, not all the time works, but in this case yes, so after i put the new chain id i see this :

![image](https://user-images.githubusercontent.com/37570367/227416347-c62af0a3-c0e1-425e-9c3a-cd767639f8dd.png)

if you see the symbol can be channged too, but is not necessary for know, just save it

Ok so you gonna se something like this :

![image](https://user-images.githubusercontent.com/37570367/227416536-480544ae-86ae-4741-9508-fb59fbb5ea27.png)

Now we have the network connected with the random name for the crypto called `EXAMPLE`, but we dont have the right account

So we gonna import the account using the private key obtained from the netcat connection

- in this case the private key is : `0xb331b8bcd8882a6d755ee6517d9124feba3563d7d8c0b969c1de10837a21e456`

![image](https://user-images.githubusercontent.com/37570367/227416889-1acf0ee7-ef4d-40a2-aa3a-d47e74859bc7.png)

![image](https://user-images.githubusercontent.com/37570367/227416954-e53adbf9-d331-4ad6-be38-8e9a4ddeabe6.png)


If you gonna check the address is the same : 
![image](https://user-images.githubusercontent.com/37570367/227417032-48da758b-47a4-4b0f-a259-df0bd78a2f13.png)

![image](https://user-images.githubusercontent.com/37570367/227417101-3b76a0d7-960b-4795-9d02-c39a8ec2470d.png)


and now we have some Cyrptos to test too, so lets go know for the smart contracts and the IDE for see what we can do :

- Link to the IDE for smart contracts : https://remix.ethereum.org/

Lets create the two files : 
- Setup.sol
- Unknown.sol

The Steup.sol looks like this : 

{% highlight ruby %}

pragma solidity ^0.8.18;

import {Unknown} from "./Unknown.sol";

contract Setup {
    Unknown public immutable TARGET;

    constructor() {
        TARGET = new Unknown();
    }

    function isSolved() public view returns (bool) {
        return TARGET.updated();
    }
}

{% endhighlight %}


And the Unknown.sol :

{% highlight ruby %}
pragma solidity ^0.8.18;


contract Unknown {
    
    bool public updated;

    function updateSensors(uint256 version) external {
        if (version == 10) {
            updated = true;
        }
    }

}
{% endhighlight %}

![image](https://user-images.githubusercontent.com/37570367/227419091-1d0c4702-b171-4e22-b52a-f731064b1895.png)

So after adding the smart contracts, lets take a look of the code an what means :

![image](https://user-images.githubusercontent.com/37570367/227419591-4a41d661-d9b1-4c8d-ae85-aa42266f1d97.png)


So now lets check the Unknown.sol :

![image](https://user-images.githubusercontent.com/37570367/227419930-70cac507-2e45-40c6-bc60-384c52b13e27.png)

Ok so only wee need to interact with the smart contract and change the value for `10`


So we know what need to do , lets go to compile the smart contracts to interact with him, for this you have two ways to do it :

the first one is only pres `ctrl + S` ( saving ), and the second is go here and press compile : 
![image](https://user-images.githubusercontent.com/37570367/227420593-06ab550e-0adf-44cd-aa2e-7b2ce94f3084.png)

After this we can go to deploy and check if you are deploying the right smart contract :

![image](https://user-images.githubusercontent.com/37570367/227420765-4b90570c-c8d1-418f-8776-e32fe44b9939.png)

Before Start deploying you need to select the web3 inyected option here and select `Metamask` : 

![image](https://user-images.githubusercontent.com/37570367/227421066-7eb341ea-ea2f-47a9-a9f7-7d5d07934abb.png)

This gonna say to de IDE someting like  -> " use this RPC of my metamask to deploy the smart contract and use my account too "

ok now we can deploy, first gonna deploy the `Setup.sol` :

Note, be sure you are connected with metamask and the Remix IDE, let me show you

![image](https://user-images.githubusercontent.com/37570367/227422030-8111b5b0-c03e-4481-b11c-e5828ca80aea.png)

![image](https://user-images.githubusercontent.com/37570367/227422186-f40f9652-bc93-4fc6-93df-1464c2aed879.png)

So when is connected looks like this : 
![image](https://user-images.githubusercontent.com/37570367/227422324-159c13c2-d758-43f1-be55-0bcaf7660dc6.png)


Ok lets continue with the deploy, you have two ways to do it :

- deploy your smart contract ( new one )
- use the structure of the smart contract to instantiate another ( we gonna use this one )

but what exactly means ? , this means you gonna use the deployed smart contract and put in your structure of your solidity code, let me draw it for you :

![image](https://user-images.githubusercontent.com/37570367/227424094-904cd28f-9460-4ba2-9d0e-1051230a19b1.png)

Ok now lets add the address, note : be sure is the same structure ( same file .sol ) :

![image](https://user-images.githubusercontent.com/37570367/227424301-c9433135-7bd4-4117-b402-0a8fdfabed11.png)

![image](https://user-images.githubusercontent.com/37570367/227424397-f73d18c6-f5fa-4c00-bab8-9412236c75ca.png)

After this you gonna see this below : 
![image](https://user-images.githubusercontent.com/37570367/227424496-2db7bde5-5de2-4216-ac9b-8f607329c893.png)

We can expand and see this, ( you can press the blue buttons to interact with the smart contract ) :

Note : The Button has colors :
- the blue buttons are `public view` ( This mean dont have cost per interaction )
- the orange buttons are `writing functions``( This mean has cost per interaction )
- the red buttons are `payable functions` ( This means you need to send crypto to interact with him ), in this challenge dont have red buttons but is a good idea to know it 

So do the same with the Unknown.sol, be sure you are selected the same smart contract that you wanna instantiate, and the right address :

![image](https://user-images.githubusercontent.com/37570367/227424860-e5247115-3be5-4a45-94a0-af7e2ad0372f.png)


Ok so now just lets interact with him and get the flag :

![image](https://user-images.githubusercontent.com/37570367/227425048-bdd9ad28-2438-452a-999c-30d639dccde5.png)

When you change the value for `10` and press the orange button this gonna popup the metamask for the intreaction with the smart contract,
so here you need to confirm the transaction and the values gonna change : 
![image](https://user-images.githubusercontent.com/37570367/227425361-ec3931b1-0b81-49cf-bff9-1f6c2a3d23d3.png)


After the transaction complete, you can re-check the values and ... magic, you do the challenge

![image](https://user-images.githubusercontent.com/37570367/227425608-48ab2cef-12e2-4f41-8407-356d44a88987.png)


now we need to reconnect to the server using netcat and get the flag : 

![image](https://user-images.githubusercontent.com/37570367/227425748-2a6d7c32-3ba4-4d42-8584-725bbefeaa7b.png)


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
