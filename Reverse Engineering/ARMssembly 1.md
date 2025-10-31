# Challenge name

For what argument does this program print win with variables 87, 3 and 3? 
File: chall_1.S Flag format: picoCTF{XXXXXXXX} -> (hex, lowercase, no 0x, and 32 bits. ex. 5614267 would be picoCTF{0055aabb})

## Solution:

1. Analyzing the main Function

bl atoi: The program takes your command-line argument (a string) and converts it to an integer.

bl func: It calls the func function, passing your integer input (which is in the w0 register).

cmp w0, 0: This is the most important part. It compares the return value from func (which is in w0) to 0.

bne .L4: "Branch if Not Equal." If the return value is not equal to 0, it branches to .L4, which prints "You Lose :(".

Conclusion: To win, the func function must return a value of 0.

2. Analyzing the func Function

str w0, [sp, 12]: Stores our input (let's call it our_input) at stack position [sp, 12].

mov w0, 87 / str w0, [sp, 16]: Stores the number 87 at [sp, 16].

mov w0, 3 / str w0, [sp, 20]: Stores the number 3 at [sp, 20].

mov w0, 3 / str w0, [sp, 24]: Stores the number 3 at [sp, 24].

3. Tracing the Calculation

lsl w0, w1, w0: This is the Logical Shift Left instruction. It loads w1 (87) and w0 (3) and calculates w0 = 87 << 3.

87 * (2^3) = 87 * 8 = 696.

```python

# Python verification
>>> 87 << 3
696
```
sdiv w0, w1, w0: This is the Signed Divide instruction. It loads w1 (696) and w0 (3) and calculates w0 = 696 / 3.

696 / 3 = 232.

```python

# Python verification
>>> 696 / 3
232.0
```

sub w0, w1, w0: This is the Subtract instruction. It loads w1 (232) and w0 (our input) and calculates w0 = 232 - our_input.

This final result is placed in w0 and returned to main.

4. Solving for the Flag

From main, we know the function must return 0 to win.

From func, we know the return value is 232 - our_input.

We solve the equation: 232 - our_input = 0

This gives us our_input = 232.

The correct argument to pass to the program is the decimal number 232.

To get the flag, we convert 232 to a 32-bit (8-character) lowercase hex value.

```
>>> "{:08x}".format(232)
'000000e8'
```

## Flag:
```
picoCTF{000000e8}
```

## Concepts learnt:

1. ARM Assembly: Reading basic ARMv8 (AArch64) assembly instructions.

2. Registers: Understanding the use of registers like w0, w1 (32-bit registers) and sp (stack pointer).

3. Stack Operations: How values are stored on and loaded from the stack using offsets (e.g., [sp, 12]).

4. Assembly Instructions:

str: Store Register (saves a value from a register to memory)

ldr: Load Register (loads a value from memory into a register)

mov: Move (copies a value into a register)

lsl: Logical Shift Left (bitwise shift)

sdiv: Signed Divide (integer division)

sub: Subtract

cmp: Compare (sets flags for branching)

bne: Branch if Not Equal (conditional jump)

## Notes:

1. This was a static analysis challenge. We didn't need to run or debug the binary; we just had to read the assembly code and understand the logic.

2. The core logic boiled down to solving the equation (87 << 3) / 3 - x = 0.

3.  The challenge description's hint about "variables 87, 3 and 3" was the key to confirming we were tracing the correct numbers.


## Resources:

1. https://youtube.com/watch?v=gh2RXE9BIN8
2. https://youtube.com/playlist?list=PLMB3ddm5Yvh3gf_iev78YP5EPzkA3nPdL
3. https://azeria-labs.com/writing-arm-assembly-part-1/
4. https://godbolt.org/
5. https://www.rapidtables.com/convert/number/decimal-to-hex.html
