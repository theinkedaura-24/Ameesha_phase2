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

## Resources:

https://www.7-zip.org/

https://www.regular-expressions.info/

https://ctf101.org/forensics/overview/

***
# NineTails

Looks like I got a little too clever and hid the flag as a password in Firefox, tucked away like one of NineTails’ many tails. Recover the logins and the key4 and let it guide you to the flag.
Hint: I named my Ninetails j4gjesg4, quite a peculiar name isn't it?

## Solution

The downloadable file provided in the challenge was a large .rar archive, and after extracting it, I received a forensic disk image .ad1 file.

.ad1 is an AccessData forensic image format, typically used for disk recovery and digital forensics. It needs to be opened with a forensic tool like FTK Imager.

The challenge description hinted at Firefox passwords stored in the system: specifically logins.json and key4.db files, which are normally stored inside a Firefox profile directory.


1. Extract and mount the AD1 image

I opened the .ad1 file using FTK Imager:

Open FTK Imager
Navigate to File → Add Evidence Item
Select Image File
Choose the provided ninetails.ad1 file
Once loaded, FTK Imager allowed us to cleanly browse the filesystem contained inside the disk image.

Then I navigated through the evidence tree and located:

Users → <user> → AppData → Roaming → Mozilla → Firefox → Profiles

Inside this directory, I found a suspicious profile folder:

j4gjesg4.default-release


which matched the challenge hint.

<img width="1902" height="1108" alt="image" src="https://github.com/user-attachments/assets/8631ef48-4e20-4b7d-ac25-f862828a0089" />


I exported the entire folder:

Right-click → Export Files → saved to /Downloads

[Screenshot-3: Export folder menu]

2. Decrypt Firefox Saved Passwords

Since Firefox stores encrypted passwords in:

logins.json
key4.db


I needed to decrypt them using Ubuntu (WSL).

First installed required dependencies:
```

sudo apt update
sudo apt install python3 python3-pip sqlite3 git
pip3 install pycryptodomex

```
Then cloned the decryption tool:
```
git clone https://github.com/unode/firefox_decrypt.git
cd firefox_decrypt
```

Finally, ran the script against the exported Firefox profile:
```
python3 firefox_decrypt.py /mnt/c/Users/admin/Downloads/j4gjesg4.default-release/
```
Terminal Output
```
ameesha_goel@DESKTOP-7U7A055:~/firefox_decrypt$ python3 firefox_decrypt.py /mnt/c/Users/admin/Downloads/j4gjesg4.default-release/
2025-12-01 02:16:03,001 - WARNING - profile.ini not found in /mnt/c/Users/admin/Downloads/j4gjesg4.default-release/
2025-12-01 02:16:03,001 - WARNING - Continuing and assuming '/mnt/c/Users/admin/Downloads/j4gjesg4.default-release/' is a profile location

Website:   https://www.rehack.xyz
Username: 'warlocksmurf'
Password: 'GCTF{m0zarella'

Website:   https://ctftime.org
Username: 'ilovecheese'
Password: 'CHEEEEEEEEEEEEEEEEEEEEEEEEEESE'

Website:   https://www.reddit.com
Username: 'bluelobster'
Password: '_f1ref0x_'

Website:   https://www.facebook.com
Username: 'flag'
Password: 'SIKE'

Website:   https://warlocksmurf.github.io
Username: 'Man I Love Forensics'
Password: 'p4ssw0rd}'
```

The flag was split across accounts, requiring reconstruction from fragments.

## Flag
```
GCTF{m0zarella_p4ssw0rd}
```
## Concepts learnt

Digital Forensics	Working with forensic disk images such as .ad1

FTK Imager	A forensic tool used to mount and extract files from disk images

Firefox Password Storage	Firefox uses logins.json and key4.db along with certificates (cert9.db)

NSS (Network Security Services)	Backend used to encrypt and decrypt password store

Password Decryption	Using python tools like firefox_decrypt.py

## Notes

I initially extracted only logins.json and key4.db individually, but the script failed with:

ERROR - Couldn't initialize NSS, maybe this is not a valid profile

Learned that the script requires the entire Firefox profile directory because it also needs cert9.db to reconstruct keys.

The password fragments misleadingly looked like different login credentials until I recognized they formed a valid CTF flag format.

## Resources

https://www.exterro.com/digital-forensics-software/ftk-imager-pro

https://github.com/unode/firefox_decrypt

Wireshark

https://support.mozilla.org/en-US/kb/profiles-where-firefox-stores-user-data

***
## Redraw

Her screen went black and a strange command window flickered to life. Moments later the system crashed.
By sheer luck, a memory dump was recovered.
There are three flags, each hidden in a different part of RAM.
Use Volatility 2 plugins to uncover command output, MSPaint artifacts, and a hidden archive deep in memory.

# Solution:

This challenge used a Windows 7 memory dump and required using Volatility 2 to extract evidence of:

a mysterious command window (Flag 1)

remnants of a MSPaint drawing (Flag 2)

