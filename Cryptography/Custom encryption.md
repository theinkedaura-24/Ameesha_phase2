# Challenge Description
We are given two files: enc_flag and custom_encryption.py. The former contains the flag as ciphertext as well as the parameters which were used to encrypt it, while the latter, as the name suggests, is the source code of the custom encryption algorithm used to encipher the flag. 


## Solution:

The provided custom_encryption.py combines two layers:

A Diffie-Hellman style key exchange using small primes (p = 97, g = 31) to derive a shared_key from a and b. The program generates u = g^a mod p, v = g^b mod p, and the shared key shared_key = v^a mod p = u^b mod p (if a and b match the ones used to produce u/v).

A two-stage encryption:

dynamic_xor_encrypt: reverses the plaintext, XORs each character with a repeating text key "trudeau", and returns the semi-ciphertext string.

encrypt: multiplies the ordinal value of each character of the semi-ciphertext by shared_key * 311, producing a list of integers (the final ciphertext).

Given enc_flag provides a = 94, b = 29, and the ciphertext array, we can:

Recompute shared_key deterministically using the same p, g, a, b.

Reverse the numeric multiplication by dividing each ciphertext element by (shared_key * 311) to get back the semi-ciphertext characters.

Reverse the dynamic_xor_encrypt by XORing, then reversing the string to retrieve the original plaintext.

I inspected custom_encryption.py to confirm exact operations and constants (p = 97, g = 31, text_key = "trudeau", multiplier 311).

The script's is_prime check is faulty (it checks primality poorly) but the values p and g are fixed; enc_flag supplied a and b, so randomness is gone — we can recompute the shared key exactly.

The multiplication step in encrypt is reversible by integer division because the encryption used integer multiplication of ord(char) by (shared_key * 311). The ciphertext contains values that are multiples of this product; division yields integer ordinals, which we convert to characters.

The dynamic_xor_encrypt function processes plaintext[::-1] (it reverses the input before XORing) and builds the cipher by XORing each character with the repeating "trudeau" key — so decryption must XOR in the same order and finally reverse the result to get the original message.



# Code used 
```
def is_prime(p):
    v = 0
    for i in range(2, p + 1):
        if p % i == 0:
            v = v + 1
    return False if v > 1 else True

def generator(g, x, p):
    return pow(g, x, p)

def leak_shared_key(a, b):
    p = 97
    g = 31
    if not is_prime(p) and not is_prime(g):
        print("Enter prime numbers")
        return
    u = generator(g, a, p)
    v = generator(g, b, p)
    key = generator(v, a, p)
    b_key = generator(u, b, p)
    if key == b_key:
        return key
    else:
        raise ValueError("Invalid key agreement")

def decrypt(ciphertext, key):
    semi_ciphertext = []
    for num in ciphertext:
        # integer division; encryption used ord(char) * key * 311
        val = num // (key * 311)
        semi_ciphertext.append(chr(val))
    return "".join(semi_ciphertext)

def dynamic_xor_decrypt(semi_ciphertext, text_key):
    plaintext = ""
    key_length = len(text_key)
    for i, char in enumerate(semi_ciphertext):
        key_char = text_key[i % key_length]
        decrypted_char = chr(ord(char) ^ ord(key_char))
        plaintext += decrypted_char
    # encryption reversed the plaintext before XOR, so reverse back
    return plaintext[::-1]

if __name__ == "__main__":
    a = 94
    b = 29
    ciphertext_arr = [
        260307,491691,491691,2487378,2516301,0,1966764,1879995,1995687,1214766,0,
        2400609,607383,144615,1966764,0,636306,2487378,28923,1793226,694152,780921,
        173538,173538,491691,173538,751998,1475073,925536,1417227,751998,202461,347076,491691
    ]
    text_key = "trudeau"

    shared_key = leak_shared_key(a, b)
    semi = decrypt(ciphertext_arr, shared_key)
    plain = dynamic_xor_decrypt(semi, text_key)
    print("shared_key =", shared_key)
    print("semi_ciphertext (repr) =", repr(semi))
    print("plaintext =", plain)
```

(Terminal output from running the script — I ran this and got the following):

```
shared_key = 93
semi_ciphertext (repr) = '\t\x11\x11VW\x00DAE*\x00S\x15\x05D\x00\x16V\x01>\x18\x1b\x06\x06\x11\x06\x1a3 1\x1a\x07\x0c\x11'
plaintext = picoCTF{custom_d2cr0pt6d_751a22dc}
```

## Flag:
```
picoCTF{custom_d2cr0pt6d_751a22dc}
```

## Concepts learnt:

Diffie–Hellman key exchange (very small parameters here)

The script implemented a simplified DH: u = g^a mod p, v = g^b mod p, shared key s = v^a mod p = u^b mod p. Using tiny p/g and supplied exponents allows us to recompute the shared key easily.

Symmetric multiplication-based obfuscation

Instead of using a standard cipher, the author multiplied ASCII codes by a predictable scalar (shared_key * 311). This is reversible by integer division when the divisor is known.

Simple XOR stream with repeated key

dynamic_xor_encrypt is essentially a repeating-key XOR (Vigenère-like) applied to the reversed plaintext. Repeating-key XOR is trivially reversible when the key is known.

Importance of randomness and secret secrecy

Because a and b were stored/known in enc_flag, the shared key isn't secret. Good implementations should not leak ephemeral private values.

Faulty cryptographic primitives

The example shows multiple crypto anti-patterns: small DH parameters, deterministic re-use/leak of secrets, use of deterministic reversible arithmetic for secrecy, and reversing plaintext before XOR (no security).

Practical CTF reverse-engineering approach

Read the encryption script carefully, identify constants and flows, reuse provided values, invert operations in reverse order (last applied -> first undone).

## Notes:

Alternate approaches / tangents

If a and b were not provided, an attacker might try to recover the shared key by brute force easily because p = 97 is tiny (only 96 possible exponents); one could bruteforce a or b from the public u or v values if they were given. Here a and b are already given in enc_flag.

If the multiplication had rounding or non-integer behavior, decryption might need to rely on nearest integer; but here integer division yields exact ASCII values because encrypt used integer multiplication.

The script uses a poor primality test (is_prime) — it’s slow and incorrect for large numbers. In a real challenge you’d replace with a robust primality test (e.g., sympy.isprime or Miller–Rabin).

The dynamic_xor_encrypt reversed the plaintext. If you forget to reverse at the end, you'll get garbled output — a common implementation bug to watch for.

## Mistakes to avoid

Don’t assume XOR key length > plaintext length; implement modulo indexing correctly.

Ensure you check for zeros in ciphertext (they correspond to ASCII NUL \x00 characters — which are valid semi-ciphertext bytes here)

## Resources

1. https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange
2. https://cryptopals.com/
