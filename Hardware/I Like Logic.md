# I Like Logic

We are given a set of logic analyzer capture files (.sal, meta.json, and multiple digital-X.bin files) from a Saleae logic analyzer. 
We must analyze this capture of a digital communication, identify the protocol, decode the data, and recover a hidden flag.

## Solution:
The core task is to reverse engineer a digital signal. The provided files are our only clues. The .sal file is a project file for the Saleae Logic 2 software, while the .bin files represent the raw, captured high/low logic levels for each channel.1. Initial File and Metadata AnalysisFirst, we inspect meta.json. This file provides the capture's context:Sample Rate: 6.25 MHz ($6.25 \times 10^6$ samples per second).Enabled Channels: 5 digital channels, D0 through D4.Device: Logic Pro 16.The presence of 5 channels and a high sample rate strongly suggests a synchronous serial protocol like SPI (Serial Peripheral Interface), which typically uses 4 lines (CLK, MOSI, MISO, CS), or a similar parallel bus. It almost certainly rules out simpler asynchronous protocols like UART, which only require one or two lines (TX/RX) and have no clock.2. Data Ingestion and ParsingThe digital-X.bin files contain the raw captured samples. Each byte in these files represents 8 consecutive samples (1 bit per sample). We use Python and the NumPy library to read these files and convert them into bit arrays, where each element is a 1 (logic high) or 0 (logic low).Pythonimport numpy as np

def load_digital_bin(file_path):
    """Reads a Saleae .bin file and unpacks it into a bit array."""
    # Load all bytes from the file
    data = np.fromfile(file_path, dtype=np.uint8)
    # Unpack each byte (8 bits) into an array of 8 separate bits
    # This gives us the logic level for each sample
    bits = np.unpackbits(data)
    return bits

Load all 5 channels into memory
ch0 = load_digital_bin("digital-0.bin")
ch1 = load_digital_bin("digital-1.bin")
ch2 = load_digital_bin("digital-2.bin")
ch3 = load_digital_bin("digital-3.bin")
ch4 = load_digital_bin("digital-4.bin")

Protocol Identification By visually inspecting the bitstreams (or opening the .sal file in the Logic 2 software), we can deduce the function of each channel:
ch0 (D0): Shows a regular, repeating square wave. This is a Clock (CLK) signal.ch3 (D3): Stays high, then goes low for a long period while data is transferred, then goes high again. This is a classic Chip Select (CS) line, active-low.ch1 (D1): Shows data transitions that align with the clock edges, only during the period when ch3 is low. This is a data line, almost certainly MOSI (Master Out, Slave In) or MISO.4. Writing the DecoderWe can now write a Python function to decode the SPI communication. 
The logic follows the SPI protocol: "When Chip Select is low, sample the MOSI data bit on every rising edge of the CLK.

