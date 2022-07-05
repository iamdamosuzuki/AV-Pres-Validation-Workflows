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

# Exercise 01: Compression Differences

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

# Exercise 02: Mediainfo Mystery Meat

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


# Exercise 05: Station Qualification

## Investigating Spectograms

A spectrogram is a visual representation of a file's frequency spectrum.

There is a sample music file included in this folder. Create a spectrogram of the song with the following commnand.

```
ffmpeg -i Sample_Music.wav -vn -lavfi showspectrumpic=s=1280x480 Sample_Music.wav.png
```

Open up the spectrogram and see if you can follow along with with the music. You should be able to see notes appear in the spectrogram. Spectrograms can be very useful for troubleshooting nad qualifying digitization stations.

### Sample Rate Issues

The music sample was recorded at 48kHz. This means that audio up to 24kHz can be represented. If you look on the left side of the image you'll see that the frequency goes up to 24kHz.

Now, let's create a 96kHz file from the 24kHz file:

```
ffmpeg -i Sample_Music.wav -c:a pcm_s24le -ar 96000 Sample_Music_96kHz.wav
```

If you listen to the file, there should be no audible difference between the two.

Now, make a spectogram of the 96kHz file.

```
ffmpeg -i Sample_Music_96kHz.wav -vn -lavfi showspectrumpic=s=1280x480 Sample_Music_96kHz.wav.png
```

If you open this up, you'll see that the scale on the left goes up to 48kHz instead, but that the frequency spectrum is empty above 24kHz. FFmpeg knows that the file is 96kHz, so the file can represent frequencies up to 48kHz. However, because the file was originally recorded at 48kHz there no audio above 24kHz in the file.

This is an example of what would happen if your audio interface was not properly digitizing your analog signal. If you interface was converting the file as a 48kHz, but recording the file as 96kHz this is what you'd see. Or, for example, if your digitization vendor is recording files at 48kHz but then exporting or delivering 96kHz this is what you'll see.

### Dropped Samples

Now, let's create a sample file that's just a sine wave:

```
ffmpeg -f lavfi -i "sine=frequency=1000:duration=5" -c:a pcm_s24le -ar 96000 SineWave.wav
```

Now, make a spectrogram of this file:

```
ffmpeg SineWave.wav -vn -lavfi showspectrumpic=s=1280x480 SineWave.wav.png
```

Open up the spectogram. you should just see a line. That's because sine waves only have one frequency!

Now, let's create a sine wave file with some dropped samples.

```
ffmpeg -f lavfi -i "sine=frequency=1000:duration=5" -bsf:a noise=drop='eq(mod(n\,50)\,1)' -c:a pcm_s24le -ar 96000 SineWave_DroppedSamples.wav
```

Listen to this file. You should hear a small click whenever a sample is dropped. One way to qualify a station would be to record audio and listen for this clicks. However, this takes a long time and would be very difficult to do. Let's make it easier by making a spectogram of this file:

```
ffmpeg SineWave_DroppedSamples.wav -vn -lavfi showspectrumpic=s=1280x480 SineWave_DroppedSamples.wav.png
```

Now, view the spectogram. You'll see big lines of noise where the dropouts are. That's because dropouts manifest as momentary broadband noise. As you can, creating a spectogram of a file is an easy way to see whether samples have been dropped. Testing files created by your audio digitization workstation for dropouts is one way to qualify that workstation.


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
