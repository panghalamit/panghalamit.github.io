---
layout: post
title: "Demystifying Authentication"
date: 2018-09-24
excerpt: "How good is MFA?"
tags: [Authentication, Fido, Authenticator, Authy, digital, cryptography, security]
comments: true
---

## Notes on authentication

Authentication is a mechanism to guarantee that entities are who they claim they are, or that the information has not been modified by unauthorized parties. In this post I am talking about entity authentication which is also referred to as identification. Entity authentication techniques assures one party through acquistion of corroborative evidence, of both the identity of a second party involved, and that the second was active at the time evidence was created or acquired. For example, when you are trying to access your bank account online through your bank's website, You are asked to enter login credentials, your username and password, or may be some other information say some numbers on your credit/debit card. Bank identifies you through the credentials and this particular type of authentication is unilateral authentication where Bank is trying to identify you in order to give access to their services. Another simple example is you talking to your friend on phone, you and your friend identify each other through voice recognition. It is an example of Mutual Authentication. 

Any such protocol between two parties say Alice and Bob should achieve following objectives:

1. Alice is able to successfully authenticate herself to Bob, and Bob at the end of protocol accepts Alice's Identity.
2. Bob should not be able to use the exchange with Alice, to successfully impersonate Alice to an another party Carol.
3. Mallory should not be able to identify herself as Alice to Bob. The probability that this happens should be negligible.

Entity Authentication can be majorly divided into three categories:
1. __Something You Know__ (Knowledge Based): These include knowledge of a secret. The secret could be standard passwords, personal identification numbers (PINs) and the secret or private keys whose knowledge is demonstrated in challenge-response protocols.
2. __Something You Own__ (ownership): These could be the physical accesories that one might own. for example, magnetic stripe and chip cards, your smartphone, tempor resistant hardware storing key etc.
3. __Something You Are__ : This includes methods which make use of physical human characterstics and biometrics, such as handwritten signatures, fingerprints, voice, retina patterns, and even dynamic keyboarding characterstics.

