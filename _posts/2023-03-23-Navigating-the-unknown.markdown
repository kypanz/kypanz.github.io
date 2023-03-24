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
- Metamask
- Remix IDE
- Smart Contracts
And we gonna interact with the smart contracts mor easy and more simple to understand what happend really in a general level

ok so first at all we need to download metamask and do the steps for configuration, i gonna skip that part because is not necessary to show how to do that, if you are curious you can check directly with this link : 

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
