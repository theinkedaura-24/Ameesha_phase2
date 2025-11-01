## RSA Oracle

In this challenge, we interact with an RSA encryption/decryption oracle hosted on titan.picoctf.net. The oracle allows us to encrypt and decrypt messages under the same RSA key, which opens the door for a chosen-plaintext or chosen-ciphertext attack to recover the secret key or plaintext.

## Solution:

Understanding the Oracle

When connecting using Netcat:
```

nc titan.picoctf.net 54779


the server prints:

*****************************************
****************THE ORACLE***************
*****************************************
what should we do for you? 
E --> encrypt D --> decrypt.


Choosing E allows us to encrypt a number, while D allows us to decrypt one.

Example:

E
enter text to encrypt (encoded length must be less than keysize): 1
1
encoded cleartext as Hex m: 31
ciphertext (m ^ e mod n): 4374671741411819653095065203638363839705760144524191633605358134684143978321095859047126585649272872908765432040943055399247499744070371810470682366100689


Recovering the Modulus N

RSA encryption works as:

c = m^e mod N


If we can query encryptions for different m values, we can compute differences that share the same modulus. The following function helps recover N:

def recover_n(pairings, e):
    pt1, ct1 = pairings[0]
    N = ct1 - pow(pt1, e)
    for pt, ct in pairings:
        val = gmpy2.mpz(ct - pow(pt, e))
        N = gmpy2.gcd(val, N)
    return N
```

This leverages the property that ct - m^e is always a multiple of N. By taking GCDs across multiple encryptions, we can determine the modulus.

In our case, using encryptions for 1 and 2, we derived the modulus N ≈ 5507598452356422225755194020880876452588463543445995226287547479009566151786764261801368190219042978883834809435145954028371516656752643743433517325277971.

Validating N

Checking the RSA operation:

pow(0x30, 65537, N)


yielded the same ciphertext seen earlier, confirming that our recovered N is correct.

Chosen-Ciphertext Attack (CCA)

Using the RSHack tool
 (modified for Python 3), I implemented the following logic:

c_bis = c * pow(2, e, n) % n
# send c_bis to oracle
# receive out = oracle_decrypt(c_bis)
p = out // 2


This approach works because RSA is multiplicatively homomorphic, meaning:

```
decrypt(c * 2^e mod N) = (m * 2) mod N
```

Dividing by 2 retrieves the original plaintext.

Oracle Interaction

Using the oracle:
```

ameesha_goel@DESKTOP-7U7A055:~$ nc titan.picoctf.net 54779
E --> encrypt D --> decrypt.
e
enter text to encrypt (encoded length must be less than keysize): 2
ciphertext: 4707619883686427763240856106433203231481313994680729548861877810439954027216515481620077982254465432294427487895036699854948548980054737181231034760249505
```


Then for decryption:
```
D
Enter text to decrypt: 16793269912070937844635095856519609843106938056844271635016672556411657824199338170936774923657169812744088290168486013857835787244752509092427180236594146403102972184218115898930702635526307598755278072389812742237902783893341521177857386971678423157530556886739554966046062248096265748880803311155569188515
decrypted ciphertext as hex (c ^ d mod n): 9ffff9d3556
```

Unfortunately, the result 9ffff9d3556 did not decode into any readable ASCII or valid password.

## Analysis

The incorrect output likely stemmed from human input error:
Since my terminal does not allow pasting, I had to type the entire ciphertext manually — a long decimal number of 300+ digits. Even a single typo alters the modular result completely, leading to invalid decryption output.

Due to this boundation I tried running the code on the internal terminal of picoctf which resulted in gibberish hex values which couldn't be converted.
This made recovering the correct flag impossible during my attempt.

I tried using the internal terminal of picoctf as well for my challenge but it resulted in the same gibberish and hence the error couldn't be resolved which resulted in me not being able to solve the challenge in time.

I feel that I employed the correct method to solve the challenge and understood its core concept but dueto a system/tecical error I couldn't get the final flag.



## Concepts Learnt:

RSA math & notation 

RSA public key: (N, e) where N = p * q (product of two primes) and e is the public exponent.

Encryption: c = m^e mod N where m is the integer representation of the plaintext (e.g., bytes → big-endian integer).

Decryption (private): m = c^d mod N where d is the private exponent.

2. Multiplicative property of RSA

RSA is multiplicatively homomorphic:
Enc(m1) * Enc(m2) ≡ Enc(m1 * m2) (mod N).

This enables attacks — if an attacker can get arbitrary decryptions, they can manipulate ciphertexts to reveal plaintexts (chosen-ciphertext attacks).

3. Recovering N using plaintext/ciphertext pairs (GCD trick)

For a known pair (m, c), we have c ≡ m^e (mod N) → c - m^e is a multiple of N.

With multiple pairs, computing gcd(c1 - m1^e, c2 - m2^e, ...) often yields N (or a nontrivial factor).

This is practical when the oracle leaks multiple encryptions of known small messages — a common CTF setup.

4. Chosen-ciphertext attack (multiplicative/Chopla)

Attack idea:

Let c be the target ciphertext for unknown m.

Choose small integer s (e.g., 2). Compute c' = c * s^e mod N.

Ask oracle to decrypt c' → it returns m' = m * s mod N (if no wrap-around).

Divide m' // s to recover m.

This only works reliably if m * s < N . Otherwise m' = (m*s) mod N and simple division fails.

5. Modular wrap-around & how to handle it

If m*s >= N, then m*s mod N <— you won’t be able to recover m by plain division.

Workarounds:

Pick a smaller s.

Use multiple decryptions with different s and combine results via CRT-like reconstruction if conditions allow.

6. Integer/hex/byte conversions — pitfalls

Converting between integers and plaintext bytes is sensitive:

Leading zero bytes are significant. hex() drops the leading zero nibble — add it back if necessary (pad to even length).

Endianness: RSA/CTF convention is big-endian (first byte is most significant).

Non-printable/binary plaintext is possible — treat output as raw bytes and inspect with repr() before assuming ASCII.

7. Oracle I/O and formats

Oracles may print outputs in decimal or hex — always inspect the raw text returned and parse accordingly.

## Notes

What went wrong in my attempt (root causes)

Clipboard/paste issue: Terminal on your machine (and picoCTF shell) apparently blocked paste operations. That forced manual entry of 300+ digit integers — the likeliest cause of error.

Even one mistyped digit gives a completely different modular result.

Parsing ambiguity: Oracle output could be hex or decimal. If mis-parsed, the subsequent division and conversion produce garbage.

Wrap-around risk: Using s = 2 is usually safe, but if target m is large, m*2 >= N causes modular wrap-around and invalid division results.

Leading zero loss: If a recovered hex string has odd length, you must pad it with a 0 nibble — failing to do so can change bytes drastically.

## Resources

https://sudorem.dev/posts/pico24-rsa-oracle/
https://youtu.be/XsiwqgGourA?si=6dI8ga5mQT0CNnLk
https://youtu.be/-ShwJqAalOk?si=L9kWl4RqeCG2rTyz



