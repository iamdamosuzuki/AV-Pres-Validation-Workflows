# Getting Started (Do This Before The Workshop)

Please complete everything in this first section before you come to the workshop. It'll take a while to download and install all of the necessary files and applications. If you wait until the workshop starts you may fall behind while you wait for the downloads to complete.

## Required Software

### Command Line Installation

The following programs can be installed via the command line

#### Homebrew

`/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`

#### FFmpeg

`brew install ffmpeg`

#### Mediainfo

`brew install mediainfo`

#### SoX (not required)

`brew install sox`

_Note: This was originally used to generate spectrograms, but is no longer being used. You don't need to isntall sox if you don't want to_

### GUI Installations

For the following programs you will need to follow the link to download a DMG.

#### QCTools

https://mediaarea.net/QCTools/Download

#### Mediaconch

https://mediaarea.net/MediaConch/Download

#### VLC

https://www.videolan.org/vlc/

## Downloading Required Files

The files that you need in order complete the exercises are quite large. You should stat downloading the files immediately so that they'll be ready by the time you need them.

### File for Exercise 1

You'll need to download the Uncompressed MOV File named `Source_Uncompressed.mov` from this page:

https://archive.org/details/source-uncompressed

Here's a direct link to download that file:

https://archive.org/download/source-uncompressed/Source_Uncompressed.mov (right click and hit "save as")

Once you have the files downloaded, move them to the `01CompressionDifference` folder in the repo.

### Files for Exercise 2

You'll also need to download the Matroska Files from this page:

https://archive.org/details/qualification-test-file-02

Here's a direct link to download those files:

https://archive.org/compress/qualification-test-file-02/formats=MATROSKA&file=/qualification-test-file-02.zip

Once you have the files downloaded, move them to the `06VideoStationQualification` folder in the repo.


# Exercise 01: Compression Differences

## Download files.

If you haven't already downloaded the file for this exercise, do so now. You'll need to download the Uncompressed MOV File named `Source_Uncompressed.mov` from this page:

https://archive.org/details/source-uncompressed

Here's a direct link to download that file:

https://archive.org/download/source-uncompressed/Source_Uncompressed.mov (right click and hit "save as")

Once you have the files downloaded, move them to the `01CompressionDifference` folder in the repo.

## Create Derivative Files

First, change directory to the folder named 02CompressionDifference

`cd 01CompressionDifference`

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

# Exercise 02: Media Mystery Meat

For this exercise you're going to probe a bunch of mystery files and figure out what they are, and which might be a good candidate for a Preservation file. The files you'll need should have been downloaded with repository. They are in the `02MediaMysteryMeat` folder. Move your terminal window into that folder using the `cd` commands

```
cd ..
cd 02MediaMysteryMeat
```

There are five mystery files. Let's start with the first one

_Note: these files all came from the FFmpeg sample file repository, I have no idea what the content is, please don't judge me._

## MysteryFile01

If you double click this file it should open in Quicktime, however this isn't super helpful for this exercise. We want to know what's going in onside the file. To do so we can run `mediainfo` on the file with the following commands

```
mediainfo MysteryFile01.mov
```

Mediainfo will then display what it can figure out about the file. This information is written into the file's header information. In the file's header there is a field for all of these metadata. Sometimes there's more than one field for some of the data, and sometimes there's not a field. In these cases mediainfo will do what it can figure out what the proper information is. The standard mediainfo output is not every single field, but rather the fields that mediainfo thinks are important with some of the redundant fields removed. To see everything, you can run this command

```
mediainfo -f MysteryFile01.mov
```

You'll see that there's a lot more info now! But some of it repeated info, or reformatted info so it's not super useful all the time.

Back to the original output, we can figure out a few things about this file by looing at the mediainfo output. First of all, you'll see three major sections:

- General
- Video
- Audio

The general section holds information about the container. in this case, the container format is `QuickTime` which refers to the MOV container.