```python
def decode_spi(clk, mosi, cs):
    """
    Decodes an SPI data stream from clock, data, and chip select bit arrays.
    Assumes data is valid on the rising clock edge and CS is active-low.
    """
    data_bits = []
    # Iterate through all samples, starting from the second one
    for i in range(1, len(clk)):
        
        # Check for the two conditions:
        # 1. Chip Select is active (low)
        # 2. A rising clock edge just occurred (went from 0 to 1)
        if cs[i] == 0 and clk[i-1] == 0 and clk[i] == 1:
            # If both are true, this is a valid bit. Sample the MOSI line.
            data_bits.append(mosi[i])
            
    # Now, group the collected bits into bytes (8 bits)
    bytes_out = []
    for i in range(0, len(data_bits), 8):
        byte_chunk = data_bits[i:i+8]
        # Stop if we have a partial byte at the end
        if len(byte_chunk) < 8:
            break
        
        # Join the bits ('1', '0', ...) and convert from binary string to integer
        val = int(''.join(map(str, byte_chunk)), 2)
        bytes_out.append(val)
        
    # Return the final data as a raw bytes object
    return bytes(bytes_out)

# Decode the data by assigning channels to roles
spi_data = decode_spi(clk=ch0, mosi=ch1, cs=ch3)

# Print the decoded data as ASCII
print(spi_data.decode('ascii', errors='ignore'))
```
Here is a new write-up for the challenge, built on the same core solution but with expanded sections for concepts, notes, and resources.Challenge Analysis: I Like LogicCategory: Hardware / Digital ForensicsProblem: We are given a set of logic analyzer capture files (.sal, meta.json, and multiple digital-X.bin files) from a Saleae logic analyzer. We must analyze this capture of a digital communication, identify the protocol, decode the data, and recover a hidden flag.Solution WalkthroughThe core task is to reverse engineer a digital signal. The provided files are our only clues. The .sal file is a project file for the Saleae Logic 2 software, while the .bin files represent the raw, captured high/low logic levels for each channel.1. Initial File and Metadata AnalysisFirst, we inspect meta.json. This file provides the capture's context:Sample Rate: 6.25 MHz ($6.25 \times 10^6$ samples per second).Enabled Channels: 5 digital channels, D0 through D4.Device: Logic Pro 16.The presence of 5 channels and a high sample rate strongly suggests a synchronous serial protocol like SPI (Serial Peripheral Interface), which typically uses 4 lines (CLK, MOSI, MISO, CS), or a similar parallel bus. It almost certainly rules out simpler asynchronous protocols like UART, which only require one or two lines (TX/RX) and have no clock.2. Data Ingestion and ParsingThe digital-X.bin files contain the raw captured samples. Each byte in these files represents 8 consecutive samples (1 bit per sample). We use Python and the NumPy library to read these files and convert them into bit arrays, where each element is a 1 (logic high) or 0 (logic low).Pythonimport numpy as np

def load_digital_bin(file_path):
    """Reads a Saleae .bin file and unpacks it into a bit array."""
    # Load all bytes from the file
    data = np.fromfile(file_path, dtype=np.uint8)
    # Unpack each byte (8 bits) into an array of 8 separate bits
    # This gives us the logic level for each sample
    bits = np.unpackbits(data)
    return bits

# Load all 5 channels into memory
ch0 = load_digital_bin("digital-0.bin")
ch1 = load_digital_bin("digital-1.bin")
ch2 = load_digital_bin("digital-2.bin")
ch3 = load_digital_bin("digital-3.bin")
ch4 = load_digital_bin("digital-4.bin")
3. Protocol IdentificationBy visually inspecting the bitstreams (or opening the .sal file in the Logic 2 software), we can deduce the function of each channel:ch0 (D0): Shows a regular, repeating square wave. This is a Clock (CLK) signal.ch3 (D3): Stays high, then goes low for a long period while data is transferred, then goes high again. This is a classic Chip Select (CS) line, active-low.ch1 (D1): Shows data transitions that align with the clock edges, only during the period when ch3 is low. This is a data line, almost certainly MOSI (Master Out, Slave In) or MISO.4. Writing the DecoderWe can now write a Python function to decode the SPI communication. The logic follows the SPI protocol: "When Chip Select is low, sample the MOSI data bit on every rising edge of the CLK."Pythondef decode_spi(clk, mosi, cs):
    """
    Decodes an SPI data stream from clock, data, and chip select bit arrays.
    Assumes data is valid on the rising clock edge and CS is active-low.
    """
    data_bits = []
    # Iterate through all samples, starting from the second one
    for i in range(1, len(clk)):
        
        # Check for the two conditions:
        # 1. Chip Select is active (low)
        # 2. A rising clock edge just occurred (went from 0 to 1)
        if cs[i] == 0 and clk[i-1] == 0 and clk[i] == 1:
            # If both are true, this is a valid bit. Sample the MOSI line.
            data_bits.append(mosi[i])
            
    # Now, group the collected bits into bytes (8 bits)
    bytes_out = []
    for i in range(0, len(data_bits), 8):
        byte_chunk = data_bits[i:i+8]
        # Stop if we have a partial byte at the end
        if len(byte_chunk) < 8:
            break
        
        # Join the bits ('1', '0', ...) and convert from binary string to integer
        val = int(''.join(map(str, byte_chunk)), 2)
        bytes_out.append(val)
        
    # Return the final data as a raw bytes object
    return bytes(bytes_out)

