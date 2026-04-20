# 2d to 3d - ctf writeup

got a file called
`2d_to_3d.png`

just opened it and started poking around

---

## first checks

ran the usual

```bash
file 2d_to_3d.png
```

png RGBA
1024 x 1024

nothing unusual there

checked structure too

```bash
pngcheck -v 2d_to_3d.png
```

all chunks look normal
just a lot of IDAT blocks
no weird appended data

metadata

```bash
exiftool -a 2d_to_3d.png
```

### exiftool

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/exiftool-2d-3d.png)

nothing hiding there either

---

## looking at it

image looks messy
kind of structured noise

### original

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/2d_to_3d.png)

didnt look like typical LSB stuff
so i checked channels separately

---

## alpha channel

this is where things got interesting

alpha values were very specific

i pulled those alpha stats with this

```bash
python3 - << 'PY'
from PIL import Image
import numpy as np

img = np.array(Image.open('2d_to_3d.png').convert('RGBA'))
a = img[:, :, 3]

u, c = np.unique(a, return_counts=True)
print('unique values:', len(u))
print('range:', int(u.min()), 'to', int(u.max()))
print('total count per value (min/max):', int(c.min()), int(c.max()))

row_counts = [np.bincount(a[y], minlength=256) for y in range(a.shape[0])]
row_counts = np.array(row_counts)
print('each row has each value this many times (min/max):', int(row_counts.min()), int(row_counts.max()))
PY
```

it printed this

```text
unique values: 256
range: 0 to 255
total count per value (min/max): 4096 4096
each row has each value this many times (min/max): 4 4
```

for a normal random-looking image this usually wont be that perfect
you'd normally see uneven counts like different min/max per value and per row
so this was a big sign alpha wasnt just transparency noise

- exactly 256 unique values
- range 0 to 255
- each row has every value exactly 4 times
- total count per value is 4096

that is way too clean to ignore

so instead of thinking transparency
treated alpha as data

---

## what it's doing

image width is 1024
256 \* 4

so split horizontally into 4 sections

for each pixel

- which section it belongs to comes from x
- where it should go comes from alpha

so for every pixel

```python
q = x // 256
v = alpha[y, x]
```

why exactly this and not something else:

- `q = x // 256` because width is 1024 and that is `4 * 256`, so x naturally falls into 4 blocks (`0..255`, `256..511`, `512..767`, `768..1023`)
- i did not use `q = x % 4` because i needed 4 full 256-wide groups, not interleaving every 4th pixel
- `v = alpha[y, x]` because alpha already gives values `0..255`, which is exactly a valid x index in each rebuilt 256-wide layer
- i did not use `v = x % 256` because then alpha data is ignored, and that clean alpha distribution pattern would be pointless
- i kept `y` the same (`layers[q][y][v]`) because the row pattern was already consistent; alpha was telling column placement, not row placement

so in short: x picks which of the 4 chunks, alpha picks the new column inside that chunk

place it into

```python
layers[q][y][v] = rgb[y, x]
```

---

## rebuilding

script

```python
from PIL import Image
import numpy as np

img = np.array(Image.open('2d_to_3d.png').convert('RGBA'))
rgb = img[:, :, :3]
a   = img[:, :, 3]

H, W = a.shape

layers = np.zeros((4, H, 256, 3), dtype=np.uint8)

for y in range(H):
    for x in range(W):
        q = x // 256
        v = int(a[y, x])
        layers[q, y, v] = rgb[y, x]

for i in range(4):
    Image.fromarray(layers[i], 'RGB').save(f'cand_layer_{i}.png')
```

this gives 4 separate images

---

## combining

tried a few ways to put them together

the one that worked was just sticking layer 0 and layer 1 side by side

```python
pair01 = np.concatenate([layers[0], layers[1]], axis=1)
```

### result image

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/cand_pair_concat_01.png)

---

## result

text shows up clearly in that combined image

---

## flag

```
UDCTF{5T0N3_0C34N}
```
