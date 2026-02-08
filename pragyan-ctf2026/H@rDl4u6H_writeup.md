# H@rDl4u6H – CTF Write-up
The Joker left only one thing at the crime scene this time: a single file named smile.bin.
No threats, no riddles, no recordings – just that one file and a smear of blood on the wall that read:
"A joke's only funny when you get it."
GCPD's cyber forensics division believes the file contains a hidden message, but every attempt to inspect it has turned up nothing but noise, corruption, and formats that make no sense together. It's almost as if the Joker stitched different pieces of evidence into one corpse of a file, daring someone to pull it apart.
If there is a message buried in there – in the static, in the distortion, in the silence between the bits – you'll have to find it yourself.
The clock is ticking. He's waiting for someone to laugh.
---
## Initial File Analysis
I began with basic file thingies
```bash
file smile.bin
```
```
smile.bin: data
```
Not helpful. Let me scan for embedded files with binwalk:
```bash
binwalk smile.bin
```
The scan revealed a **7-Zip archive** embedded at offset **0xD75F4** (882,164 bytes into the file).
We also notice that
At the very beginning of the file, buried in the junk data, I found a Base64 string:
```
aHR0cHM6Ly9naXRodWIuY29tL3NuaXBlcmxpbmUwNDcvQXVkaW8tU3RlZ2Fub2dyYXBoeS1DTEk=
```
Decoded:
```
https://github.com/sniperline047/Audio-Steganography-CLI
```
This was a tool for hiding messages in audio using Least Significant Bit (LSB) steganography. I kept it in mind just incase...
---
## Carving the File
The file structure was clearly stetched together like the chall said:
1. **Corrupted data + Audio (WAV)** – First 882,164 bytes
2. **7-Zip archive** – Middle section
3. **Slack data** – Trailing bytes at the end
I wrote a Python script to separate them:
```python
with open("smile.bin", "rb") as f:
    data = f.read()
# Extract WAV portion
wav_part = b"RIFF" + data[4:882164]
with open("extracted_audio.wav", "wb") as f:
    f.write(wav_part)
# Extract archive
archive_part = data[882164:6694798]
with open("extracted_archive.7z", "wb") as f:
    f.write(archive_part)
# Extract slack
slack_part = data[6694798:]
with open("slack_data.bin", "wb") as f:
    f.write(slack_part)
```
This gave me three pieces to examine:
- **extracted_audio.wav**
- **extracted_archive.7z** (password protected)
- **slack_data.bin**
---
## Extracting the Audio Passphrase
I turned to the extracted audio file. Using the LSB steganography tool we get straight forward 
```
transform
```
A single word. Clean. 
This had to be the password for the 7-Zip archive.
---
## Opening the Archive
With the password in hand:
```bash
7z x extracted_archive.7z -ptransform
```
Success. Inside was a single file:
```
y0uc4n7533m3.png
```
Examining its metadata:
```bash
exiftool y0uc4n7533m3.png
```
The PNG was 3000×4500 pixels, 8-bit grayscale. The metadata contained three suspicious comments:
- "uh oh why is the image washed out"
- "can you hear the wail of the damned"
- Title: "Grave of the Fireflies"
The image appeared nearly blank at first glance—just a washed-out grayscale. But the comments suggested I was missing something.
---
## Finding the Hidden Password
When I adjusted the contrast and brightness filters on the PNG, hidden text emerged:
```
prgynxoxo
```
This had to be important, likely a key for final decryption.
---
## Wrestling with the Slack Data
I went back to the slack data. When I examined it in a hex editor, the very first bytes revealed a suspicious header:
```
**rosetta**
```
Followed immediately by binary data clearly encrypted. I tried several approaches ... Any cyberchef recipe that comes to mind ....
"rosetta" as the key, simple XOR operations, various encoding schemes. Nothing worked. The data stubbornly remained gibberish.


Then I noticed the structure looked like a **PGP-encrypted message**. I tried decrypting it locally with gpg , but kept hitting errors. 

Frustrated, I turned to an online PGP decryption tool, pasted the encrypted data, used "rosetta" as the 
passphrase, and—success:

