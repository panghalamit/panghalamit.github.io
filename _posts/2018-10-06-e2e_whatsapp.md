---
layout: post
title: "How it works?: Whatsapp's E2E Encryption"
excerpt: "How does signal protocol work?"
date: 2018-10-06
tags: [whatsapp, Signal Protocol, cryptography, security]
comments: true
---

You must have heard about WhatsApp using end to end encryption. In layman's words, every message that you sent to your friend, is encrypted on your device, this encrypted message passes through network and bunch of servers and reaches your friend's device, and finally it is decrypted on friend's device. So as long as underlying cryptography is intact, you can be assured that no one else other than your friend knows about your [dirty little secret](https://www.youtube.com/watch?v=gPDcwjJ8pLg). Is it as simple as it sounds?, No. Purpose of this post is to give you peak  of what is actually happening behind the scene. 

## Features for a secure messaging system

So what possible features such system should have? I will introduce a few characters in the system. I will chose Ankita, Bud and Mayank. (Cryptography literature uses Alice, Bob and Mallory. Name are chosen based on their roles). Ankita and Bud wants to exchange messages. Mayank is evil, and he wants to listen on the conversation between Ankita and Bud, and possibly want to send messages to Ankita acting as Bud and to Bud acting as Ankita. We want to our system to have following properties:  

1. Confidentiality : Mayank can't know what messages Ankita and Bud send to each other.
2. Integrity : If Bud receives a message from Ankita, He can check if the message was modified by Mayank on the way.
3. Authenticity : When Bud receives a message from Ankita, He can be sure it is from Ankita and not from Mayank.

Another feature that we might want is 'deniability', that is if someone recovers, Ankita's or Bob's old messages in future, it can't be linked to them and they can deny having send that message. 

