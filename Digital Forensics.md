# HIDE AND SEEK
Sakamoto’s at it again with a game of hide and seek, but this time, it’s not with Shin or his daughter. An old friend hid some secret data in this image. Can you find it before the others do? 
Hint: Even in retirement, Sakamoto never loses at hide and seek. Maybe stegseek can help you keep up. 

## Solution:
The challenge provided an image named sakamoto.jpg and a hint pointing towards a tool called stegseek. My initial thought process was to use this tool to brute-force any steganography hidden in the image.

Step 1: Setting up the environment I attempted to run stegseek, but it was not installed on my system. I decided to build it from source. I cloned the repository from GitHub.
During the build process using cmake, I ran into a fatal error: mcrypt.h: No such file or directory. This indicated I was missing necessary development libraries. I resolved this by installing libmcrypt-dev, libmhash-dev, and libjpeg-dev.


```

# Cloning the repository
git clone https://github.com/RickdeJager/stegseek.git

# Installing dependencies after initial build failure
sudo apt install libmcrypt-dev libmhash-dev libjpeg-dev build-essential cmake

# Building the tool
cd stegseek/build
cmake ..
make -j$(nproc)
```
Step 2: Acquiring a Wordlist Once stegseek was compiled, I attempted to run it using the standard rockyou.txt wordlist. However, the file was missing from my /usr/share/wordlists/ directory. I had to download the wordlist manually to proceed.


```

# Downloading rockyou.txt
wget -O rockyou.txt https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt

```
Step 3: Cracking the Steganography With the tool built and the wordlist ready, I ran stegseek against sakamoto.jpg. The tool quickly found the passphrase "iloveyou1" and extracted the hidden file.

```
# Running stegseek
stegseek /mnt/c/Users/admin/Downloads/sakamoto.jpg ~/rockyou.txt

# Output
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek
[i] Found passphrase: "iloveyou1"
[i] Original filename: "flag.txt".
[i] Extracting to "sakamoto.jpg.out".
```
Step 4: Reading the Flag I checked the extracted output file, sakamoto.jpg.out. It contained the flag in plain text.

```
# Verifying the output file type
file sakamoto.jpg.out
# sakamoto.jpg.out: ASCII text

# Reading the flag
cat sakamoto.jpg.out
# nite{h1d3_4nd_s33k_but_w1th_st3g_sdfu9s8}
Flag:
nite{h1d3_4nd_s33k_but_w1th_st3g_sdfu9s8}
```
## Concepts learnt:
Steganography Cracking: Learned how to use stegseek, a lightning-fast steghide cracker that can brute-force passwords using a wordlist.

Compiling from Source: Reinforced the process of building C++ tools using cmake and make, including identifying and installing missing dependencies (like libmcrypt and libjpeg).

Wordlist Management: Handling situations where standard wordlists (like rockyou.txt) are missing from the environment.

## Notes:
I initially faced a fatal error: mcrypt.h: No such file or directory which stalled the build. I had to manually identify that libmcrypt-dev was the required package to fix this header issue.

I mistakenly tried to verify the output file with file sa kamoto.jpg.out initially due to a typo (spaces in the command), which threw "No such file or directory" errors before I realized the file was named sakamoto.jpg.out .

## Resources:

https://github.com/RickdeJager/stegseek

***

# Nutrela Chunks

One of my favorite foods is soya chunks. But as I was enjoying some Nutrela today, I noticed a few chunks weren’t quite right. Seems like something’s off with their structure. Could you help me fix these broken chunks so I can enjoy my meal again?

## Solution:

A PNG file was provided but wouldn’t open, suggesting file structure corruption.

The challenge hint said "broken chunks", referring to PNG chunk metadata.

1. Checking the file
```
file nutrela.png
xxd -l 128 nutrela.png
```

Issue 1: Wrong PNG signature
p n g instead of PNG

```
cp nutrela.png fixed.png
echo "00000001: 50 4E 47" > patch.txt
xxd -r - fixed.png < patch.txt
```

2. Checking chunks
```
pngcheck -v fixed.png
```

Issue 2: IHDR was lowercase (ihdr)
So we patched it:

```
echo "0000000C: 49 48 44 52" > fixhdr.patch
xxd -r - fixed.png < fixhdr.patch
```

3. Patch IDAT chunk name
```
echo "00000025: 49 44 41 54" > idat.patch
xxd -r idat.patch final.png
```

4. Patch IEND chunk at correct offset

Found via:
```
xxd -s 0x83810 -l 64 final.png

```
Patched:
```
echo "0008381E: 49 45 4E 44" > proper_iend.patch
echo "00083822: AE 42 60 82" >> proper_iend.patch
cp final.png solved.png
xxd -r proper_iend.patch solved.png
```

5. Final Check
pngcheck -v solved.png


Output:

No errors detected in solved.png


Result: A valid PNG again 

Final Image (Flag inside)

<img width="1000" height="1000" alt="solved" src="https://github.com/user-attachments/assets/4272110b-1521-4d91-a914-366466f2037e" />



## Flag:
```
nite{n0w_y0u_kn0w_ab0ut_PNG_chunk5}
```

## Concepts learnt:

1. PNG File Structure

->Signature

->Chunks (IHDR, IDAT, IEND)

->Chunk format: Length | Type | Data | CRC

2. Hex editing

->Using xxd to patch binary files

3. File magic bytes (headers)

4. pngcheck for validation

5. How case sensitivity matters in chunk types

6. Debugging corrupted metadata

## Notes:

1. Initially tried removing random bytes that unfortunately broke file further 

2. Learned that this CTF intentionally corrupted uppercase chunk names

3. Important lesson: always inspect the actual layout via hex before patching

4. Used sequential validation after each fix

## Resources:

PNG File Structure Reference
https://www.w3.org/TR/2003/REC-PNG-20031110

xxd usage guide
https://linux.die.net/man/1/xxd

pngcheck utility
http://www.libpng.org/pub/png/apps/pngcheck.html

***

# RAR of the Abyss

Two philosophers peer into the networked abyss and swap a secret. Use the secret to decrypt the Abyss’ RAwR and pull your flag from the void.

## Solution:
Step 1 — Understanding the Challenge

We are provided with a file named abyss.pcap

The challenge description hints at philosophers (Camus & Nietzsche) and the word "secret", implying hidden communication in a network capture.

.pcap files generally contain packet captures — best opened in tools like Wireshark or using scripting to extract textual content.

Step 2 — Extracting Strings From PCAP

My initial approach was to extract all readable strings from the binary .pcap file to see if any human chat or credentials appear.

Since I was using IDLE and the file path wasn’t in the same directory initially, I wrote a Python script that asks for file path input to avoid directory confusion.
```

import re

# Ask the user for the full file path
filepath = input("Enter the full path of the pcap file: ")

try:
    # Read file in binary mode
    with open(filepath, "rb") as file:
        data = file.read()

    # Find printable ASCII strings
    strings = re.findall(rb"[ -~]{6,}", data)

    print("\nExtracted Text Strings:\n")
    for s in strings:
        try:
            print(s.decode())
        except:
            pass

except FileNotFoundError:
    print("\nError: File not found. Please check the path and try again.")
except Exception as e:
    print("\nAn error occurred:", e)

Output
Camus: One must imagine Sisyphus happy but are we happy ?
Nietzsche: You will be happy after reading my latest work
Camus: whats the password ?
Nietzsche: b3y0ndG00dand3vil
Camus: thanks
```

Observation

The philosophers’ clue from the description appears directly in this output and one line clearly contains a password:
```
b3y0ndG00dand3vil
```

Step 3 — Using the Password

After extracting the password, I checked the provided challenge files and found a compressed archive inside the same package.

I used 7-Zip and attempted to extract it. It prompted for a password — I entered the one retrieved from the pcap conversation:
```
b3y0ndG00dand3vil
```

Extraction succeeded and produced a flag.txt file.

Step 4 — Getting the Flag

Opening flag.txt revealed:
```
nite{thus_sp0k3_th3_n3tw0rk_f0r3ns1cs_4n4lyst}
```
## Flag:
```
nite{thus_sp0k3_th3_n3tw0rk_f0r3ns1cs_4n4lyst}
```
## Concepts Learnt:

Network Forensics — analyzing packet captures for hidden communication.

PCAP analysis basics

->Strings extraction from binary using regex and scripting

->Understanding common protocols (ICMP, SSDP, DNS)

Using Python for DFIR (Digital Forensics Incident Response)

Identifying hidden messages disguised as normal network traffic

Password-protected archives in CTFs

7-Zip extraction using discovered credentials

## Notes:

Initially, the script printed itself because I accidentally executed it while the .pcap was in a different folder, so Python opened the script file instead of the pcap.

Solved by switching to a version that accepts file-path input.

Highlights importance of keeping challenge files together or knowing file location handling in Python.

Could also have solved using Wireshark and Follow → ICMP Stream, but plaintext string extraction was quicker.

Resources:

https://www.wireshark.org/

https://www.7-zip.org/

https://www.regular-expressions.info/

https://ctf101.org/forensics/overview/
