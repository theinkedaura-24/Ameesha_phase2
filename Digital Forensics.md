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
