# Working with Subtitles

## Extract embedded subtitle and convert to SRT
```
ffmpeg -i input.mp4 -map 0:s:0 output.srt
```

## Extract embedded Blu-Ray subtitles (PGS/HDMV) from mkv
```
ffmpeg -i input.mkv -map 0:s:0 -f sup output.sup
```

## Convert subtitle from SRT to SSA
```
ffmpeg -i input-sub.srt output-sub.ass
```

## Embed SRT subtitle into mp4 as soft subtitle stream
(See https://www.baeldung.com/linux/subtitles-ffmpeg)

Note: `mov_text` is the only valid embedded subtitle format in mp4 container
```
ffmpeg -i input.mp4 -i sub.srt -c copy -c:s mov_text -metadata:s:s:0 language=eng output.mp4
```

## Embed multiple subtitles into mp4
```
ffmpeg -i input.mp4 -i sub1.srt -i sub2.srt -map 0:v -map 0:a -map 1 -map 2 -c:v copy -c:a copy -c:s mov_text -metadata:s:s:0 language=eng -metadata:s:s:1 language=eng -metadata:s:s:1 title='SDH' output.mp4
```

## Remux mp4 with no embedded subtitles to mkv, don't copy any metadata
```
ffmpeg -i input.mp4 -map_metadata -1 -map 0 -c copy output.mkv
```

## Remux mp4 with embedded subtitle stream to mkv, don't copy any metadata
```
ffmpeg -i input.mp4 -map_metadata -1 -map 0:v -map 0:a -map 0:s -c:v copy -c:a copy -c:s srt output.mkv
```
to add a title to the subtitle stream, add `-metadata:s:s:0 title='Subs title'`

## Remux mp4 and 1 external subtitle to mkv, don't copy any metadata
```
ffmpeg -i input.mp4 -i sub.srt -c copy -map_metadata -1 -map 0:v -map 0:a -map 1:s -metadata:s:s:0 language=eng output.mkv
```

## Remux mp4 and multiple external subtitles to mkv, don't copy any metadata
```
ffmpeg -i input.mp4 -i sub1.srt -i sub2.srt -c copy -map_metadata -1 -map 0:v -map 0:a -map 1:s -map 2:s -metadata:s:s:0 language=eng -metadata:s:s:1 language=eng -metadata:s:s:1 title='SDH' output.mkv
```

## Extract HDMV/PGS/SUP subtitles for OCR etc.
First identify track ID with mkvmerge, then use that track ID with mkvextract
```
mkvmerge -i input.mkv
mkvextract input.mkv tracks 1:output.sup
```

## Embed VobSub (.idx/.sub) subtitles into mkv
```
mkvmerge -o output.mkv input.mkv sub.idx #[ ...sub2.idx ... ]
```

# Working with Metadata

## Strip all metadata tags from mkv
```
mkvpropedit input.mkv --tags all:
```

## Strip all metadata tags from mp4 (requires remux)
(See [Superuser post](https://superuser.com/questions/441361/strip-metadata-from-all-formats-with-ffmpeg/428039#428039))
```
ffmpeg -i input.mp4 -map 0 -map_metadata -1 -c:v copy -c:a copy -c:s copy -fflags +bitexact -flags:v +bitexact -flags:a +bitexact -flags:s +bitexact output.mp4
```

## Delete main title from mkv
This deletes the title tag from the segment info / format section
```
mkvpropedit input.mkv -d title
```

## Delete name from first video track in mkv
```
mkvpropedit input.mkv -e track:v1 -d name
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
