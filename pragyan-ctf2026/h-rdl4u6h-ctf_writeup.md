# H@rDl4u6H - CTF\_writeup

The Joker left only one thing at the crime scene this time: a single file named `smile.bin`.\
No threats, no riddles, no recordings – just that one file and a smear of blood on the wall that read:\
"A joke's only funny when you get it."

GCPD's cyber forensics division believes the file contains a hidden message, but every attempt to inspect it has turned up nothing but noise, corruption, and formats that make no sense together. It's almost as if the Joker stitched different pieces of evidence into one corpse of a file, daring someone to pull it apart.

If there is a message buried in there – in the static, in the distortion, in the silence between the bits – you'll have to find it yourself. The clock is ticking. He's waiting for someone to laugh.

***

## Initial File Analysis

I began with basic file thingies:

{% code title="bash" %}
```bash
file smile.bin
```
{% endcode %}

Output:

```
smile.bin: data
```

Not helpful. I scanned for embedded files with binwalk:

{% code title="bash" %}
```bash
binwalk smile.bin
```
{% endcode %}

The scan revealed a **7-Zip archive** embedded at offset `0xD75F4` (882,164 bytes into the file).\
Also, at the very beginning of the file, buried in the junk data, I found a Base64 string:

```
aHR0cHM6Ly9naXRodWIuY29tL3NuaXBlcmxpbmUwNDcvQXVkaW8tU3RlZ2Fub2dyYXBoeS1DTEk=
```

Decoded:

```
https://github.com/sniperline047/Audio-Steganography-CLI
```

This pointed to a tool for hiding messages in audio using Least Significant Bit (LSB) steganography. I kept it in mind just in case...

***

## Carving the File

The file structure was clearly stretched together like the challenge said. I separated the file into its constituent parts:

{% stepper %}
{% step %}
### Corrupted data + Audio (WAV)

First 882,164 bytes contained corrupted data and an audio stream (WAV).
{% endstep %}

{% step %}
### 7-Zip archive

A 7-Zip archive was embedded in the middle of the file at offset `0xD75F4`.
{% endstep %}

{% step %}
### Slack data

Trailing bytes at the end — some residual "slack" data.
{% endstep %}
{% endstepper %}

I wrote a Python script to split the file:

{% code title="split_smile_bin.py" %}
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
{% endcode %}

This produced three items to examine:

* `extracted_audio.wav`
* `extracted_archive.7z` (password protected)
* `slack_data.bin`

<figure><img src="../.gitbook/assets/Untitled.png" alt=""><figcaption></figcaption></figure>

***

## Extracting the Audio Passphrase

I inspected `extracted_audio.wav` using the LSB steganography tool referenced earlier. The tool returned a single clean word:

<figure><img src="../.gitbook/assets/image.jpg" alt=""><figcaption></figcaption></figure>

```
transform
```

This looked very likely to be the password for the 7-Zip archive.

***

## Opening the Archive

With the password:

{% code title="bash" %}
```bash
7z x extracted_archive.7z -ptransform
```
{% endcode %}

Success. Inside was a single file:

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

```
y0uc4n7533m3.png
```

I examined its metadata:

{% code title="bash" %}
```bash
exiftool y0uc4n7533m3.png
```
{% endcode %}

Metadata notes:

* PNG size: 3000×4500 pixels, 8-bit grayscale
* Comments:
  * "uh oh why is the image washed out"
  * "can you hear the wail of the damned"
  * Title: "Grave of the Fireflies"

The image looked nearly blank at first glance&#x20;

just a washed-out grayscale

but the metadata suggested something was hidden .....

***

## Finding the Hidden Password in the Image

After adjusting contrast and brightness, hidden text emerged:

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

```
prgynxoxo
```

This appeared to be a key or password for a later step.



***

## Wrestling with the Slack Data

I opened `slack_data.bin` in a hex editor and saw it began with:

```
**rosetta**
```

Followed by binary data that appeared encrypted. I tried several local approaches (XOR attempts, encodings), but the data remained gibberish.

The structure, however, suggested a PGP-encrypted message.&#x20;

Local `gpg` attempts failed with errors.

&#x20;I used an online PGP decryption tool, supplied the encrypted data and used `"rosetta"` as the passphrase.&#x20;

That successfully produced the following poem:

<img src="../.gitbook/assets/Screenshot 2026-02-08 183405 (1).png" alt="" data-size="original">

<details>

<summary>Decrypted PGP message (click to expand)</summary>

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

</details>



***

## The Mystery of the Washed-Out Image

The PNG looked washed out, but when I zoomed way in I noticed faint dots clustered around the center. In a hex editor I also found massive blocks of `ffffffff` repeating

&#x20;unusual for normal PNG data.

Searching for that artifact suggested the image had been manipulated in the frequency domain (FFT)&#x20;

This came as a surprise after I searched for this specific artifact online.\
\
which fit with the earlier "transform" password

I computed the FFT of the grayscale image to visualize frequency-domain structures.

Script used to compute and save the FFT magnitude spectrum:\
I found it online and edited it to fit the challenge&#x20;

{% code title="fft_visualize.py" %}
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
{% endcode %}

The FFT revealed:

* Circles and dots, something I had never seen before, left me stuck there for quite a long time.





<figure><img src="../.gitbook/assets/CLEAR.png" alt=""><figcaption></figcaption></figure>

Enhancing : <br>

<figure><img src="../.gitbook/assets/CLEAR_cropped_contrast.png" alt=""><figcaption></figcaption></figure>

***

## Decoding the Circular Pattern

Interpreting the poem&#x20;

* "Start at the eastern rim" → 3 o'clock position (rightmost)
* "Eight measured steps counter-clockwise" → move 8 positions (of 16) counter-clockwise = 180°
* "Mirrored chorus" → the opposite half is redundant/mirrored
* "Dark for the answer, and bright for the no" → dark = 1, bright/dot = 0 (binary convention described by poem)
* "Read from innermost to outermost" → ring 1 → ring 21

Luckily, I didn’t have to figure all of this out on my own… we got a little help from an AI agent.

Converting the bitstream to bytes and then to hexadecimal yielded:

You can do this manually (YEAH COUNT THEM BY HAND) or use a script or an AI tool.

```
00 2d 04 0d 08 03 18 10 16 2f 47 57 26 5b 4b 1d 49 5f 05 47 1a
```

(21 bytes of hex.)

***

## Final Decryption: XOR with "prgynxoxo"

I suspected a simple XOR with the image-derived password `prgynxoxo`. I XORed the 21-byte hex sequence with the ASCII repeating key `"prgynxoxo"`:

{% code title="xor_decrypt.py" %}
```python
hex_data = "00 2d 04 0d 08 03 18 10 16 2f 47 57 26 5b 4b 1d 49 5f 05 47 1a"
key = "prgynxoxo"
# Convert hex string to bytes
data = bytes.fromhex(hex_data.replace(" ", ""))
# XOR with the password (repeating)
result = ""
for i, byte in enumerate(data):
    result += chr(byte ^ ord(key[i % len(key)]))
print(result)
```
{% endcode %}

Output:

```
p_ctf{why_50_53r10u5}
```

✅ Flag Accepted!



***

So what do we learn from this? Not much, really. I liked the idea of the FFT, but it was too vague and needed more explanation, at least in the image metadata. Without further reading of the poem, I would have been completely stuck without the flag. The poem was hard to understand and even harder to extract the correct hex characters from after a while, as I kept mixing them up. I only confirmed them after trying multiple times. Definitely a fun challenge, but also a very weird and difficult one.&#x20;

~~_**The joker keeps laughing at you.**_~~
