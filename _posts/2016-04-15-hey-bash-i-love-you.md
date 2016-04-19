---
layout:     post
title:      Hey BASH, I love you.
date:       2016-04-15 12:22:00
author:     Franklin
summary:    An article about why i love bash.
categories: bash scripts youtube-dl ffmpeg

tags:
 - Bash-scripts
 - youtube-dl
 - ffmpeg
---

### About me :-

I  am from Chenapalli, it is a beautiful village located ~70km from Kanyakumari. I came to north India three years back for my carrier. Presently (8-April-2016) i am in Gurgaon (~2800 km distance). Once or twice a year i go to my home. Every time i go, i bring something to my family individuals. So far i have not given anything to my lovely mom. So this time (2016 Easter holidays) i decided to surprise her with an android phone, hence I bought a Moto G3.

### Planning to convince my mom to use Moto G3 :-
I know if i give this phone to her, she is not going to accept it. Even if she accepts, she is not going to use it. Last year we gave a button phone she is not using it now. She does not feed it, does not make any call, rarely accepts any call, does not check call history, sms, etc. When i asked her why are you not using it, she answered, i don't have time to do all this. I know life in village is quite busy, whole day is not just enough to take care of the family, farms, and animals. I do agree with her, So i decided to convince her by giving this phone with something she likes and she can make use of it while doing other works. She is very pious and so fond of Christian songs. So i bought a 32GB Scandisk memory card(though Moto G3 has 16GB internal) and wanted to fill it with tamil christian songs.

### About my songs collections :-

When i was 14 years old my Dad bought me a 100 sony CDs pack, i started collecting Christian songs. In that CD pack Nearly 40 VCD(MPEG) and 20 Audio Disks(MP3, WMA) i have in good condition, I have lots of songs in DVDs(VOB) (Second time Dad bought me DVD pack).

### DownloadHelper :-

In 2011 i started using youtube. One day when i was surfing internet i found a christian video song, i liked it and i wanted to download it. I googled how to download youtube videos. I came across [DownloadHelper][0] a mozilla Add-on. It was pretty easy use, and helped to download the current playing song and it allowed me to download the  desired format (with in the available format).

### Movgrab :-

The problem was, if internet went off in the middle, i could not resume the download. So i was looking for alternate options. One day i came across [movgrab][1]. It is a c based command line tool. I was using movgrab for sometime till one day i came across a [Messianic playlist][5]. I was wondering if i could download the entire playlist.

### youtube-dl :-

I googled for downloading a playlist from youtube. I came across [youtube-dl][2]. It is a [python][3] based command line tool. Installing / updating youtube-dl is very simple. (just one command [pip][4] install youtube-dl) Downloading a playlist is even simpler. I am using youtube-dl since 2012, and i am able to download pretty much anything from youtube without any problem. youtube-dl is one of my *FAVORITE* command line tool now. One day i came across a youtube channel which has _2544_ songs, and they update every week. I wanted to download the entire channel. Obviously the first thing i came to my mind was youtube-dl. I started downloading the channel by following command.

{% highlight sh %}
  youtube-dl -cit https://www.youtube.com/channel/xxxxxxxxxxxxxxxxxxxxxxxx/videos
{% endhighlight %}

### Troubles while dealing with huge playlist :-

It started downloading the channel. The problem was my internet. My internet connection was not so good. Average download speed was 70Kb/s. By default youtube-dl downloaded the best available format and i wanted the best format. Average size of each song was 30Mb. I knew it will take weeks may be months to download the entire channel. First day was good, i think it downloaded around 100 songs. I wanted to give some rest to my system. I killed youtube-dl and shutdown my system.

Next day i ran the same command and it parse the playlist again and went through each downloaded songs and if it
found any song was partially downloaded, it completed the partial files and went ahead. It took few minutes to start downloading the new song (actually 101st songs). I thought it was ok because it did not download any song for the second time but resumed the partially download files. Second day it downloaded another 100 songs. I thought of giving some rest to my system. I killed youtube-dl and shutdown my system. It was fine till it reached 500 songs. After 500 songs when i ran the command to download the songs again it took a lot of time to reach the new download. It ran through all the indexes and compared with the downloaded songs in the folder.

#### Simple solution - Don't let youtube-dl die :-

