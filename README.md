# caudec: multiprocess audio converter

Copyright © 2012 - 2025 Guillaume Cocatre-Zilgien

https://github.com/gcocatre/caudec

**_caudec_** is a command-line utility that transcodes (converts) audio files from one format (codec) to another, among other things.

It leverages multi-core CPUs and runs multiple processes concurrently (one per file and per codec, and more than one thread per codec when it supports it). The objective is to hog the CPU as much and as long as possible. One strategy is to sort input files by size, so that the largest files potentially get more threads towards the end of the job.

## Features

* **Supported input formats** / codecs: WAV, AIFF, CAF, FLAC, WavPack, Monkey's Audio, ALAC.
* **Supported output formats** / codecs: all of the above, as well as LossyWAV / LossyFLAC, MP3, AAC (.m4a), Ogg Vorbis, Opus.
* **Supported platforms**: macOS, Linux.
* Transcoding to several different codecs at once is possible. In that case, decoding of input files is done only once.
* Metadata is preserved (as much as possible) from one codec to another.
* Artwork can be embedded into each file, and / or copied to the output directory. It can be done selectively (e.g. embed and / or copy one image for lossless files, and another image for lossy files).
* Audio can be resampled (e.g. 48kHz to 44.1kHz) and downmixed (e.g. 6 channels to stereo). A profile can be provided to set a maximum value for the number of channels, bit depth and sampling rate. When a profile is provided, the source will only be altered after decoding and before encoding, if some metric of the source is above the given profile.
* **Multiprocess ReplayGain scanner** for FLAC, WavPack, MP3, Ogg Vorbis, Opus.
* Ability to hard link lossy files to a different directory when encoding to WavPack Hybrid. The point is to have two libraries that takes the storage of just one, with a lossy collection that has its own root directory and that's easy to drag and drop to a device such as a smartphone or a Digital Audio Player (DAP).
* Ability to touch files and album directories using metadata to reflect the music's release date and duration (see example below).

## Installation

To install caudec when no package is available for your distribution, run:

`sudo ./install.sh [ --prefix=/some/custom/path ]`

That script will install caudec in `/usr/local` by default, or in the directory
specified with --prefix. A default configuration file will be installed as
`/etc/caudecrc`, or `~/.caudecrc`. An additional `decaude` link will be installed
alongside caudec, which is equivalent to `caudec -d` (for decoding).

## Usage

### Examples

caudec is meant to be used on one album at a time. Depending on how you've decided to organize your music collection, these examples can give you an idea of what to do. In all of these, the source codec doesn't matter, as long as it's a lossless codec, or lossyFLAC. Assuming your directory structure is `[Artist]/[Album]/[Track]*`:

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

---

#### Ideal example of using FLAC's internal multithreading on an ALAC album that's a single track:

```
$ caudec -c flac -P ~/Music/FLAC/ "Daft Punk/Alive 1997 [2001]"/*.m4a
 0 10x OK 01. Alive 1997.flac
 * 4.29 seconds (read: 85.5 MB/s, write: 83.3 MB/s, rate: 637.8x)

WAV:                459.9 MiB    100.0%
Source:             349.3 MiB     76.0%    1071 kbps
FLAC:               340.7 MiB     74.1%    1045 kbps
```

---

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
---

#### Transcoding an album to Ogg Vorbis, using `transcaude`:

```
$ cd ~/Music/FLAC
transcaude -c vorbis -q 6 -P ~/Music/OggVorbis -f folder.jpg "Artist/Album"/*.flac
```

`transcaude` is a small utility that uses `caudec` to transcode an album (or several albums, or an entire collection), then compute ReplayGain and touch files (when the output format is compatible). It does all 3 caudec commands in the previous example, in one fell swoop. In order to prevent `transcaude` from touching files, use `-M` or set `transcaudeTouchFiles='false'` in `~/.caudecrc`. You may also tell it to not compute ReplayGain values from lossless to lossless with the `-R` parameter, or set `computeLosslessReplayGain='false'` in `~/.caudecrc` (useful when you know that the source already has ReplayGain data).

Since it invokes `caudec`, multiple codecs may be specified at once:

---

#### Transcoding an album to both FLAC and Ogg Vorbis, using `transcaude`:

```
$ cd ~/Music/WavPack
transcaude -c flac -q 8 -P ~/Music/FLAC -c vorbis -q 6 -P ~/Music/OggVorbis -f folder.jpg "Artist/Album"/*.wv
```

