# caudec: multiprocess audio converter

Copyright © 2012 - 2025 Guillaume Cocatre-Zilgien

https://github.com/gcocatre/caudec

**_caudec_** is a command-line utility that transcodes (converts) audio files from one format (codec) to another, among other things.

It leverages multi-core CPUs and runs multiple processes concurrently (one per file and per codec, and more than one thread per codec when it supports it). The objective is to hog the CPU as much and as long as possible. One strategy is to sort input files by size, so that the largest files potentially get more threads towards the end of the job.

## Features

* **Supported input formats** / codecs: WAV, AIFF, CAF, FLAC, WavPack, Monkey's Audio, ALAC.
* **Supported output formats** / codecs: all of the above, as well as LossyWAV / LossyFLAC / LossyWV, MP3, AAC (.m4a), Ogg Vorbis, Opus.
* **Supported platforms**: macOS, Linux.
* Transcoding to several different codecs at once is possible. In that case, decoding of input files is done only once.
* Metadata is preserved (as much as possible) from one codec to another.
* Artwork can be embedded into each file, and or copied to the output directory. It can be done selectively (e.g. embed and / or copy one image for lossless files, and another image for lossy files).
* Hashes can be computed and stored as metadata, for convenience when comparing different versions of the same song.
* Audio can be resampled (e.g. 48kHz to 44.1kHz) and downmixed (e.g. 6 channels to stereo).
* **Multiprocess ReplayGain scanner** for FLAC, WavPack, MP3, Ogg Vorbis, Opus.
* Ability to hard link lossy files to a different directory when encoding to WavPack Hybrid. The point is to have two libraries that takes the storage of just one, with a lossy collection that has its own root directory and that's easy to drag and drop to a device such as a smartphone or a Digital Audio Player (DAP).
* Ability to touch files and album directories using metadata to reflect the music's release date and duration (see example below).

## Installation

To install caudec when no package is available for your distribution, run:

`sudo ./install.sh [ --prefix=/some/custom/path ]`

That script will install caudec in /usr/local by default, or in the directory
specified with --prefix. A default configuration file will be installed as
/etc/caudecrc, or ~/.caudecrc. An additional 'decaude' link will be installed
alongside caudec, which is equivalent to 'caudec -d' (for decoding).

## Usage

### Examples

caudec is meant to be used on one album at a time. Depending on how you've decided to organize your music collection, these examples can give you an idea of what to do. Assuming your directory structure is `[Artist]/[Album]/[Track]*`:

#### Transcoding an ALAC album to FLAC while creating a mirrored directory structure in `~/Music/FLAC/`:

```
$ caudec -c flac -P ~/Music/FLAC/ "Miles Davis/Bitches Brew [1970]"/*.m4a
 3  1x OK 202. John McLaughlin.flac
 1  2x OK 203. Miles Runs the Voodoo Down.flac
 5  2x OK 201. Spanish Key.flac
 4  1x OK 204. Sanctuary.flac
 0  2x OK 101. Pharaoh's Dance.flac
 2  3x OK 102. Bitches Brew.flac
 * 5.03 seconds (read: 87.2 MB/s, write: 78.8 MB/s, rate: 1119.8x)

WAV:                947.7 MiB    100.0%
Source:             418.4 MiB     44.1%     623 kbps
FLAC:               377.9 MiB     39.9%     562 kbps
```

#### Ideal example of using FLAC's internal multithreading on an ALAC album that's a single track:

```
$ caudec -c flac -P ~/Music/FLAC/ "Daft Punk/Alive 1997 [2001]"/*.m4a
 0 10x OK 01. Alive 1997.flac
 * 4.29 seconds (read: 85.5 MB/s, write: 83.3 MB/s, rate: 637.8x)

WAV:                459.9 MiB    100.0%
Source:             349.3 MiB     76.0%    1071 kbps
FLAC:               340.7 MiB     74.1%    1045 kbps
```

#### Result of 'touching' a WavPack album with `caudec -T`:

For my own convenience, I have set an alias to the `ls` command:
```
alias d='ls -lh --color=auto --time-style="+%Y-%m-%d %H:%M:%S"''
```

