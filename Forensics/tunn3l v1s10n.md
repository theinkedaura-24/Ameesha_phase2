
# Challenge Description
The challenge provided a file tunn3l_v1s10n.bmp. The player reports that by editing the file in a hex editor the file opens as a picture. This report documents a deeper forensic re-analysis (byte-level and bitplane inspection) and supplies artifacts for manual review.

## Solution
The file tunn3l_v1s10n.bmp is a valid 24-bit BMP. Parsed header values:
width: 1134
height: 834
bits-per-pixel: 24
pixel data offset: 54 bytes
A direct textual search for picoCTF{ in the raw bytes did not find a complete flag string.
Simple LSB-to-bytes extraction produced noisy output (no clear picoCTF{...} visible in the automated extraction).
To help manual visual inspection, I generated bitplane visualization images (combined B,G,R bitplanes 0..7) and an LSB visualization for the blue channel. These often reveal hidden messages when steganography uses bit-plane or channel LSB hiding.
My solve :
1) File header & quick checks
Verified BMP signature 42 4D at start and read the header fields (width, height, offset, bpp).
Noted file size: 2,893,454 bytes.
Confirmed pixel data begins at offset 54 (standard for BMP with no color table).
2) Quick string search and signature scan
Searched the whole file for common file signatures and ASCII strings (e.g., picoCTF{). No direct occurrences of a full flag were found.
Saved a short list of ASCII snippets found inside the file for reference. Many snippets are just binary noise or image bytes interpreted as text.
3) LSB stream extraction (automated)
Extracted the least-significant bit of each byte from the pixel data and reconstructed bytes by grouping each 8 bits.
Tried both common bit ordering variants (MSB-first and LSB-first) and also tried taking LSBs from every 3rd byte (to approximate per-channel or per-pixel hiding patterns).
Results were noisy / non-printable for the most part — no picoCTF{...} obvious in the stream outputs.
4) Bitplane visualization (recommended manual review)
Built visualization images for bitplanes 0 (LSB) through 7 (MSB) by combining the three channels' bit at that plane into a single grayscale image. This converts subtle bit-level patterns into visible shapes and text if present.
Also produced a single-channel visualization showing only the LSB of the blue channel (a common hiding location).
Files created (in /mnt/data/):
bitplane_combined_0.png … bitplane_combined_7.png
lsb_blue_vis.png
These images often reveal hidden text when opened in an image viewer or when contrast/levels are adjusted.
Artifacts for manual inspection (download/view)
Raw file: /mnt/data/tunn3l_v1s10n.bmp
Automated LSB outputs (binary):
/mnt/data/lsb_msb_bytes.bin
/mnt/data/lsb_lsb_bytes.bin
/mnt/data/bits3_msb.bin
Bitplane visualizations (view these in any image viewer; zoom or adjust contrast if needed):
sandbox:/mnt/data/bitplane_combined_0.png
sandbox:/mnt/data/bitplane_combined_1.png
sandbox:/mnt/data/bitplane_combined_2.png
sandbox:/mnt/data/bitplane_combined_3.png
sandbox:/mnt/data/bitplane_combined_4.png
sandbox:/mnt/data/bitplane_combined_5.png
sandbox:/mnt/data/bitplane_combined_6.png
sandbox:/mnt/data/bitplane_combined_7.png
sandbox:/mnt/data/lsb_blue_vis.png

## Concepts learnt
Bitplane steganography and visualization.
How BMP stores pixel data (bottom-up rows, row padding to 4 bytes).
Practical LSB extraction techniques and ordering issues (MSB-first vs LSB-first).
Use of automated tools vs manual inspection for stego puzzles.

## Notes 
Searched the raw file for embedded PNG/JPG signatures — none found.
Tried both MSB-first and LSB-first grouping when forming bytes from bitstreams.
Tried per-3-byte sampling to emulate per-channel hiding.
Generated visualizations to allow you to easily spot human-readable content.

## Resources 
Python + Pillow (for bitplane images)
strings, xxd (recommended locally)
zsteg, binwalk, stegsolve (recommended)