# Decode the data by assigning channels to roles
spi_data = decode_spi(clk=ch0, mosi=ch1, cs=ch3)

# Print the decoded data as ASCII
print(spi_data.decode('ascii', errors='ignore'))

Result and Verification
Running the script decodes the raw signal and prints a readable ASCII string:TFCCTF{Th1s_1s_som3_s1mpl3_4rdu1no_f1rmw4re}This result can be independently verified by opening the .sal file in the Saleae Logic 2 software, adding an SPI Analyzer, and assigning D0 as CLK, D1 as MOSI, and D3 as CS. The software's built-in decoder displays the exact same byte stream.


## Flag:
```
TFCCTF{Th1s_1s_som3_s1mpl3_4rdu1no_f1rmw4re}
```
## Concepts Learned

Digital Logic Analysis: This challenge is a practical exercise in capturing and interpreting digital signals. Logic analyzers are the primary tool for debugging hardware communications, showing the precise high/low voltage levels over time.

Synchronous Serial Communication: We learned to identify and decode the Serial Peripheral Interface (SPI), a synchronous protocol because it relies on a shared clock line (CLK). This contrasts with asynchronous protocols (like UART) which have no clock and rely on precise timing (baud rates).

SPI Protocol: We identified the key SPI lines:

CLK (Clock): Synchronizes the data transfer.

CS (Chip Select): An active-low line used by the master to select which "slave" device it wants to talk to.

MOSI (Master Out, Slave In): The data line from the master to the slave.

Raw Binary File Parsing: We learned how to ingest raw binary data from a file using numpy.fromfile and, crucially, how to transform packed bytes of data into a time-series bitstream using numpy.unpackbits.

Hardware Reverse Engineering: Instead of reverse engineering a compiled binary, we reverse-engineered the output of a device's firmware by analyzing its hardware-level electrical signals.

## Notes
Initial Hypothesis (UART): A common first guess for serial data is UART. However, this was quickly ruled out. UART is asynchronous and wouldn't have a dedicated clock line. Furthermore, the bit timing didn't align with any standard baud rates.

The "Easy Path": The inclusion of the .sal project file was a major hint. This file can be opened directly in the free Saleae Logic 2 software, allowing for a no-code solution using the built-in SPI analyzer. Writing the Python script, however, provides a much deeper understanding of the protocol.

MSB vs. LSB: Our script implicitly assumes the data is sent Most Significant Bit (MSB) first, which is the SPI default. If the output was garbled, our next step would have been to test for Least Significant Bit (LSB) first by reversing the bit order of each 8-bit chunk.

The Hint File: The challenge.txt file contained À LFj1iPV Hc9. This appears to be a snippet of the data stream, mis-decoded (e.g., with the wrong bit order or a sampling error), which served as a hint that we were on the right track.

## Resources
Saleae Logic 2 Software: The (free) software used to analyze .sal capture files. https://www.saleae.com/downloads/

Sigrok/PulseView: A popular open-source alternative to the Saleae software that can also analyze logic captures. https://sigrok.org/wiki/PulseView

SparkFun: SPI Protocol: An excellent, clear tutorial on the fundamentals of the Serial Peripheral Interface (SPI). https://learn.sparkfun.com/tutorials/serial-peripheral-interface-spi

NumPy unpackbits Documentation: The official documentation for the key function used to convert packed bytes into a bit array. https://numpy.org/doc/stable/reference/generated/numpy.unpackbits.html



