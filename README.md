# CW Loops

Code and setup to make CW loops (MP3)

This page has the main concept and documentation.

References:

* [Soapbox](SOAPBOX.md) - for other comments on practicing CW.
* [Psychoacoustioc Masking](PSYCHOACOUSZTIC.md) - for detail explanation on MP3.

## CW Loops Setup

These steps assume you are using an Ubuntu Linux system (or close to it). If you're
using Redhat (CentOS and flavors based on RH) the package installation commands
are slightly different, but close.

### Requirements

*  Python
*  `ffmpeg`
*  A shell


First thing is to get the libraries you need for the python script to work.

**From here on, any use of the `$` is meant to indicate the shell prompt.**

The script and library provided by this repository are `play.py` and `morse.py` -- those are provided here by this repository.

#### The Python Libraries

Only on Ubuntu (Debian based) systems do this:


```
$ sudo apt install libportaudio2 \
portaudio19-dev \
python3-dev \

$ sudo pip3 install numpy scipy sounddevice
```

Only on Redhat based systems, do this instead.

```
$ sudo dnf install portaudio portaudio-devel python3-devel
pip3 install numpy scipy sounddevice
```

#### The `ffmpeg` Tool

Only on Ubuntu (Debian based), do this:

```
$ sudo apt update
$ sudo apt install ffmpeg
```

Only on Redhat (CentOS, etc..) based systems do this instead:

```
$ sudo dnf install epel-release
$ sudo dnf install https://download1.rpmfusion.org/free/el/rpmfusion-free-release-$(rpm -E %rhel).noarch.rpm
$ sudo dnf install ffmpeg ffmpeg-devel
```

From here on out the use of Python and `ffmpeg` **does not depend** on what system you're using.  Only the installation of these packages are affected by the system flavor.

Thus begins the platform independent steps --

## Install Software

The software in this repository `code/` is the Python:

1. `play.py`  -- this is the application invoked to make tracks.
2. `morse.py` -- this is a library used by the application.
3. `morseTable.py` -- just a convenience for defining the elements of each character.


Make a directory to work in and copy those Python files into that directory.
And change to that directory:

```
$ mkdir exp
$ cp code/*.py exp
$ cd exp
```

Now we can experiment.

## Concept of Operations

Two steps:

1.  Encode the WAV
2.  Transcode the WAV into MP3

The script `play.py` synthesizes a WAV file (not MP3).
The text to encode as CW into WAV is provided to the script as *stdin*

Example of step 1:


```
$ echo "CQ DE W7BRS" | python3 play.py -f 672 --fs 25 --wpm 20 -o sample.wav
```

*  The `-f` flag tells `play.py` the audio frequency to encode.  All symbols will be encoded at that frequency in `sample.wav`.
*  The `--fs` is the <a target="top" href="https://www.arrl.org/files/file/Technology/x9004008.pdf">Farnsworth</a> speed (not words per minute) -- this also goes
by the name *character speed*.  In other words the speed of each character can be
timed to a different rate than the rate of all of the symbols (words per minute).  You can set the Farnsworth speed (character speed) to be the same words per minute if you like.  There is an advantage to learn CW with a faster character speed than wpm speed.
*  The `--wpm` flag with argument is the setting for words per minute (wpm). 
This is the rate that most rigs are setting when you set the CW speed.  Advanced
settings in radios can offer configuration to change the **character speed** but
seldom do. Don't worry about it.
* The `-o` flag with argument is the name of the output WAV file.

When literal spaces are found in the input string, then `play.py` encodes the
pause timed as an average character in silence.  There is no preamble of silence
encoded.


Example of step 2:

Now that you have the WAV file, you can use it as is, or encode it into MP3 for lossy
compression.  It'll sound fine, just takes less data.

We're going to encode the MP3 at 192kbit/sec which is at the bottom limit of
where slight imperfections can be heard by an audiophile.  Any digitized signal
is going to have imperfections *compared to the analog source*.  WAV files are not
analog either.  For MPEG-1 layer III (what is commonly called just MP3),
above 192kbit it becomes difficult for the average person to truly hear 
the difference.  Stick with 192kbit.  [Additonal info](PSYCHOACOUSTZIC.md)

The sampling rate is 44.1kHz.

The tool to use is called `ffmpeg` (and you can install this the usual way
on your linux system)

Syntax:

```
$ ffmpeg -i INPUT_FILE  -vn -ar 44100 -ac 2 -b:a 192k OUTPUT_FILE
```

Real example:

```
$ ffmpeg -i sample.wav -vn -ar 44100 -ac 2 -b:a 192k final.mp3
```

The key flags offered:

* `-i`  Specifies the input file that ffmpeg will read from.
* `-vn`  Disables processing of any video stream so only audio is handled.
* `-ar 44100`  Sets the audio sample rate to 44,100 Hz (CD-quality).
* `-ac 2`  Forces the audio to have 2 channels (stereo).
* `-b:a 192k`  Sets the audio bit-rate to 192 kbps for encoding quality/size tradeoff.
* `final.mp3`  Defines the output file path and format (MP3 inferred from extension).

The result is `final.mp3` which is a lossy rendition of the same WAV.


## Additional Concepts

In the case where you want to encode multiple stations having a QSO then
there is a method for encoding the audio so the individual stations sound
a bit different for realism.

Let's suppose Alice and Bob are in a QSO.  Alice is W1ABC and Bob is W7DEF

