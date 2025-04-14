# Introduction to cryptography - Diffie-Hellman Key Exchange
It has been a while since I wrote an introduction to cryptography blogpost! My intention is bridging a big gap - namely the Diffie-Hellman Key Exchange algorithm.  
While this is going to be a short blogpost, I believe it's going to be worth your while.  

## Motivation and background
Historically, cryptography has spent most of its time around *Symmetric cryptography*.  
I have mentioned what Symmetric cryptography is [in a past blogpost](https://github.com/yo-yo-yo-jbo/crypto_vigenere/) but in essence it means that encryption and decryption use the same `key`.  
There are many examples of Symmetric ciphers, including really old ones like [Caesar cipher](https://github.com/yo-yo-yo-jbo/crypto_terminology/), which developed with time to more sophisticated ones such as [Vigenere cipher](https://github.com/yo-yo-yo-jbo/crypto_vigenere/).  
Even today modern cryptography relies on Symmetric ciphers, with [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) being a prime example of such a cipher.  
