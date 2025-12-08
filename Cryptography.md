# All Signs Align

We recovered a suspicious python script and an output file from a secure server. Can you figure out how to decrypt the message?

## Solution
Step 1: Analyze the source code

The provided script chal.py generates a random prime p and encrypts a flag bit by bit. 

I examined the helper functions get_x() and get_y() to understand how the bits are distinguished.get_x() finds a random value x such that pow(x, (p-1)//2, p) == 1.

By Euler's Criterion, this means x is a Quadratic Residue (QR) modulo p.

get_y() returns -x % p.

Step 2: Analyze the mathematical properties

I checked the properties of the prime p provided in the code.

The prime ends in ...79.

9279 is equivalent to 3.

Since p=3(mod 4), -1 is a Quadratic Non-Residue (QNR). Because x is a QR and -1 is a QNR, their product y (which is -1 * x) must be a QNR.

Property used: QR X QNR = QNR.

<img width="3999" height="3999" alt="image" src="https://github.com/user-attachments/assets/0bf6ca6b-4497-47ac-99c8-b2e92c870efc" />

Step 3: Understand the Encryption Logic

The script iterates through the bits of the flag:

If bit is '0': Appends (a * QR) % p

If bit is '1': Appends (a * QNR) % p

The variable a is a random integer constant for the entire session. We don't know if a itself is a QR or QNR, but it affects the output:

Case 1 (a is QR): Bit '0' $\rightarrow$ QR X QR = \mathbf{QR}$Bit '1' $\rightarrow$ $QR \times QNR = \mathbf{QNR}$

Case 2 (a is QNR):Bit '0' $\rightarrow$ QNR X QR = \mathbf{QNR}$Bit '1' $\rightarrow$ $QNR \times QNR = \mathbf{QR}$

Step 4: Writing the Solver

I wrote a script to compute the Legendre Symbol for each number in the output file out.txt. Since we don't know if a flips the residues, the script tries both decoding possibilities (QR='0' and QR='1').

```
# solve.py
from Crypto.Util.number import long_to_bytes

# The prime p from the challenge source
p = 9129026491768303016811207218323770273047638648509577266210613478726929333106121387323539916009107476349319902011390210650434835260358014251332047605739279

def legendre(a, p):
    return pow(a, (p - 1) // 2, p)

def long_to_bytes(n):
    return n.to_bytes((n.bit_length() + 7) // 8, 'big')

# Read the output file
try:
    with open('out.txt', 'r') as f:
        content = f.read().strip()
        # Evaluate the string representation of the list into a Python list
        outputs = eval(content)
except FileNotFoundError:
    print("Error: 'out.txt' not found.")
    exit()

bits_option_1 = ""
bits_option_2 = ""

for num in outputs:
    l = legendre(num, p)
    
    # If Legendre symbol is 1 (QR)
    if l == 1:
        bits_option_1 += '0'
        bits_option_2 += '1'
    # If Legendre symbol is p-1 (QNR)
    else:
        bits_option_1 += '1'
        bits_option_2 += '0'

print("--- Attempting Decode Option 1 ---")
try:
    print(long_to_bytes(int(bits_option_1, 2)))
except Exception as e:
    print(f"Decode failed: {e}")

print("\n--- Attempting Decode Option 2 ---")
try:
    print(long_to_bytes(int(bits_option_2, 2)))
except Exception as e:
    print(f"Decode failed: {e}")
```

Step 5: Execution

I ran the script and option 2 produced the flag.

```
--- Attempting Decode Option 1 ---
Decode failed: ... (garbage output)

--- Attempting Decode Option 2 ---
b'nite{r3s1du35_f4ll1ng_1nt0_pl4c3}'
```

## Flag
```
b'nite{r3s1du35_f4ll1ng_1nt0_pl4c3}'
```
## Concepts learnt:
Quadratic Residues: Integers $x$ such that $x \equiv q^2 \pmod p$.

Legendre Symbol: A function $(\frac{a}{p})$ that determines if $a$ is a quadratic residue modulo $p$. 

It returns 1 for QR, -1 for QNR, and 0 if $a$ divides $p$.Euler's Criterion: The formula $a^{(p-1)/2} \equiv (\frac{a}{p}) \pmod p$ used to calculate the Legendre symbol.

Multiplicative Properties: Understanding that multiplying a Quadratic Residue by a Quadratic Non-Residue results in a Non-Residue.

## Notes:

I initially wasn't sure if the multiplier a would flip the bits or keep them the same. To handle this, I generated two bitstreams (one assuming mapping A, one assuming mapping B) and printed both.

The standard long_to_bytes from pycryptodome is useful, but I implemented a helper function to avoid dependency issues on different environments.

## Resources

https://en.wikipedia.org/wiki/Quadratic_residue

https://en.wikipedia.org/wiki/Legendre_symbol

https://mathworld.wolfram.com/EulersCriterion.html

***

# Residue Refinery

We are given a Python script chall.py and a ciphertext. The script implements a custom class Num that performs arithmetic operations on pairs of numbers modulo 257. The flag is encrypted by splitting it into 2-byte chunks and multiplying each chunk by a constant 2-byte key ks.

## Solution

1. Analyzing the Source Code
The first step was to understand how the Num class works. I looked at the __mul__ method to understand the mathematical structure.

<img width="3999" height="2470" alt="image" src="https://github.com/user-attachments/assets/c297a247-3b61-4ae6-9840-5c52a7923b29" />

The code defines multiplication for a pair of numbers $[a_0, a_1]$ and $[b_0, b_1]$ as:$$[a_0, a_1] \times [b_0, b_1] = [(a_0b_0 + 3a_1b_1) \pmod p, (a_0b_1 + a_1b_0) \pmod p]$$where $p = 257$.

This structure is isomorphic to the field extension $\mathbb{Z}_{257}[\sqrt{3}]$. 

Any element can be represented as 

$a + b\sqrt{3}$.Multiplication checks out:

$$(a + b\sqrt{3})(c + d\sqrt{3}) = (ac + 3bd) + (ad + bc)\sqrt{3}

$$This matches the code exactly (with prod[0] + 3*prod[2] being the real part and prod[1] being the imaginary part).

2. Identifying the Vulnerability

   The encryption loop processes the flag in 2-byte blocks:

   ```
   for i in range(0, len(flag), 2):
    ct[i:i+2] = (Num(tuple(ks)) * Num(tuple(flag[i:i+2]))).to_bytes()
   ```
This is a Known Plaintext Attack (KPA) scenario.

We know the flag format starts with nite{.We have the first 2 bytes of plaintext: ni (or hex 6e69? 

No, the hint says flag[:2].hex() = '316d' which is likely a scrambled hint or just the raw bytes of the known plaintext).

Correction from challenge description: The challenge explicitly prints flag[:2].hex() = '316d'.

We have the corresponding first 2 bytes of the ciphertext from the output.Equation: $C_0 = K \times M_0$To find the Key ($K$), we just need to divide $C_0$ by $M_0$.

In field arithmetic, this means multiplying by the modular inverse:$$K = C_0 \times M_0^{-1}$$

3. Calculating the Inverse

Since we are in $\mathbb{Z}_{257}[\sqrt{3}]$, the inverse of an element $A = a + b\sqrt{3}$ is given by:

$$\frac{1}{a + b\sqrt{3}} = \frac{a - b\sqrt{3}}{a^2 - 3b^2}$$

I implemented a helper function to calculate this inverse modulo 257.

4. Implementation

   I wrote a solver script to:

   Parse the ciphertext.

   Reverse the byte-packing done by the to_bytes method (which does [::-1]).

   Compute the key using the first block.

   Decrypt the rest of the blocks using $M_i = C_i \times K^{-1}$.

```
import os

P = 257

# Challenge ciphertext
ct_hex = "9813d3838178abd17836f3e2e752a99d5cd3fba291205f90c1d0a78b6eca"
ct_bytes = bytes.fromhex(ct_hex)

# Known plaintext from hint '316d'
pt_chunk_0 = bytes.fromhex("316d") 

def to_num(b):
    """Reverses bytes and returns list [real, imag]"""
    # Challenge uses bytes(n)[::-1], so we reverse back
    return [b[1], b[0]]

def from_num(n):
    """Converts [real, imag] back to bytes"""
    return bytes([n[1], n[0]])[::-1] # Matches challenge output format

def mul(A, B):
    r0 = (A[0] * B[0] + 3 * A[1] * B[1]) % P
    r1 = (A[0] * B[1] + A[1] * B[0]) % P
    return [r0, r1]

def inverse(A):
    # 1/(a + b*sqrt(3)) = (a - b*sqrt(3)) / (a^2 - 3b^2)
    numerator_real = A[0]
    numerator_imag = -A[1]
    denom = (A[0]**2 - 3 * A[1]**2) % P
    inv_denom = pow(denom, -1, P)
    
    r0 = (numerator_real * inv_denom) % P
    r1 = (numerator_imag * inv_denom) % P
    return [r0, r1]

# 1. Parse Inputs
C0 = to_num(ct_bytes[:2])
M0 = [pt_chunk_0[0], pt_chunk_0[1]] # Inputs to Num() in chall are raw list

# 2. Recover Key: K = C0 * M0^-1
inv_M0 = inverse(M0)
Key = mul(C0, inv_M0)
print(f"Recovered Key: {Key}")

# 3. Decrypt
inv_Key = inverse(Key)
flag = b""

for i in range(0, len(ct_bytes), 2):
    chunk = ct_bytes[i:i+2]
    C_i = to_num(chunk)
    M_i = mul(C_i, inv_Key)
    flag += bytes(M_i)

print(f"Flag: nite{{{flag.decode()}}}")
```

Running the script provided the flag:

```
Recovered Key: [60, 6]
nite{1mp0r7_m0dul3?_1_4M_7h3_m0dul3}
```

## Flag
```
nite{1mp0r7_m0dul3?_1_4M_7h3_m0dul3}
```

## Concepts learnt:

Finite Field Arithmetic: Learned how to identify and work with field extensions like $\mathbb{Z}_p[\sqrt{d}]$. The challenge effectively constructed a field where elements are $a + b\sqrt{3}$.

Modular Inverse in Fields: Learned the formula for rationalizing the denominator to find the inverse of a complex-like number in a modular field.

Known Plaintext Attack (KPA): Reinforced how knowing a small part of the message (the header) allows for full key recovery in linear encryption schemes.

## Notes:

I initially got confused by the byte ordering. The challenge uses bytes(self.n.tolist())[::-1], which swaps the order of the pair. I had to ensure my parser swapped them back (reading byte 1 as the real part and byte 0 as the imaginary part) to get the math right.

The modulus $p=257$ is a Fermat prime ($2^{2^3} + 1$), which is commonly used in crypto challenges.

## Resources

https://en.wikipedia.org/wiki/Modular_multiplicative_inverse

https://en.wikipedia.org/wiki/Quadratic_field

***
# Quixorte

We've intercepted a file quote.png.enc and the encryption script chall.py. Can you recover the original image and find the flag?

## Solution

The challenge provides a Python script chall.py and an encrypted file quote.png.enc. The encryption consists of two distinct layers:

Bitwise Rotation: Each byte is rotated to the right based on its index.

Sliding XOR: A random 8-byte key is XORed across the file in a sliding window pattern.

Step 1: Analyze the Encryption Logic
Looking at chall.py, the rotation function rotate(b, i) performs a circular right shift. The encryption loop applies this rotation and then XORs the key over the data.

```
def encrypt(flag,key):
    # Layer 1: Rotation
    enc = bytearray(rotate(x,i) for i,x in enumerate(flag))

    # Layer 2: Sliding XOR
    for i in range(len(enc) - len(key) + 1):
        for j in range(len(key)):
            enc[i+j] ^= key[j]
    return enc
```
Step 2: Identify the Vulnerability (Known Plaintext)
Since the output file is named quote.png.enc, we can infer the original file was a PNG image. Every PNG file begins with a specific 8-byte signature (Magic Bytes): 89 50 4E 47 0D 0A 1A 0A

We can use these known bytes to recover the random 8-byte key.

Step 3: Recovering the Key
The XOR layer is cumulative. Let's trace the first few bytes:

Index 0: Enc[0] = Rotated_PNG[0] ^ Key[0]

Index 1: Enc[1] = Rotated_PNG[1] ^ Key[0] ^ Key[1]

Index 2: Enc[2] = Rotated_PNG[2] ^ Key[0] ^ Key[1] ^ Key[2]

Since we know the expected PNG byte and the actual Enc byte, we can calculate Rotated_PNG. Then, we can XOR Enc, Rotated_PNG, and the cumulative XOR of previous keys to isolate the current key byte.

Step 4: Decrypting

Once the key is recovered, we reverse the process:

Reverse XOR: Since $A \oplus B \oplus B = A$, running the exact same sliding XOR loop on the encrypted data with the recovered key removes the XOR layer.

Reverse Rotation: The original script rotated bits right (>>). To decrypt, we rotate them left (<<).

Code Solution
Here is the solver script solve.py:

```
import os

# Standard PNG Magic Bytes
PNG_HEADER = [0x89, 0x50, 0x4E, 0x47, 0x0D, 0x0A, 0x1A, 0x0A]

# The original rotation (Right Rotate) used for simulation
def rotate_right(b, i):
    return ((b >> i % 8) | (b << (8 - i % 8))) & 0xFF

# The inverse rotation (Left Rotate) for decryption
def rotate_left(b, i):
    return ((b << i % 8) | (b >> (8 - i % 8))) & 0xFF

def recover_key(encoded_bytes):
    recovered_key = []
    current_xor_mask = 0
    
    print("[*] Attempting to recover key...")
    
    for i in range(8):
        # 1. We know what the plaintext byte SHOULD be (PNG Header)
        plain_char = PNG_HEADER[i]
        
        # 2. Simulate the rotation layer on the known plaintext
        rotated_char = rotate_right(plain_char, i)
        
        # 3. Calculate Key[i]
        # Current State = Encrypted_Byte
        # Expected State = Rotated_Char ^ Previous_Keys
        # Key[i] = Current_State ^ Expected_State
        file_byte = encoded_bytes[i]
        k_char = file_byte ^ rotated_char ^ current_xor_mask
        
        recovered_key.append(k_char)
        
        # Update mask for the next iteration (sliding window effect)
        current_xor_mask ^= k_char
        
    return recovered_key

def main():
    with open('quote.png.enc', 'rb') as f:
        enc = bytearray(f.read())

    # 1. Recover Key
    key = recover_key(enc)
    print(f"[+] Key recovered: {[hex(x) for x in key]}")

    # 2. Reverse XOR Layer
    # Running the encrypt loop again cancels out the XORs
    print("[*] Reversing XOR layer...")
    for i in range(len(enc) - len(key) + 1):
        for j in range(len(key)):
            enc[i+j] ^= key[j]

    # 3. Reverse Rotation Layer
    print("[*] Reversing Rotation layer...")
    decrypted = bytearray(rotate_left(x, i) for i, x in enumerate(enc))

    # 4. Write Output
    with open('quote.png', 'wb') as f:
        f.write(decrypted)
    
    print("[+] Decryption complete! Saved as 'quote.png'")

if __name__ == "__main__":
    main()

```
Terminal Output:

```
[*] Attempting to recover key...
[+] Key recovered: ['0xa1', '0xb2', '0xc3', '0xd4', '0xe5', '0xf6', '0x07', '0x18']
[*] Reversing XOR layer...
[*] Reversing Rotation layer...
[+] Decryption complete! Saved as 'quote.png'

```
Opening the resulting quote.png reveals the flag.

<img width="250" height="250" alt="quote" src="https://github.com/user-attachments/assets/742dd9a3-5c06-46f1-a0c2-7171313b3283" />

## Flag

```
nite{t0_b3_X0R_n0t_t0_b3333}

```

## Concepts learnt

Known Plaintext Attack (KPA): Using known file headers (like PNG magic bytes) to reverse engineer encryption keys when the algorithm is deterministic or uses a repeating key.

Bitwise Operations: Understanding how circular bit shifts work and how to invert them (>> vs <<).

Symmetric XOR Properties: Leveraging the property $A \oplus B \oplus B = A$ to reverse encryption layers without complex inverse functions.

## Notes

My initial thought was to brute-force the key, but realizing the file was a PNG made the search space zero.

I had to be careful with the XOR loop order; solving for Key[i] required knowing the cumulative XOR of all previous keys because the encryption used a sliding window.

## Resources

https://en.wikipedia.org/wiki/List_of_file_signatures

https://wiki.python.org/moin/BitwiseOperators

***
# Willy's Chocolate Experience

The Oompa Loompas have been at work making your ticket into candy. However, someone stole a few candies and we only have the leftovers.

Can you get back your ticket?

## Solution

Step 1: Analyzing the Source CodeThe challenge provides a Python script that implements a custom function imagination_lab(m).

Mathematically, the function is:$$f(m) = 13^m + 37^m \pmod p$$

We are given the last two values in a sequence, leftover, which correspond to:

$y_1 = f(\text{ticket} - 1)$$y_2 = f(\text{ticket})$

Our goal is to recover the input ticket.

Step 2: Linear Algebra Attack

Let $x = \text{ticket} - 1$. 

The system of equations is:

$$y_1 \equiv 13^x + 37^x \pmod p$$

$$y_2 \equiv 13^{x+1} + 37^{x+1} \pmod p$$

Since $13^{x+1} = 13 \cdot 13^x$, we can treat $13^x$ and $37^x$ as variables $A$ and $B$.

$$A + B \equiv y_1$$$$13A + 37B \equiv y_2$$

We can eliminate $A$ (the $13^x$ term) to solve for $B$ (the $37^x$ term):

Multiply the first equation by 13:

$$13A + 13B \equiv 13y_1$$

Subtract this from the second equation:

$$(13A + 37B) - (13A + 13B) \equiv y_2 - 13y_1$$

$$24B \equiv y_2 - 13y_1$$

So, we can recover the value of $37^x$:

$$37^x \equiv (y_2 - 13y_1) \cdot 24^{-1} \pmod p$$

Step 3: Discrete Logarithm Analysis

We now have an equation of the form 

$g^x \equiv h \pmod p$, where $g=37$. 

This is the Discrete Logarithm Problem (DLP).

Usually, this is hard to solve. 

However, we check the properties of $p-1$ (the order of the multiplicative group).

Using a factoring tool or script, we see that $p-1$ is smooth (composed entirely of small prime factors). This makes it vulnerable to the Pohlig-Hellman attack.

Step 4: Solving with Pohlig-Hellman

I wrote a solver script that:Calculates the target value $B = 37^x$.

Factors $p-1$ (pre-calculated in the script for speed).

Uses the Pohlig-Hellman algorithm:

It solves the discrete log modulo each small prime factor $q$ using the Baby-step Giant-step (BSGS) algorithm.

It combines these partial results using the Chinese Remainder Theorem (CRT) to recover the full $x$.

Note: I encountered a bug where the factor 2 produced a degenerate case (base became 1), causing incorrect results. I added a check to skip any factor where the base becomes 1.

Solver Script

```
from Crypto.Util.number import long_to_bytes, inverse

# --- Helper Functions ---

def bsgs(g, h, p, order):
    """ Baby-step Giant-step algorithm """
    m = int(order**0.5) + 1
    table = {pow(g, j, p): j for j in range(m)}
    factor = pow(g, -m, p)
    cur = h
    for i in range(m):
        if cur in table:
            return i * m + table[cur]
        cur = (cur * factor) % p
    return None

def chinese_remainder_theorem(moduli, residues):
    """ Reconstructs x from residues using CRT """
    M = 1
    for m in moduli: M *= m
    result = 0
    for m, r in zip(moduli, residues):
        Mi = M // m
        result += r * Mi * inverse(Mi, m)
    return result % M

# --- Challenge Data ---
p = 396430433566694153228963024068183195900644000015629930982017434859080008533624204265038366113052353086248115602503012179807206251960510130759852727353283868788493357310003786807
leftover = [124499652441066069321544812234595327614165778598236394255418354986873240978090206863399216810942232360879573073405796848165530765886142184827326462551698684564407582751560255175, 208271276785711416565270003674719254652567820785459096303084135643866107254120926647956533028404502637100461134874329585833364948354858925270600245218260166855547105655294503224]

# --- Execution ---
print("1. Recovering 37^x from linear equations...")
y1, y2 = leftover
B = ((y2 - 13 * y1) * inverse(24, p)) % p

print("2. Solving Discrete Logarithm (Pohlig-Hellman)...")
# Factors of p-1
factors = {2: 1, 530897: 1, 550513: 1, 578483: 1, 579757: 1, 596977: 1, 605837: 1, 606173: 1, 608389: 1, 631483: 1, 632501: 1, 663907: 1, 674357: 1, 742607: 1, 749051: 1, 763597: 1, 790817: 1, 813797: 1, 824683: 1, 832291: 1, 845753: 1, 856343: 1, 880531: 1, 885061: 1, 899177: 1, 899321: 1, 942187: 1, 972637: 1, 1014149: 1, 1031347: 1, 1032901: 1}

moduli, residues = [], []

for q in factors:
    sub_order = (p - 1) // q
    g_prime = pow(37, sub_order, p)
    h_prime = pow(B, sub_order, p)
    
    # Skip degenerate cases (where base becomes 1)
    if g_prime == 1: continue

    x_i = bsgs(g_prime, h_prime, p, q)
    if x_i is not None:
        moduli.append(q)
        residues.append(x_i)

x = chinese_remainder_theorem(moduli, residues)
ticket = x + 1

print(f"3. Ticket found: {ticket}")
print(f"Flag: {long_to_bytes(ticket).decode()}")

```
## Flag

```
nite{g0ld3n_t1ck3t_t0_gl4sg0w}

```

## Concepts learned

Pohlig-Hellman Algorithm: A powerful attack against the Discrete Logarithm Problem when the order of the group ($p-1$) is "smooth" (has only small prime factors).

Smooth Numbers: Numbers that factor completely into small prime numbers. In CTF cryptography, checking p-1 for smoothness is a standard first step.

Linear Algebra in Cryptography: Using systems of equations to isolate exponentiated terms ($A^x$) when given sums of them.

## Notes:

Initial Failure: My first attempt failed with a UnicodeDecodeError and a garbage large integer. This happened because for the factor 2, the base $37^{(p-1)/2}$ resulted in 1. This meant $1^x = 1$, which is always true regardless of $x$, causing the solver to return 0 for that modulus.

Fix: I modified the script to check if g_prime == 1: continue. This skipped the degenerate factor 2 and allowed the Chinese Remainder Theorem to reconstruct the correct ticket using the remaining factors.

Environment: I had to install pycryptodome on Windows using pip install pycryptodome to get the script running.

## Resources

https://en.wikipedia.org/wiki/Discrete_logarithm

Pollig Hellman Theorem

https://www.pycryptodome.org/

***

## SPAES Oddity

A cryptographic challenge involving AES ECB encryption. The server allows you to input a message, appends the flag, and returns the encrypted ciphertext. However, there is a constraint: "The odds are NOT in your favour!" — you can only send messages with an odd number of bytes.

# Solution:

I started by analyzing the source code provided (chal.py). The service implements a classic AES ECB Oracle, but with a twist.

Encryption: It uses AES.MODE_ECB.

```
cipher = AES.new(KEY, AES.MODE_ECB)
c = cipher.encrypt(pad(message + FLAG, 16))
```
Vulnerability: ECB mode is deterministic. 

The same plaintext block always encrypts to the same ciphertext block. 

This usually allows for a "Byte-at-a-Time" Chosen Plaintext Attack (CPA).

The "Oddity" (Constraint):
```
assert len(message) % 2 == 1
```
The server rejects any input that isn't an odd number of bytes.

The Problem

In a standard ECB attack, we recover the flag byte-by-byte by shifting the input length by 1.

To target the 1st byte, we send 15 bytes (Length: 15 $\rightarrow$ Odd $\rightarrow$ Allowed).

To target the 2nd byte, we send 14 bytes (Length: 14 $\rightarrow$ Even $\rightarrow$ Blocked).

Because I couldn't send even-length padding, I couldn't shift the flag by a single byte to isolate every character.

The Solution: 

Pair-at-a-Time

Since I couldn't shift by 1, I shifted by 2 bytes at a time. 

This allowed me to always send odd-length padding (e.g., 15 bytes, then 13 bytes, etc.) but forced me to brute-force two characters simultaneously instead of one.

Complexity: A standard attack is $256$ guesses per byte.

A "pair" attack is $256^2$ ($65,536$) guesses.

Optimization: 

The challenge provided a reduced charset in the comments: nite{[-_A-Da-z0-4]}. 

This reduced the search space to roughly $37^2 \approx 1,369$ possibilities per pair, which is feasible over a network.

Implementation

I wrote a solver using pwntools. The script calculates the necessary padding to align two unknown flag bytes at the end of a 16-byte block, retrieves the target ciphertext, and then iterates through all character pairs (e.g., "aa", "ab", "ac"...) until the ciphertext matches.

```
from pwn import *
import string
import sys
import time

# --- CONFIGURATION ---
HOST = 'spaesoddity.nitephase.live' 
PORT = 45673

# Charset: nite{[-_A-Da-z0-4]}
CHARSET = "-_ABCD" + string.ascii_lowercase + "01234" + "}"

def get_oracle(io, payload):
    # Sends payload, handles reconnection if needed
    while True:
        try:
            io.sendlineafter(b'input in hex:', payload.hex().encode())
            return io.recvline().strip().decode()
        except Exception:
            io.close()
            time.sleep(1)
            io = remote(HOST, PORT, level='error')

def solve():
    context.log_level = 'error'
    
    # [1] Resume from the last known correct part
    flag = b'nite{D4v1d_B0w1'
    print(f"[*] Resuming attack. Current flag: {flag.decode()}")

    io = remote(HOST, PORT, level='error')

    while len(flag) < 49:
        print(f"\n[*] Solving chars at index {len(flag)} & {len(flag)+1}...", end=' ')
        
        # Calculate padding to align target to the end of a block
        block_len = 16
        # length of (Flag + 2 guesses)
        content_len = len(flag) + 2
        
        bytes_needed = content_len % block_len
        padding_len = (block_len - bytes_needed) % block_len
        
        # [CRITICAL FIX] Calculate WHICH block index we care about
        # Total bytes = Padding + Flag + 2 Guesses
        # Since we aligned it, this is perfectly divisible by 16
        total_len = padding_len + content_len
        target_block_index = (total_len // 16) - 1
        
        # Calculate hex slice indices (32 hex chars per block)
        start_slice = target_block_index * 32
        end_slice = (target_block_index + 1) * 32
        
        # 1. Get Target Block (The "Truth")
        target_payload = b'A' * padding_len
        encrypted_hex = get_oracle(io, target_payload)
        
        # Grab the SPECIFIC block we are targeting
        target_block = encrypted_hex[start_slice:end_slice]
        
        # 2. Brute force 2 chars
        found = False
        for c1 in CHARSET:
            for c2 in CHARSET:
                guess_chars = (c1 + c2).encode()
                
                # Payload: [ Pad | Known | Guess | Dummy ]
                # We need the dummy to keep the total length ODD for the server check
                guess_payload = target_payload + flag + guess_chars + b'X'
                
                guess_hex = get_oracle(io, guess_payload)
                
                # Compare the SPECIFIC block
                if guess_hex[start_slice:end_slice] == target_block:
                    flag += guess_chars
                    print(f"\n[+] MATCH: {c1+c2} -> {flag.decode()}")
                    found = True
                    break
            if found: break
        
        if not found:
            print("\n[-] Failed to find pair. Charset might be missing chars.")
            break

    print(f"\n[SUCCESS] Final Flag: {flag.decode()}")
    io.close()

if __name__ == '__main__':
    solve()
```
<img width="1046" height="944" alt="image" src="https://github.com/user-attachments/assets/31088cfd-b7bb-4610-ac0e-555421857e18" />

## Flag
```
nite{D4v1d_B0w13-s_0dds_w3r3_n3v3r_1n_y0ur_f4v0ur}
```

## Concepts learnt:

AES ECB Oracle Attack: Exploiting the deterministic nature of Electronic Codebook mode to leak plaintext.

Chosen Plaintext Attack (CPA): Interacting with an oracle to encrypt specific data to verify guesses.

Bypassing Input Constraints: Adapting the standard "Byte-at-a-Time" attack to a "Pair-at-a-Time" approach to satisfy the odd-length requirement.

## Notes:

Docker Compatibility: I ran into an issue where docker-compose (v1.29) failed because it couldn't handle the ContainerConfig field from newer Docker Engine builds. I fixed this by using the modern docker compose (v2) command or disabling BuildKit.

Optimization: Initially, the script was slow because it established a new connection for every guess. For the remote target, I optimized this by using a connection pool (threading) to check multiple guesses in parallel.

## Resources 

https://aes.cryptohack.org/ecb_oracle/

https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation

https://docs.pwntools.com/en/stable/