```
In twenty-one circles my message is spun,
A quiet design the shadows have won.
Each ring remembers a fragment I cast;
Half speaks the present, half echoes the past.
Where sunrise touches the eastern rim,
My first soft mark grows faint and slim.
I walk against the daylight’s run,
Eight measured steps, and then I’m done.
The lights I leave are the silence I keep;
The darkened traces are truths that speak.
And once the eighth reveals its part,
A mirrored chorus completes the art.
Thus ring after ring, the pattern will flow;
Dark for the answer, and bright for the no.
Follow their arc with a patient heart,
And you will uncover the code in my art.
```
These looked like instructions, but instructions for what? I didn't have anything circular to decode yet.
---
## The Mystery of the Washed-Out Image

I stared at the PNG for a while. It was mostly blank, but those metadata comments kept nagging at me. 

"Transform" from the audio password. The washed-out appearance. 

Something was hidden in a way normal viewing couldn't reveal.
Then I noticed something odd: when I zoomed way in on the PNG in an image viewer, there were faint dots scattered across what should have been empty space. Hundreds of them.

They formed no obvious pattern at normal zoom levels, but they were definitely there, clustered around the center of the image.

I opened the PNG in a hex editor to see if there were any other clues. That's when I saw it: **massive blocks of `ffffffff` repeating throughout the file**. 

This wasn't normal PNG data. I searched online for "PNG file with ffffffff hex pattern" and found discussions about images that had been manipulated in the frequency domain—specifically through **Fast Fourier Transform (FFT)**.

FFT made perfect sense: it's literally a mathematical "transform," and it reveals patterns hidden in the spatial frequency of an image that are invisible in normal pixel space. The `ffffffff` blocks were likely padding or artifacts from the frequency domain manipulation.

So we script:

```python
import numpy as np
from PIL import Image
import matplotlib.pyplot as plt
# Load image and convert to grayscale
img = Image.open("y0uc4n7533m3.png").convert('L')
img_gray = np.array(img)
# Compute 2D FFT and shift zero-frequency component to center
f_shift = np.fft.fftshift(np.fft.fft2(img_gray))
# Log scaling for better visualization
magnitude_spectrum = np.log1p(np.abs(f_shift))
# Display at high resolution
plt.figure(figsize=(15, 15), dpi=300)
plt.imshow(magnitude_spectrum, cmap='gray', interpolation='nearest')
plt.axis('off')
plt.tight_layout()
plt.savefig('fft_output.png', dpi=300, bbox_inches='tight')
plt.show()
```

The result was :

**21 concentric rings**,

each with **16 angular positions**

filled with dots arranged in perfect angular symmetry.

This was the encoding mechanism.


---
## Decoding the Circular Pattern
Now the decrypted instructions made perfect sense:
- **"Start at the eastern rim"** = Begin at the 3 o'clock position (right side)
- **"Eight measured steps counter-clockwise"** = Move 180 degrees around the circle (8 of 16 positions)
- **"Mirrored chorus"** = The other 8 positions are redundant/mirrored duplicates
- **"Dark for the answer, and bright for the no"** = Gap/dark space = 1, dot/bright = 0 (binary encoding)
- **"Read from innermost to outermost"** = Ring 1 to Ring 21
Reading the pattern from the innermost ring outward, starting at 3 o'clock and moving counter-clockwise, I extracted the binary data and converted it to hex:
```
00 2d 04 0d 08 03 18 10 16 2f 47 57 26 5b 4b 1d 49 5f 05 47 1a
```
21 bytes of hexadecimal data.
---
## Final Decryption: XOR with "prgynxoxo"
I had:
- The hex data from the circular pattern
- The password "prgynxoxo" extracted from the image
- A hunch that XOR was the cipher
```python
hex_data = "00 2d 04 0d 08 03 18 10 16 2f 47 57 26 5b 4b 1d 49 5f 05 47 1a"
key = "prgynxoxo"
# Convert hex string to bytes
data = bytes.fromhex(hex_data.replace(" ", ""))
# XOR with the password (repeating as needed)
result = ""
for i, byte in enumerate(data):
    result += chr(byte ^ ord(key[i % len(key)]))
print(result)
```
Output:
```
p_ctf{why_50_53r10u5}
```
✅ **Flag Accepted!**
