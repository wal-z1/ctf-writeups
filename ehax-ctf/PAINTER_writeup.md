# painter — CTF\_writeup-EHAX CTF2026

So the challenge description tells you the flag is `EH4X{hihihihi}`. A painter working for a security company tried to social engineer an employee and left behind a USB capture. 

Challenge Author: stapat

---

## getting started

You get a `pref.pcap` file and that's it. I ran tshark to get a quick look at what's inside.

```bash
tshark -r pref.pcap | head -n 30
```

I noticed it was just `URB_INTERRUPT in` repeating over and over. That's the tell for HID traffic, keyboard, mouse, stylus, that kind of thing. Not your typical network data, which was already interesting.

## finding the actual payload

I tried the usual HID fields like `usb.data_fragment` and got nothing. So I dumped one frame in verbose mode to actually read through it carefully.

```bash
tshark -r pref.pcap -c 1 -V
```

I found what I needed:

- URB transfer type: URB_INTERRUPT (0x01)
- Endpoint: 0x81, Direction: IN
- Data length bytes: 7
- Leftover Capture Data: `01000000fdff00`

The Leftover Capture Data is what tshark shows as `usb.capdata`. That's where the real bytes are hiding.

## extracting the data stream

I pulled all the capdata records:

```bash
tshark -r pref.pcap -Y "usb.transfer_type==0x01 && usb.endpoint_address==0x81 && usb.capdata" \
  -T fields -e usb.capdata > capdata.txt
```

Quick check on the lengths:

```bash
tshark -r pref.pcap -T fields -e usb.data_len | sort | uniq -c
```

Output:

```
9888 7
```

I noticed every single report is exactly 7 bytes. Didn't think much of it at first.

## figuring out the report format

I had no idea what the Report ID or deltas were at this point, so I had to google around to understand the HID report format. I found out that keyboard reports are usually 8 bytes, so this was something different. The 7 bytes looked like this:

```
01000000fdff00
0100e1ffcfff00
```

The first byte is always `01`, that's the Report ID. The rest looks like pointer or stylus data. I figured the layout is probably:

- 1 byte: Report ID
- 1 byte: buttons or pen state
- 2 bytes: X delta signed little-endian
- 2 bytes: Y delta signed little-endian
- 1 byte: wheel or extra

So I realized that instead of someone typing the flag, the painter actually drew it. Which yeah .. totally makes sense

## decoding the deltas

I converted each 7-byte hex record into readable numbers using an AWK script that handles the little-endian and signed 16-bit conversion:

```bash
awk '
function hexval(c){ return index("0123456789abcdef", tolower(c))-1 }
function hex2dec(h,   n,i){ n=0; for(i=1;i<=length(h);i++) n=n*16+hexval(substr(h,i,1)); return n }
function s16(u){ return (u>=32768)?u-65536:u }
{
  d=$1
  if(length(d)!=14) next
  btn = hex2dec(substr(d,3,2))
  x = s16(hex2dec(substr(d,7,2) substr(d,5,2)))
  y = s16(hex2dec(substr(d,11,2) substr(d,9,2)))
  print btn, x, y
}' capdata.txt > deltas_bxy.txt
```

Quick sanity check:

```bash
head -n 10 deltas_bxy.txt
```

```
0 0 -3
0 0 -6
0 -1 -11
```

I confirmed these are deltas, meaning each value isn't telling you where the pen is on the screen, it's telling you how much it moved since the last reading. So instead of "pen is at x=300", it's more like "pen moved 3 pixels to the right". You have to add them all up to reconstruct the actual path.

## figuring out which button means drawing

I noticed that if you include hover movements you get a complete mess. So I counted up the button values to figure out which one means actual pen contact:

```bash
awk '{c[$1]++} END{for(k in c) print c[k],k}' deltas_bxy.txt | sort -nr | head
```

I found that button state `2` was the pen touching the surface, and it produced clean readable strokes. The others were hover or other states. Simple enough once you know what to look for.

## turning it into an image

I used an AI-generated script to write the strokes out to an SVG file, since I wasn't sure how to approach the rendering part myself:

```python
from pathlib import Path

data=[]
for line in Path("deltas_bxy.txt").read_text().splitlines():
    if not line.strip():
        continue
    b,dx,dy=line.split()
    data.append((int(b), int(dx), int(dy)))

def make_svg(out_name, pen_btn=2, flipy=True, stroke=3, W=1800, H=700):
    X=Y=0
    strokes=[]
    cur=[]
    pen=False

    for b,dx,dy in data:
        X += dx
        Y += dy
        down = (b == pen_btn)

        if down:
            y = -Y if flipy else Y
            cur.append((X,y))
            pen=True
        else:
            if pen and len(cur) > 1:
                strokes.append(cur)
            cur=[]
            pen=False

    if pen and len(cur) > 1:
        strokes.append(cur)

    xs=[x for s in strokes for x,_ in s]
    ys=[y for s in strokes for _,y in s]
    minx,maxx=min(xs),max(xs)
    miny,maxy=min(ys),max(ys)
    w=(maxx-minx) or 1.0
    h=(maxy-miny) or 1.0
    pad=20

    def sc(x,y):
        sx = pad + (x-minx)/w*(W-2*pad)
        sy = pad + (y-miny)/h*(H-2*pad)
        return sx,sy

    paths=[]
    for s in strokes:
        d=[]
        x0,y0=sc(*s[0])
        d.append(f"M {x0:.2f} {y0:.2f}")
        for x,y in s[1:]:
            sx,sy=sc(x,y)
            d.append(f"L {sx:.2f} {sy:.2f}")
        paths.append(" ".join(d))

    svg = f'''<svg xmlns="http://www.w3.org/2000/svg" width="{W}" height="{H}" viewBox="0 0 {W} {H}">
  <rect width="100%" height="100%" fill="white"/>
  <g fill="none" stroke="black" stroke-width="{stroke}" stroke-linecap="round" stroke-linejoin="round">
    {''.join(f'<path d="{p}"/>' for p in paths)}
  </g>
</svg>
'''
    Path(out_name).write_text(svg)

make_svg("drawing.svg", pen_btn=2)
```

I opened the SVG and the scribbles turned into actual letters. Pretty satisfying ngl.

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/SVG_of_flag.png)

## reading the message

Once I filtered the right pen state the text was clear:

```
wh4t_c0l0ur_15_th3_fl4g
```

So there's your flag:

```
EH4X{wh4t_c0l0ur_15_th3_fl4g}
```
