# VaultDoor3

## Challenge Description 
This vault uses for-loops and byte/char arrays. The source code for this vault is here: VaultDoor3.java (Author: Mark E. Haase)

## Solution
The provided Java source code, VaultDoor3.java, implements a checkPassword method. This method takes our input (the 32-character string inside picoCTF{...}), scrambles it into a 32-character buffer array, and then compares that buffer to a hard-coded target string.

Our goal is to find the original 32-character password string that, when scrambled by the Java code, produces the following target:

"jU5t_a_sna_3lpm18g947_u_4_m9r54f"

The scrambling logic is contained in four distinct for loops, each one writing to a different part of the buffer using characters from password at specific indices:

Loop 1 (i = 0 to 7): buffer[i] = password.charAt(i);

Writes to buffer[0...7] using password[0...7].

Loop 2 (i = 8 to 15): buffer[i] = password.charAt(23-i);

Writes to buffer[8...15] using password indices in reverse (e.g., buffer[8] gets password[15]).

Loop 3 (i = 16 to 30, evens): buffer[i] = password.charAt(46-i);

Writes to buffer[16, 18, ..., 30] using password indices from a different reversed range.

Loop 4 (i = 31 to 17, odds): buffer[i] = password.charAt(i);

Writes to buffer[31, 29, ..., 17] using password[31, 29, ..., 17].

Reversal Strategy
Since the scrambling process is deterministic (it's just a shuffle, with no data loss), we can reverse it.

The Java code defines the relationship buffer[i] = password[f(i)], where f(i) is the function defined by the loops (e.g., f(i) = i, f(i) = 23-i, etc.).

We have the buffer (the target string) and want to find the password. We can invert the logic: for each loop, we can say password[f(i)] = buffer[i].

We will write a script that creates an empty 32-character array (for our password) and then iterates through the exact same loops. In each loop, instead of reading from password and writing to buffer, we will read from our target string and write to our password array at the correct, "unscrambled" index.

Solution Script (Python)
This script performs the direct reversal and includes a "sanity check" function to verify the answer.

```python

def solve():
    # The hard-coded string from the Java file
    target = "jU5t_a_sna_3lpm18g947_u_4_m9r54f"
    
    # Create an empty list to hold our unscrambled password
    # Using a placeholder '_' to see if all indices get filled
    password_chars = ['_'] * 32
    
    # We now reverse the logic from the Java code.
    # We'll read from the `target` (buffer) and write to `password_chars`.
    
    # Loop 1: buffer[i] = password.charAt(i)
    # Inverse: password[i] = target[i]
    for i in range(0, 8):
        password_index = i
        password_chars[password_index] = target[i]
        
    # Loop 2: buffer[i] = password.charAt(23-i)
    # Inverse: password[23-i] = target[i]
    for i in range(8, 16):
        password_index = 23 - i
        password_chars[password_index] = target[i]
        
    # Loop 3: buffer[i] = password.charAt(46-i)
    # Inverse: password[46-i] = target[i]
    for i in range(16, 32, 2):
        password_index = 46 - i
        password_chars[password_index] = target[i]
        
    # Loop 4: buffer[i] = password.charAt(i)
    # Inverse: password[i] = target[i]
    for i in range(31, 16, -2): # (i = 31, 29, ..., 17)
        password_index = i
        password_chars[password_index] = target[i]
        
    final_password = "".join(password_chars)
    return final_password

def verify(password):
    """
    This function simulates the original Java code's
    scrambling logic to confirm our password is correct.
    """
    target = "jU5t_a_sna_3lpm18g947_u_4_m9r54f"
    buffer = [''] * 32
    
    # Simulate Loop 1
    for i in range(0, 8):
        buffer[i] = password[i]
    # Simulate Loop 2
    for i in range(8, 16):
        buffer[i] = password[23 - i]
    # Simulate Loop 3
    for i in range(16, 32, 2):
        buffer[i] = password[46 - i]
    # Simulate Loop 4
    for i in range(31, 16, -2):
        buffer[i] = password[i]
        
    simulated_target = "".join(buffer)
    
    print(f"   Original Target: {target}")
    print(f"Simulated Target: {simulated_target}")
    return simulated_target == target

--- Main execution ---
print("Attempting to reverse the password...")
password = solve()

print(f"Reconstructed Password: {password}")

print("\nVerifying the reconstructed password...")
if verify(password):
    print("Verification SUCCESSFUL!")
    print(f"\nFinal Flag: picoCTF{{{password}}}")
else:
    print("Verification FAILED.")
Script Output
Attempting to reverse the password...
Reconstructed Password: jU5t_a_s1mpl3_an4gr4m_4_u_79958f

Verifying the reconstructed password...
   Original Target: jU5t_a_sna_3lpm18g947_u_4_m9r54f
Simulated Target: jU5t_a_sna_3lpm18g947_u_4_m9r54f
Verification SUCCESSFUL!

Final Flag: picoCTF{jU5t_a_s1mpl3_an4gr4m_4_u_79958f}
```
## Flag
```
picoCTF{jU5t_a_s1mpl3_an4gr4m_4_u_79958f}
```

## Concepts learnt
Algorithmic Reversal: The core concept is inverting a deterministic algorithm. Because the scrambling process is a permutation (just rearranging data) and is non-lossy (no data is destroyed, e.g., via XOR or AND operations), it can be perfectly reversed. We simply inverted the assignment: dest[i] = source[f(i)] becomes source[f(i)] = dest[i].

Static Analysis: We solved this challenge by just reading the VaultDoor3.java source code (static analysis). We didn't need to run or debug the compiled program (dynamic analysis). This process involved translating the procedural logic of the for loops into a declarative mapping of indices.

Index Mapping (Direct Addressing): This challenge is a practical example of index mapping. The Java code uses a set of piecewise functions (one for each loop) to map indices from the password to the buffer. Our Python script computes the inverse of this map to place characters from the target string back into their original password slots.

Procedural Logic & Order of Operations: The order of the loops is critical. The program is a sequence of steps. Loop 3 writes to even indices (16, 18, ...) and Loop 4 writes to odd indices (17, 19, ...) in the same range. If they had overlapped, the last write would be the one that counts. Our reversal script must follow the same logic to correctly reconstruct the state.

Verification by Simulation: The verify function is a crucial step. It's a "unit test" for our flag. By taking our candidate password and running it through a simulation of the original Java logic, we prove that our reversal is correct. This gives us high confidence in our answer before submitting it.

## Notes
Common Pitfalls:

Off-by-One Errors: Misreading loop bounds (e.g., i < 16 vs. i <= 16) or the start/end of the range() function in Python.

Loop Direction: Forgetting that some loops move backward (like i -= 2) or that the index calculations are not simple (e.g., 23-i and 46-i).

Copy-Paste Errors: Using the wrong target string from the Java file (e.g., from a different version of the challenge) will result in a completely wrong flag.

Alternative Approaches:

Manual Mapping: You could create a 32-element table on paper and manually trace each loop to see which password index lands in which buffer slot. For example: buffer[0] gets password[0], buffer[1] gets password[1], ..., buffer[8] gets password[15], etc. This is slow and highly prone to error but requires no coding.

Symbolic Execution (Advanced): A more advanced method is to use a constraint solver like Z3. You would declare 32 symbolic variables (representing the unknown password characters). Then, you would programmatically assert all the logic from the Java loops. Finally, you would add a constraint that the resulting buffer must equal the target string. The solver would then solve for the 32 unknown variables, giving you the flag. This is overkill for this problem but a powerful technique for more complex challenges.

## Resources

1. https://ctf101.org/reverse-engineering/overview/
2. https://wiki.studsec.nl/books/ctf-guides/page/reverse-engineering
3. https://stackoverflow.com/questions/22544146/reverse-deterministic-shuffle-derive-key
4. https://ericpony.github.io/z3py-tutorial/guide-examples.htm
