#Problem Description


At his last rally, Trump made an interesting statement:

>I know RSA, I have the best RSA.
>The more bits I have, the more secure my cyber, and my modulus is YUUUUUUUUUUUUUGE
>We don't believe his cyber is as secure as he says it is. 
>See if you can break it for us

#Overview

Determine the RSA plaintext given ciphertext, public exponent, and modulus

#Solution

*Note: the numbers used for this challenge, due to their size (~157000 digits), have been omitted from this writeup.  Please refer to the file titled best_rsa.txt for the numerical values.

Opening best_rsa.txt, we find an overwhelmingly large modulus n and ciphertext c, and a public exponent e (65537).  Without a private key d, the only option is to factor n, take the totient phi(n), and find the modular inverse of e with phi(n).  From first glance, this seems impossible - for how could we factor a coprime modulus of such magnitude in a reasonable amount of time?  However, upon closer inspection, we find something interesting about n: it ends in ...4619598388671875.  Thus, we can conclude that 5 is a factor of the modulus.  Despite this, we don't know whether or not the modulus is composed of exactly two primes, and it seems unlikely given its size.  Just to be sure, we compute n mod i for every i between 1 and 100 by the following python snippet: 

    >>> print([n%i for i in range(1,100)])
    >>> [0L, 1L, 0L, 3L, 0L, 3L, 0L, 3L, 0L, 5L, 0L, 3L, 0L, 7L, 0L, 3L, 0L, 9L, 0L, 15L, 0L, 11L, 0L, 3L, 0L, 13L, 0L, 7L, 0L, 15L, 0L, 3L, 0L, 17L, 0L, 27L, 0L, 19L, 0L, 35L, 0L, 21L, 0L, 11L, 0L, 23L, 0L, 3L, 0L, 25L, 0L, 39L, 0L, 27L, 0L, 35L, 0L, 29L, 0L, 15L, 0L, 31L, 0L, 3L, 0L, 33L, 0L, 51L, 0L, 35L, 0L, 27L, 0L, 37L, 0L, 19L, 0L, 39L, 0L, 35L, 0L, 41L, 0L, 63L, 0L, 43L, 0L, 11L, 0L, 45L, 0L, 23L, 0L, 47L, 0L, 3L, 0L, 49L, 0L]

We find that n is divisible by every odd integer in that range, meaning that the modulus is composed of many prime factors and that we need to compute phi(n) in a manner other than the traditional (p-1)(q-1) for dual-prime moduli.  From this point, I didn't trust that Python would be able to numeric operations with these numbers, so I transitioned to Mathematica instead. Luckily, Mathematica has a built-in totient function.  Taking advantage of this,  we compute:
    
    phi = EulerPhi[n]

Once we have phi(n), we must compute d by finding the modular multiplicative inverse of e and phi(n).

    e = 65537
    d = PowerMod[e,-1,phi] (* -1 denotes inverse *)

We now have our tuple, (c,d,n), and we proceed with the normal RSA decryption process: m = c<sup>d</sup> mod n.  

	m = BaseForm[PowerMod[c,d,n],16] ( * This took about 3 hours, but could be sped up with usage of the Chinese Remainder Theorem * )


It looks as if m begins with 474946383961 - that's the GIF magic number.  Thus, we write m into a GIF file:
	
	BinaryWrite["flag.gif", IntegerDigits[m, 256]]
	Close["flag.gif"]

We see a picture of Trump with the Chinese flag in the background, and the flag l33tly displayed:


![flag](https://github.com/Alaska47/HackTheVote-2016-Writeups/blob/master/crypto/250-Best-RSA/flag.gif)
    
#Flag

    flag{s4ved_by_CH1N4_0nc3_aga1n}


Yes, it is a pun - a pity we didn't use CRT :(
