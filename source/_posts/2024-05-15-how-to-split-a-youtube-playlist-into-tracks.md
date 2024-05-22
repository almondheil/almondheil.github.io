---
layout: post
category: til
slug: split youtube cuelist
title: How to Split a YouTube Playlist Into MP3 Tracks
---

Today, I learned how to split a downloaded YouTube playlist into individual tracks
so I could play them in VLC more easily.

I wanted to write a tutorial about it, both for
my future reference and in case anyone else wants to do the same thing.

## Table of contents
- [Table of contents](#table-of-contents)
- [Requirements](#requirements)
- [Downloading the Playlist Audio](#downloading-the-playlist-audio)
- [The Cuesheet Format](#the-cuesheet-format)
- [Creating Our Own Cuesheet](#creating-our-own-cuesheet)
- [Splitting the Downloaded Audio](#splitting-the-downloaded-audio)
- [Optional: Adding Metadata and Album Art](#optional-adding-metadata-and-album-art)

## Requirements

The following tools are needed to follow this tutorial. I've given the names
of packages I had to install on Arch Linux, so your package names may vary.
- ffmpeg
- perl (version 5)
- yt-dlp
- cuetools
- python-mutagen
- lame

## Downloading the Playlist Audio

First, download the playlist with yt-dlp. The details here correspond to the
playlist I was working with when first attempting this project.

```
yt-dlp --extract-audio --audio-quality 0 \
    "https://www.youtube.com/watch?v=Hti1jmkMrnU" \
    --output "azure - nujabes playlist for the groovy yayayayy"
```

Flag breakdown:
 - `--extract-audio` saves an audio file, rather than a video one.
 - `--audio-format mp3` specifies the audio format that should be used. I chose mp3 because YouTube audio is already lossy.
 - `--audio-quality 0` maximizes the audio quality we download.
 - `--output` sets the name of the saved file.

## The Cuesheet Format

At this point, we have one big audio file, but we the individual tracks. 

There's only one issue: every tutorial I've been able to find assumes you already
have a cuesheet file (with the `.cue` extension), which tells audio software where
each track starts within the larger file.

We don't have, one of those, though. Luckily, cuesheets are a plaintext format,
and we can roll our own! There is documentation for the format on the [libodraw GitHub repo](https://github.com/libyal/libodraw/blob/main/documentation/CUE%20sheet%20format.asciidoc){:target="_blank"}, which I found very helpful. I also referenced the Wikipedia page about cue sheets, specifically the [examples section](https://en.wikipedia.org/wiki/Cue_sheet_(computing)#Examples){:target="_blank"}.

There are three really important things I gleaned from reading these sources:

1. The cue file is laid out with a `FILE` at the top, that tells it what file the tracks come from
2. For each track in the playlist, there is a `TRACK` block. 

    It contains info about the track, but the most important thing is the start
    time within the file. This is written as an `INDEX 01` line.

3. The format for `INDEX` is `mm:ss:ff`, where that third item is frames. I just left frames set to `00`, since the timestamps on the YouTube video only go down to the second.
  
    Notice that the 'minutes' spot only gets two characters. That means we can't have a track that starts later than 1 hour 40 minutes. However, you should be able to manually split your audio file into two or more 99-minute chunks, in which case your cuesheet will have multiple `FILE` entries. I've never tried this before, however.

## Creating Our Own Cuesheet

With this knowledge in hand, I was able to write the following `playlist.cue` based on the one with the video, but tweaked to be a tiny bit more accurate for some songs.

You can write a cuesheet by hand, but as I tried to convert more 
playlists the novelty wore off and I got really bored.

To make it easier, I wrote a script that converts from a YouTube
comment containing timestamps (the kind you often find below one of 
these playlists) into a cuesheet `TRACK` list. You can find
it [on my GitHub](https://github.com/almondheil/comment-to-cuesheet){:target="_blank"}.

```
REM GENRE Lo-Fi
REM DATE 2024
PERFORMER Nujabes
TITLE "nujabes playlist for the groovy yayayayy"
FILE "azure - nujabes playlist for the groovy yayayayy.mp3" MP3
  TRACK 01 AUDIO
    TITLE "F.I.L.O. (Instrumental)"
    PERFORMER "Nujabes, Shing02"
    INDEX 01 00:00:00
  TRACK 02 AUDIO
    TITLE "Luv (sic)"
    PERFORMER "Nujabes, Shing02"
    INDEX 01 03:31:00
  TRACK 03 AUDIO
    TITLE "Lady Brown (instrumental)"
    PERFORMER "Nujabes, Cise Starr"
    INDEX 01 08:18:00
  TRACK 04 AUDIO
    TITLE "The Final View"
    PERFORMER "Nujabes"
    INDEX 01 11:32:00
  TRACK 05 AUDIO
    TITLE "After Hanabi ~~Listen to My Beats~~"
    PERFORMER "Nujabes"
    INDEX 01 15:14:00
  TRACK 06 AUDIO
    TITLE "battlecry"
    PERFORMER "Nujabes, Shing02"
    INDEX 01 20:31:00
  TRACK 07 AUDIO
    TITLE "Think Different"
    PERFORMER "Nujabes, Substantial"
    INDEX 01 23:52:00
  TRACK 08 AUDIO
    TITLE "Gone Are The Days"
    PERFORMER "Nujabes, Uyama Hiroto"
    INDEX 01 27:09:00
  TRACK 09 AUDIO
    TITLE "Sky Is Falling"
    PERFORMER "Nujabes, C.L. Smooth"
    INDEX 01 33:11:00
  TRACK 10 AUDIO
    TITLE "Kiss OF life"
    PERFORMER "Nujabes, Giovanca, Benny Sings"
    INDEX 01 37:53:00
  TRACK 11 AUDIO
    TITLE "Horn in the middle"
    PERFORMER "Nujabes"
    INDEX 01 42:01:00
  TRACK 12 AUDIO
    TITLE "Far Fowls"
    PERFORMER "Nujabes"
    INDEX 01 46:26:00
  TRACK 13 AUDIO
    TITLE "Summer Gypsy"
    PERFORMER "Nujabes"
    INDEX 01 50:45:00
  TRACK 14 AUDIO
    TITLE "Perfect Circle"
    PERFORMER "Nujabes"
    INDEX 01 54:46:00
  TRACK 15 AUDIO
    TITLE "Sky is Tumbling"
    PERFORMER "Nujabes, Cise Starr"
    INDEX 01 59:15:00
  TRACK 16 AUDIO
    TITLE "Kumomi"
    PERFORMER "Nujabes"
    INDEX 01 63:09:00
  TRACK 17 AUDIO
    TITLE "Luv (sic) pt2"
    PERFORMER "Nujabes, Shing02"
    INDEX 01 67:43:00
  TRACK 18 AUDIO
    TITLE "Feather"
    PERFORMER "Nujabes, Cise Starr, Akin"
    INDEX 01 70:38:00
```

> NOTE: I chose a bad first playlist, because the timestamps that I copied from
> the comments don't actually line up with the audio tracks. The above cuesheet
> attempts to fix those errors.

## Splitting the Downloaded Audio

Now that we have a finished cuesheet, this tutorial starts looking more like others that
can be found online. I'm going to see this to the end, just so I can document all the steps I ended up taking.

I used a [perl script by xrgtn on GitHub](https://github.com/xrgtn/split-cue/){:target="_blank"} to split the audio file. I initially attempted to use the `shnsplit` program as the Arch Wiki recommends, but I had issues where it cut tracks in the
wrong places. I still don't know what went wrong there.

```
./split-cue -d playlist.cue
```

Here, the `-d` flag disables the CDDB query functionality of the script. This is
a feature that I didn't require, and I didn't want to spend the time of installing
the perl module correctly.

## Optional: Adding Metadata and Album Art

After the script runs, you will have a directory with an mp3 file for each track.
Next, use the metadata contained in the cuesheet to flesh out each file better.

To get this script, I had to install the `cuetools` package.

```
cd '2024 - Nujabes - nujabes playlist for the groovy yayayayy'
cuetag.sh ../playlist.cue *.mp3
```

To add album art, you first need to find a suitable image. I went with the art
linked on the YouTube playlist, 
<https://yukoart.com/work/inq-mobile/>{:target="_blank"}. Save your image 
as `image.jpg` in the same directory as the individual songs.

To add the album art to a file, I found a useful command from Baeldung (<https://www.baeldung.com/linux/terminal-music-add-album-art>{:target="_blank"}). For mp3 files, we use lame. There's one drawback: lame can't add the art in-place.

I wrote a really simple bash loop that will add art to all the mp3 files in the current
directory. 

> **WARNING**: Make sure you have the `image.jpg` in the same directory, because this
> loop doesn't check for errors at all.
>
> If you don't have it in the right place and with the right name, this just 
> deletes all your audio files and you'll have to regenerate them.

```
for file in *.mp3; do
	tempfile="${file}.temp"
	mv "$file" "$tempfile"
	ffmpeg -i "$tempfile" -i image.jpg -map_metadata 0 -map 0 -map 1 -acodec copy "$file"
	rm "$tempfile"
done
rm image.jpg
```

Now, the album art should be there across all the sound files!