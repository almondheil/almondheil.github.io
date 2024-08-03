---
layout: post
category: til
slug: id3v2 set metadata
title: Using id3v2 to Set Metadata
---

Recently I bought a couple They Might Be Giants albums from their website, but when I was playing
them they each showed up as having no author. I was confused and frustrated, so I decided to look
into how to fix the metadata so they would be listed properly.

I looked for a good command-line tool to set the metadata, and my searches turned up the tool id3v2.

It sets a lot of things, but these are the flags I care about:

- -a (author)
- -A (album)
- -t (song title)
- -T (track number)
- -y (year)

So, I went ahead and made a quick python script to do this. I've been using the subprocess library
at work recently, so I decided it would be easy to use that to run the actual command so I would
have access to easy data structures.

Here's the script I ended up using! This metadata is for the album "Almanac", which was free when I
checked the website today. As you can see, I wrote the script assuming the files were WAVs in the
current directory. 

{% highlight python linenos %}
import subprocess

class Song:
    def __init__(self, title, fname):
        self.title = title
        self.fname = fname

    def __str__(self):
        return self.title

artist = "They Might Be Giants"
album = "Almanac"
year = "2004"
tracklist = {
    1: Song("Clap Your Hands", "01 Clap Your Hands.wav"),
    2: Song("Experimental Film", "02 Experimental Film.wav"),
    3: Song("John Lee Supertaster", "03 John Lee Supertaster.wav"),
    4: Song("Particle Man", "04 Particle Man.wav"),
    5: Song("It's Kickin' In", "05 It's Kickin' In.wav"),
    6: Song("Dr. Worm", "06 Dr. Worm.wav"),
    7: Song("Famous Pokla", "07 Famous Polka.wav"),
    8: Song("Stalk of Wheat", "08 Stalk of Wheat.wav"),
    9: Song("I Palindrome I", "09 I Palindrome I.wav"),
    10: Song("Wicked Little Critta", "10 Wicked Little Critta.wav"),
    11: Song("Some Crazy Bastard", "11 Some Crazy Bastard.wav"),
    12: Song("Drink!", "12 Drink!.wav"),
    13: Song("Thunderbird", "13 Thunderbird.wav"),
    14: Song("Violin", "14 Violin.wav"),
    15: Song("Damn Good Times", "15 Damn Good Times.wav"),
    16: Song("Fingertips", "16 Fingertips.wav"),
    17: Song("Robot Parade", "17 Robot Parade.wav"),
    18: Song("Thank You", "18 Thank You.wav"),
    19: Song("End of the Tour", "19 End of the Tour.wav"),
}

def main():
    total_tracks = len(tracklist)
    for track_num, song in tracklist.items():
        track_formatted = "{0}/{1}".format(track_num, total_tracks)
        command = ['id3v2', '-a', artist, '-A', album, '-T', track_formatted, '-y', year, '-t', song.title, song.fname]
        print(f'setting data on {song.fname}')
        subprocess.run(command, check=True)

if __name__ == '__main__':
    main()
{% endhighlight %}

This script worked perfectly for all the songs I was interested in, and now they're all correctly
displayed when I look at them on my media server.
