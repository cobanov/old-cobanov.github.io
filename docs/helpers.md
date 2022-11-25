---
title: One Liners
summary: A brief description of my document.
authors:
  - Mert Cobanov
date: 2022-11-11
some_url: https://cobanov.dev/blog
hide:
  - footer
---
# Helper Codes

## Remote Connection

### SSH

```bash
ssh -J mert@{servername}.ddns.net:port mert@target
```

### SCP

#### From Local

```bash
scp -o "ProxyJump mert@servername.ddns.net -p {port}" test.txt mert@target-pc:/home/mert/
```

#### Download File From Remote Server

```bash
scp -o "ProxyJump mert@servername.ddns.net -p {port}" mert@target-pc:/home/mert/ test.txt
```

---

## Various

### Download File

```bash
wget --user-agent Mozilla/4.0 'big address' -O dest_file_name
```

### Rename Files

```bash
ls -v | cat -n | while read n f; do mv -n "$f" "$n.ext"; done
```

### Extract Files

```bash
7za x test.7z
```

### String Slicing

```bash
# From Character
for f in raw_daily/*.csv; do echo  $f /dimensions_"${f#*blocks_}"; done

# TO Character
for f in raw_daily/*.csv; do echo  $f /dimensions_"${f%*blocks_}"; done
```

### Random File Name

```bash
for i in *.jpg; do mv -i "$i" ${RANDOM}${RANDOM}.jpg; done
```

### Move Files

```bash
for f in png-exports/*; do cp $f/*.png all_images; done
```

### Delete Files Recursively

```bash
 find e -maxdepth 10 -type f -name ".*" -delete
```

### Get Dimensions from Folder

```jsx
ls -U | while read n; do identify -format "%f,%w,%h\n" "$n"; done > file_size.csv
```

---

## FFMPEG

### MP3 → WAV ###

```bash
for f in *.mp3; do ffmpeg -i "$f" -acodec pcm_s16le -ac 1 -ar 16000 "wav-exports/${f%.}.wav"; done

for f in *.flac; do ffmpeg -i "$f" "wav-exports/${f%.}.wav"; done
```
### WAV  → mp3

```bash
for f in *.*; do ffmpeg -i "$f" "wav-exports/${f%.}.wav"; done
for f in *; do ffmpeg -i "${f}" -vn -ab 128k -ar 44100 -y "${f}.mp3" ; done
```

### PNG Sequence → MP4

```bash
ffmpeg -f image2 -r 30 -i image_%6d.png -vcodec libx264 -crf 18  -pix_fmt yuv420p output.mp4
```

### MP4 → PNG

```bash
ffmpeg -i test.mp4 -vf fps=1/2  png-exports/video13_%06d.png
```

```bash
for f in *.mp4; do ffmpeg -i "$f" -vf fps=2 png-exports/${f%.*}_%06d.png; done
```

### MOV to Optimized GIF

```jsx
ffmpeg -i test.mov -vf scale=320:-1 -r 10 output/ffout%3d.png
```

```jsx
convert -delay 8 -loop 0 output/ffout*.png output/test.gif
```

---

## Image

Convert all images in directory

```bash
mogrify -format png *.*
```

---

### ESRGAN

```bash
python inference_realesrgan.py -n RealESRGAN_x4plus -i v13 -s 3 --suffix 8k -t 1500 -o v13_out
```
