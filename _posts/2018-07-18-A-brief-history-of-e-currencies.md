---
layout: post
title: "A brief history of e-currencies"
date: 2018-07-16
excerpt: "Digital cash system attempts before bitcoin."
tags: [cryptocurrencies, bitcoin, history, bitcoinbook.cs.princeton.edu]
comments: true
---
This blog post is summary of more detailed chapters about the same topic in the [book](https://bitcoinbook.cs.princeton.edu/)

## Traditional Financial Systems

### Barter -> Cash -> Credit

* Barter : difficult to make the coordination where everyone finds what he needs based on what he/she is willing to exchange
* Credit Based System : Goods and Services are received in exchange of favour/promise. Receiving Party has a debt, which it needs to settle in future. No way to quantify the favour. Risk is introduced in the system.
* Cash Based system : Goods and Services are received in exchange of Cash. Good and Services are quantified using cash. System need to bootstrapped with cash to work.
* Cash+Credit : Cash is used to quantify debt/favour in credit based systems.

### Ideas in context of an online p2p file sharing system
* Addressing "Freeloaders", downloading but not sharing files
* [Mojonation](https://en.wikipedia.org/wiki/Mnet_(peer-to-peer_network)) and Karma, solves using virtual cash or karma, receive some cash on sending some file, and spend that cash upon receiving.
* These are ancestors of current protocols like BitTorrent, Tahoe-LAFS

Online payment systems existing today can be grouped into credit and cash based groups. While credit cards transactions account for majority of payments online.
How that works? Well, you send your credit card details to companies like Amazon and they send your details to system which includes and not limited to Banks, processors, credit card companies. Another architecture is similar to paypal, where there is an intermediatory between buyer and seller, buyer sends credit card details to intermediatory which approves the transaction, notifies seller. Intermediatory settles balances with seller at the end of the day. With this architecture you avoid a sending your credit card details to seller which could be a security risk. It could also improve privacy, if you dont have to give your identity to seller. Drawbacks are complexity and that seller and buyer must have same intermediatory. 
Early web of 90s when privacy concerns were high amount users, technologies involving intermediatories were of great interest. Let go through some of the early attempts.

### FirstVirtual 1994
* First company with virtual office, employees spread across country communicating over internet
* Similar to paypal but communication using email
* Buyer and seller both send transaction detail to intermediatory and bills customer and on confirmation, seller is notified
* 90 days to settle consumer dispute

### [SET](https://en.wikipedia.org/wiki/Secure_Electronic_Transaction) ARCHITECTURE
* You send your view of transactions encrypted to Seller
* Seller Forwards encrypted view with its own view to an intermediatory
* Intermediatory decrypts your view and verfies if both views are same and approves Transaction
* Developed by VISA and Mastercard along with Tech giants during that time

### [CYBERCASH](https://en.wikipedia.org/wiki/CyberCash)
* Implemented SET Architecture for credit card processing
* Also had a digital cash product, CyberCoin for micropayments
* Upper cap of 10 dollars worth of CyberCoin
* Was affected by y2k bug - lead to double billing
* Bankrupt in 2001 and IP was acquired by VeriSign which was resold to Paypal

Why SET didnt work ?

Problem of certificates. All users, merchants had to get digital certificates from company like VeriSign which was inconvenient for users.

Moving on from Credit to Cash. Cash offers additional advantages over credit :

* Anonymity : Credit based systems are tied to banks and hence they can track your spendings
* Offline Transactions

## E-cash systems 
Consider this simple physical analogy of a cash system. I issue a piece of paper with my signature on it which says "Bearer of this note can redeem X dollars by presenting it to me". If people trust that I will keep my promise and my signature is unforgeable, these papers can pass on as bank notes. Now lets assume that people can make perfect copies of these notes and it is difficult to distinguish between original and copy. They can make copies and pass on to different people. This is called double spending problem. 
How do we solve it?
Well we can use unique serial numbers i.e, I write unique number on the paper when I issue it and record it in my ledger. So when someone spends it, the receiver will call me and give the serial number and I will mark it as spent in my ledger. Receiver can later come to me and get new notes with unique serial number in exchange of spent ones. 
Do you see the problem with this? 
We lost the feature that cash offers, anonymity. I can keep track of all the places you are spending your money. Well how do we solve it then?

### Chaum's Proposal and Further Improvements
Chaum's idea was to use [blind signatures](http://www.hit.bme.hu/~buttyan/courses/BMEVIHIM219/2009/Chaum.BlindSigForPayment.1982.PDF). That is when I issue you note you choose a random number such that I cant see it and I sign it. Choosing a number such a way that it is highly unlikely that it has been picked before. This guarantees Anonymity and prevents double spending.

Chaum with collaboration with other cryptographers Fiat and Naor came up with offline [e-cash](https://link.springer.com/chapter/10.1007/0-387-34799-2_25) scheme. This clever idea revolves around detecting double spending rather than protecting it. Which could be then followed by attempts to recover or punish the perpetrator. If you think about it, it is what traditional finance system do, i.e., if you write someone a personal cheque, they have no guarantee that money is in their account but they can come after you if cheque bounces. If such offline were widely adopted, double spending would be recognized as a crime.
Their scheme worked like this: every digital coin issued to you encodes your identity in way that no one except you can decode it, not even the bank. So every time you want to spend it, recipient ask you to decode a random subset of your encoding and they keep a record of it. This information alone doesnt reveal your identity but if you were to double spend it, both recepients can go to bank with their subset of decodings and bank can put these together to identify you and take actions accordingly. You cannot frame someone as double spender, i.e try to spend coin upon receipt without going to the bank. Because you wont be able to decode random subset upon request when you try to spend it.

### [Digicash](https://en.wikipedia.org/wiki/DigiCash)
It is company that came out of Chaum-Fiat-Naor scheme founded by Chaum in 1989. The cash in the system was Ecash. It works the following way
* Clients are anonymous but Merchant's aren't. Merchants have to return coin to the bank as soon as they receive them.
* Coins can't be split, so banks issue different kind of denominations, for you to pay for exact amount  of a transactions.
* You click on payment link, it takes you to digicash website which creates a reverse connection to your computer and starts digicash software on it. you apporve the transaction and send the money.

why did it fail ?

Banks and merchants didnt adopt it. since there werent many merchants that accepted e-cash, users didnt want it either. It also didnt support user-user transactions.

Digicash also experimented with tamper resistant hardware to prevent double spending. Sort of chip card and a device which keep track of your balance which decreases when you spend and increases when load it with money. This counter was was very difficult to tamper with physically. 

### [Mondex](https://en.wikipedia.org/wiki/Mondex) system
* A smart card and a Wallet unit.
* Either could be loaded with cash.
* User-User swap happened using wallet, put money into wallet by insertng card and receiver put their card in the wallet and load money onto card.
* Anonymous way to exchange digital cash.

Issues with Mondex : slow and clunky, and retailer hated having separate payment terminals

However, the smart card technology prevailed. Not for preventing double spending but to be used as authentication mechanism. That is you know the pin associated with your account.


## Minting Money in digital systems
To create a free-floating digital currency which is likely to acquire real value, you need something which is scarce by design. This is why gold/diamonds have been used as a backing for money. 
In digital ecosystems, one way to achieve scarcity is by requiring to solve a computational puzzle that takes some time to crack to mint digital money.

### [Dwork and Naor Scheme](https://dl.acm.org/citation.cfm?id=705669) (1992) 
* solution to combat email spams
* reuqires you to solve mini computational puzzles every time your sending an email
* Heavy cost for spammer who is try to send million emails 

### [Hashcash](https://en.wikipedia.org/wiki/Hashcash) (1997)
* Proposed by Adam Back
* Puzzles specific to email: based on sender, receiver and content and time which makes it impossible for spammers to reuse puzzle solutions
* Puzzle independence
* difficulty adjustment by recipients
* Bitcoin uses similar puzzles with minor variation

Why Hashcash didnt catch on ?

Email spams arent that big of a problem. Good spam filters in place today.

## Secure Timestamping
The goal of timestamping is to give an approximate idea of when a document came into existense. It could convey the order in which they are created. The security property ensures the timestamps cant be changed after.
These were ideas that might have lead up to blockchain, ledger in which transaction are securely recorded.

### Haber and Stornetta's [scheme](https://www.anf.es/pdf/Haber_Stornetta.pdf) 
* Timestamping service which receives the document to timestamp
* Service creates a certificate by signing the document with current time and a link/pointer  to previous document.
* Each document ensures integrity of the contents of Next one,

Later proposals improve on efficiency by grouping documents in blocks and linking the blocks together in a chain. Within each block, link documents in a tree like structure. This data structure forms skeleton of Bitcoin's blockchain. 

## Combining computational puzzles and secure timestamping

### [B-money](http://www.weidai.com/bmoney.txt)
* Proposed by Wei Dai in 1998
* Money is minted using hashcash like puzzles
* Solutions include solution to previous solution i.e, new coin is linked to previously minted coin
* p2p system with every node having its own ledger
* Solution to puzzles is propogated and everyone increment solver's balance in their ledger
* Similarly transactions are propogated

### [Bitgold]()
* Proposed by Nick Szabo in series of blog posts
* Similar to b-money

Both B-money and Bitgold doesnt reveal in detail about how to solve issues like resolving disagreements with ledger.  

In Next series of blogs, I will write about cyptographic methods, consensus algorithms used in bitcoin and other altcoins. I also plan to write some dev-friendly blog-posts for getting started with smart-contract on various platforms.
