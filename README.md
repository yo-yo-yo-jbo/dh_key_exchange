# Introduction to cryptography - Diffie-Hellman Key Exchange
It has been a while since I wrote an introduction to cryptography blogpost! My intention is bridging a big gap - namely the Diffie-Hellman Key Exchange algorithm.  
While this is going to be a short blogpost, I believe it's going to be worth your while.  

## Motivation and background
Historically, cryptography has spent most of its time around *Symmetric cryptography*.  
I have mentioned what Symmetric cryptography is [in a past blogpost](https://github.com/yo-yo-yo-jbo/crypto_vigenere/) but in essence it means that encryption and decryption use the same `key`.  
There are many examples of Symmetric ciphers, including really old ones like [Caesar cipher](https://github.com/yo-yo-yo-jbo/crypto_terminology/), which developed with time to more sophisticated ones such as [Vigenere cipher](https://github.com/yo-yo-yo-jbo/crypto_vigenere/).  
Even today modern cryptography relies on Symmetric ciphers, with [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) being a prime example of such a cipher.  
In our current modern world, Symmetric ciphers are very reliable in terms of security - with a very short key length (generated with sufficient randomness!) we can rely on an attacker not being able to decrypt or break the cipher (in fact, even with Quantum computers you won't be able to do much - more on this [here](https://en.wikipedia.org/wiki/Grover's_algorithm) if you are curious).  
So, the problem is not the Symmetric ciphers we use, but how to exchange the Symmetric `key` - it's the chicken and the egg problem in some sense - to exchange the `key` with you need a cipher to begin with!  
Up till 1976 (or before, if you count military intelligence) there was no known way to exchange keys reliably and remotely (in an untrusted medium, that is).  
If you're thinking about my previously published [RSA blogpost](https://github.com/yo-yo-yo-jbo/rsa_math/) then know that RSA does solve the problem, but it was discovered much later.  
So, the motivation for us is to exchange a secret `key` created with sufficient randomness, without anyone in the untrusted medium being able to decipher it.

## Background: modular arithmetic and basic Group theory
I have discussed both modular arithmetic and basic Group theory [in a previous blogpost](https://github.com/yo-yo-yo-jbo/crypto_modular/) - I recommend reading those before.  
However, here's a refresher:
- We work within a frame of `modular arithmetic`, which means we only look at the remainder of our common operations of addition, substraction and multiplication.
- Division can be achieved if and only if our modulus is a `prime number`, with something called the [Extended Euclidean Algorithm](https://en.wikipedia.org/wiki/Extended_Euclidean_algorithm).
- The set of numbers `{ 1, 2, ... p-1 }` form a `multiplicative Group`, which means addition, substraction, multiplication and division work "nicely".
- Our Group also has something called a `Generator`, which is a member of that group that you can multiply by itself over and over again that iterates the entire Group.
- In practice, we will also look at Generators that do not generate the *entire* Group, but only a *Subgroup* instead. More on that later.

To give an example, let's take `p=7` (i.e. `(mod 7)`) - the Group members are `{ 1, 2, 3, 4, 5, 6 }`.  
The number `3` is a *generator* - let's multiply it by itself over and over again:
```
3^1 = 3 = 3                  (mod 7)
3^2 = 3*3 = 9 = 2            (mod 7)
3^3 = 3*3*3 = 27 = 6         (mod 7)
3^4 = 3*3*3*3 = 81 = 4       (mod 7)
3^5 = 3*3*3*3*3 = 243 = 5    (mod 7)
3^6 = 3*3*3*3*3*3 = 729 = 1  (mod 7)
```

(By the way, you do not need to calculate `3^6` directly - you can just look at `3^5 = 5 (mod 7)` and multiply by 3: `3^6 = 3*5 (mod 7) = 15 (mod 7) = 1 (mod 7)`)  

Note how the generator created all of the elements in our Group, but with a "weird" order - this happens due to modular arithmetic. In the next section we'll take advantage of that.  
If we take a different value (let's say `g=4`) then we do not get all elements, but we still get a `Subgroup`:
```
4^1 = 4        (mod 7)
4^2 = 16 = 2   (mod 7)
4^3 = 64 = 1   (mod 7)
```

I hope all of these definitions and arithmetic operations are clear - if not, please read my previous blogposts again.

## Diffie-Hellman: key exchange via exponentiation
Let us assume we have a well-known large prime number `p` and a generator for the group which we will mark as `g`.  
In our illustration we'll say `Alice` and `Bob` want to share a secret, and do that over an untrusted medium (let's say, the internet).  
Let's just jump right into it:  
1. `Alice` and `Bob` agree on the numbers `p` and `g`, in which `p` is a very large prime number and `g` is a generator.
2. `Alice` creates a random number `a` and keeps it to herself.
3. `Alice` sends `Bob` the number `g^a (mod p)`.
4. `Bob` receives `g^a (mod p)` and does something similar - he generates a random secret number `b` and keeps it to himself, but sends `Alice` `g^b (mod p)`.
5. `Alice` takes the input from `Bob` and raises it to the `a`-th power, effectively calculating `(g^b)^a (mod p) = g^(ba) (mod p) = g^(ab) (mod p)`.
6. `Bob` does the same thing but raises `Alice`'s input to the `b`-th power, calculating `(g^a)^b (mod p) = g^(ab) (mod p)`.
7. Now `Alice` and `Bob` have a joint key `g^(ab) (mod p)`, and they can use it for further communications (e.g. as a secret key for an `AES` Symmetric cipher).

Here's a nice illustration too:

```
Alice                                                      Bob
-----                                                      ---
                              Agrees on: p, g
                         <------------------------->

                              Sends: g^a (mod p)
(generates: a)           -------------------------->

                              Sends: g^b (mod p)
                         <--------------------------       (generates: b)

(key = g^(ab) (mod p))                                     (key = g^(ab) (mod p))
```

Let us now imagine an evesdropper in the middle - what can they do?  
One important aspect here is that `g^a (mod p)` appears just as a number. Attacker knows `g` and `p`, but magically cannot deduce `a` efficiently.  
In real numbers, we use [Logarithms](https://en.wikipedia.org/wiki/Logarithm) to solve the problem of finding the value of `a`, but those do not have an easy-to-calculate modular counterpart.  
What Diffie-Hellman exploits here is called the [Discrete Logarithm problem](https://en.wikipedia.org/wiki/Discrete_logarithm), which is computationally believed to be difficult for a computer to solve, assuming a very large value of `p`.

### Some notes on parameter selection
The most obvious danger is that `g` generates a really small number of elements in the Group.  
Therefore, there are some interesting notes I'd like to mention on the parameter selection (`p` and `g`).  
1. The prime `p` must be large to make the Discrete logarithm problem hard; it'd usually be 2048 bits or more. I've written [a blotpost](https://github.com/yo-yo-yo-jbo/generating_random_primes/) on how to generate random prime numbers, you should check it out for details on how that's achieved.
2. Often, `p` is a *safe prime*, which is a prime where `p = 2q + 1` and `q` is also prime. This ensures the multiplicative Group modulo `p` has a large prime-order subgroup, helping avoid small subgroup attacks.
3. The generator `g` should be a *primitive root* modulo `p` — meaning that for every number `x` in the Group, there exists an integer `k` where `g^k = x (mod p)`.
4. More practically: `g` is often selected to generate a large cyclic subgroup of order `q` (the same `q` that was used to define `p` if `p` is a safe prime).
5. In practice, `p` and `g` are not just randomly selected, but only selected from a standard agreed-upon list (e.g. [RFC 3526](https://www.ietf.org/rfc/rfc3526.txt) has some of those).

That's quite important - when I learned about Diffie-Hellman I thought `g` is a generator of the entire multiplicative Group and that is *not* the case - it's a generator of a *sufficiently large* subgroup.
