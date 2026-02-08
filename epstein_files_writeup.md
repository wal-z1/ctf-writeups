# The Epstein Files – CTF Write-up

This challenge was a wild ride through PDF forensics, cryptography, and classic CTF misdirection. I started with a PDF file called **`contacts.pdf`**, supposedly part of an "ongoing investigation," and my goal was to find the hidden flag in the format **`pctf{...}`**.

---

## Starters....

The first thing I did, as usual in file-forensics challenges, was to run basic file analysis:

```bash
file contacts.pdf
```

```
contacts.pdf: PDF document, version 1.4
```

Nothing unusual there so.... Time to dig deeper with **`strings`** to see if there was any human-readable text hiding in the file:

```bash
strings contacts.pdf > strings_output.txt
```

---

## Discovery: The Hidden Comment

While scanning through the strings output, I found something interesting around line 176760. There was a PDF comment with **`/Hidden`**:

```bash
strings contacts.pdf | sed -n '176760,176780p'
```

```
/Pg 1723 0 R
/K[2 ]
endobj
1730 0 obj
<</Type/StructElem
/S/Div
/P 5390 0 R
/Pg 1723 0 R
/K[1731 0 R  ]
% /Hidden (3e373f283d312d25222332362c3d2e292322)
endobj
```

That hex string **`3e373f283d312d25222332362c3d2e292322`** looked suspicious. I saved it for later "might be useful"

---

## Extracting PDF Text & Finding the XOR Key

At first, I didn't know what to do with that hex code, so I decided to extract all readable text from the PDF (with the Encoded Image Files). I used an **online PDF parser tool** to get a clean text extraction from the document.

I opened the extracted text file and started **reading through it manually**. After scrolling through pages of contact information and documents, I finally found the crucial hint buried in the text:

```
XOR KEY: JEFFREY
```

Perfect... Now I could decode that hidden hex string.

---

## XOR Decoding

a quick Python script to XOR the hex string with the key **`JEFFREY`**:

```python
hex_string = "3e373f283d312d25222332362c3d2e292322"
key = "JEFFREY"

# Convert hex to bytes
data = bytes.fromhex(hex_string)

# XOR with the key
result = ""
for i, byte in enumerate(data):
    result += chr(byte ^ ord(key[i % len(key)]))

print(result)
```

```bash
python3 xor_decode.py
```

Output:

```
trynottogetdiddled
```

Interesting... But this wasn't the flag
;\_;
I kept exploring.

---

## Down the Rabbit Holes

At this point, I went down multiple investigative paths:

### Steganography Attempts

```bash
# (checking the inside images might have hidden data within them)
pdfimages -all contacts.pdf image

# steghide
for img in image-*.jpg; do
    steghide extract -sf "$img" -p "trynottogetdiddled" 2>/dev/null
done

# zsteg
for png in image-*.png; do
    zsteg -a "$png"
done
```

No luck.

### Metadata Analysis

```bash
exiftool contacts.pdf
exiftool image-*.jpg | grep -i "comment\|description"
```

Nothing useful.

### Stream Analysis

```bash
# decompress all PDF streams
qpdf --qdf --object-streams=disable contacts.pdf uncompressed.pdf

#
mutool extract contacts.pdf
```

Still nothing.

### Font File Examination

```bash
#  fonts from PDF
pdffonts -loc contacts.pdf

# checked font files for hidden data
for font in font-*.pfa font-*.ttf; do
    strings "$font" | grep -E "pctf|flag|hidden"
done
```

Dead end.

---

## The EOF Trick

After exhausting those options, I remembered a classic PDF forensics technique: **data hidden after the EOF marker**. PDFs officially end with **`%%EOF`**, but sometimes data is appended after that marker.

Let me check the file size and EOF location:

```bash
ls -lh contacts.pdf
```

```
-rw-r--r-- 1 user user 14M Feb 6 15:23 contacts.pdf
```

```bash
grep -oba "%%EOF" contacts.pdf | tail -1
```

```
13875932:%%EOF
```

The file is 14MB, but EOF is at byte 13875932. There's definitely data after ...

### Extracting Post-EOF Data

```bash
# everything after EOF goes into a bin file
tail -c +13875933 contacts.pdf > after_eof.bin

# Check what we got
file after_eof.bin
```

```
after_eof.bin: PGP symmetric key encrypted data - AES with 256-bit key salted & iterated - SHA512
```

**Jackpot!**
It's a PGP-encrypted file. And I the password is **`trynottogetdiddled`**.

---

## PGP Decryption

Time to decrypt:

```bash
gpg --decrypt --passphrase "trynottogetdiddled" after_eof.bin > decrypted.bin
```

```
gpg: AES256.CFB encrypted data
gpg: encrypted with 1 passphrase
```

Success! Let's see what's inside:

```bash
cat decrypted.bin
```

```
cpgs{96a2_a5_j9l_u8_0h6p6q8}
```

This looks like the flag, but it's encoded.

---

## Final Decoding: ROT18

The string starts with **`cpgs`**, which should be **`pctf`**. Let me check the cipher:

- `c` → `p` (shift of +13)
- `p` → `c` (shift of +13)
- `g` → `t` (shift of +13)
- `s` → `f` (shift of +13)

That's **ROT13** for letters! But what about the numbers?

I applied ROT13 first:

```python
import codecs

encrypted = "cpgs{96a2_a5_j9l_u8_0h6p6q8}"
rot13 = codecs.decode(encrypted, 'rot13')
print(rot13)
```

```
pctf{96n2_n5_w9y_h8_0u6c6d8}
```

The letters decoded, but the numbers stayed the same.

**ROT5** for digits (ROT13 + ROT5 = **ROT18**).

### ROT5 Decoding

```python
def rot5(char):
    if char.isdigit():
        return str((int(char) + 5) % 10)
    return char

encrypted = "pctf{96n2_n5_w9y_h8_0u6c6d8}"
result = "".join(rot5(c) if c.isdigit() else c for c in encrypted)
print(result)
```

```
pctf{41n7_n0_w4y_h3_5u1c1d3}
```

Reading it in : _"ain't no way he suicide"_

---

## Final Flag

```
pctf{41n7_n0_w4y_h3_5u1c1d3}
```

✅ **Flag accepted!**

---

Overall, a really fun challenge that combined multiple forensics and crypto techniques... The most satisfying moment was discovering the PGP-encrypted data after the EOF marker and realizing the XOR-decoded string was the decryption passphrase.
