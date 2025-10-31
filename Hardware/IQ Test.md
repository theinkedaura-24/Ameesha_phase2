# Challenge Description
The challenge provided a complex digital logic circuit with 36 input signals (x0–x35) and 12 outputs (y0–y11). The task was to analyze the circuit and determine the correct output for a given input value.

## Solution:
For this challenge, I used CircuitVerse — an online digital logic simulator — to recreate and test the provided logic gate circuit.

Step 1: Recreate the circuit
I carefully replicated the logic gate connections exactly as shown in the given schematic using CircuitVerse.
The circuit contained combinations of AND, OR, XOR, and NOT gates, arranged to process 36 inputs and produce 12 output bits.

(Insert your screenshot of the recreated circuit here)

Step 2: Input value
The input given in the challenge was:

x = 30478191278
I converted this decimal number into binary form to use it as the input for the logic circuit.

30478191278 (decimal) = 011100011000101001000100101010101110 (binary)
Each bit in this binary string corresponds to an input from x0 to x35.

Step 3: Simulating the circuit
Using the binary input, I applied each bit to the corresponding input pin (x0–x35) in CircuitVerse and observed the outputs (y0–y11).

After simulation, I obtained the following 12-bit output:

y = 100010011000
Step 4: Deriving the flag
According to the challenge format, the final flag should contain the output bits inside the braces.

Flag: nite{100010011000}

## Flag:
```
nite{100010011000}
```
## Concepts learnt:
Digital Logic Simulation: Using CircuitVerse to build and test digital logic circuits.
Binary Conversion: Converting large decimal numbers into binary for bit-level processing.
Logic Gate Functionality: Understanding how combinations of AND, OR, XOR, and NOT gates create complex Boolean functions.
Input–Output Mapping: Observing how input changes propagate through gates to affect output.

## Notes:
At first, I verified the correctness of all connections by checking each layer of gates.
Binary inputs were applied bit by bit to ensure proper mapping from x0 (LSB) to x35 (MSB).
After running several test cases, the output pattern was confirmed consistent.

## Resources:
CircuitVerse Online Simulator
Binary to Decimal Converter
Logic Gate Basics – GeeksforGeeks
