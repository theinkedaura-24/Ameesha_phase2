# Challenge Description
The challenge came with a mysterious file called message.wav and a hint that said something about “How did pictures from the moon landing get sent back to Earth?”
That clue instantly made me think this wasn’t a normal sound file — probably some kind of signal.

## Solution:
Step 1: Inspecting the file
I started by simply playing the .wav file.
Instead of music or speech, I heard a series of strange beeps and tones — kind of like an old modem connecting or Morse code on steroids.
That convinced me it was some form of encoded data rather than regular audio.

So I checked the file type and metadata:

file message.wav
It showed it was a standard PCM WAV file, which didn’t reveal much more.
The moon hint, though, pushed me toward SSTV (Slow-Scan Television) — the same system used to transmit images from the Apollo missions.

Step 2: Setting up to decode SSTV
To decode SSTV signals, I used a Linux tool called QSSTV.
I installed everything I needed with:
```
sudo apt update
sudo apt install qsstv pavucontrol pulseaudio-utils
```
QSSTV listens to audio input, so I had to make my computer’s audio output loop back into it.
I did that by creating a virtual audio cable:

pactl load-module module-null-sink sink_name=virtual-cable
This creates a virtual output device called virtual-cable.
Then I opened pavucontrol (PulseAudio Volume Control) to route the playback of the WAV file into QSSTV:

In the Recording tab, I set QSSTV’s input to Monitor of Null Output (virtual-cable).
In Playback, I sent the system audio to the virtual-cable output.
Step 3: Decoding the audio
With everything wired up, I launched QSSTV:
```
qsstv &
```
Inside QSSTV, I chose Auto Mode, but remembered the hint about “CMU’s mascot” — which is Scotty the Scottie Dog — so if auto didn’t work, I would try Scottie 1 manually.

Then I played the file through the virtual cable:
```
paplay -d virtual-cable message.wav
```
After a few seconds, I saw QSSTV start drawing lines slowly on the screen — exactly how SSTV images appear in real time.
It felt like watching an old transmission come to life line by line.

Step 4: Extracting the flag
Once the decoding finished, QSSTV saved the image automatically (you can find it in ~/.qsstv/images).

Opening the image showed a block of text in the middle.
It looked like Base64, so I copied it:
```
cGljb0NURntiZWVwX2Jvb3BfaW1faW5fc3BhY2V9
```
Then I decoded it using:
```
echo -n 'cGljb0NURntiZWVwX2Jvb3BfaW1faW5fc3BhY2V9' | base64 --decode
```
Output:
```
picoCTF{beep_boop_im_in_space}
```
## Flag:
```
picoCTF{beep_boop_im_in_space}
```
## Concepts learnt:
SSTV (Slow-Scan Television):
SSTV converts images into audio tones so they can be transmitted over radio and then converted back into pictures.
This is the same technique used by astronauts to send photos from space in the 1960s.

PulseAudio Virtual Sink:
I learnt how to create a “fake” audio device that lets you feed audio from one program to another without physical cables.

Base64 Encoding:
Even after decoding the image, the text itself was Base64, so a simple decoding step gave the final flag.

## Notes:
Initially, I tried basic steganography tools like binwalk and strings thinking there might be hidden data directly in the audio file. That didn’t reveal anything useful.
Only after replaying the audio did I realize it was modulated data.

If QSSTV doesn’t decode properly, check that the input is really the virtual-cable monitor, or try different SSTV modes (Scottie 1 or Martin 1 usually work).

After finishing, you can unload the virtual sink with:

pactl unload-module module-null-sink

## Resources:
HackMD write-up reference (used for verifying commands)
