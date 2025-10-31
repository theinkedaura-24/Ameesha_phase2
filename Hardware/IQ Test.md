# Challenge Description
The challenge presented a large-scale combinational logic schematic. This digital circuit consisted of 36 distinct input pins (labeled x0 through x35) and 12 output pins (y0 through y11). The objective was to determine the circuit's 12-bit binary output for a single, specific 36-bit input value.

## Solution:
FStep 1: Recreate the circuit
The first and most critical step was to meticulously transcribe the provided schematic into a functional simulator. I used CircuitVerse, a browser-based digital logic simulator, for this task due to its ease of use and visual interface.

I digitally reconstructed the entire circuit, carefully connecting all the AND, OR, XOR, and NOT gates exactly as specified. This involved wiring the 36 inputs through multiple layers of logic to finally connect to the 12 outputs.

Step 2: Input value
The challenge provided the input as a single decimal number:

x = 30478191278

This value had to be translated into its binary representation to be applied to the 36 input pins. The 36-bit binary equivalent is:

011100011000101001000100101010101110

A crucial step was correctly mapping this binary string to the input pins. The standard convention, which I followed, is that x0 corresponds to the Least Significant Bit (LSB)—the rightmost bit—and x35 corresponds to the Most Significant Bit (MSB)—the leftmost bit.

x35 (MSB) = 0

x34 = 1

...

x1 = 1

x0 (LSB) = 0

Step 3: Simulating the circuit
With the circuit faithfully rebuilt in CircuitVerse, I applied the 36-bit input vector to the corresponding input pins.

The simulator automatically propagated the signals through the complex web of gates, calculating the final Boolean expression for each of the 12 outputs. After the signals stabilized, the output pins y0 through y11 displayed the resulting 12-bit binary value:

y = 100010011000

Step 4: Deriving the flag
The challenge format required this 12-bit output to be placed inside the flag braces.

Flag: nite{100010011000}

## Flag:
```
nite{100010011000}
```
## Concepts learnt:
Digital Logic Simulation: Using a graphical simulator (CircuitVerse) to model, build, and test a complex digital logic circuit, verifying its output without needing to perform manual Boolean algebra.

Binary Conversion (Number Systems): The necessity of converting between number bases (Decimal to Binary) to interface between human-readable numbers and bit-level digital systems.

Logic Gate Functionality: Reinforced understanding of how fundamental logic gates (AND, OR, NOT, XOR) serve as the building blocks for all complex digital functions.

Input–Output Mapping: The importance of correctly mapping a data vector (the binary string) to the physical pins of a system, including understanding the conventions of LSB (Least Significant Bit) and MSB (Most Significant Bit).

Combinational Logic: This circuit is a prime example of combinational logic, where the output is purely a function of the current inputs. This is distinct from sequential logic, which involves memory (like flip-flops) and where outputs depend on both current inputs and past states.

## Notes:
Verification is Key: The most difficult part of this challenge was not the simulation itself, but the tedious and error-prone task of accurately recreating the circuit. I had to double-check every single connection against the schematic.

Input Bit Ordering: Mistakenly reversing the bit order (i.e., mapping the LSB to x35) would have resulted in a completely different, incorrect output.

"Brute Force" Simulation: This solution method avoids performing the extremely complex Boolean algebra required to simplify the 36-input, 12-output system into a set of equations. Instead, we let the computer "brute force" the calculation for the single input we care about.

Test Cases: Before using the final input, I ran simple test cases (like all 0s or all 1s) to ensure the circuit was wired correctly and behaving as expected.

Leading Zero: The 36-bit binary conversion of 30478191278 starts with a 0. This leading zero is significant and must be included as the input for x35.

## Resources:
https://www.falstad.com/circuit/

https://www.rapidtables.com/convert/number/decimal-to-binary.html

https://www.geeksforgeeks.org/logic-gates-in-digital-electronics/

https://www.allaboutcircuits.com/textbook/digital/chpt-3/logic-gates/