There is one video section and one audio section. The video section `Codec ID` is `v210`, which is the 10-bit Uncompressed Codec. The `Format` for the audio section is `PCM` which is uncompressed audio. You can even look down at the `Sampling rate` and `Bit Depth` fields and see that they are `48.0 kHz` and `24bit` respectively. This is a nice 10-bit Uncompressed MOV file with 48kHz/24bit audio. A great choice for video preservation.

One of the most helpful fields that mediainfo can give you is the `Compression Mode` field. This will either say `Lossless` or `Lossy`. If the mode is `Lossy` then you know the file is not sufficient for preservation!

Another way to probe the file for information is to run this command

```
ffprobe -hide_banner MysteryFile01.mov
```

This runs `ffprobe` on the file, which is FFmpeg's tool for gathering information from multimedia streams. At the top you'll see a `Metadata` section that looks like this:

```
Metadata:
    major_brand     : qt  
    minor_version   : 512
    compatible_brands: qt  
    encoder         : Lavf59.16.100
```
This indicates that the file is a QuickTime, or MOV file. Then you'll see the stream information below:

```
Stream #0:0[0x1](eng): Video: v210 (v210 / 0x30313276), yuv422p10le(smpte170m/smpte170m/bt709, bottom coded first (swapped)), 720x486, 223725 kb/s, SAR 9:10 DAR 4:3, 29.97 fps, 29.97 tbr, 30k tbn (default)
    Metadata:
      handler_name    : VideoHandler
      vendor_id       : FFMP
      encoder         : Lavc59.18.100 v210
  Stream #0:1[0x2](eng): Audio: pcm_s24le (in24 / 0x34326E69), 48000 Hz, stereo, s32 (24 bit), 2304 kb/s (default)
    Metadata:
      handler_name    : SoundHandler
      vendor_id       : [0][0][0][0]
```

The stream notation is `x:y`, where `x` refers to the file number (in case you give it multiple files) and `y` refers to the stream number. Computers start counting at zero, so the first stream of the first file is always `0:0`

In this case, `stream #0:0` is the video stream in the file. It's v210 video, as expected. There's also a bunch of useful data like it's frame size, frame rate, and more.

 `stream #0:1` is the audio stream. it's in24, 48kHz, 24bit as expected!

 On last thing we can do with this file is play it back in `ffplay`. You can do that with this command.

 ```
ffplay -loop -1 MysteryFile01.mov
 ```

BTW, you can run this with out `-loop -1` flag, but this will keep looping the file and these files are pretty short so it can help with the exercises.

When you play the file in FFplay it'll open up in a new window. If you press `w` once it will show you the audio waveforms. If you do this you'll see that there are two audio channels. If you press `w` a second time it'll show you a specotrgram. This shows the the frequency information in the audio portion of the file. We'll talk about this more later! If you press `w` a third time the video will show again. Press `esc` to quit.

You now have some fun tools to play with! We'll use them on the rest of the files to figure out what they are.

## MysteryFile02

If you have VLC installed this file will open in VLC is you double click it. This is a good indication that the file is at least somewhat properly formed. However, VLC will play most things that you can throw at it.

Run ffprobe on the file and let's see what's inside:

```
Input #0, matroska,webm, from '/Users/morgan/Documents/GitHub/AV-Pres-Validation-Workflows/02MediaMysteryMeat/MysteryFile02.mkv':
  Metadata:
    title           : Mai Hime 01: That's a serious matter for a maiden
    encoder         : libebml v0.7.7 + libmatroska v0.8.0
    creation_time   : 2006-09-16T16:57:05.000000Z
  Duration: 00:00:23.18, start: 0.000000, bitrate: 1874 kb/s
  Chapters:
    Chapter #0:0: start 0.000000, end 23.180000
      Metadata:
        title           : Prologue
  Stream #0:0: Video: h264 (High), yuv420p(progressive), 704x480, SAR 10:11 DAR 4:3, 23.98 fps, 23.98 tbr, 1k tbn (default)
    Metadata:
      title           : Mai Hime 01: That's a serious matter for a maiden
  Stream #0:1(jpn): Audio: vorbis, 48000 Hz, stereo, fltp (default)
    Metadata:
      title           : 2.0 Vorbis
  Stream #0:2(eng): Audio: vorbis, 48000 Hz, stereo, fltp
    Metadata:
      title           : 2.0 Vorbis
  Stream #0:3(ger): Audio: vorbis, 48000 Hz, stereo, fltp
    Metadata:
      title           : 2.0 Vorbis
  Stream #0:4(fre): Audio: vorbis, 48000 Hz, stereo, fltp
    Metadata:
      title           : 2.0 Vorbis
  Stream #0:5(eng): Subtitle: ass (default)
    Metadata:
      title           : ASS
  Stream #0:6(eng): Subtitle: subrip
    Metadata:
      title           : SRT
  Stream #0:7(ger): Subtitle: ass
    Metadata:
      title           : ASS
  Stream #0:8: Attachment: ttf
    Metadata:
      filename        : MAIAN.TTF
      mimetype        : application/x-truetype-font
  Stream #0:9: Attachment: ttf
    Metadata:
      filename        : SanvitoPro-LtCapt.otf
      mimetype        : application/x-truetype-font
  Stream #0:10: Attachment: ttf
    Metadata:
      filename        : HighlanderStd-Book.otf
      mimetype        : application/x-truetype-font
Unsupported codec with id 98304 for input stream 8
Unsupported codec with id 98304 for input stream 9
Unsupported codec with id 98304 for input stream 10
```

If you check the top section you'll see this a `matroska` file. This file has WAY more than two streams. Stream `0:0` is an x.264 video stream. Streams `0:1` thru `0:4` are audio streams in different languages. Streams `0:5` thru `0:7` are subtitle streams in different languages. Streams `0:8` thru `0:10` are fonts for the subtitle streams.

That's a complicated file!

Now play it with ffplay

```
ffplay -loop -1 MysteryFile02.mkv
```

If you press `w` you'll onyl see two audio channels. That's because it's only showing one stream at a time. An easy way to access the extra streams is to play the file in VLC again. In VLC, select `Audio -> Audio Track` to see the additional audio streams. You can access the subtitles by selecting `Subtitle -> Subtitle Track`.

By probing this file we've seen just how complex MKV files can be. This is useful for Preservation in that we can properly store all the information from a videotape in a way that properly reflects its organization on the tape.

## MysteryFile03

This looks like it's probably an audio file, because it has a `.wav` extension. Let's open it in ffplay.

```
ffplay -loop -1 MysteryFile03.wav
```

You'll see the waveforms appear immediately. The audio is a voice saying the channel that audio is coming out of. You can easily see in waveform how the channel configuration relates to the organization of the streams. This file has 6 channels, it's a 5.1 surround. 5.1 refers to the follow streams:

  - 1  ) Front Left
  - 2  ) Front Right
  - 3  ) Center
  - 4  ) Back Left
  - 5  ) Back Right
  - 5.1) Subwoofer

## MysteryFile04

The next file looks like another .mov file, try double-clicking it to open it in QuckTime.

Wait, that didn't work! What's going on here. Probe the file to see what's happening

```
ffprobe -hide_banner MysteryFile04.mov
```

Here's what you should see

```
Input #0, matroska,webm, from 'MysteryFile04.mov':
  Metadata:
    encoder         : Haali Matroska Writer b0
  Duration: 00:00:40.24, start: 0.000000, bitrate: 4099 kb/s
  Stream #0:0(eng): Video: h264 (High), yuv420p(progressive), 1280x720, SAR 1:1 DAR 16:9, 25 fps, 25 tbr, 20k tbn (default)
```

The file is actually a Matroska file! It turns out files can lie. Even if the extension is .mov, the file itself can be an .mkv, or really anything. Be careful out there! Sometimes files won't even have extensions, but you can use ffprobe or mediainfo to see what's really going on in there.

