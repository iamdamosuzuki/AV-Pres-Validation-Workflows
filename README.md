# Getting Started

## Required Software

homebrew

`/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`

FFmpeg

`brew install ffmpeg`

Mediainfo

`brew install ffmpeg`

QCTools

https://mediaarea.net/QCTools/Download

Mediaconch

https://mediaarea.net/MediaConch/Download

### Recommended But Not Required

SoX

`brew install sox`

I needed to install git lfs to upload the large .mov files

https://git-lfs.github.com/

# Exercise 01: Mediainfo Mystery Meat

# Exercise 02: Compression Differences

## Create Derivative Files

First, change directory to the folder named 02CompressionDifference

`cd 02CompressionDifference`

Now, run the following command to create derivatives from the uncompressed source file

```
ffmpeg -i Source_Uncompressed.mov  -c:v libx264 -pix_fmt yuv420p -movflags faststart -crf 30 -b:a 160000 -ar 48000 -vf setdar=4/3 'H264_LowQ.mp4' -c:v libx264 -pix_fmt yuv420p -movflags faststart -crf 30 -b:a 160000 -ar 48000 -vf setdar=4/3 'H264_LowQ.mp4' -c:v libx264 -pix_fmt yuv420p -movflags faststart -crf 18 -b:a 160000 -ar 48000 -vf setdar=4/3 'H264_HighQ.mp4' -c:v prores -profile:v 3 -c:a pcm_s24le -aspect 4:3 -ar 48000 -vf setdar=4/3 'ProRes.mov'  -c:v ffv1 -level 3 -g 1 -slices 16 -slicecrc 1 -color_primaries smpte170m -color_trc bt709 -colorspace smpte170m -color_range mpeg -metadata:s:v:0 'encoder= FFV1 version 3' -c:a copy -vf setfield=bff,setsar=40/27,setdar=4/3 -metadata creation_time=now -f matroska -vf setfield=bff,setdar=4/3 'FFV1.mkv'
```

## For V210 vs V210

This string compares our source file to itself. It's a good test to make sure that what we're doing is working. If we subtract the file from itself we should get just black. If we see black in the lower left-hand corner then we know that the process is working properly.

```
ffplay -f lavfi "movie=Source_Uncompressed.mov,setpts=PTS-STARTPTS,format=gbrp10le,split[a1][a2];movie= Source_Uncompressed.mov,setpts=PTS-STARTPTS,format=gbrp10le,split[b1][b2];[a1][b1]blend=all_mode=difference,format=yuv422p10le,histeq=strength=0.1:intensity=0.2,pad=2*iw:ih:0:0[down];[a2][b2]hstack[up];[up][down]vstack"
``````

## For V210 vs ProRes
```
ffplay -f lavfi "movie=Source_Uncompressed.mov,setpts=PTS-STARTPTS,format=gbrp10le,split[a1][a2];movie=ProRes.mov,setpts=PTS-STARTPTS,format=gbrp10le,split[b1][b2];[a1][b1]blend=all_mode=difference,format=yuv422p10le,histeq=strength=0.1:intensity=0.2,pad=2*iw:ih:0:0[down];[a2][b2]hstack[up];[up][down]vstack"
```

## For V210 vs High Quality H.264
```
ffplay -f lavfi "movie=Source_Uncompressed.mov,setpts=PTS-STARTPTS,format=gbrp10le,split[a1][a2];movie=H264_HighQ.mp4,setpts=PTS-STARTPTS,format=gbrp10le,split[b1][b2];[a1][b1]blend=all_mode=difference,format=yuv422p10le,histeq=strength=0.1:intensity=0.2,pad=2*iw:ih:0:0[down];[a2][b2]hstack[up];[up][down]vstack"
```

## For V210 vs Low Quality H.264
```
ffplay -f lavfi "movie=Source_Uncompressed.mov,setpts=PTS-STARTPTS,format=gbrp10le,split[a1][a2];movie=H264_LowQ.mp4,setpts=PTS-STARTPTS,format=gbrp10le,split[b1][b2];[a1][b1]blend=all_mode=difference,format=yuv422p10le,histeq=strength=0.1:intensity=0.2,pad=2*iw:ih:0:0[down];[a2][b2]hstack[up];[up][down]vstack"
```

## For V210 vs FFV1
```
ffplay -f lavfi "movie=Source_Uncompressed.mov,setpts=PTS-STARTPTS,format=gbrp10le,split[a1][a2];movie=FFV1.mkv,setpts=PTS-STARTPTS-TB,format=gbrp10le,split[b1][b2];[a1][b1]blend=all_mode=difference,format=yuv422p10le,histeq=strength=0.1:intensity=0.2,pad=2*iw:ih:0:0[down];[a2][b2]hstack[up];[up][down]vstack"
```

# Exercise 03: Round Trip Transcode


# Exercise 04: Station Qualification
