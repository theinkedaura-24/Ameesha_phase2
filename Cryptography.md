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

Case 1 (a is QR): Bit '0' $\rightarrow$ $QR \times QR = \mathbf{QR}$Bit '1' $\rightarrow$ $QR \times QNR = \mathbf{QNR}$

Case 2 (a is QNR):Bit '0' $\rightarrow$ $QNR \times QR = \mathbf{QNR}$Bit '1' $\rightarrow$ $QNR \times QNR = \mathbf{QR}$

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
