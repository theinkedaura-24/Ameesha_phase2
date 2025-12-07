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
