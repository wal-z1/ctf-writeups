# idinahui - CTF\_writeup-EHAX CTF2026

I got dropped into a wide seaside parking lot with that immediate post-Soviet coastal city vibe. Blocky concrete apartments on the left, newer glassy buildings on the right, and a big open shoreline straight ahead.

Here's the full street view of the drop location, the parking lot, the shoreline, and the buildings in the background:

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/idinahui-1.png)

---

## key clues

**Cyrillic everywhere**

The architecture and road hardware gave off a strong Russia or ex-USSR feel. I had no idea what the car models were so I reverse image searched them on Google and the results kept pointing back to Russian/CIS markets.

**the flags**

I noticed two flags on poles nearby. One was clearly the Russian tricolor. The second one looked like a regional flag, the kind you usually see near government buildings or official complexes.

Here's the close-up of the two flags on the poles, the Russian tricolor on the left and the regional flag on the right:

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/idinahui-3.png)

**the coastline**

The shoreline didn't feel like touristy Europe at all. Flat horizon, muted beach, windy open space. It felt more like the Caspian than the Black Sea. That regional feel was hard to place at first but it stuck with me.

**the Mimino sign**

The big text on one of the buildings read "Мимино / Mimino". I had to google it and found out it's a very common Georgian-themed restaurant name in Russia, based on a famous Soviet-era film. That detail pushed me hard toward the North Caucasus region.

Here's the Mimino sign on the building, both the Cyrillic and Latin versions are visible:

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/idinahui-4.png)

---

## narrowing it down

once I was pretty sure it was somewhere on the Caspian coast in the North Caucasus, I googled around and Makhachkala kept coming up as the main coastal city in that area.

---

## final pin

I placed my guess at the Mimino restaurant by the Caspian waterfront in Makhachkala, on Petra I Avenue.

**Location:** Mimino (Мимино), Makhachkala, Republic of Dagestan, Russia  
**Coords:** 42.967536, 47.544367

Here's the final pin on the map, dropped right on the Mimino restaurant complex by the Caspian waterfront:

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/idinahui-5.png)

---

## final flag

```
EH4X{ru5514n_t34_15_v3ry_g00d}
```