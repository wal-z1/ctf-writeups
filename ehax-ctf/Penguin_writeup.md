# penguin - CTF\_writeup-EHAX CTF2026

> In a colony of many, one penguin's path is an anomaly. Silence the crowd to hear the individual.

We get a single file: `challenge.mkv`. That's it.

Challenge Author: stapat

---

## initial analysis

First thing I did was run exiftool to see what's inside the metadata.

```bash
exiftool challenge.mkv
```

I found a comment field almost immediately:

```
Comment: EH4X{k33p_try1ng}
```

I knew right away that was a decoy. The challenge description literally says "silence the crowd to hear the individual" so there's clearly something hidden underneath.

The rest of the metadata that caught my eye:

```
Title                : Penguin
Video Frame Rate     : 23.976
Audio Channels       : 2
Audio Sample Rate    : 44100
Codec ID             : A_FLAC
Duration             : 00:01:03
```

Two things stood out — there's a FLAC audio track, and the track is named "English (5.1 Surround)" but only has 2 channels. That's a bit odd for a 5.1 track.

---

## stream inspection

I used ffprobe to get a clearer look at the MKV structure:

```bash
ffprobe -hide_banner -show_streams -show_format challenge.mkv
```

I noticed there are actually two separate stereo audio tracks:

- Stream #0:0 — Video (H.264)
- Stream #0:1 — Audio (FLAC stereo, default)
- Stream #0:2 — Audio (FLAC stereo, titled "English (5.1 Surround)")

Two audio tracks in the same video file. That's the anomaly the challenge was hinting at.

---

## extracting the audio tracks

I pulled both audio streams out separately:

```bash
ffmpeg -i challenge.mkv -map 0:a:0 -c copy a0.flac
ffmpeg -i challenge.mkv -map 0:a:1 -c copy a1.flac
```

Then converted both to WAV for easier processing:

```bash
ffmpeg -i a0.flac a0.wav
ffmpeg -i a1.flac a1.wav
```

---

## comparing the tracks

I ran sox stats on both to see if they were actually different:

```bash
sox a0.wav -n stat
sox a1.wav -n stat
```

Both tracks showed nearly identical RMS amplitude, mean amplitude, frequency, and duration. On the surface they looked exactly the same. Which yeah .. that's the point.

---

## subtracting the tracks

If two audio tracks are almost identical, subtracting one from the other cancels out all the shared content and leaves only the difference. That difference is where the hidden data is.

```bash
sox -m a0.wav "|sox a1.wav -p vol -1" diff_tracks.wav
```

Then I normalized the result so it's actually audible:

```bash
sox diff_tracks.wav diffN.wav gain -n -3
```

---

## generating a spectrogram

Audio data hidden in the frequency domain shows up in spectrograms. I generated one from the difference signal:

```bash
sox diffN.wav -n spectrogram -o diff_tracks.png -X 2000 -Y 1000 -z 100 -h
```

Then I zoomed into the interesting section around the 25 second mark:

```bash
sox diffN.wav -n spectrogram \
-S 25 -d 3 \
-X 5000 -Y 1200 \
-z 120 \
-h \
-o final_flag.png
```

I also filtered the frequency range to clean it up further:

```bash
sox diffN.wav filtered.wav sinc 5000-12000

sox filtered.wav -n spectrogram \
-S 25 -d 3 \
-X 5000 -Y 1200 \
-z 120 \
-h \
-o clean_flag.png
```

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/penguine-chall-specto.png)

---

## reading the spectrogram

The spectrogram revealed text written into the frequency domain of the difference signal. I zoomed in to read it more clearly:

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/penguine-chall-specto.pngzoomed1.png)

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/penguine-chall-specto.pngzoomed2.png)

```
EH4X{0n3_tr4ck_m1nd_tw0_tr4ck_f1les}
```

Pretty satisfying ngl.

---

## final flag

```
EH4X{0n3_tr4ck_m1nd_tw0_tr4ck_f1les}
```