* I find the process id of youtube-dl by running the following command.
{% highlight sh %}
ps -ax | grep youtube
{% endhighlight %}
* Suspend the process(PID is 2294) using the following command.
{% highlight sh %}
kill -TSTP 2294
{% endhighlight %}
* Put the system in sleep. (May be giving some rest to my system.)
* Later resume the process using following command, youtube-dl will continue from the place where it was suspended.
{% highlight sh %}
kill -CONT 2294
{% endhighlight %}


This was quite helpful while situation such as power cut, wifi/internet failure, home (to/from) work travel. I have used this idea in all such situations and it was helpful. So i thought i have solved one problem. I did not shut down my system for few days. One day my system crashed and restarted automatically. OOPS! the process(2294) died. This was not the first time i encountered Mac crash. I should have foreseen it. So this was not the best solution. I was surfing for another options!

#### Best solution :-
Yey! found a solution from youtube-dl site. Yeah, it was --download-archive flag.
--download-archive FILE - Download only videos not listed in the archive file. Record the IDs of all downloaded videos in it. Well this is what exactly i was looking for. I should have used the flag from the beginning. I was in the middle of downloading a playlist with 2500+ songs and one fourth of the songs were already downloaded. Fortunately i used -t flag which downloaded the songs with it's original title and youtube id. Next step was extract youtube-id from all the download songs.

I ran the following bash script to extract the youtube-id from all the downloaded files and append them in downloaded.txt file.

{% highlight sh %}
for n in *.*; do if [[ "$n" =~ -[-_0-9a-zA-Z]{11}.*$ ]]; then echo "youtube ${n: ${#n}-15: 11}" >> downloaded.txt;fi;done
{% endhighlight %}

This script parsed each file in the current directory and took all the file titles and extracted the 11 char youtube id and appended them with youtube as prefix in downloaded.txt file.

There we go, in no time it wrote all the ids of all downloaded songs in downloaded.txt file. Now all i have to do is just run the following command.

{% highlight sh %}
    youtube-dl --download-archive downloaded.txt -cita toDownload.txt
{% endhighlight %}

Here downloaded.txt file has all the youtube ids and toDownload.txt has the youtube channel's URL. In few weeks, i downloaded the complete channel. I follow this channel, if any new upload i run the same command and it downloads the new song. This is how i have been collecting songs from youtube.

### youtube-dl in android:-

My post paid mobile plan includes 1GB 3G data. I was looking for an option to download youtube video from my android phone as well. I tried few application from playstore to download youtube songs. I felt no app was as good as youtube-dl. I wanted to use youtube-dl in mobile. I was looking for terminal app android.

#### Termux:-

Found [termux][11]. It is a java based terminal emulator. It works perfectly fine. Installed it on a Marshmallow device. Then installed python and youtube-dl by running the following commands.
{% highlight sh %}
  apt update
  apt install python-dev
  pip install youtube-dl
{% endhighlight %}

Unfortunately it does not work in kitkat(19), it requires at least lollipop(21). My android phone is still running on kitkat.  

#### QPython:-

Then i came across [QPython][12] - a python console. QPython has its own shell and allows to run pip command, I installed youtube-dl using pip. Then i tried youtube-dl command, it worked.

{% highlight sh %}
# Downloading a playlist
youtube-dl --download-archive /sdcard/pious/tamilChristianSongs/downloaded.txt -o '/sdcard/pious/tamilChristianSongs/%(title)s-%(id)s.%(ext)s' -ci https://www.youtube.com/user/XXXXXXXXXXXXXXXXXX/videos

# Downloading one or more song, playlist(s).
youtube-dl --download-archive /sdcard/pious/tamilChristianSongs/downloaded.txt -o '/sdcard/pious/tamilChristianSongs/%(title)s-%(id)s.%(ext)s' -cia /sdcard/pious/tamilChristianSongs/mobilePlaylist.txt
{% endhighlight %}

### My songs collection:-

Currently i have ~400GB songs in Tamil, English, Malayalam, Hebrew(mostly Messanic) and Hindi in my external Hard drive. These songs include .mp3, .mp4, .avi, .vob, .DAT, .wma, .webm, .mkv file formats. I use VLC media player in my system, which plays all the formats without any problem.

### My Tamil songs collection:-

I copied only the Tamil Songs(included all formats). It was about 16GB of .mp3 and .wma files and 115GB of different video formats. So have 130 GB of songs. I wanted to fill my mom's phone with all these songs. My primary target was mp3 songs because she can listen them even if she does some other work. 16 GB was fine i could transfer them to mobile. I felt there could be a lot of duplicate files. (May be same song, same song sung by different artist). I wanted to remove them as well.

### Naming conventions and identifying duplicate files:-