Try playing the file in FFplay

```
ffplay -loop -1 MysteryFile04.mov
```

You'll see that it plays fine! You can also drag the file into VLC and it'll play fine. If you rename the file to `MysteryFile04.mkv` and double click it it'll open just fine in VLC. Extensions just tell the computer which software to open the file with, and don't affect the file's utility otherwise.

## MysteryFile05

This is a fun one. Try and open this file in VLC.

What do you think? The audio sounds fine, but the video looks crazy. What's going on here? Let's open the file in ffprobe.

```
ffprobe -hide_banner MysteryFile04.
```

Here's what you should see

```
[mxf @ 0x7ffb2f42df80] broken or empty index
[mxf @ 0x7ffb2f42df80] error getting stream index 67174400
[jpeg2000 @ 0x7ffb2f4301c0] Missing EOC Marker.
[mxf @ 0x7ffb2f42df80] Estimating duration from bitrate, this may be inaccurate
Input #0, mxf, from 'MysteryFile05.mxf':
  Metadata:
    operational_pattern_ul: 060e2b34.04010101.0d010201.01010900
    uid             : 81ddd167-c3b8-11de-a525-001b2128a1f2
    generation_uid  : 81ddd168-c3b8-11de-a2e4-001b2128a1f2
    company_name    : SAMMA Systems
    product_name    : MXF for SAMMA mjpeg2k
    product_version_num: 0.2.0.41.0
    product_version : 0.2.0.41
    application_platform: win32
    product_uid     : 43339ae6-9040-4e2c-be5f-a3de38328894
    toolkit_version_num: 0.2.0.41.0
    modification_date: 2009-10-28T11:53:28.788000Z
    material_package_umid: 0x060A2B340101010501010D121322F4CC034B8D0232510585CFD3001B2128A1F2
    timecode        : 01:03:34:21
  Duration: 00:00:26.65, start: 0.000000, bitrate: 3073 kb/s
  Stream #0:0: Video: jpeg2000, yuv422p(unknown/unknown/bt470m, top first), 720x243, lossless, SAR 9:20 DAR 4:3, 59.94 tbr, 59.94 tbn
    Metadata:
      file_package_umid: 0x060A2B340101010501010D121349F1F1034B8D0232510585BBE9001B2128A1F2
      file_package_name: Source Package
      track_name      : Track 1
  Stream #0:1: Audio: pcm_s16le, 48000 Hz, 4 channels, s16, 3072 kb/s
    Metadata:
      file_package_umid: 0x060A2B340101010501010D121349F1F1034B8D0232510585BBE9001B2128A1F2
      file_package_name: Source Package
      track_name      : Track 2
```

First, there's a few errors. That's not good but maybe not the end of the world. From the container metadata we can see that this is a SAMMA JPEG2000 MXF file! These file are notorious for being problematic. but let's see if we can make out more information about it. It has two streams, one video stream and one audio stream.

The video stream `0:0` is jpeg2000, yuv422, 4:3. This stuff is normal. The weird part is that the framerate is 59.94, which is double the standard 29.97 framerate, and the size is 720x243, which is half the height of the standard 720x486. What's happening here? These files hold the data for each field in their own frame. So what FFmpeg sees as a frame is actually half a frame. The two fields are combined to make a single frame for playback!

the audio stream `0:1` is 4 channels, 16 bit, 48kHz PCM. Looks pretty standard, but it's 16 bits instead of 24 bits.

Let's see what ffplay can do with this file

```
ffplay -loop -1 MysteryFile05.mxf
```

The file actually plays mostly ok in FFplay, which unlike VLC can properly make out the information. FFplay does report a bunch of errors, and the frame size isn't correct, but this is still better than VLC!

## MysteryFile06

Take a look at this file yourself! Probe it, play it, investigate, and see if you can make any interesting conclusions about this file. When you're done, expand the text below to see what I think about this files

