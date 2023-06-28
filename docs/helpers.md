# One Liners

# Python

## Poetry

Here are some helpful one liners for using Poetry, a dependency manager for Python:

- `poetry new <project-name>`: Create a new project.
- `poetry add <library>`: Add a new library to the project.
- `poetry remove <library>`: Remove a library from the project.
- `poetry update <library>`: Update a library to the latest version.
- `poetry run which python`: Get the path to the virtual environment's Python interpreter.
- `poetry env list`: Show a list of all environments.
- `poetry export --output requirements.txt`: Export dependencies to a requirements.txt file.

---

## Remote Connection

### SSH

```
ssh -J mert@{servername}.ddns.net:port mert@target
```

### SCP

**From Local**

```
scp -o "ProxyJump mert@servername.ddns.net -p {port}" test.txt mert@target-pc:/home/mert/
```

**Download File From Remote Server**

```
scp -o "ProxyJump mert@servername.ddns.net -p {port}" mert@target-pc:/home/mert/ test.txt
```

---

## Various

### Download File

```
wget --user-agent Mozilla/4.0 'big address' -O dest_file_name
```

### Rename Files

```
ls -v | cat -n | while read n f; do mv -n "$f" "$n.ext"; done
```

### Extract Files

```
7za x test.7z
```

## String Slicing

**From Character**

```
for f in raw_daily/*.csv; do echo  $f /dimensions_"${f#*blocks_}"; done
```

**To Character**

```
for f in raw_daily/*.csv; do echo  $f /dimensions_"${f%*blocks_}"; done
```

**Random File Name**

```
for i in *.jpg; do mv -i "$i" ${RANDOM}${RANDOM}.jpg; done
```

**Move Files**

```
for f in png-exports/*; do cp $f/*.png all_images; done
```

**Delete Files Recursively**

```
find e -maxdepth 10 -type f -name ".*" -delete
```

**Get Dimensions from Folder**

```
ls -U | while read n; do identify -format "%f,%w,%h\n" "$n"; done > file_size.csv
```

---

## FFMPEG

**MP3 → WAV**

```
for f in *.mp3; do ffmpeg -i "$f" -acodec pcm_s16le -ac 1 -ar 16000 "wav-exports/${f%.}.wav"; done
```

**FLAC→ WAV**```
for f in *.flac; do ffmpeg -i "$f" "wav-exports/${f%.}.wav"; done

```
**WAV → mp3**
```

for f in *.*; do ffmpeg -i "$f" "wav-exports/${f%.}.wav"; done`

```
for f in *; do ffmpeg -i "${f}" -vn -ab 128k -ar 44100 -y "${f}.mp3" ; done
```

**PNG Sequence → MP4**

```
ffmpeg -f image2 -r 30 -i image_%6d.png -vcodec libx264 -crf 18  -pix_fmt yuv420p output.mp4
```

**MP4 → PNG**

```
ffmpeg -i test.mp4 -vf fps=1/2  png-exports/video13_%06d.png`
```

```
for f in *.mp4; do ffmpeg -i "$f" -vf fps=2 png-exports/${f%.*}_%06d.png; done
```

**MOV to Optimized GIF**

```
ffmpeg -i test.mov -vf scale=320:-1 -r 10 output/ffout%3d.png`
```

```
convert -delay 8 -loop 0 output/ffout*.png output/test.gif
```

---

## Image

Convert all images in directory

```
mogrify -format png *.*
```

**ESRGAN**

```
python inference_realesrgan.py -n RealESRGAN_x4plus -i v13 -s 3 --suffix 8k -t 1500 -o v13_out
```

---

## Utilities

Delete all hidden Mac junk files in Windows (Like .DS_STORE)

```
del /s /q /f /a .DS_STORE`
```

```
del /s /q /f /a ._.*`
```