When i was going through the songs list i noticed the following. The songs were titled in different formats ( i have collected them from different people, every one has their own naming conventions)

1. all small characters,
2. ALL CAPITAL CHARACTERS,
3. all_small_separated_by_underscore,
4. ALL_CAPITAL_SEPARATED_BY_UNDERSCORE,
5. 01. Song title starts with song number.
6. artist name or album name as prefix
7. artist name or album name as suffix
8. AllCapitalizedWords
9. words-separated-by-hyphen
10. soMe-title with ouT any pa`ttern.
11. some end with a leading space .

I wanted to have the titles as simple as possible and uniform, i preferred to have all the titles in small character separated by space. Some of the folders have the songs title as track1.mp3, track2.mp3, etc. Mostly the audio cd's which i converted to mp3 using [audiograber][6]. I used audiograber while i was using windows. Now i keep audio files as it is(.wma).

I copied all the files except the track files in one folder. Since i was going to use for loop i took a backup of all mp3, i didn't want to be in trouble by wrong execution of any command.

##### Solving problems 1, 2, 4, 8, 10:-
First i tried to convert all file names to lower case.

{% highlight sh %}:-
# convert all to lower case
for f in *; do mv "$f" "`echo $f | tr "[:upper:]" "[:lower:]"`"; done
{% endhighlight %}

##### Solving problem 5:-
Trying to remove the numbers from the song title.
{% highlight sh %}
# remove all the digits from the name
for f in *; do mv "$f" "${f//[[:digit:]]/}"; done
for f in *; do mv "$f" "${f/[0-9][0-9]/}"; done
{% endhighlight %}

Either of these worked but there was a problem. It removed the trailing 3 from .mp3, so now all the songs end with .mp Oops now i needed to append 3 with each file.
{% highlight sh %}
# append 3 with the name
for f in *; do mv "$f" "$f"3; done
{% endhighlight %}

##### Solving problems 5, 6:-
Trying to remove the leading space, this one occurred when i removed the leading number from the title.
{% highlight sh %}
# remove leading space
for f in *; do mv "$f" "${f/ /}"; done
{% endhighlight %}

##### Solving problems 3, 4:-

Trying to replace all the underscore with space.
{% highlight sh %}
# replace _ with space
for f in *; do mv "$f" "${f//_/ }"; done
{% endhighlight %}

##### Solving problems 5, 6:-

Trying to remove leading n(10) characters, had to do this when the title was stated with Album name or the artist name
{% highlight sh %}
# Removes leading n(10) char
for f in *; do mv "$f" "${f:10}"; done
{% endhighlight %}

##### Solving problem 7:-
Trying to remove trailing n(16) characters, Songs which i downloaded from youtube has its ids appended. (11 char + one hyphen + .mp4)
{% highlight sh %}
# Removes trailing n(16) char
for f in *; do mv "$f" "${f:0:${#f}-16}" ; done

# append .mp4 after removing n(16) char
for f in *; do mv "$f" "${f:0:${#f}-16}.mp4" ; done
{% endhighlight %}

##### Solving random problems:-

{% highlight sh %}
#Some songs ends with !!s.mp3, remove the !!s.mp3 from file name
for f in *; do mv "$f" "${f/\!\!s\.mp3/}"; done

# removes the .!!s from mp3
for f in *; do mv "$f" "${f/.\!\!s/}"; done

# removes the ..!!s from mp3
for f in *; do mv "$f" "${f/..\!\!s/}"; done

After running the previous script some songs had two dots like xx..mp3 tried to replace .. with .

# replace .. with .
for f in *;do mv "$f" "${f/./}"; done

One folder had songs title with "tamil keerthanaigal collections" tried to remove that as well

for f in *; do mv "$f" "${f/ \(tamil keerthanaigal collections\)\-/}"; done
{% endhighlight %}

Thats all, renaming part was done for mp3. Applied the same for videos. The audio collections were named as as track1, track2, etc. Old video songs were named ad AVSEQ01.DAT, AVSEQ02.DAT, etc. I had to play the songs and change the titles manually.

At this point of time i was able to identify duplicate files easily just by sorting. And i removed most of the duplicate files.

### Converting file formats to save memory:-

Now i have huge collection of songs with proper names. Only problem was memory. I wanted to convert the file formats. There were some video files with a static image(s). Basically they were .mp4 or .mkv files with static images. I wanted to convert them as .mp3 to save some memory. I googled sometime, found [ffmpeg][7] and [libmp3lame][8] will do the magic. Collected all such files and .wma files in one folder and ran the following script.

{% highlight sh %}
# convert any format to mp3.
# for f in *;do ffmpeg -i "$f" -acodec libmp3lame -ab 128k "${f}.mp3"; done
{% endhighlight %}

It converted all the files in the current directory to .mp3 and this is the end of mp3.

### Problems while dealing with video files:-

1. I have not installed any extra app to play video in my mom's Moto G3, i tried to play a .mkv file and the device complained it could not play the song. (assuming it wont play .mov, .vob, .DAT, .webm files)
2. I tried to play some hd .mp4 songs, though the video was playing could not hear any audio. I did not happen for                all files.
3. I always download the best available format (by default youtube-dl downloads best available). Most of the songs come around 100Mb. The mobile device screen size is very small comparing with desktop. I want to convert all the songs to mobile screen size(400x200 - best fit for 5" screen), no mater what actual size(480 or 720 or 1080) is.

To fix all the above problems i just had to run one command.
{% highlight sh %}
#for mobile specific mp4 with less screen size - it converts any video format to .mp4 with 400x200 screen size.
for f in *;do ffmpeg -y -i "$f" -s 400x240 -vcodec libx264 -acodec libvo_aacenc -b:a 92k "${f}.mp4"; done
{% endhighlight %}

I know it is going to take a lot of time to complete this. I ran this script one night and slept. Next morning everything was ready. I had to remove the trailing seven characters and append mp4 with each file. (script explained above).

### Mission complete:-

After converting the video songs to 400x200, ~100Mb became ~20Mb.

#### Final output:

Total .mp3 songs - 1543

Total size of .mp3 songs - 8.45GB

Total .mp4 songs - 980

Total size of .mp4 songs - 20.19GB

Then i transfered the songs to Mobile.
{% highlight sh %}
  # Transferring songs to Mobile.
  adb push /source/ /sdcard/destination/
{% endhighlight %}

A 32 Gb memory card was more than enough to contain all the tamil christian songs i have been collecting for ten+ year!.

### file_name_converter.sh file:-

Every time i ran commands in terminal, i added them in [file_name_converter.sh][9] file for later reference. At this point the file looked like

{% highlight sh %}
#!/bin/bash

#for n in *.mp3
#do if [[ "$n" =~ -[-_0-9a-zA-Z]{11}.mp3$ ]]
#   then echo "youtube ${n: -15: 11}" >> downloaded.txt
#   fi
#done

#for n in *.mp4; do if [[ "$n" =~ -[-_0-9a-zA-Z]{11}.mp4$ ]]; then echo "youtube ${n: #-15: 11}" >> downloaded.txt;fi;done
#youtube-dl --download-archive downloaded.txt -cita myPlaylist.txt

#youtube-dl --download-archive downloaded.txt --no-post-overwrites -ciwx --audio-#format mp3 -o "%(title)s.%(ext)s" [path here]

# for f in *
 	# do mv "$f" "`echo $f | tr "[:upper:]" "[:lower:]"`";
 	# do mv "$f" "${f//[[:digit:]]/}"
 	# do mv "$f" "$f"3
 	# do mv "$f" "${f:22}"
# done

# convert all to lower case
# for f in *; do mv "$f" "`echo $f | tr "[:upper:]" "[:lower:]"`"; done

# remove all the digits from the name
# for f in *; do mv "$f" "${f//[[:digit:]]/}"; done
# for f in *; do mv "$f" "${f/[0-9][0-9]/}"; done

# append 3 with the name
# for f in *; do mv "$f" "$f"3; done
# for f in *; do mv "$f" "$f".mp4; done

# replace _ with space
# for f in *; do mv "$f" "${f//_/ }"; done

# remove leading space
# for f in *; do mv "$f" "${f/ /}"; done

# Removes leading n char
# for f in *; do mv "$f" "${f:28}"; done

# Removes traling n char
# for f in *; do mv "$f" "${f:0:${#f}-15}" ; done

# for f in *; do mv "$f" "${f:0:${#f}-27}.mp4" ; done

# remove the !!s.mp3 from file name
# for f in *; do mv "$f" "${f/\!\!s\.mp3/}"; done

# removes the .!!s from mp3
# for f in *; do mv "$f" "${f/.\!\!s/}"; done
# for f in *; do mv "$f" "${f/..\!\!s/}"; done

# for f in *; do mv "$f" "${f/ \(tamil keerthanaigal collections\)\-/}"; done

# replace .. with .
# for f in *;do mv "$f" "${f/./}"; done

# convert mp4 to mp3
# for f in *;do ffmpeg -i "$f" -acodec libmp3lame -ab 128k "${f/.mp4/}.mp3"; done

#mkv to mp4

# ffmpeg -y -i ip.mkv -vcodec libx264 -acodec libvo_aacenc op.mp4

# ffmpeg -y -i ip.mkv -vcodec libx264 -acodec libvo_aacenc -b:a 92k op.mp4

# ffmpeg -y -i ip.mkv -vcodec libx264 -acodec libvo_aacenc -b:a 92k -r 30 op.mp4

# for mobile specific mkv to mp4 with less screen size
# ffmpeg -y -i ip.mkv -s 400x240 -vcodec libx264 -acodec libvo_aacenc -b:a 92k op.mp4

#for f in *;do ffmpeg -y -i "$f" -s 400x240 -vcodec libx264 -acodec libvo_aacenc -b:a 92k "${f/.mkv/}.mp4"; done

{% endhighlight %}

### At home:-

I reached my home, showed Moto G3 to my Dad and Mom. They told , "yeah it is a nice phone, it has a big display, it looks good". Then i told my mom, "its for you!". She was totally surprised and refused. I expected this and i prepared a solution for this. I opened [Marshmallow's file explorer][10] and played one of her favorite tamil hymn. The song was over, no response from her, played another hymn, again no response, played another one, at the end of this song she told, this is a very old song and its very nice (She sings this song in family prayers). Yea, now i have a firm feeling that she will accept it. I opened Music player and played the auto playlist. I enabled shuffle and random options. I kept the phone in kitchen and went to meet a friend. I came after two hours the songs were playing (no wonder, she did not know how to stop it yet). She told me, "most of the songs are awesome". Then i told her, i can teach you how to play songs in this phone. Once you start a song you don't have to do anything, you can continue your work, songs will play automatically. You can stop when you want to stop and it is very simple to do. She agreed.

### Happiness is helping my parents to use android phone :-

Immediately i taught her to unlock the phone, navigate to apps screen, open music player app and play the songs. After two three attempts playing a mp3 song was a piece of cake for her. Next day i went to my Grandma home, when i came back my mom told me, "I was playing the songs, one song was really really good, so i paused the song at the end, i want to hear the song again". Hurray! Success!, I wanted to teach her about file explorer or Music player's view songs options, but i felt it might confuse her. So i told her, "mom i will teach them once you familiar with android". Then i taught her  to play video songs, making a call from contacts, receiving a call. She kept trying all four apps and  became very comfortable. After the holidays i came back to work. Everyday when we talk i ask her about the phone and songs. She says she listen the songs, mostly the video songs. I am so happy that she started using android phone.

### Thank you guys:-
Thanks to bash, youtube-dl and ffmpeg. Without these tools it would have taken a lot of time to complete. After spending some good time with bash i feel bash is slim, easy to use, each command just does what it suppose to do, provides loads of utility and flags, provides native access to system level informations and configurations. Hey BASH you saved me a lot of time, and i love you, seriously __I LOVE YOU!.__

### Special Thanks:-
At this very moment i thank everyone(some names mentioned bellow) who shared their song collections with me. Some of them shared songs even before i heard about youtube!.

 Anila Mary<br>
 Anish John<br>
 Baby<br>
 Baby Sharmila<br>
 Bella<br>
 Benjamin<br>
 Edberg John<br>
 Hubert Jeno<br>
 Jasmine Sarah<br>
 Minu Prince<br>
 Sudhar<br>
 Sylvia Jennifer<br>
 Vinoth Raj<br>

 [0]: https://www.downloadhelper.net/
 [1]: https://github.com/JackieXie168/movgrab
 [2]: https://github.com/rg3/youtube-dl/
 [3]: https://www.python.org/
 [4]: https://pip.pypa.io/en/stable/
 [5]: https://en.wikipedia.org/wiki/Messianic_Judaism#Music
 [6]: http://www.audiograbber.org/
 [7]: https://www.ffmpeg.org/
 [8]: https://github.com/gypified/libmp3lame
 [9]: https://raw.githubusercontent.com/Franklin2412/bash-collections/master/file_name_conveter.sh
 [10]: http://www.howtogeek.com/231401/how-to-use-android-6.0%E2%80%99s-built-in-file-manager/
 [11]: https://termux.com
 [12]: http://qpython.com/
