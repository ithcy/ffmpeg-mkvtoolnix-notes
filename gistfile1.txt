# Normalizing audio volume
# First, to find the mean audio volume:
ffmpeg -i input.mkv -vn -af "volumedetect" -f null /dev/null

# In the output will be something like mean_volume: -24.8 dB
# To normalize:
ffmpeg -i input.mkv -vcodec copy -af "volume=24dB" output.mkv

# tigole HEVC/x265 encode settings
ffmpeg -i input.mkv input-res=1920x1036 frame-threads=2 numa-pools=4 wpp no-pmode no-pme no-psnr no-ssim log-level=2 input-csp=1 interlace=0 level-idc=0 high-tier=1 uhd-bd=0 ref=4 no-allow-non-conformance no-repeat-headers annexb no-aud no-hrd info hash=0 no-temporal-layers open-gop min-keyint=23 keyint=250 gop-lookahead=0 bframes=4 b-adapt=2 b-pyramid bframe-bias=0 rc-lookahead=25 lookahead-slices=4 scenecut=40 hist-scenecut=0 radl=0 no-splice no-intra-refresh ctu=64 min-cu-size=8 rect no-amp max-tu-size=32 tu-inter-depth=1 tu-intra-depth=1 limit-tu=0 rdoq-level=2 dynamic-rd=0.00 no-ssim-rd signhide no-tskip nr-intra=0 nr-inter=0 no-constrained-intra strong-intra-smoothing max-merge=3 limit-refs=3 limit-modes me=3 subme=3 merange=57 temporal-mvp no-frame-dup no-hme weightp no-weightb no-analyze-src-pics deblock=0:0 no-sao no-sao-non-deblock rd=4 selective-sao=0 no-early-skip rskip no-fast-intra no-tskip-fast no-cu-lossless no-b-intra no-splitrd-skip rdpenalty=0 psy-rd=2.00 psy-rdoq=1.00 no-rd-refine no-lossless cbqpoffs=0 crqpoffs=0 rc=abr bitrate=6250 qcomp=0.60 qpstep=4 stats-write=0 stats-read=2 cplxblur=20.0 qblur=0.5 ipratio=1.40 pbratio=1.30 aq-mode=3 aq-strength=1.00 cutree zone-count=0 no-strict-cbr qg-size=32 no-rc-grain qpmax=69 qpmin=0 no-const-vbv sar=0 overscan=0 videoformat=5 range=0 colorprim=2 transfer=2 colormatrix=2 chromaloc=0 display-window=0 cll=0,0 min-luma=0 max-luma=1023 log2-max-poc-lsb=8 vui-timing-info vui-hrd-info slices=1 no-opt-qp-pps no-opt-ref-list-length-pps multi-pass-opt-rps scenecut-bias=0.05 hist-threshold=0.03 no-opt-cu-delta-qp aq-motion no-hdr10 no-hdr10-opt no-dhdr10-opt no-idr-recovery-sei analysis-reuse-level=0 analysis-save-reuse-level=0 analysis-load-reuse-level=0 scale-factor=0 refine-intra=0 refine-inter=0 refine-mv=1 refine-ctu-distortion=0 no-limit-sao ctu-info=0 no-lowpass-dct refine-analysis-type=0 copy-pic=1 max-ausize-factor=1.0 no-dynamic-refine no-single-sei no-hevc-aq no-svt no-field qp-adaptation-range=1.00 scenecut-aware-qp=0 conformance-window-offsets right=0 bottom=0 decoder-max-rate=0 no-vbv-live-multi-pass output.mkv


# if the error appears 'Starting new cluster due to timestamp', add -max_interleave_delta 0
# if missing timestamp errors appear, try -fflags +genpts
# -benchmark to time the operation
# -metadata:s:s:0 title='English'


# extract embedded (blu ray) subtitles to file
ffmpeg -i input.mkv -map 0:s:0 -c copy -f sup output-sub.sup

# convert subtitle from SRT to SSA
ffmpeg -i input-sub.srt output-sub.ass

# strip all metadata from mp4 (requires remux)
# from https://superuser.com/questions/441361/strip-metadata-from-all-formats-with-ffmpeg/428039#428039
ffmpeg -i input.mp4 -map 0 -map_metadata -1 -c:v copy -c:a copy -c:s copy -fflags +bitexact -flags:v +bitexact -flags:a +bitexact -flags:s +bitexact output.mp4

# embed srt file into mp4 as soft subtitle stream - mov_text is the only valid embedded sub format in mp4
# from https://www.baeldung.com/linux/subtitles-ffmpeg
ffmpeg -i input.mp4 -i sub.srt -c copy -c:s mov_text -metadata:s:s:0 language=eng output.mp4

# embed multiple subtitles into mp4
ffmpeg -i input.mp4 -i sub1.srt -i sub2.srt -map 0:v -map 0:a -map 1 -map 2 -c:v copy -c:a copy -c:s mov_text -metadata:s:s:0 language=eng -metadata:s:s:1 language=eng -metadata:s:s:1 title='SDH' output.mp4

# reencode blu ray to x265
# from https://www.reddit.com/r/ffmpeg/comments/mij9mr/which_settings_for_converting_fullhd_blu_rays_to/?rdt=47933
# see aq discussion: 
ffmpeg -i input.mkv -analyzeduration 2147483647 -probesize 2147483647 -map 0 -preset slow -crf 20 -aq-mode 4 -pix_fmt yuv420p10le -c:v libx265 -tag:v hvc1 -x265-params hdr-opt=1:keyint=96 -profile:v main10 -c:a copy -c:s copy output.mkv

# remux mp4 with no embedded subs to mkv, strip metadata
ffmpeg -i input.mp4 -map_metadata -1 -map 0 -c copy output.mkv

# remux mp4 with embedded sub to mkv, don't copy any metadata
# to add a title to the sub stream, add -metadata:s:s:0 title='Subs title'
ffmpeg -i input.mp4 -map_metadata -1 -map 0:v -map 0:a -map 0:s -c:v copy -c:a copy -c:s srt output.mkv

# remux mp4 streams and 1 external sub to mkv, don't copy any metadata
ffmpeg -i input.mp4 -i sub.srt -c copy -map_metadata -1 -map 0:v -map 0:a -map 1:s -metadata:s:s:0 language=eng output.mkv

# remux mp4 and multiple external subs to mkv, don't copy any metadata
ffmpeg -i input.mp4 -i sub1.srt -i sub2.srt -c copy -map_metadata -1 -map 0:v -map 0:a -map 1:s -map 2:s -metadata:s:s:0 language=eng -metadata:s:s:1 language=eng -metadata:s:s:1 title='SDH' output.mkv

# extract HDMV/PGS/SUP subtitles for OCR etc
# first identify track ID with mkvmerge, then use that track ID with mkvextract
mkvmerge -i input.mkv
mkvextract input.mkv tracks 1:output.sup

# embed VobSub subtitles into mkv
mkvmerge -o output.mkv input.mkv sub.idx #[ ...sub2.idx ... ]

# delete title from mkv segment info
mkvpropedit input.mkv -d title

# strip all metadata tags from mkv
mkvpropedit input.mkv --tags all:
