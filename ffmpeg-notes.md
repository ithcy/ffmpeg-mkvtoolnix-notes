# Working with Subtitles

## Remuxing with Soft Subtitles
(Subtitle streams embedded into a video container file, not hardcoded subs)

### Remux MP4 with No Soft Subtitles to MKV
```
ffmpeg -i input.mp4 -c copy -map 0 -map_metadata -1 output.mkv
```

### Remux MP4 with 1 Soft Subtitle Stream to MKV
```
ffmpeg -i input.mp4 -c:v copy -c:a copy -c:s srt -map_metadata -1 -map 0 -metadata:s:s:0 language=eng output.mkv
```
to add a title to the subtitle stream, add `-metadata:s:s:0 title='Subs title'`

### MISSING: Remux MP4 with Multiple Soft Subtitle Streams to MKV

## Remuxing with Sidecar Subtitles
Sidecar subtitles are separate subtitle files that usually sit alongside the main video file.

### Remux MP4 and 1 Sidecar SRT into MKV
```
ffmpeg -i input.mp4 -i sub.srt -c copy -map_metadata -1 -map 0:v -map 0:a -map 1:s -metadata:s:s:0 language=eng output.mkv
```

### Remux MP4 and 1 Sidecar SRT into MP4
Note: `mov_text` is the only valid embedded subtitle format in mp4 container
```
ffmpeg -i input.mp4 -i sub.srt -c:v copy -c:a copy -c:s mov_text -map_metadata -1 -metadata:s:s:0 language=eng output.mp4
```

### Remux MKV and Sidecar VobSub (.idx/.sub) into MKV
```
mkvmerge -o output.mkv input.mkv sub.idx #[ ...sub2.idx ... ]
```

### Remux MP4 and Multiple Sidecar SRTs into MKV
```
ffmpeg -i input.mp4 -i sub1.srt -i sub2.srt -c copy -map_metadata -1 -map 0:v -map 0:a -map 1:s -map 2:s -metadata:s:s:0 language=eng -metadata:s:s:1 language=eng -metadata:s:s:1 title='SDH' output.mkv
```

### Remux MP4 and Multiple Sidecar SRTs into MP4
```
ffmpeg -i input.mp4 -i sub1.srt -i sub2.srt -c:v copy -c:a copy -c:s mov_text -map_metadata -1 -map 0:v -map 0:a -map 1:s -map 2:s -metadata:s:s:0 language=eng -metadata:s:s:1 language=eng -metadata:s:s:1 title='SDH' output.mp4
```

## Extracting and Converting Subtitles

### Extract Blu-Ray (PGS/HDMV) Subtitles from MKV
```
ffmpeg -i input.mkv -map 0:s:0 -f sup output.sup
```

### Alternate method of extracting Blu-Ray subtitles using mkvmerge
First identify track ID with mkvmerge, then use that track ID with mkvextract
```
mkvmerge -i input.mkv
mkvextract input.mkv tracks 1:output.sup
```

### Extract Soft Subtitle from MP4 and Convert to SRT
```
ffmpeg -i input.mp4 -map 0:s:0 output.srt
```

### Convert Subtitle file from SRT to SSA
```
ffmpeg -i input.srt output.ass
```

# Working with Metadata

## Working with Matroska Metadata
No remuxing is needed for these operations; they happen in place on the original file.

### Strip All Metadata Tags from an MKV file
```
mkvpropedit input.mkv --tags all:
```

### Delete Main Title from an MKV file
This deletes the title tag from the segment info / format section
```
mkvpropedit input.mkv -d title
```

### Delete Name from First Video Track in an MKV file
```
mkvpropedit input.mkv -e track:v1 -d name
```

## Working with MP4 Metadata
These operations require remuxing.

### Strip All Metadata Tags from an MP4 file
(See [Superuser post](https://superuser.com/questions/441361/strip-metadata-from-all-formats-with-ffmpeg/428039#428039))
```
ffmpeg -i input.mp4 -map 0 -map_metadata -1 -c:v copy -c:a copy -c:s copy -fflags +bitexact -flags:v +bitexact -flags:a +bitexact -flags:s +bitexact output.mp4
```

# Other

## Re-encode Blu-Ray to x265 (1080p)
(See [Reddit post](https://www.reddit.com/r/ffmpeg/comments/mij9mr/which_settings_for_converting_fullhd_blu_rays_to/?rdt=47933))
```
ffmpeg -i input.mkv -analyzeduration 2147483647 -probesize 2147483647 -map 0 -preset slow -crf 22 -aq-mode 4 -pix_fmt yuv420p10le -c:v libx265 -tag:v hvc1 -x265-params hdr-opt=1:keyint=96 -profile:v main10 -c:a copy -c:s copy output.mkv
```

## Normalize audio volume
First, to find the mean audio volume:
```
ffmpeg -i input.mkv -vn -af "volumedetect" -f null /dev/null
```

In the output will be something like

`mean_volume: -24.8 dB`

Next, use that value to normalize:
```
ffmpeg -i input.mkv -vcodec copy -af "volume=24dB" output.mkv
```

# Notes

If 'Starting new cluster due to timestamp' warning appears in ffmpeg output, add `-max_interleave_delta 0`

If missing timestamp errors appear in ffmpeg output, try `-fflags +genpts`

To time ffmpeg operation, add `-benchmark`
