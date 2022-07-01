# Getting Started

## Required Software

homebrew

`/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`

FFmpeg

`brew install ffmpeg`

Mediainfo

`brew install mediainfo`

QCTools

https://mediaarea.net/QCTools/Download

Mediaconch

https://mediaarea.net/MediaConch/Download

VLC

https://www.videolan.org/vlc/

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
ffmpeg -i Source_Uncompressed.mov  -c:v libx264 -pix_fmt yuv420p -movflags faststart -crf 30 -b:a 160000 -ar 48000 -vf setdar=4/3 'H264_LowQ.mp4' -c:v libx264 -pix_fmt yuv420p -movflags faststart -crf 18 -b:a 160000 -ar 48000 -vf setdar=4/3 'H264_HighQ.mp4' -c:v prores -profile:v 3 -c:a pcm_s24le -aspect 4:3 -ar 48000 -vf setdar=4/3 'ProRes.mov'  -c:v ffv1 -level 3 -g 1 -slices 16 -slicecrc 1 -color_primaries smpte170m -color_trc bt709 -colorspace smpte170m -color_range mpeg -metadata:s:v:0 'encoder= FFV1 version 3' -c:a copy -vf setfield=bff,setsar=40/27,setdar=4/3 -metadata creation_time=now -f matroska -vf setfield=bff,setdar=4/3 'FFV1.mkv'
```

## For V210 vs V210

This string compares our source file to itself. It's a good test to make sure that what we're doing is working. If we subtract the file from itself we should get just black. If we see black in the lower left-hand corner then we know that the process is working properly.

```
ffplay -f lavfi "movie=Source_Uncompressed.mov,setpts=PTS-STARTPTS,format=gbrp10le,split[a1][a2];movie= Source_Uncompressed.mov,setpts=PTS-STARTPTS,format=gbrp10le,split[b1][b2];[a1][b1]blend=all_mode=difference,format=yuv422p10le,histeq=strength=0.1:intensity=0.2,pad=2*iw:ih:0:0[down];[a2][b2]hstack[up];[up][down]vstack"
```

## For V210 vs ProRes
```
ffplay -f lavfi "movie=Source_Uncompressed.mov,setpts=PTS-STARTPTS,format=gbrp10le,split[a1][a2];movie=ProRes.mov,setpts=PTS-STARTPTS,format=gbrp10le,split[b1][b2];[a1][b1]blend=all_mode=difference,format=yuv422p10le,split[blended1][blended2];[blended2]histeq=strength=0.1:intensity=0.2[blended2];[a2][b2]hstack[up];[blended2][blended1]hstack[bottom];[up][bottom]vstack"
```

## For V210 vs High Quality H.264
```
ffplay -f lavfi "movie=Source_Uncompressed.mov,setpts=PTS-STARTPTS,format=gbrp10le,split[a1][a2];movie=H264_HighQ.mp4,setpts=PTS-STARTPTS,format=gbrp10le,split[b1][b2];[a1][b1]blend=all_mode=difference,format=yuv422p10le,split[blended1][blended2];[blended2]histeq=strength=0.1:intensity=0.2[blended2];[a2][b2]hstack[up];[blended2][blended1]hstack[bottom];[up][bottom]vstack"
```

## For V210 vs Low Quality H.264
```
ffplay -f lavfi "movie=Source_Uncompressed.mov,setpts=PTS-STARTPTS,format=gbrp10le,split[a1][a2];movie=H264_LowQ.mp4,setpts=PTS-STARTPTS,format=gbrp10le,split[b1][b2];[a1][b1]blend=all_mode=difference,format=yuv422p10le,split[blended1][blended2];[blended2]histeq=strength=0.1:intensity=0.2[blended2];[a2][b2]hstack[up];[blended2][blended1]hstack[bottom];[up][bottom]vstack"
```

## For V210 vs FFV1
```
ffplay -f lavfi "movie=Source_Uncompressed.mov,setpts=PTS-STARTPTS,format=gbrp10le,split[a1][a2];movie=FFV1.mkv,setpts=PTS-STARTPTS-TB,format=gbrp10le,split[b1][b2];[a1][b1]blend=all_mode=difference,format=yuv422p10le,split[blended1][blended2];[blended2]histeq=strength=0.1:intensity=0.2[blended2];[a2][b2]hstack[up];[blended2][blended1]hstack[bottom];[up][bottom]vstack"
```

# Exercise 03: Breaking an FFV1 Files

Copy or move the FFV1.mkv file from the previous exercise into the folder named `03BreakFFV1`

Change your terminal directory to `03BreakFFV1` using the `cd` command

Run the following command to add heavy corruption to a file

```
ffmpeg -i FFV1.mkv -c copy -bsf:v noise=amount=-0.0000001 FFV1_HeavyCorruption.mkv
```

Run the following command to add medium corruption to a file

```
ffmpeg -i FFV1.mkv -c copy -bsf:v noise=amount='mod(n\,120)/110' FFV1_MediumCorruption.mkv
```

Run the following command to add light corruption to a file

```
ffmpeg -i FFV1.mkv -c copy -bsf:v noise=amount='eq(n\,150)' FFV1_LightCorruption.mkv
```

Watch each of these clips in VLC and see if you can identify the errors

Play each of these files in FFplay and see what happens when FFmpeg encounters an error

# Exercise 04: Round Trip Transcode

Copy or move the `Source_Uncompressed.mov` file from the exercise 02 `04RoundTripTranscode`

Change your terminal directory to `04RoundTripTranscode` using the `cd` command

Create an FFV1 file from the `Source_Uncompressed.mov` file with the following command

```
ffmpeg -i Source_Uncompressed.mov  -c:v ffv1 -level 3 -g 1 -slices 16 -slicecrc 1 -color_primaries smpte170m -color_trc bt709 -colorspace smpte170m -color_range mpeg -metadata:s:v:0 'encoder= FFV1 version 3' -c:a copy -vf setfield=bff,setsar=40/27,setdar=4/3 -metadata creation_time=now -f matroska -vf setfield=bff,setdar=4/3 FFV1_From_MOV.mkv
```

Now, create an MOV file from the `FFV1_From_MOV.mkv` file with the following command

```
ffmpeg -i FFV1_From_MOV.mkv -movflags write_colr -c:v v210 -color_primaries smpte170m -color_trc bt709 -colorspace smpte170m -color_range mpeg -metadata:s:v:0 "encoder=Uncompressed 10-bit 4:2:2" -c:a copy -vf setfield=bff,setsar=40/27,setdar=4/3 -f mov MOV_From_FFV1.mov
```

We now have two MOV files, one that we started with (this was created with vrecord) and one that we created from the FFV1 file. Everything we know about FFV1 says that these files should be identical. How can we prove it. Visually we can do a difference blend like we did in exercise two using this command:

```
ffplay -f lavfi "movie=Source_Uncompressed.mov,setpts=PTS-STARTPTS,format=gbrp10le,split[a1][a2];movie= MOV_From_FFV1.mov,setpts=PTS-STARTPTS,format=gbrp10le,split[b1][b2];[a1][b1]blend=all_mode=difference,format=yuv422p10le,histeq=strength=0.1:intensity=0.2,pad=2*iw:ih:0:0[down];[a2][b2]hstack[up];[up][down]vstack"
```

You should see nothing on the bottom row, meaning that they're perfectly visually identical. But we can also prove that all these files are mathematically identical by making checksums of the video tracks. Run the following commands to get that checksum info.

```
ffmpeg -i Source_Uncompressed.mov  -map 0:v -f md5 Source_Uncompressed.mov.videomd5
```

```
ffmpeg -i FFV1_From_MOV.mkv  -map 0:v -f md5 FFV1_From_MOV.mkv.videomd5
```

```
ffmpeg -i MOV_From_FFV1.mov  -map 0:v -f md5 MOV_From_FFV1.mov.videomd5
```

If done correctly, these files should all match!

This is different than if we just did a checksum of the entire. You can do that with the following three commands:


```
md5 -q Source_Uncompressed.mov > Source_Uncompressed.mov.md5
```

```
md5 -q FFV1_From_MOV.mkv > FFV1_From_MOV.mkv.md5
```

```
md5 -q MOV_From_FFV1.mov > MOV_From_FFV1.mov.md5
```

You'll notice that these are all different. That's becausethe files themselves are not fully identical. Even the two MOV files are different. That's because the wrapper information and many non-essence parts of the file are difference. But, the video data in them is identical when decoded!

If you want to go even deeper we can look at the frame level data. Make a framemd5 file for each video file using the following three commands


```
ffmpeg -i Source_Uncompressed.mov  -map 0:v -f framemd5 Source_Uncompressed.mov.framemd5
```

```
ffmpeg -i FFV1_From_MOV.mkv  -map 0:v -f framemd5 FFV1_From_MOV.mkv.framemd5
```

```
ffmpeg -i MOV_From_FFV1.mov  -map 0:v -f framemd5 MOV_From_FFV1.mov.framemd5
```

Scroll through them and you'll see they're identical. You can also use the `diff` command to make sure. (`diff`) will return any characters that are different between two files. If it returns nothing the files are identical)

```
diff Source_Uncompressed.mov.framemd5 FFV1_From_MOV.mkv.framemd5
```

```
diff Source_Uncompressed.mov.framemd5 MOV_From_FFV1.mov.framemd5
```

```
diff FFV1_From_MOV.mkv.framemd5 MOV_From_FFV1.mov.framemd5
```

Just one more thing, if we do a framemd5 of the actual un-decoded data, you'll see that they're different. This is because the data is only identical when decoded!

```
ffmpeg -i Source_Uncompressed.mov  -map 0:v -c copy -f framemd5 Source_Uncompressed.mov.encodedframemd5
```

```
ffmpeg -i FFV1_From_MOV.mkv  -map 0:v -c copy -f framemd5 FFV1_From_MOV.mkv.encodedframemd5
```

```
ffmpeg -i MOV_From_FFV1.mov  -map 0:v -c copy -f framemd5 MOV_From_FFV1.mov.encodedframemd5
```

Now, the MOV files are still identical, but the FFV1 file is not! Use the `diff` command to prove this


```
diff Source_Uncompressed.mov.encodedframemd5 FFV1_From_MOV.mkv.encodedframemd5
```

```
diff Source_Uncompressed.mov.encodedframemd5 MOV_From_FFV1.mov.encodedframemd5
```


# Exercise 05: Station Qualification