a hidden RAR archive (Flag 3)

Identify Image Profile

Before using Volatility, we identify the correct profile:
```
./volatility_2.6_lin64_standalone -f MemoryDump_Lab1.raw imageinfo
```

This returns:
```
Suggested Profile(s) : Win7SP1x64, Win7SP0x64, Win2008R2SP0x64...
Image date and time : 2019-12-11 14:38:00 UTC+0000
```

We select:
```
--profile=Win7SP1x64
```
Part 1 – Retrieving the Command Window Output (FLAG 1)

The description hinted that a black command window flashed before the crash.

Step 1: Identify running processes
```
./volatility_2.6_lin64_standalone -f MemoryDump_Lab1.raw --profile=Win7SP1x64 pslist
```

This confirms a cmd.exe process:
```
cmd.exe                1984
conhost.exe            2692
```
Step 2: Extract command history using consoles
```
./volatility_2.6_lin64_standalone -f MemoryDump_Lab1.raw --profile=Win7SP1x64 consoles
```

This shows:
```
Cmd #0: St4G3$1
ZmxhZ3t0aDFzXzFzX3RoM18xc3Rfc3Q0ZzMhIX0=
```

This Base64 string decodes to:
```
flag{th1s_1s_th3_1st_st4g3!!}
```
Part 2 – Recovering MSPaint Canvas (FLAG 2)

The challenge description states “she was drawing when it happened.”

We confirm MSPaint was running:
```
mspaint.exe  2424
```
Step 1: Dump MSPaint process memory
```
./volatility_2.6_lin64_standalone -f MemoryDump_Lab1.raw --profile=Win7SP1x64 memdump -p 2424 -D ./output
```

This produces:
```
Writing mspaint.exe [2424] to 2424.dmp
```
Step 2: The MSPaint canvas is raw pixel data, not a PNG/JPG

Foremost does NOT help here.

Instead, open the dump in GIMP → Import as Raw Data.

File → Open → select 2424.dmp

Choose “Open as Raw”

Experiment with

RGB

24–32 bits per pixel

Little endian

Width ≈ 1000–2000 px

At first the canvas appears slanted

After rotating 180° and flipping vertically

<img width="732" height="153" alt="image" src="https://github.com/user-attachments/assets/7af20ae0-04e2-4a84-9ae7-923dc554a33a" />

We recover the hidden text:

flag{G00d_BoY_good_girL}

Part 3 – Recovering the Hidden RAR Archive (FLAG 3)

The hint mentioned “a mysterious archive hiding deeper in memory.”

Step 1: Search memory for RAR files
```
./volatility_2.6_lin64_standalone -f MemoryDump_Lab1.raw --profile=Win7SP1x64 filescan | grep -E '\.rar'
```

This gives:

Important.rar

Step 2: Dump the file using its physical offset
```
./volatility_2.6_lin64_standalone -f MemoryDump_Lab1.raw --profile=Win7SP1x64 dumpfiles -Q <offset> -D ./output
```

The dumped file has .dat extension — rename it:

mv file.dat Important.rar

Step 3: Tried extracting
```
unrar e Important.rar
```

Output:

Password is NTLM hash of Alissa's account passwd.

Step 4: Retrieve NTLM hashes
```
./volatility_2.6_lin64_standalone -f MemoryDump_Lab1.raw --profile=Win7SP1x64 hashdump

```
This shows:
```
Alissa Simpson:1003:...:F4FF64C8BAAC57D22F22EDC681055BA6:::
```

Using that hash as password:

F4FF64C8BAAC57D22F22EDC681055BA6


Extracting yields:

flag3.png
<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/21251b05-2200-4d14-a563-236721eb5200" />


Opening the PNG:

flag{w3ll_3rd_stage_was_easy}

## Flags:
```
flag{th1s_1s_th3_1st_st4g3!!}
flag{G00d_BoY_good_girL}
flag{w3ll_3rd_stage_was_easy}
```
## Concepts learnt:

Volatility 2 usage
Profiles, memdump, filescan, consoles, hashdump

Memory forensics basics
RAM contains: screen buffers, clipboard, command history, pixel data, documents, passwords

Raw image reconstruction
Viewing raw framebuffer data with GIMP to recover MSPaint drawings

NTLM hashes usage
Windows stores user credentials as NTLM hashes → usable as passwords for encrypted content

RAR memory extraction
How to reconstruct file objects stored only in RAM

## Notes:

I initially delved into procdump and got highly confused so wasted time, should have instead gone to memdump.

Foremost extracted unrelated images — MSPaint canvas is RAW, not BMP

The slanted image in GIMP was confusing until rotated and flipped

Multiple RAR offsets existed but they all pointed to the same file

The challenge significantly improved my Volatility workflow

## Resources:

Volatility 2 Documentation

https://github.com/volatilityfoundation/volatility/wiki

Articles on Windows memory forensics

Tutorials on GIMP “Import RAW” feature

NTLM hash cracking references

DumpIt documentation

