- [1. Working with Subtitles](#1-working-with-subtitles)
  - [1.1. Remuxing with Soft Subtitles](#11-remuxing-with-soft-subtitles)
    - [1.1.1. Remux MP4 with No Soft Subtitles to MKV](#111-remux-mp4-with-no-soft-subtitles-to-mkv)
    - [1.1.2. Remux MP4 with 1 Soft Subtitle Stream to MKV](#112-remux-mp4-with-1-soft-subtitle-stream-to-mkv)
    - [1.1.3. MISSING: Remux MP4 with Multiple Soft Subtitle Streams to MKV](#113-missing-remux-mp4-with-multiple-soft-subtitle-streams-to-mkv)
  - [1.2. Remuxing with Sidecar Subtitles](#12-remuxing-with-sidecar-subtitles)
    - [1.2.1. Remux MP4 and 1 Sidecar SRT into MKV](#121-remux-mp4-and-1-sidecar-srt-into-mkv)
    - [1.2.2. Remux MP4 and 1 Sidecar SRT into MP4](#122-remux-mp4-and-1-sidecar-srt-into-mp4)
    - [1.2.3. Remux MKV and Sidecar VobSub (.idx/.sub) into MKV](#123-remux-mkv-and-sidecar-vobsub-idxsub-into-mkv)
    - [1.2.4. Remux MP4 and Multiple Sidecar SRTs into MKV](#124-remux-mp4-and-multiple-sidecar-srts-into-mkv)
    - [1.2.5. Remux MP4 and Multiple Sidecar SRTs into MP4](#125-remux-mp4-and-multiple-sidecar-srts-into-mp4)
  - [1.3. Extracting and Converting Subtitles](#13-extracting-and-converting-subtitles)
    - [1.3.1. Extract Blu-Ray (PGS/HDMV) Subtitles from MKV](#131-extract-blu-ray-pgshdmv-subtitles-from-mkv)
    - [1.3.2. Alternate method of extracting Blu-Ray subtitles using mkvmerge](#132-alternate-method-of-extracting-blu-ray-subtitles-using-mkvmerge)
    - [1.3.3. Extract Soft Subtitle from MP4 and Convert to SRT](#133-extract-soft-subtitle-from-mp4-and-convert-to-srt)
    - [1.3.4. Convert Subtitle file from SRT to SSA](#134-convert-subtitle-file-from-srt-to-ssa)
  - [1.4. Other Subtitle Operations](#14-other-subtitle-operations)
    - [1.4.1. Setting Subtitles to Automatically Display](#141-setting-subtitles-to-automatically-display)
- [2. Working with Metadata](#2-working-with-metadata)
  - [2.1. Working with Matroska Metadata](#21-working-with-matroska-metadata)
    - [2.1.1. Strip All Metadata Tags from an MKV file](#211-strip-all-metadata-tags-from-an-mkv-file)
    - [2.1.2. Delete Main Title from an MKV file](#212-delete-main-title-from-an-mkv-file)
    - [2.1.3. Delete Name from First Video Track in an MKV file](#213-delete-name-from-first-video-track-in-an-mkv-file)
  - [2.2. Working with MP4 Metadata](#22-working-with-mp4-metadata)
    - [2.2.1. Strip All Metadata Tags from an MP4 file](#221-strip-all-metadata-tags-from-an-mp4-file)
- [3. Other](#3-other)
  - [3.1. Encoding](#31-encoding)
    - [3.1.1. Re-encode Blu-Ray file to x265 (1080p)](#311-re-encode-blu-ray-file-to-x265-1080p)
  - [3.2. Audio Processing](#32-audio-processing)
    - [3.2.1. Normalize Audio Volume](#321-normalize-audio-volume)
- [4. Notes](#4-notes)

# 1. Working with Subtitles

## 1.1. Remuxing with Soft Subtitles
Soft subtitles refers to subtitle streams that are embedded into a video container file, as opposed to [sidecar](#remuxing-with-sidecar-subtitles) or hardcoded subtitles.

### 1.1.1. Remux MP4 with No Soft Subtitles to MKV
```
ffmpeg -i input.mp4 -c copy -map 0 -map_metadata -1 output.mkv
```

### 1.1.2. Remux MP4 with 1 Soft Subtitle Stream to MKV
```
ffmpeg -i input.mp4 -c:v copy -c:a copy -c:s srt -map_metadata -1 -map 0 -metadata:s:s:0 language=eng output.mkv
```
To give a title to the subtitle stream, add e.g. `-metadata:s:s:0 title='English Commentary'`

### 1.1.3. MISSING: Remux MP4 with Multiple Soft Subtitle Streams to MKV

## 1.2. Remuxing with Sidecar Subtitles
Sidecar subtitles are separate subtitle files that usually sit alongside the main video file.

### 1.2.1. Remux MP4 and 1 Sidecar SRT into MKV
```
ffmpeg -i input.mp4 -i sub.srt -c copy -map_metadata -1 -map 0:v -map 0:a -map 1:s -metadata:s:s:0 language=eng output.mkv
```

### 1.2.2. Remux MP4 and 1 Sidecar SRT into MP4
*The subtitle is converted to mov_text format (aka Timed Text/TTXT, [MPEG-4 Part 17](http://en.wikipedia.org/wiki/MPEG-4_Part_17)) since MP4 does not support SRT subtitles*
```
ffmpeg -i input.mp4 -i sub.srt -c:v copy -c:a copy -c:s mov_text -map_metadata -1 -metadata:s:s:0 language=eng output.mp4
```

### 1.2.3. Remux MKV and Sidecar VobSub (.idx/.sub) into MKV
```
mkvmerge -o output.mkv input.mkv sub.idx #[ ...sub2.idx ... ]
```

### 1.2.4. Remux MP4 and Multiple Sidecar SRTs into MKV
```
ffmpeg -i input.mp4 -i sub1.srt -i sub2.srt -c copy -map_metadata -1 -map 0:v -map 0:a -map 1:s -map 2:s -metadata:s:s:0 language=eng -metadata:s:s:1 language=eng -metadata:s:s:1 title='SDH' output.mkv
```

### 1.2.5. Remux MP4 and Multiple Sidecar SRTs into MP4
```
ffmpeg -i input.mp4 -i sub1.srt -i sub2.srt -c:v copy -c:a copy -c:s mov_text -map_metadata -1 -map 0:v -map 0:a -map 1:s -map 2:s -metadata:s:s:0 language=eng -metadata:s:s:1 language=eng -metadata:s:s:1 title='SDH' output.mp4
```

## 1.3. Extracting and Converting Subtitles

### 1.3.1. Extract Blu-Ray (PGS/HDMV) Subtitles from MKV
```
ffmpeg -i input.mkv -map 0:s:0 -f sup output.sup
```

### 1.3.2. Alternate method of extracting Blu-Ray subtitles using mkvmerge
*First, find the ID of the subtitle track ID with mkvmerge, then use that track ID with mkvextract.*
```
mkvmerge -i input.mkv
mkvextract input.mkv tracks 1:output.sup
```

### 1.3.3. Extract Soft Subtitle from MP4 and Convert to SRT
```
ffmpeg -i input.mp4 -map 0:s:0 output.srt
```

### 1.3.4. Convert Subtitle file from SRT to SSA
```
ffmpeg -i input.srt output.ass
```

## 1.4. Other Subtitle Operations

### 1.4.1. Setting Subtitles to Automatically Display
*Video players will automatically display the first subtitle stream that has the "default" flag set.*
```
mkvpropedit -q input.mkv -e track:s1 -s flag-default=1
```

# 2. Working with Metadata

## 2.1. Working with Matroska Metadata
*No remuxing is needed for these operations; they happen in place on the original file.*

### 2.1.1. Strip All Metadata Tags from an MKV file
```
mkvpropedit input.mkv --tags all:
```

### 2.1.2. Delete Main Title from an MKV file
*This deletes the title tag from the segment info / format section.*
```
mkvpropedit input.mkv -d title
```

### 2.1.3. Delete Name from First Video Track in an MKV file
```
mkvpropedit input.mkv -e track:v1 -d name
```

## 2.2. Working with MP4 Metadata
*These operations require remuxing.*

### 2.2.1. Strip All Metadata Tags from an MP4 file
(See [Superuser post](https://superuser.com/questions/441361/strip-metadata-from-all-formats-with-ffmpeg/428039#428039))
```
ffmpeg -i input.mp4 -map 0 -map_metadata -1 -c:v copy -c:a copy -c:s copy -fflags +bitexact -flags:v +bitexact -flags:a +bitexact -flags:s +bitexact output.mp4
```

# 3. Other

## 3.1. Encoding

### 3.1.1. Re-encode Blu-Ray file to x265 (1080p)
(See [Reddit post](https://www.reddit.com/r/ffmpeg/comments/mij9mr/which_settings_for_converting_fullhd_blu_rays_to/?rdt=47933))
```
ffmpeg -i input.mkv -analyzeduration 2147483647 -probesize 2147483647 -map 0 -preset slow -crf 22 -aq-mode 4 -pix_fmt yuv420p10le -c:v libx265 -tag:v hvc1 -x265-params hdr-opt=1:keyint=96 -profile:v main10 -c:a copy -c:s copy output.mkv
```

## 3.2. Audio Processing

### 3.2.1. Normalize Audio Volume
First, find the mean audio volume:
```
ffmpeg -i input.mkv -vn -af "volumedetect" -f null /dev/null
```

Near the end of the output will be something like:

`mean_volume: -24.8 dB`

Next, use that value to normalize
```
ffmpeg -i input.mkv -vcodec copy -af "volume=24dB" output.mkv
```

# 4. Notes

If 'Starting new cluster due to timestamp' warning appears in ffmpeg output, add `-max_interleave_delta 0`

If missing timestamp errors appear in ffmpeg output, try `-fflags +genpts`

To time ffmpeg operation, add `-benchmark`