<details>
  <summary>Don't open until you've investigated the file!</summary>

  This file is actually a properly formed FFV1/MKV file. It would work great as a preservation file!

</details>


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

## Log any potential errors

You may have seen some errors appear in the terminal window while FFplay plays back the files. This is a good way to identify any errors, but it's pretty easy to miss. The best way to dig down and see if a file has any errors is to ask FFmpeg to validate the file and put any issues it finds into a log. Here's the general command to do that:

```
ffmpeg -v error -i [input file] -f null - 2> [log file]
```

You can run this command on the four files that we have and see what the output looks like. These commands save the log as a sidecare file of the original video file, which is my favorite way to organize these kinds of files.

```
ffmpeg -v error -i FFV1.mkv -f null - 2> FFV1.mkv.error.log
```

```
ffmpeg -v error -i FFV1_HeavyCorruption.mkv -f null - 2> FFV1_HeavyCorruption.mkv.error.log
```

```
ffmpeg -v error -i FFV1_MediumCorruption.mkv -f null - 2> FFV1_MediumCorruption.mkv.error.log
```

```
ffmpeg -v error -i FFV1_LightCorruption.mkv -f null - 2> FFV1_LightCorruption.mkv.error.log
```

If the log is empty it means there are no errors. Otherwise, the log will list all of the errors that FFmpeg found while decoding the file.

# Exercise 04: Round Trip Transcode

As we discussed in the lecture, FFV1's compression is fully lossless. This means that if you create an FFV1 file from an uncompressed file you should be able to decompress the FFV1 and get back the same exact uncompressed file. We'll now prove that mathematically!

## Perform a Round-Trip Transcode

Copy or move the `Source_Uncompressed.mov` file from the second exercise to the folder named `04RoundTripTranscode`

Change your terminal directory to `04RoundTripTranscode` using the `cd` command

```
cd ..
cd 04RoundTripTranscode
```

Create an FFV1 file from the `Source_Uncompressed.mov` file with the following command

```
ffmpeg -i Source_Uncompressed.mov  -c:v ffv1 -level 3 -g 1 -slices 16 -slicecrc 1 -color_primaries smpte170m -color_trc bt709 -colorspace smpte170m -color_range mpeg -metadata:s:v:0 'encoder= FFV1 version 3' -c:a copy -vf setfield=bff,setsar=40/27,setdar=4/3 -metadata creation_time=now -f matroska -vf setfield=bff,setdar=4/3 FFV1_From_MOV.mkv
```

Now, create an MOV file from the `FFV1_From_MOV.mkv` file with the following command

```
ffmpeg -i FFV1_From_MOV.mkv -movflags write_colr -c:v v210 -color_primaries smpte170m -color_trc bt709 -colorspace smpte170m -color_range mpeg -metadata:s:v:0 "encoder=Uncompressed 10-bit 4:2:2" -c:a copy -vf setfield=bff,setsar=40/27,setdar=4/3 -f mov MOV_From_FFV1.mov
```

## Perform a visual comparison

We now have two MOV files, one that we started with (this was created with vrecord) and one that we created from the FFV1 file. Everything we know about FFV1 says that these files should be identical. How can we prove it? Visually we can do a difference blend like we did in exercise two using this command:

```
ffplay -f lavfi "movie=Source_Uncompressed.mov,setpts=PTS-STARTPTS,format=gbrp10le,split[a1][a2];movie= MOV_From_FFV1.mov,setpts=PTS-STARTPTS,format=gbrp10le,split[b1][b2];[a1][b1]blend=all_mode=difference,format=yuv422p10le,histeq=strength=0.1:intensity=0.2,pad=2*iw:ih:0:0[down];[a2][b2]hstack[up];[up][down]vstack"
```

You should see nothing on the bottom row, meaning that they're perfectly visually identical. We already did this, so it's sort of old news

## Perform a mathematical comparison

