# caudec: multiprocess audio converter

Copyright © 2012 - 2024 Guillaume Cocatre-Zilgien

https://github.com/gcocatre/caudec

caudec is a command-line utility that transcodes (converts) audio files from one format (codec) to another. It leverages multi-core CPUs and runs multiple processes concurrently (one per file and per codec).

## Features

* Supported input formats / codecs: WAV, FLAC.
* Supported output formats / codecs: WAV, FLAC, lossyWAV, MP3 (LAME), Ogg Vorbis.
* Transcoding to several different codecs at once is possible. In that case, decoding of input files is done only once.
* Metadata is preserved (as much as possible) from one codec to another.
* Multiprocess Replaygain scanner for FLAC, MP3, Ogg Vorbis.

## Requirements

caudec uses common UNIX tools (which, uname, grep, stat, sed/gsed, date/gdate, tr, cut, wc, df, mktemp, xargs, nohup), as well as the following software:

### Mandatory Software

* bc: http://www.gnu.org/software/bc/
~~* SoX: http://sox.sourceforge.net/~~

### Optional Software

#### Native Codecs

* FLAC: http://flac.sourceforge.net/
* LAME: http://lame.sourceforge.net/
* Ogg Vorbis: http://www.vorbis.com/
* lossyWAV: https://wiki.hydrogenaud.io/index.php?title=LossyWAV

#### Replaygain

~~* wavegain: http://freecode.com/projects/wavegain~~
* vorbisgain: http://sjeng.org/vorbisgain.html
* mp3gain: http://mp3gain.sourceforge.net/

#### Miscellaneous
* wget (for checking for new versions): http://www.gnu.org/software/wget/wget.html
~~* APEv2 (for files with APEv2 metadata): http://caudec.net/~~
~~* cksfv (for hashing CRC32): http://zakalwe.fi/~shd/foss/cksfv/~~
~~* md5sum (Linux) or md5 (OS Ⅹ) for hashing MD5 sums~~
~~* sha1sum, sha256sum, sha512sum (Linux) or shasum (OS Ⅹ) for hashing SHA sums~~
* eyeD3 (for MP3 tagging): http://eyed3.nicfit.net/


## Installation

To install caudec when no package is available for your distribution, run:
$ sudo ./install.sh [ --prefix=/some/custom/path ]
That script will install caudec in /usr/local by default, or in the directory
specified with --prefix. A default configuration file will be installed as
/etc/caudecrc, or ~/.caudecrc. An additional 'decaude' link will be installed
alongside caudec, which is equivalent to 'caudec -d' (for decoding).