1) could be achieved using encryption/decryption mechanism. 2) and 3) could be achieved through MACs and Digital Signatures. These primitives require secure exchange of some secret (In case of Symmetric Key Cryptography) or establishment of each other's public information (case of public key cryptography). Public Key Cryptography is nice in sense that you dont need to share a secret with someone you want to send message to, as long you know each other's public key or (identity). Problem with that approach is that Ankita need to be assured that she actually has Bud's public key and its not of Mayank's and so She can securely communicate with Bud by encrypting the messages with Bud's public key. Only Bud, who has the corresponding private key can decrypt the message. Similarly , In case of symmetric key cryptography, Mayank and Ankita need a mechanism to share the secret (May be meet in person and exchange), which they will use for encryption/decryption. In former case, it is okay if Mayank learns about Ankita and Bud's public key (It's Public).       

## WhatsApp E2E Encryption protocol?

Whatsapp uses open source [Signal Protocol](https://en.wikipedia.org/wiki/Signal_Protocol) developed by ['Open Whisper Systems'](https://signal.org) (They have their own messaging application, Signal). Signal Protocol uses primitives like [Double Ratchet Algorithm](https://en.wikipedia.org/wiki/Double_Ratchet_Algorithm), prekeys, Triple [Diffie Hellman](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange) Handshake, [Curve25519](https://en.wikipedia.org/wiki/Curve25519), [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) and [HMAC_SHA256](https://en.wikipedia.org/wiki/HMAC).

We will define these primitive before putting them all together to understand Signal Protocol.

## Understanding Primitives

### Prekeys
These are Curve25519 key pairs generated on device during install time. There is one Signed Prekey Pair and several one time prekey pairs. The Identity Public key and the Public keys of the prekey pairs are signed by a long term Identity Secret Key(Curve25519 private key correspond to Identity public key) and sent to server during registeration. Server stores these keys along with Identity Public key.

### Curve25519
Elliptic curve used as part of Diffie Hellman Key Exchange Protocol. I will discuss Elliptic Curve Cryptography in separate post. Its security is based on difficulty of discrete logarithm problem in Large Finite Groups. 

### Diffie Hellman Key Agreement
Diffie Hellman Key Agreement or Triple Diffie Hellman Handshake allows two parties to have a shared secret over public channel (Mayank is able to listen to message exchanged by Ankita and Bud). It is based on arthmetic modulo large prime p and it basically works as follows,

0. Ankita and Bud agree on protocol parameters, large prime p and generator g. 'g' is the [generator](https://en.wikipedia.org/wiki/Primitive_root_modulo_n) of multiplicative group 'modulo p'. Generator is between 1 and p-1 and has this nice property, that every element in [1, p] can be represented as 'g^k mod p' where k is in [0,p-2]. 
1. Ankita selects a random number 'x' between '1' and 'p-1' , her private key.
2. Bud selects a random number 'y' between '1' and 'p-1', his private key.
3. Ankita sends Bud, 'x_p = g^x mod p' , her public key.
4. Bud sends Ankita 'y_p = g^x mod p', his public key.
5. Ankita computes 'ss_a = y_p^x mod p'
6. Bud computes 'ss_b = x_p^y mod p'

After completion of protocol Ankita and Bud have a shared secret 'ss_a = ss_b = g^(xy) mod p' .

Mayank on the network sees 'x_p' and 'y_p', and it is computationaly infeasible for him to determine shared secret without knowledge of 'x' or 'y' .
We assume here that Mayank only see the messages on channel between Bud and Ankita but doesnt temper them.

### Extended Triple Diffie Hellman Key Agreement (X3DH)
[X3DH]("https://signal.org/docs/specifications/x3dh/") is an extension of above protocol for asynchronous setting. Imagine Ankita wants to establish a shared key with Bud to send encrypted message to him. Above agreement works smoothly when Bud is online. What if Bud is offline?. She would have to wait for Bob to come online. 
Also, In above key agreement, Ankita and Bud have no way to determine if they are talking to each other. They may be both talking to Mayank thinking they are talking to each other. The protocol might end up as Ankita and Mayank sharing a secret, Mayank and Bud agreeing on a key. 
X3DH solves both these problems using a trusted third party and Prekeys. Ankita and Bud register signed prekeys (Signed using their long term private keys) on a trusted server. Each time one of them one wants to establish a shared secret with other, the former fetches the signed prekey bundle of latter. Lets assume Ankita is sender.

Now, Ankita does her part, performs DH Operation (raising a public key with a private key, operation 5 in above) 3 or 4 times (depending on prekey she fetched from server). These operations are between,
1. Ankita's long term private key and Bud's signed prekey.
2. Ankita's ephemeral private key (from key pair generated specifically for this exchnage and deleted afterwards) and Bud's signed prekey.
3. Ankita's ephemeral private key and Bud's long term public key.
4. (if prekey bundle  has a one time public key) Ankita's ephemeral private key and Bud's One time public key.
<figure>
    <a><img src="/assets/img/X3DH.png"></a>
    <figcaption>X3DH</figcaption>
</figure>
The outputs from above steps are combined used as a key material i.e. master secret. These keys derived from master secret are used in Double Ratchet decribed below  to send subsequent encrypted messages to Bud, all of them include Ankita's ephemeral public key, long term identity key and information on which of the Bud's one time public key is used ( in case it is in step 4.)are included in header (plaintext). This ends x3dh from Ankita's side. The messages received by Bud if he is online or can be stored on server where Bud can fetch it later. When Bud receives the message later, he can derive the same master secret by using his private keys and Ankita's public keys in the message header. 
This schemes allows Ankita and Bud to authenticate each other and derive shared secret to be used as key material. 

### HMAC_SHA256
It is keyed cryptographic hash function. Apart from the value to be hashed, this function also takes input a key. Unlike hash function which are easy to compute for a given input, keyed hash function require knowledge of the key. Here it is used as a key derivation function and MAC.

### Double Ratchet 
 'Ratchet' is name of a device that moves only in one direction. Double Ratchet uses two cryptographic Ratchets, i.e., deriving new keys from current keys and moving forward, while forgetting old keys. The two Ratchets used are Diffie-Hellman ratchet and symmetric ratchet. Each time a Diffie Hellman ratchet move forward, a secret is established between sender and receiver using Diffie Hellman described above, and the secret is used to derive two new keys (root key and chain key).  Symmetric Ratchet moves forward by using a  Key Derivation Function using chain key to generate a message key to encrypt message to be sent, and a chain key to be used for next ratchet movement. This 'Ratcheting' provides a useful property to protocol known as forward secrecy, i.e, if Ankita or Bud comprise their keys in future, their previous messages cant be comprised (decrypted), as long as ratchet works as expected (old keys are deleted).    


## Putting Everything Together

I have attempted to summarize the primitives used in the protocol which could be difficult to digest all at once. Assuming the primitives work as expected, following steps will describe the working of protocol in much simpler language.  These steps occur when Ankita wants to send message to Bud using Whatsapp,

1. Registeration of Clients with whatsapp server (Mobile apps on Ankita's and Bud's phone). It includes registering signed prekeys.
2. Session setup by Ankita using x3dh
3. In step 2, Ankita calculates master secret and Using DH Ratchet step, derives  a root key and a chain key to be used Symmetric Ratchet.
4. Ankita derives a message key and next chain key using the chain key using  symmetric Ratchet.
5. She encrypts her message using message key ( [AES256 in CBC mode](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) ). 
6. Everytime she sends a message her Symmetric Ratchet moves forward.
7. Everytime she receives a response from Bud, which includes a new public key in header, advances her DH Ratchet, calculates a new root key and a new chain key.

On Bud's side,
1. When he retrives first message from Ankita, he completes the session setup by deriving the master secret, root key, chain key and the message key. 
2. Uses messages key to decrypt.  
3. If he wants to send message, he generates a new ephemeral pair, moves his DH ratchet forward using the root key and ephemeral private key and replaces hi chain key and message key.
4. New message is derived from chain key and message is encrypted.
5. encrypted message is sent with Bud's ephemeral key in header. 


In the image below (Assuming it's Ankita's Phone), message in white boxes are one received and greens one are one sent. Each pair of continous boxes can help you visualize ratchet movements.
1. White -> White :  Symmteric Ratchet of Bud moves forward by 1 step (new message key and next chain keys are derived)
2. Green -> Green : Symmetric Ratchet of Ankita moves forward by 1 step (she derives new message key and chain key)
3. White -> Green : Ankita's DH Ratchet moves forward by 1 step(she derives new root key and chain key)
4. Green -> White : Bud's DH Ratchet moves 1 step forwards (He derives new root key and chain key)

<figure>
    <a><img src="/assets/img/conv.png"></a>
    <figcaption>Screenshot of Ankita's Phone</figcaption>
</figure>

### Media Attachements and Files
Media files are encrypted and uploaded to blob store. When Ankita sends an image or video to Bud, the pointer to the location of encrypted image or video file in blob store is encrypted and  sent using above scheme.

### Group Messages
1. Whenever a group member sends his/her 1st message, he/she generates a sender key which is distributed to all group members using one-to-one protcol described above.
2. For subsequent messages, messages are encrypted using new message key derived by symmetric Ratchet (this is different from one used in one-to-one).
3. Since every member has other member's sender key, they can move ratchet corresponding to sender to decrypt their messages.
4. Whenever a member leaves the group, sender keys are renegotiated (step 1).

### Calling
Calling is synchronous/real-time. Whenever Ankita  calls bud, she generates a random [SRTP](https://en.wikipedia.org/wiki/Secure_Real-time_Transport_Protocol) secret.
This secret is send to Bud using pairwise system. If he responds to call, encrypted session begins.

### Miscellaneous
Apart from the end to end encryption, the the channel between the whatsapp client and whatsapp server is secure. Plaintext Headers are not visible to anyone listing on channels. This is server's extra wrapping over Ankita's already wrapped gift(message) to Bud.


## Wrapping Up
So you might have heard people saying that Whatsapp is probably reading our messages, As you happen to see ads of things on your facebook feed which you happen to discuss on whatsapp chats. Lets cover some possibilities and conspiracy theories.
1. If you trust that whatsapp's end to end encryption is actually implemented as per signal specification, then no way they are able to read your chats. Unlike Signal application, whatsapp's code is not open source so you can't really confirm.
2. Assuming signal protocol is implemented correctly, whatsapp servers know which whatsapp user is interacting with which user, how frequently, how recently. Same is true for signal app's servers. If you have phone mobile registered on facebook and is the one use for whatsapp. They could use this information to associate your friend's activity on facebook and serve you relevant ad which might come as surprise to you. 
3. They might have enough information from other channels and such an ad is merely an coincidence.



## References and Recommended Readings

1. Signal Developer [Docs](https://signal.org/docs/)
2. Signal [Blogs](https://signal.org/blog/)
3. Whatsapp [whitepaper](https://www.whatsapp.com/security/WhatsApp-Security-Whitepaper.pdf) 
4. Diffie Hellman simple [explanation](https://www.youtube.com/watch?v=YEBfamv-_do&feature=youtu.be) 
5. Off the record messaging, [OTR](https://en.wikipedia.org/wiki/Off-the-Record_Messaging) 
