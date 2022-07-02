# Getting Started

## Required Software

### Command Line Installation

The following programs can be installed via the command line

#### Homebrew

`/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`

#### FFmpeg

`brew install ffmpeg`

#### Mediainfo

`brew install mediainfo`

#### SoX (suggested but not required)

`brew install sox`

### GUI Installations

For the following programs you will need to follow the link to download a DMG.

#### QCTools

https://mediaarea.net/QCTools/Download

#### Mediaconch

https://mediaarea.net/MediaConch/Download

#### VLC

https://www.videolan.org/vlc/

# Exercise 01: Mediainfo Mystery Meat

# Exercise 02: Compression Differences

## Create Derivative Files

First, change directory to the folder named 02CompressionDifference

`cd 02CompressionDifference`

Now, run the following command to create derivatives from the uncompressed source file

```
ffmpeg -i Source_Uncompressed.mov  -c:v libx264 -pix_fmt yuv420p -movflags faststart -crf 30 -b:a 160000 -ar 48000 -vf setdar=4/3 'H264_LowQ.mp4' -c:v libx264 -pix_fmt yuv420p -movflags faststart -crf 18 -b:a 160000 -ar 48000 -vf setdar=4/3 'H264_HighQ.mp4' -c:v prores -profile:v 3 -c:a pcm_s24le -aspect 4:3 -ar 48000 -vf setdar=4/3 'ProRes.mov'  -c:v ffv1 -level 3 -g 1 -slices 16 -slicecrc 1 -color_primaries smpte170m -color_trc bt709 -colorspace smpte170m -color_range mpeg -metadata:s:v:0 'encoder= FFV1 version 3' -c:a copy -vf setfield=bff,setsar=40/27,setdar=4/3 -metadata creation_time=now -f matroska -vf setfield=bff,setdar=4/3 'FFV1.mkv'
```

## Comparing files visually against each other

In this section we will visually compare files of different codecs against each other. Each command will make a 2x2 grid of videos. The top left is the source video, the top right is one of the derivatives, the bottom right is the source minus the derivative, and the bottom left is the source minus the derivative with the levels boosted. Boosting the levels is important in some cases because the visual differences in some cases are so subtle they cannot be seen easily.

### Compare V210 against itself

This command compares our source file to itself. It's a good test to make sure that what we're doing is working. If we subtract the file from itself we should get just black. If we see black in the lower row then we know that the process is working properly.

```
ffplay -f lavfi "movie=Source_Uncompressed.mov,setpts=PTS-STARTPTS,format=gbrp10le,split[a1][a2];movie= Source_Uncompressed.mov,setpts=PTS-STARTPTS,format=gbrp10le,split[b1][b2];[a1][b1]blend=all_mode=difference,format=yuv422p10le,histeq=strength=0.1:intensity=0.2,pad=2*iw:ih:0:0[down];[a2][b2]hstack[up];[up][down]vstack"
```

### Compare V210 against ProRes

This command compares our source file to the ProRes version. ProRes is a lossy compression encoding, but has excellent quality. The difference between the two is very subtle, but it's there.

```
ffplay -f lavfi "movie=Source_Uncompressed.mov,setpts=PTS-STARTPTS,format=gbrp10le,split[a1][a2];movie=ProRes.mov,setpts=PTS-STARTPTS,format=gbrp10le,split[b1][b2];[a1][b1]blend=all_mode=difference,format=yuv422p10le,split[blended1][blended2];[blended2]histeq=strength=0.1:intensity=0.2[blended2];[a2][b2]hstack[up];[blended2][blended1]hstack[bottom];[up][bottom]vstack"
```

### Compare V210 against High Quality H.264

This command compares our source file to the High Quality H.264. H.264 is more lossy than ProRes, and there should be a more pronounced difference between the uncompressed and H.264 file.

```
ffplay -f lavfi "movie=Source_Uncompressed.mov,setpts=PTS-STARTPTS,format=gbrp10le,split[a1][a2];movie=H264_HighQ.mp4,setpts=PTS-STARTPTS,format=gbrp10le,split[b1][b2];[a1][b1]blend=all_mode=difference,format=yuv422p10le,split[blended1][blended2];[blended2]histeq=strength=0.1:intensity=0.2[blended2];[a2][b2]hstack[up];[blended2][blended1]hstack[bottom];[up][bottom]vstack"
```

### Compare V210 against Low Quality H.264

This command compares our source file to the Low Quality H.264. There should be an even more pronounced difference between the uncompressed and H.264 file.

```
ffplay -f lavfi "movie=Source_Uncompressed.mov,setpts=PTS-STARTPTS,format=gbrp10le,split[a1][a2];movie=H264_LowQ.mp4,setpts=PTS-STARTPTS,format=gbrp10le,split[b1][b2];[a1][b1]blend=all_mode=difference,format=yuv422p10le,split[blended1][blended2];[blended2]histeq=strength=0.1:intensity=0.2[blended2];[a2][b2]hstack[up];[blended2][blended1]hstack[bottom];[up][bottom]vstack"
```

### Compare V210 against FFV1

This command compares our source file to the FFV1. There should be no difference here. Even the boosted levels difference should be completely black. This shows that there is no visual difference whatsoever between Uncompressed and FFV1.

```
ffplay -f lavfi "movie=Source_Uncompressed.mov,setpts=PTS-STARTPTS,format=gbrp10le,split[a1][a2];movie=FFV1.mkv,setpts=PTS-STARTPTS-TB,format=gbrp10le,split[b1][b2];[a1][b1]blend=all_mode=difference,format=yuv422p10le,split[blended1][blended2];[blended2]histeq=strength=0.1:intensity=0.2[blended2];[a2][b2]hstack[up];[blended2][blended1]hstack[bottom];[up][bottom]vstack"
```

# Exercise 03: Breaking an FFV1 Files

As discussed in the lecture, FFV1 has internal checksum verification. This means that if a frame is corrupted somehow the file should be able to report that there is a problem during playback or decoding.

## Creating corrupted files

Copy or move the FFV1.mkv file from the previous exercise into the folder named `03BreakFFV1`

Change your terminal directory to `03BreakFFV1` using the `cd` command

```
cd ..
cd 03BreakFFV1
```

Run the following command to add heavily corrupted to a file

```
ffmpeg -i FFV1.mkv -c copy -bsf:v noise=amount=-1 FFV1_HeavyCorruption.mkv
```

Run the following command to add medium corruption to a file

```
ffmpeg -i FFV1.mkv -c copy -bsf:v noise=amount='mod(n\,120)/110' FFV1_MediumCorruption.mkv
```

Run the following command to add light corruption to a file

```
ffmpeg -i FFV1.mkv -c copy -bsf:v noise=amount='eq(n\,150)' FFV1_LightCorruption.mkv
```

## View corrupted files visually

Watch each of these clips in VLC and see if you can identify the errors. The errors should be very obvious in the heavily corrupted file, but are far more subtle in the other files.

## Decode corrupted files using FFplay

If FFmpeg is used to decode these files it will report on any errors it encounters. The easiest way to do that is to simply play the files back in FFplay. Play each file and watch as FFplay reports on the errors. See if you can identify the errors as it reports them.

```
ffplay FFV1_HeavyCorruption.mkv

```
```
ffplay FFV1_MediumCorruption.mkv
```
```
ffplay FFV1_LightCorruption.mkv
```

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
