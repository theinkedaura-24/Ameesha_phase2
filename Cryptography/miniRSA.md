# Challenge Description
A file containing an RSA modulus N, a public exponent e = 3, and a ciphertext c. The ciphertext appears "small" relative to N. The goal is to recover the plaintext / flag.

# Solution:

This is a classic low-exponent RSA vulnerability: when e = 3 and the plaintext message m is small enough that m^3 < N, the ciphertext c = m^3 (mod N) is simply m^3 in the integers (no wraparound). Recovering m reduces to computing the integer cube root of c, then converting that integer back to bytes to obtain the flag.

My solve:

Inspect the provided file

I opened the uploaded file named ciphertext (1) and located the lines containing N, e, and ciphertext (c).

Observed e = 3 (small exponent). This immediately suggests checking whether the ciphertext might be smaller than N (i.e., c < N). If so, the ciphertext is the plain integer cube m^3, because modular reduction did not change the value.


Check whether c < N

Convert the recorded decimal strings for N and c into integers and compare. If c < N holds, proceed with the cube root attack.


Compute the integer cube root of c

Find integer m = ∛(c) such that m^3 <= c < (m+1)^3. For exact cubes, m^3 == c.


Convert m to bytes and decode as ASCII/UTF-8 to get the flag.

Verify the result

Optionally re-encrypt m with e = 3 and N to verify m^3 mod N == c.

Commands / Code used
```py
# 1) Read the file and extract integers (adjust path if necessary)
import re
s = open('ciphertext (1)', 'r').read().strip().splitlines()
# Example: assume N and c are on known lines — adjust indices as needed
N_digits = re.sub(r'[^0-9]', '', s[0])   # line with N:
c_digits = re.sub(r'[^0-9]', '', s[3])   # line with ciphertext (c):
N = int(N_digits)
c = int(c_digits)


# 2) Compare sizes
print('N bits:', N.bit_length())
print('c bits:', c.bit_length())
print('c < N?', c < N)


# 3) Integer cube root function


def iroot(k, n):
    # returns floor(n**(1/k)) for integer k >= 1
    low, high = 0, 1 << ((n.bit_length() // k) + 2)
    while low < high:
        mid = (low + high) // 2
        if mid**k <= n:
            low = mid + 1
        else:
            high = mid
    return low - 1


m = iroot(3, c)
print('m^3 == c?', m**3 == c)


# 4) Convert to bytes & print
msg = m.to_bytes((m.bit_length() + 7) // 8, 'big')
print(msg)
try:
    print(msg.decode())
except Exception:
    print('Could not decode as UTF-8; raw bytes shown above')


# 5) Verify encryption (optional)
assert pow(m, 3, N) == c

(Screenshot suggestion: include a terminal screenshot showing the Python script being run and printing the decoded flag. Example output below.)

N bits: 2048
c bits: 512
c < N? True
m^3 == c? True
b'picoCTF{n33d_a_lArg3r_e_606ce004}'
picoCTF{n33d_a_lArg3r_e_606ce004}
```

## Flag:
```
picoCTF{n33d_a_lArg3r_e_606ce004}
```
## Concepts learnt:

RSA basics (N, e, d, ciphertext)

RSA public key = (N, e), private key d. Encryption c = m^e mod N and decryption m = c^d mod N.

Low public exponent attacks

When the public exponent e is small (like 3 or 5) and the message is small / not padded, it's possible that m^e < N, so c = m^e as an integer — making the message recoverable by taking an integer root.

Unpadded RSA is dangerous

RSA without proper padding (e.g., OAEP for encryption) is insecure. Padding adds randomness and ensures m is large enough (and unpredictable). In CTFs, challenges often purposely give unpadded messages to teach this.

Integer root algorithms

Efficient binary search or Newton methods can compute integer k-th roots of large integers.

Coppersmith's method (brief)

For more advanced cases where m^e is reduced modulo N and partial information is known, Coppersmith's method can find small roots of modular polynomial equations. It's a lattice-based technique used when m is small relative to N but not small enough that m^e < N.

Verification via modular arithmetic

After recovering m, always verify pow(m, e, N) == c.

## Notes:

I first considered attempting to factor N — but factoring is hard for sufficiently large RSA moduli (and unnecessary here).

Another tangent is to try brute-forcing small messages — but the cube-root approach is deterministic and fast.

If the ciphertext were larger than N, other low-exponent attacks could apply (e.g., Hastad's broadcast attack when the same plaintext is encrypted under several moduli with small e), or Coppersmith. Those are more complex.

When the simple cube-root attack doesn't apply

If c >= N (i.e., m^3 mod N wrapped), or if padding was used, recoveries require different techniques. Examples include:

Håstad's Broadcast Attack: when the same plaintext m is encrypted with same small e under multiple co-prime moduli N1, N2, ..., use CRT and integer root.

Coppersmith's attack: find small roots of polynomial equations modulo N using lattice reduction (LLL). Useful when part of m is known or m is smaller than N^(1/e) by some margin.

Reproducibility

The solution is reproducible with the short Python script above. Replace file path lines or parsing as needed depending on the exact formatting of the provided file. In the original run, c < N was True which made the attack straightforward.

## Resources:

1. https://www.coursera.org/learn/crypto
2. https://en.wikipedia.org/wiki/Coppersmith%27s_attack