The first thing to decide is **pitch frequency** for Alice and Bob.  Let's make
Bob a bit lower than Alice's pitch frequency.

The QSO will go like this:

```
CQ CQ DE W1ABC W1ABC W1ABC K
W7DEF
W7DEF W1ABC TU CALL BT OP IS ALICE QTH NR TUCSON HW CPY ? W7DEF DE W1ABC KN
W1ABC W7DEF FB ALICE TNX BT NAME BOB ES QTH NR SEATTLE SEATTLE WA BT UR 5NN HR BT ANT LOOP UP 30 M ES RIG K3 ES KPA BT HW CPY ? W1ABC DE W7DEF K
W7DEF W1ABC OK BOB GD SIG NICE TO WRK U UR ALSO 5NN HR BT TNX QSO 73 W7DEF DE W1ABC SK
W1ABC W7DEF OK ALICE TNX 73 DE W7DEF SK
```

We want to encode all six transmissions such that Alice's pitch is a bit higher
than Bob and we want Alice's character speed and wpm speed to be slightly faster
than Bob.

| Op | pitch (Hz) | character speed | wpm |
|----|------------|-----------------|-----|
| Alice | 682 | 27 | 25 |
| Bob   | 668 | 24 | 22 |


First step is to encode all the parts.  From the command line, let's make
six (6) MP3's

```
$ echo "CQ CQ DE W1ABC W1ABC W1ABC K" | python3 play.py -f 682 --fs 27 --wpm 25 -o alice-cq.wav
$ echo "W7DEF" | python3 play.py -f 668 --fs 24 --wpm 22 -o bob-answer.wav
$ echo "W7DEF W1ABC TU CALL BT OP IS ALICE QTH NR TUCSON HW CPY ? W7DEF DE W1ABC KN" | python3 play.py -f 682 --fs 27 --wpm 25 -o alice-resp.wav
$ echo "W1ABC W7DEF FB ALICE TNX BT NAME BOB ES QTH NR SEATTLE SEATTLE WA BT UR 5NN HR BT ANT LOOP UP 30 M ES RIG K3 ES KPA BT HW CPY ? W1ABC DE W7DEF K" | python3 play.py -f 668 --fs 24 --wpm 22 -o bob-sig.wav
$ echo "W7DEF W1ABC OK BOB GD SIG NICE TO WRK U UR ALSO 5NN HR BT TNX QSO 73 W7DEF DE W1ABC SK" | python3 play.py -f 682 --fs 27 --wpm 25 -o alice-final.wav
$ echo "W1ABC W7DEF OK ALICE TNX 73 DE W7DEF SK" | python3 play.py -f 668 --fs 24 --wpm 22 -o bob-final.wav
```


Six files are made by these six invocations of `play.py`

Remember that **encoding** is where all the properties of the content are
established.  The content of the  CW is established, the pitch, spacing, speed
all are encoded into WAV.

*Transcoding* is NOT going to alter any of the **content** -- it merely **converts**
the same media **content** from one *codec* into another *codec.* 
We are transcoding from WAV to MP3.

There are six files:
1. `alice-cq.wav`
2. `bob-answer.wav`
3. `alice-resp.wav`
4. `bob-sig.wav`
5. `alice-final.wav`
6. `bob-final.wav`

Now we want to make concatenate these parts into a single MP3.

### Steps


1. First make each part an MP3 file with `ffmpeg`
2. Concat them with `ffmpeg`

#### Make MP3

The steps to make each MP3 below.  I'll show just one, but you will repeat the
process for each file.  Remember to give each OUTPUT MP3 a unique name.

The first one:

```
$ ffmpeg -i alice-cq.wav -vn -ar 44100 -ac 2 -b:a 192k alice-cq.mp3
```

The next use the same pattern:

```
$ ffmpeg -i bob-answer.wav -vn -ar 44100 -ac 2 -b:a 192k bob-answer.mp3
```

There is no requirement to use the same *basename* of the file. That was just
a convenience.

After all six are encoded, then you will have these files:

1. `alice-cq.mp3`
2. `bob-answer.mp3`
3. `alice-resp.mp3`
4. `bob-sig.mp3`
5. `alice-final.mp3`
6. `bob-final.mp3`

#### Concatenate

The `ffmpeg` software has a feature to concatenate media content.  The 
information that `ffmpeg` needs is the file list.  The list of all MP3 files to
concatenate.   The order of the files is important.   Example, we don't 
want the `CQ` from Alice to be after Bob's first answer.

So make a text file `alice-bob.txt` that specifies the file list and the *order* of the files to concatenate.

The text file you make should look like this:

```
file 'alice-cq.mp3'
file 'bob-answer.mp3'
file 'alice-resp.mp3'
file 'bob-sig.mp3'
file 'alice-final.mp3'
file 'bob-final.mp3'
```

The literal prefix word `file` comes before the filename. 
And the filenames are **single-quoted.**

Now that we have the Manifest of files to concatenate, we just ask `ffmpeg` to
do so:

```
$ ffmpeg -f concat -i alice-bob.txt W1ABC-W7DEF.mp3
```


The result is a new MP3 file called `W1ABC-W7DEF.mp3` which is the entire
QSO bolted together from the six parts previously encoded (and transcoded).

Six is just an arbitrary number of exchanges in the QSO. You can make your
QSO as long or short as you want, involve more than two operators, etc..

Remember that for realism, the pitch frequency and character speed and wpm speed
should be slightly different for each participant so you can tell them apart
in the full playback.

