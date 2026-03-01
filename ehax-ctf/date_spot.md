# date spot - CTF_writeup-EHAX CTF2026

I got dropped into a coastal road in what felt like northern Japan immediately. Conifer hills, autumn colors, pebble beach, and those massive reinforced seawalls you see everywhere on the Japanese coast. There was a fishing harbor with a white lighthouse visible in the distance and a coastal highway running right beside the sea.

Here's the full panorama of the drop location:

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/FULL-date.png)

---

## figuring out the country

I wrote down what I noticed ...

- left-hand traffic
- blue road signs with white text
- Kanji characters on signage
- Japanese-style guardrails
- coastal infrastructure that looked very Japan

Here's the left-hand traffic visible in the scene:

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/left-hand-traffic.png)

Here's a close-up of the Japanese road sign text:

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/japanese-road-sign-text.png)

---

## narrowing the region

The terrain gave it away ...
Conifer-covered hills and autumn foliage ... pebble beach, large seawalls ... and a fishing harbor with a white lighthouse ... That combination pointed hard toward northern Japan ...

Here's the coastline with the seawall and the harbor visible in the background:

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/coastline-seawall.png)

---

## the road sign was the key

I zoomed into the blue direction sign and use an OCR to detect its text

- 木古内 (Kikonai)
- 北斗茂辺地 (Hokuto Moheji)
- E59 (expressway number on the green panel)

Here's the zoomed sign:

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/blue-direction-sign-zoom.jpg)

I googled each name. Kikonai is in southern Hokkaido, Hokuto Moheji is a district in Hokuto, and E59 is the Hakodate-Esashi Expressway. That confirmed the our location

---

## finding the spot on the map

I opened Google Maps and started following National Route 228 along the coastline between Hakodate and Kikonai. I was looking for a specific set of things that matched the panorama:

- road directly beside the shoreline
- small harbor with a white lighthouse
- a nearby river mouth
- a large inland viaduct
- a bridge crossing a small river near the coast


the satellite view :

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/satellite-view-match.png)

---

## confirming with street view

and yeah using street view we found it

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/streetview-comparison.png)

---

## final pin

**Location:** Moheji Ōhashi (茂辺地大橋), Hokuto, Hokkaido, Japan  
**Coords:** 41.76590, 140.60798

---

## final flag

```
EH4X{th4t_15_my_dr34m_d4t3_5p0t}
```
