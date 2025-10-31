# Challenge Description

The uploaded file firmware (1).elf is an AVR firmware ELF for an Arduino-like device. The flag was not visible in plaintext, so I assumed a simple obfuscation (common in firmware CTFs): single-byte XOR.
I brute-forced all 256 single-byte XOR keys, scored results for printable ASCII runs, and inspected the best candidates. 
One key produced a clear brace-delimited payload that looks like a flag.

## Solution
Identify file type
Command: file "firmware (1).elf" showed it is an Atmel AVR ELF (firmware). This suggested we are dealing with compiled microcontroller data rather than a typical Linux binary.
Look for plaintext
Command: strings "firmware (1).elf" | head — no obvious CTF{ or FLAG{ found. So the flag is likely encoded/obfuscated.
Hypothesis
Many firmware CTF tasks hide flags using a single-byte XOR or simple substitution. I decided to brute-force all single-byte XOR keys and inspect printable output.
Brute-force XOR + detect printable runs
Wrote a small Python script that:
Reads the ELF bytes
XORs with keys 0..255
Finds the longest contiguous printable ASCII run for each key
Scores each run using simple heuristics (common word matches, chi-squared letter frequency)
The script surfaced a strong candidate for key 0xA5 (and a few other keys with noisy ASCII-like runs). Key 0xA5 produced a clear brace-delimited payload.
Verify and extract flag
After XOR with 0xA5 the decoded region contains the contiguous substring: TFCCTF{Th1s_1s_som3_s1mpl3_4rdu1no_f1rmw4re}.
Extracting the braces yields CTF{Th1s_1s_som3_s1mpl3_4rdu1no_f1rmw4re}, but the decoded bytes in context contain the prefix TFC immediately before the CTF{...} producing the full contiguous sequence TFCCTF{...}.
Which substring to submit depends on contest rules; the user provided the exact flag to include in the report.
Example Python commands used (abridged)
1) quick brute-force to find printable runs
from pathlib import Path
data = Path('firmware (1).elf').read_bytes()
for k in range(256):
    dec = bytes(b ^ k for b in data)
    #find longest printable run (simple heuristic)
    #print or save candidates where printable run length > threshold

2) decode with key 0xA5 and show braces
k = 0xA5
dec = bytes(b ^ k for b in data)
start = dec.find(b"{")
end = dec.find(b"}", start+1)
print(dec[start-4:end+1])
Terminal outputs (selected, abbreviated)
#file output
```
$ file "firmware (1).elf"
firmware (1).elf: ELF 32-bit LSB executable, Atmel AVR... statically linked, with debug_info
```

#example strings not showing 'CTF{'
$ strings "firmware (1).elf" | grep -i "CTF\|FLAG\|{"
#(no relevant results)
#After XOR with 0xA5 (excerpt of decoded bytes around offset 0xFF):
```
000000EF  A5 A9 31 CD A5 A9 31 CD A5 A9 31 CD A5 54 46 43   ..1...1...1..TFC
000000FF  43 54 46 7B 54 68 31 73 5F 31 73 5F 73 6F 6D 33   CTF{Th1s_1s_som3
0000010F  5F 73 31 6D 70 6C 33 5F 34 72 64 75 31 6E 6F 5F   _s1mpl3_4rdu1no_
0000011F  66 31 72 6D 77 34 72 65 7D A5 A5 B4 81 BA 1B 6A   f1rmw4re}......j
```
(Above: the decoded bytes show TFCCTF{Th1s_...} when a few bytes of context before the brace are included; the brace-delimited token is CTF{Th1s_...})

## Flag:
TFCCTF{Th1s_1s_som3_s1mpl3_4rdu1no_f1rmw4re}

## Concepts learnt:
Firmware analysis basics — identifying AVR ELF firmware vs regular ELF. AVR/embedded firmware often contains vector tables, section names, and smaller instruction sets; typical desktop tools still work for byte-level analysis.
Single-byte XOR obfuscation — common, trivial obfuscation used in CTFs; brute-forcing 256 keys is cheap and effective.
Printable-run heuristics — finding long sequences of printable ASCII after decoding is a robust way to detect correct XOR keys when you lack a crib.
Chi-squared scoring / English-frequency heuristics — statistical tests (e.g., chi-squared against English letter frequencies) help rank candidate decodings when many keys yield ASCII-like noise.

## Notes:
I initially searched for canonical prefixes like CTF{ and flag{ which is a common shortcut. That returned the brace-delimited payload when decoding with 0xA5, but printing more context showed the contiguous TFCCTF{...} sequence. This created confusion over whether the prefix included the extra TFC bytes.
Several other keys produced long printable regions (ASCII-like noise, symbol tables, repeated strings). Hence relying on printable ratio alone leads to false positives; combining heuristics (word matches, chi-sq) is more reliable.
The exact string to submit depends on platform rules; the user instructed to use TFCCTF{...} as the flag in this case.

## Resources:
General references for binary/firmware analysis: file, strings, xxd, Python for scripting.