Now, let's perform a bit-level comparison of the uncompressed video stream. We can check to see if the video streams are identical by making checksums of the video streams in reach file. Run the following commands to get that checksum info. This command makes a single checksum of just the video steam data in each file

```
ffmpeg -i Source_Uncompressed.mov  -map 0:v -f md5 Source_Uncompressed.mov.videomd5
```

```
ffmpeg -i FFV1_From_MOV.mkv  -map 0:v -f md5 FFV1_From_MOV.mkv.videomd5
```

```
ffmpeg -i MOV_From_FFV1.mov  -map 0:v -f md5 MOV_From_FFV1.mov.videomd5
```

You can now open each of these files in TextEdit or whatever you use to view text files. You should see that these files are perfectly match! Even a single bit difference between the video streams would result in a dramatically different checksum. The fact that these three files match means that the video information in all three files is exactly identical when decoded.

This is different than if we just did a checksum of the entire file. You can do that with the following three commands:


```
md5 -q Source_Uncompressed.mov > Source_Uncompressed.mov.md5
```

```
md5 -q FFV1_From_MOV.mkv > FFV1_From_MOV.mkv.md5
```

```
md5 -q MOV_From_FFV1.mov > MOV_From_FFV1.mov.md5
```

You'll notice that these are all different. That's because the files themselves are not fully identical. Even the two MOV files are different. That's because the container information and many non-essence parts of the file are different. But, the video data in them is identical when decoded!

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


# Exercise 05: Audio Station Qualification

## Investigating Spectrograms

A spectrogram is a visual representation of a file's frequency spectrum.

First, Change your terminal directory to `05AudioStationQualification` using the `cd` command

```
cd ..
cd 05AudioStationQualification
```


There is a sample music file included in this folder. Create a spectrogram of the song with the following commnand.

```
ffmpeg -i Sample_Music.wav -vn -lavfi showspectrumpic=s=1280x480 Sample_Music.wav.png
```

_Note: Alternatively, you can create spectorgrams using `sox` however this wont work on video files, which we'll do later. This is why we've switched over to using FFmpeg to create them. the following is the command for creating them with `sox`_

```
sox Sample_Music.wav -n spectrogram -o Sample_Music.wav.png
```

Open up the spectrogram and see if you can follow along with with the music. You should be able to see notes appear in the spectrogram. Spectrograms can be very useful for troubleshooting nad qualifying digitization stations.

### Sample Rate Issues

The music sample was recorded at 48kHz. This means that audio up to 24kHz can be represented. If you look on the left side of the image you'll see that the frequency goes up to 24kHz.

Now, let's create a 96kHz file from the 24kHz file:

```
ffmpeg -i Sample_Music.wav -c:a pcm_s24le -ar 96000 Sample_Music_96kHz.wav
```

If you listen to the file, there should be no audible difference between the two.

Now, make a spectrogram of the 96kHz file.

```
ffmpeg -i Sample_Music_96kHz.wav -vn -lavfi showspectrumpic=s=1280x480 Sample_Music_96kHz.wav.png
```

If you open this up, you'll see that the scale on the left goes up to 48kHz instead, but that the frequency spectrum is empty above 24kHz. FFmpeg knows that the file is 96kHz, so the file can represent frequencies up to 48kHz. However, because the file was originally recorded at 48kHz there no audio above 24kHz in the file.

This is an example of what would happen if your audio interface was not properly digitizing your analog signal. If you interface was converting the file as a 48kHz, but recording the file as 96kHz this is what you'd see. Or, for example, if your digitization vendor is recording files at 48kHz but then exporting or delivering 96kHz this is what you'll see.

### Dropped Samples

Now, let's create a sample file that's just a sine wave:

```
ffmpeg -f lavfi -i "sine=frequency=1000:duration=5" -c:a pcm_s24le -ar 96000 -ac 2 SineWave.wav
```

Now, make a spectrogram of this file:

```
ffmpeg -i SineWave.wav -vn -lavfi showspectrumpic=s=1280x480 SineWave.wav.png
```