Entity Authentication is generally used to facilitate access control to a resource or a service, where access privilege is linked to a particular identity. Most common method that is used is password based (belongs to 1st category) and also referred to as weak authentication. The password usually serves a shared secret between user and system. The way information about password is stored on the on the system, and how it is used later to verify the password entered by user determines the possible attacks on the scheme. The Most common way is to store [cryptographic hash](https://en.wikipedia.org/wiki/Cryptographic_hash_function) of salted password along with the salt.
The 'salting of password' means using adding a fixed size random [nonce](https://en.wikipedia.org/wiki/Cryptographic_nonce) to password. Salting makes [dictionary based attacks](https://en.wikipedia.org/wiki/Dictionary_attack) harder. What does it mean? Lets say salt is of size 12 bits, so an attacker creating offline dictionary attack would have store all 4096 possible variants of a single guess. If it is remote authentication, passwords are sent in cleartext over a secure channel. If channel is unsecure such schemes are also susceptible to replay attacks (even if you hash and send your password), where adversary learns about hash or cleartext when it is in transit and can use for future authentication. Passwords normally used are sequence of characters with additional minimum requirements placed by server for its strength. But users tend to set passwords with low entropy and hence are easy to guess. Check this [paper](http://www.jbonneau.com/doc/B12-IEEESP-analyzing_70M_anonymized_passwords.pdf) from Prof. Bonneau which analyzes millions of passwords used by yahoo users anonmyously and proposes several guessing hueristics. 
A better approach is to use passphrases which are easy to remember and have higher entropy than random strings of fixed length.
<figure>
    <a href="https://imgs.xkcd.com/comics/password_strength.png"><img src="https://imgs.xkcd.com/comics/password_strength.png"></a>
    <figcaption>on password strength</figcaption>
</figure>

## One Time Passwords
One Time Passwords provide timeliness guarantees which help authentication protocols to counter replay attacks. These protocols use one of following time variant parameters. 
1. Random Numbers: used in challenge-repsonse type authentication protocols where one party sends a newly generated nonce to other party and subsequent message are bound to this nonce. This demands for parties to store nonce during one instance of protocol execution
2. Sequence Numbers: This requires parties to maintain state and state is incremented with every execution.
3. Timestamps : Timestamps are included along with messages and messages that are not within a minimum threshold of local clock are discarded. This assume client and server clocks are synchronized and is vulnerable to adversial resetting of clocks.

In practice, One Time Passwords are used along with other password based (weak authentication) techniques, to confirm User's ownership of a device (a smartphone). This is most common 2FA (2 Factor) authentication in practice, out of band 2 factor authentication. The one time password generated at server is distributed to user's registered devices. It could be done by a phone call or SMS. SMS based methods have deprecated and has been discouraged to use by NIST as of June 2016. It is due to ability of SMSs to be intercepted at a large scale.  

[TOTP](https://en.wikipedia.org/wiki/Time-based_One-time_Password_algorithm) uses both timestamps and sequence numbers, is used in [Google Authenticator](https://github.com/google/google-authenticator), [Authy](https://authy.com/) and as part of 2 step verification in most of the web services. 

TOTP allows for users to generate one time passwords locally on their smartphone without network using TOTP applications.  For TOTP, you have one time set up phase and authentications. During set up, user registers with server, and server generates a secret which could be entered on TOTP application on smartphone. After this initial steup, you could just use otps generated on your device to login to server.

## Strong Authentication
Crypotgraphic Challenge Response based protocol typically fall under this category. The general idea is to enable Prover to prove his knowledge of the secret without revealing the secret itself to the verifier during protocol. This is done by providing a response to time variant challenge, this response is depends both on prover's secret and the challenge. Someone listening on the link between prover and verifier would not be able to use it for subsequent authentication as each time there would be a different challenge.
Challenge reponse protocols are based on Symmetric key techniques, Public-key techniques and zero knowledge based.
__Symmetric Key__ based techniques rely on secure exchanges of keys and secrecy of keys. There are different implementations based on different kind of time variant parameters as challenges. Usually involved encryption/decryption or keyed hashing of challenge strings.

__Public Key__ based techniques involve either decryption of challenge encrypted by public key or digitally signing of challenges to prove knowledge of the private key.

__Zero Knowledge Authentication__ based protocols provide Prover ability to prove his knowledge of secret key to verifier without revealing zero information about key in the process. These protocols use three steps. In step one, prover commits to a value by sending a commitment to verifier. Verifier sends a challenge to prover. Prover responds to the challenge using the secret and the value he commited in step 1. Finally, verifier check if response is valid by using commitment and Prover's public information. 

[FIDO](https://fidoalliance.org/fido2/) publishes standards for web services to use strong authentication. They encourage using public key based challenge response with an additional factor. Google's security key(tempor resistant usb key device) is compliant with fido standard. The typical flow is 
1. User registers on FIDO compliant web service (bank website), which prompts user to insert usb key and press a button.
2. New key pairs are generated locally, private key is on usb key and public key is registered on bank website.
3. Assuming browsers are FIDO Compliant, when accessing the website, a call is made which is same for all browser, which prompts user to use usb key.
4. Response is generated locally using the private key and send to website.
5. User can access the service if response is valid.

Instead of hardware usb key, other second factor devices say smartphone could be used. Keys could be stored locally on device and can be unlocked by binding device to user by using a another layer, say fingerprint scanner (biometrics) to unlock keys locally and send the response to desktop via bluetooth or NFC.

While this looks good, (No typing/remembering passwords) and safe against passowrd phishing, but is still susceptible to session hijacking (Once the user has logged in). 

Also, another kind of authentication that we havent talked about is called "Continuous Authentication". It usually works by monitoring user behaviour and access patterns during a session, and logging user out if unusual activity is detected. It is very difficult to get it right and contradicts usability if honest users are denied access.

A very _critical vulnerability_ in the authentication systems is the recovery mechanism deployed when a honest user loses access to his/her crendential. Here is where tradeoff is made between usability and security. Key recovery mechanism are usually exploited by using __social engineering__ techniques. 
Security in practice follows 'something is better than nothing' and try to minimize the attack surface while ensuring usability. 

## References
1. Handbook of Applied Cryptography, Alfred J. Manezes, Paul C. van Oorschot, Scott A. Vanstone 