---

#### Specifying source directories instead of source files, and use default compression settings:

Set `ignoreUnsupportedFiles=true` in `~/.caudecrc`, and set the appropriate `compression_*` / `bitrate_` / `average_bitrate_*` values to your liking:

```
ignoreUnsupportedFiles=true
compression_FLAC=8
compression_OggVorbis=5
```

```
$ caudec -c flac -P ~/Music/FLAC/ -c vorbis -P ~/Music/OggVorbis "Miles Davis/Bitches Brew [1970]"
```

```
$ transcaude -c flac -P ~/Music/FLAC/ -c vorbis -P ~/Music/OggVorbis "Miles Davis/Bitches Brew [1970]"
```

---

#### Specifying default target directories for lossless and lossy:

```
defaultLosslessDestination="${HOME}/Music/FLAC"
defaultLossyDestination="${HOME}/Music/OggVorbis"
```

```
$ caudec -c flac -c vorbis "Miles Davis/Bitches Brew [1970]"
```

```
$ transcaude -c flac -c vorbis "Miles Davis/Bitches Brew [1970]"
```

---

#### Hard linking WavPack lossy files when transcoding to WavPack Hybrid:

```
$ caudec -c wvh -P ~/Music/lossless -L ~/Music/lossy "Miles Davis/Bitches Brew [1970]"
```

```
$ transcaude -c wvh -P ~/Music/lossless -L ~/Music/lossy "Miles Davis/Bitches Brew [1970]"
```

If you'd like to hard link lossy *.wv files automatically via `~/.caudecrc`:

```
defaultLosslessDestination="${HOME}/Music/lossless"
defaultLinkedDestination="${HOME}/Music/lossy"
```

```
$ caudec -c wvh "Miles Davis/Bitches Brew [1970]"
```

```
$ transcaude -c wvh "Miles Davis/Bitches Brew [1970]"
```

There, `~/Music/lossless` would contain both *.wv and *.wvc files, while `~/Music/lossy` would only contain hard linked *.wv files, which could be conveniently transfered to an external device (e.g. a portable player), and which wouldn't take additional storage space.

---

#### Transcoding multiple albums using `transcaude`:

```
$ transcaude -c flac -P ~/Music/FLAC -c vorbis -P ~/Music/OggVorbis "Pink Floyd" "Daft Punk"
```

Here we specify a list of (album) directories instead of a list of files. Use `transcaude -s` in order to be silent and only display errors.

---

#### Using a profile to downmix or upmix, downscale and / or downsample before encoding, only if needed:

```
$ caudec -r 2/24/48 -c flac "Pink Floyd/The Dark Side of the Moon [50th anniversary remaster] (1973)"
```

If the source has more than 2 channels, caudec will automatically downmix to stereo. If the source is 24 bit, caudec will keep the bit depth unaltered. Finally, if the source is 92kHz, caudec will resample to 48 kHz.

```
$ caudec -r 2/24/48 -c flac "Daft Punk/Random Access Memories [2013]"
```

Here, if the source is a CD (stereo, 16 bit, 44.1kHz), caudec will leave the source untouched. The provided profile only sets a maximum and will not touch the source unless needed, and it will never upmix, upscale, or upsample (although that can be done with -2, -b and -r separately). The profile can be provided with `-r` but also set in `~/.caudecrc`:

```
resamplingProfile='2/24/48'
```


## Requirements

caudec uses common UNIX tools (bc, cut, date/gdate, find, grep, head, mktemp, ps, sed/gsed, sort, stat, tail, tr, uname, wc, xargs), as well as the following software (most of them available via **Homebrew** on macOS):

### Mandatory Software

* SoX: http://sox.sourceforge.net/
* Monkey's Audio, ALAC (as source files): https://ffmpeg.org/

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

* FLAC, WavPack, Monkey's Audio, Ogg Vorbis (rsgain): https://github.com/complexlogic/rsgain
* MP3 (mp3gain): https://mp3gain.sourceforge.net/
* Opus (opusgain): https://github.com/FrancisRussell/zoog

#### Miscellaneous
* schag (for files with APEv2 metadata): https://github.com/gcocatre/schag
* eyeD3 (for MP3 tagging): https://eyed3.readthedocs.io/en/latest/