Open up the spectrogram. you should just see a line. That's because sine waves only have one frequency!

Now, let's create a sine wave file with some dropped samples.

```
ffmpeg -f lavfi -i "sine=frequency=1000:duration=5" -bsf:a noise=drop='eq(mod(n\,50)\,1)' -c:a pcm_s24le -ar 96000 SineWave_DroppedSamples.wav
```

Listen to this file. You should hear a small click whenever a sample is dropped. One way to qualify a station would be to record audio and listen for this clicks. However, this takes a long time and would be very difficult to do. Let's make it easier by making a spectrogram of this file:

```
ffmpeg -i SineWave_DroppedSamples.wav -vn -lavfi showspectrumpic=s=1280x480 SineWave_DroppedSamples.wav.png
```

Now, view the spectrogram. You'll see big lines of noise where the dropouts are. That's because dropouts manifest as momentary broadband noise. As you can, creating a spectrogram of a file is an easy way to see whether samples have been dropped. Testing files created by your audio digitization workstation for dropouts is one way to qualify that workstation.


## Performing an Audio Null Test

Create two sine wav files. Normally when performing a Null test you have a control file and a Test File. The Control File is made by a "known good" station and the Test File is made by the station you are testing. In this case however, we're just going to generate the files for simplicity's sake

```
ffmpeg -f lavfi -i "sine=frequency=1000:duration=5" -c:a pcm_s24le -ar 96000 -ac 2 Control.wav
```

```
ffmpeg -f lavfi -i "sine=frequency=1000:duration=5" -c:a pcm_s24le -ar 96000 -ac 2 Test.wav
```

Ideally these files will be identical. If we added them together now you'd simply get a louder. In order to do a proper null test we'll need to phase reverse the Test File.

```
ffmpeg -i Test.wav -c:a pcm_s24le -ar 96000 -af "aeval='-val(0)':c=same" Test_Phase_Reversed.wav
```

Now we'll add the Control file and the Phase Reversed Test file together.

```
ffmpeg -i Control.wav -i Test_Phase_Reversed.wav -filter_complex amix=inputs=2:duration=longest File_Null_Test_Output.wav
```

The output should be completely silent! This proves that the two files we stated with, the Control File and Test File, were identical! This shouldn't be surprising, since the files were generated with the same original FFmpeg command. However, if you had created these two files for the sake of station qualification they should still be identical!

Let's see what happens if the files aren't identical. First, make a noisy version of the Test File

```
ffmpeg -f lavfi -i "sine=frequency=1000:duration=5" -bsf:a noise=amount=-1 -c:a pcm_s24le -ar 96000 -ac 2 Test_w_Noise.wav
```

Now create a reversed phase version of the Noisy Test File

```
ffmpeg -i Test_w_Noise.wav -c:a pcm_s24le -ar 96000 -af "aeval='-val(0)':c=same" Test_w_Noise_Phase_Reversed.wav
```

Now we'll add the Control file and the Noisy Phase Reversed Test file together.

```
ffmpeg -i Control.wav -i Test_w_Noise_Phase_Reversed.wav -filter_complex amix=inputs=2:duration=longest Noisy_File_Null_Test_Output.wav
```

The output should be just noise! Only the non-similar information is retained after the null test. So, if you try to perform a null test and you hear noise or some sort of content, you know that your files are not identical and the station qualification has failed in some way.

## Using MediaConch to Check Format Conformance

MediaConch stands for Media Conformance Checker. This tool is used to make sure that files properly conform to desired specifications. For this exercise we're going to check some of the files we've created against an agreed-upon specification profile.

Open MediaConch, navigate to the Public Policies tab, and scroll down until you find profile named `PARADISEC Audio is 96kHz/24bit, stereo`

Download this policy by pressing `Add to my policies`

Go back to the Checker tab. Clear any existing files by pressing `x Close All Results`

Drag the following files into the MediaConch window:

- Sample_Music.wav
- Sample_Music_96kHz.wav
- Test.wav
- Test_w_Noise.wav

Now, click the dropdown next to `Apply a policy to all results` and select `PARADISEC Audio is 96kHz/24bit, stereo`

MediaConch will now test all the files against that policy. You should see the following results:

- Sample_Music.wav - FAIL
- Sample_Music_96kHz.wav	- PASS
- Test.wav	- PASS
- Test_w_Noise.wav - PASS


Sample_Music.wav fails because the file is not 96kHz, but is 48kHz. Remember however, that in the previous example we noted that Sample_Music_96kHz.wav was actually a problematic file. Even though the file itself was 96kHz, it was clear by looking at the spectrogram that the file had originally been recorded as 48kHz. This shows that while MediaConch can be useful for checking file specs, it can't always catch a bad file.

The same issue can be been in how Test.wav passes, but so does Test_w_Noise.wav. Mediaconch doesn't know that there has been problematic noise added to the Test_w_Noise.wav file, it just knows that the file meets the desired file specifications.

Keep this in mind when using MediaConch. It's possible for a file to fail MediaConch even though it's correct, and it's possible for a file to pass MediaConch even though it's wrong. MediaConch is just one of many useful tools available for station qualification.

# Exercise 06: Video Station Qualification

Now that you've learned some ways to analyze and probe files, it's time to use those methods to qualify some file. On the internet archive there are 9 .mkv files. Download the Matroska files from this page:

https://archive.org/details/qualification-test-file-02

Here's a direct link to download those files:

https://archive.org/compress/qualification-test-file-02/formats=MATROSKA&file=/qualification-test-file-02.zip

Once you have the files downloaded, move them to the `06VideoStationQualification` folder and move your terminal window into that folder using the `cd` commands

```
cd ..
cd 06VideoStationQualification
```

Now, use what you've learned in this workshop to see which of these files comes from a properly qualified digitization workstation. Only one of these files is error free! See if you can find the correct one. You may be able to do so on your own, but in case you get lost there's a series of hints below. Expand the text to see the hints.


<details>
  <summary>Hint 1</summary>

  Run the files against the NYPL_FFV1 MediaConch policy to if the files properly conform to the agreed upon specifications for FFV1/MKV Preservation Files

</details>

<details>
  <summary>Hint 2</summary>

  The following command will create an FFmpeg error log for every file in the directory. You can use this log to see if there are any errors in the files that FFmpeg can see

  ```
  for file in *.mkv ; do ffmpeg -v error -i "$file" -f null - 2>  "${file}.error.log" ; done
  ```

</details>

<details>
  <summary>Hint 3</summary>

  The following command will create a spectrogram for every file. You can use this to see if there are any errors in the frequency specturum

  ```
  for file in *.mkv ; do ffmpeg -i "$file" -vn -lavfi showspectrumpic=s=1280x480 "${file}.png" ; done
  ```

</details>

<details>
  <summary>Hint 4</summary>

  The following command will create a QCTools Report for every file. You can use this to quickly load the QCTools information. From there you can further investigate the files for potential errors.

  ```
  for file in *.mkv ; do qcli -i "$file" ; done
  ```

</details>

<details>
  <summary>Hint 6</summary>

  Now that you have QCTools reports generated, you can look for common errors. Are there repeated frames? Does the waveform look good, or is it clipped or chopped up? Do all the bitplanes contain data?

</details>

<details>
  <summary>Hint 7</summary>

  Don't forget to visually inspect the files! Use your eyes and ears. Also, FFplay is a better playback platform for troubleshooting than VLC because sometimes VLC will fix problems it sees, whereas FFplay will play exactly what it encounters, even if it's problematic.

</details>

<details>
  <summary>Hint 8</summary>

  When you're visually inspecting the files, check to make sure that the files look interlaced. It's easiest to see interlacing when things move from side to side. Also, check to see that the audio and video are in sync with each other.

</details>