```
$ d "The Dave Brubeck Quartet/Time Out [1959]"
total 113M
-rw-r--r-- 2 user group  20M 1959-01-01 00:00:00 '01. Blue Rondo a la Turk.wv'
-rw-r--r-- 2 user group  22M 1959-01-01 00:06:44 '02. Strange Meadow Lark.wv'
-rw-r--r-- 2 user group  16M 1959-01-01 00:14:07 '03. Take Five.wv'
-rw-r--r-- 2 user group  16M 1959-01-01 00:19:31 '04. Three to Get Ready.wv'
-rw-r--r-- 2 user group  14M 1959-01-01 00:24:55 "05. Kathy's Waltz.wv"
-rw-r--r-- 2 user group  13M 1959-01-01 00:29:43 "06. Everybody's Jumpin'.wv"
-rw-r--r-- 2 user group  13M 1959-01-01 00:34:06 '07. Pick Up Sticks.wv'
-rw-r--r-- 2 user group 187K 1959-01-01 00:38:22  folder.jpg

$ d -d "The Dave Brubeck Quartet/Time Out [1959]"
drwxr-xr-x 10 user group 320 1959-01-01 00:38:22 'The Dave Brubeck Quartet/Time Out [1959]'
```

#### Batch transcoding an entire FLAC music collection to Ogg Vorbis, using a `for` loop:

```
$ cd ~/Music/FLAC
$ for d in */*; do \
echo "$d"; \
caudec -s -c vorbis -q 6 -P ~/Music/OggVorbis -f folder.jpg "$d"/*.flac && \
caudec -s -g ~/Music/OggVorbis/"$d"/*.ogg && \
caudec -s -T ~/Music/OggVorbis/"$d"/*; \
done
```

With the `-s` parameter used above, caudec will only display errors, making them easier to spot.

* The first caudec command will transcode FLAC files to OggVorbis and copy each album's `folder.jpg` file (if present) to the destination directory.
* If it succeeds (`&&`), it will compute ReplayGain on the transcoded files. Using that command is a personal choice and not mandatory if you don't care about ReplayGain.
* If it succeeds again, it will 'touch' the transcoded files (as shown before). Using that command is optional as well.

#### Transcoding an album to Ogg Vorbis, using `transcaude`:

```
$ cd ~/Music/FLAC
transcaude -c vorbis -q 6 -P ~/Music/OggVorbis -f folder.jpg "Artist/Album"/*.flac
```

`transcaude` is a small utility that uses `caudec` to transcode an album, then compute ReplayGain and touch files (when the output format is compatible). It does all 3 caudec commands in the previous example, in one fell swoop.

## Requirements

caudec uses common UNIX tools (bc, cut, date/gdate, find, grep, head, mktemp, ps, sed/gsed, sort, stat, tail, tr, uname, wc, xargs), as well as the following software (most of them available via **Homebrew** on macOS):

### Mandatory Software

* SoX: http://sox.sourceforge.net/

#### Native Codecs

* FLAC: https://xiph.org/flac/
* WavPack: https://www.wavpack.com/
* Monkey's Audio: https://www.monkeysaudio.com/
* LossyWAV: https://wiki.hydrogenaud.io/index.php?title=LossyWAV
* MP3: https://lame.sourceforge.io/
* AAC: https://ffmpeg.org/
* ALAC: https://ffmpeg.org/
* Ogg Vorbis: https://xiph.org/vorbis/
* Opus: https://www.opus-codec.org/

#### ReplayGain

* MP3 (mp3gain): https://mp3gain.sourceforge.net/
* Ogg Vorbis (vorbisgain): http://sjeng.org/vorbisgain.html
* Opus (opusgain): https://github.com/FrancisRussell/zoog

#### Miscellaneous
* APEv2 (for files with APEv2 metadata): https://github.com/gcocatre/APEv2
* cksfv (for hashing CRC32): https://zakalwe.fi/~shd/foss/cksfv/
* md5sum (Linux) or md5 (macOS) for hashing MD5 sums
* sha1sum, sha256sum, sha512sum (Linux) or shasum (macOS) for hashing SHA sums
* eyeD3 (for MP3 tagging): https://eyed3.readthedocs.io/en/latest/
