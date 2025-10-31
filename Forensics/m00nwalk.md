# Challenge Description
The challenge provided a single audio file, message.wav. A crucial hint was given: “How did pictures from the moon landing get sent back to Earth?” This clue strongly suggested the audio file contained data encoded using a method similar to that used by the Apollo missions, pointing away from simple steganography and toward analog signal encoding.

## Solution:
Step 1: Inspecting the File
My first action was to play the .wav file. It was immediately clear this was not music or speech. The file contained a series of high-pitched beeps, tones, and screeches, highly reminiscent of an old dial-up modem or a fax machine. This confirmed the audio stream itself was the data.

Running file message.wav confirmed it was a standard PCM WAV file, offering no further clues.

The "moon landing" hint was the key. This directly points to SSTV (Slow-Scan Television), the very technology used to transmit still images from the Apollo missions over low-bandwidth radio links.

Step 2: Setting up to Decode SSTV
To decode the signal, I needed an SSTV decoder. On Linux, the standard tool for this is QSSTV. I also needed a way to "play" the audio file into QSSTV as if it were coming from a microphone. This requires setting up a virtual audio loopback.

I installed the necessary software:
```
sudo apt update
sudo apt install qsstv pavucontrol pulseaudio-utils
```
qsstv: The SSTV decoder.

pavucontrol: A graphical mixer for PulseAudio.

pulseaudio-utils: Contains command-line tools (pactl, paplay) to manage audio.

Next, I created the virtual audio "cable" (a null sink):

```
pactl load-module module-null-sink sink_name=virtual-cable
```
This command creates a virtual output device. Any audio sent to this "virtual-cable" sink can be listened to from its "Monitor" source.

Finally, I used pavucontrol (PulseAudio Volume Control) to route the audio:

Recording Tab: I set qsstv's input source to be "Monitor of Null Output" (which is our virtual-cable).

Playback Tab: I prepared to play the file. (This step can also be done while paplay is running).

Step 3: Decoding the Audio
With the audio plumbing ready, I launched the decoder:

Bash

qsstv &
In QSSTV, I set the mode to Auto to let it detect the signal type. (The hint about "CMU's mascot" points to "Scottie the Scottie Dog," so the Scottie 1 mode was my manual backup if Auto failed).

Then, in a separate terminal, I played the .wav file, explicitly directing its output to the virtual-cable sink:

```
paplay -d virtual-cable message.wav
```
Immediately, QSSTV's "waterfall" display lit up, and the main window began slowly rendering an image, line by line, exactly as the audio signal was processed.

Step 4: Extracting the Flag
After the audio file finished, QSSTV had fully rendered the transmitted image, which was automatically saved (typically in ~/.qsstv/images/).

The image itself contained a large block of text:

cGljb0NURntiZWVwX2Jvb3BfaW1faW5fc3BhY2V9
This is a classic Base64 string. I copied the text and decoded it on the command line:

```
echo -n 'cGljb0NURntiZWVwX2Jvb3BfaW1faW5fc3BhY2V9' | base64 --decode
```
(Note: echo -n is important to prevent a trailing newline from being added, which can corrupt the Base64 decoding.)

This command printed the final flag:
```
picoCTF{beep_boop_im_in_space}
```
## Flag:
```
picoCTF{beep_boop_im_in_space}
```
## Concepts learnt:
SSTV (Slow-Scan Television): A method of transmitting still images over low-bandwidth channels (like radio). It works by encoding the image's pixel data as a sequence of audio frequencies.

Audio Signal Analysis: The ability to recognize that an audio file contains modulated data rather than just sound. This pivots the analysis from steganography to signal processing.

PulseAudio Virtual Sink (module-null-sink): A powerful feature of the PulseAudio sound system (common on Linux) that allows for creating "virtual" audio devices. This enables complex audio routing between applications, effectively letting one program "listen" to another program's output.

Multi-stage Decoding: A common CTF pattern where solving one part of the puzzle (decoding the SSTV) reveals another layer of encoding (Base64) that must also be solved.

Base64 Encoding: A standard for encoding binary data into a text-only (ASCII) format. It's not encryption, but a way to ensure data remains intact when transmitted over text-based protocols.
Even after decoding the image, the text itself was Base64, so a simple decoding step gave the final flag.

## Notes:
Initial Failed Tangents: My first instinct was to use digital steganography tools like steghide or analyze the file with strings or binwalk. This failed because the data was not hidden in the audio's metadata or LSB (Least Significant Bit); the audio was the data.

Spectrograms: If the SSTV hint wasn't as obvious, the next step would have been to open the file in a spectrogram viewer like Audacity or Sonic Visualiser. A spectrogram plots frequency vs. time and would have visually revealed the structured, non-random patterns of the SSTV signal.

Alternative Tools: While qsstv is excellent, gqrx (an SDR tool) or even mobile apps (e.g., "Robot36" on Android, "SSTV Slow Scan TV" on iOS) can also decode SSTV if you play the sound from your computer's speaker into your phone's microphone.

System Cleanup: After finishing, you can unload the virtual sink to return your audio system to its normal state with:

```
pactl unload-module module-null-sink
```

## Resources:
1. http://users.telenet.be/on4qz/qsstv/index.html
2. https://www.audacityteam.org/
3. http://www.arrl.org/slow-scan-television
