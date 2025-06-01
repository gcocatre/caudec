```
caudec 2.1.1: multiprocess audio converter
Copyright © 2012 - 2025 Guillaume Cocatre-Zilgien
https://github.com/gcocatre/caudec

Usage: caudec [ GLOBAL OPTIONS ] [ PROCESSING ] [ ENCODING/DECODING/RG ] FILES
Operate on multiple audio files at once, in parallel.
Instead of multiple files, one or more directories may be specified.
Multiple codec switches (optionally paired with a -q switch) may be specified.
Supported input files: .wav, .aiff, .caf, .flac, .wv, .ape, .m4a

-------------------------------------------------------------------------------

Global options:

  -s        be silent, only print errors
  -n N      launch N processes concurrently (1-10);
            by default, the number of CPU cores.

  -o DIR    set existing output directory
  -O DIR    set output directory, create it if it doesn't exist already
  -P DIR    set and create output directory, and mirror the source file's
            path components (e.g. 'a/b/file.flac' => 'DIR/a/b/file.ogg')

  -l DIR    set existing directory for hard linking lossy .wv files from
            WavPackHybrid's output directory
  -L DIR    set and create a directory for hard linking lossy .wv files from
            WavPackHybrid's output directory

  -a FILE   embed cover art FILE into supported files. Specify -a first
            to embed the same file for all supported codecs specified by -c,
            or specify -a after -c to embed cover art only for that codec,
            if supported. FILE must be a filename and that file must be
            located in the source directory.

  -f ARG    copy certain files to the output directory. ARG may be a 'all',
            a comma separated list of files, and / or file extensions preceded
            by a dot. If ARG is a list of files, they must be filenames and
            those files must be located in the source directory.

  -k        keep existing destination files (don't overwrite them)
  -K        keep existing destination files if they're newer than their source

  -T        touch files using metadata to reflect the music's release date
            and duration. Provide all files as arguments, not just audio files,
            although at least one audio file is required for this to work.
            Use with -l when dealing with a hard linked collection (-c wvh).
            Ex: caudec -T ~/Music/Lossless/Artist/Album/*
            Ex: caudec -T ~/Music/Lossless/Artist/Album/* -l ~/Music/Lossy

  -z        produce machine-parsable output; must be the first parameter on the
            command line to take effect. Run 'caudec -z' on its own to print
            a description of the syntax.

-------------------------------------------------------------------------------

When transcoding to multiple codecs at once (multiple -c parameters),
specify a -o/O/P parameter after each -c parameter in order to set per-codec
output directories. For instance:
$ caudec -c flac -P ~/Music/FLAC -c mp3 -P ~/Music/MP3 "Artist/Album"/*.flac

-------------------------------------------------------------------------------

Similarly, specifying -a/-f after a -c parameter will affect only that codec.
For instance:
$ caudec -c flac -P ~/Music/FLAC -f cover.jpg \
  -c mp3 -P ~/Music/MP3 -f folder.jpg "Artist/Album"/*.flac

-------------------------------------------------------------------------------

Creating a WavPack Hybrid collection with a different root directory for lossy:
$ caudec -c wvh -P ~/Music/Lossless -L ~/Music/Lossy "Artist/Album"/*.flac
That will create a mirrored directory structure with hard linked lossy files.
~/Music/Lossless will contain the full lossless collection, which can be safely
backed up and ~/Music/Lossy will contain only the lossy part of the collection.
Little to no extra storage will be used (just extra inodes).

-------------------------------------------------------------------------------

Processing / resampling options:

  -b BITS   bit depth (16, 24)
  -r HZ     sampling rate in Hz (44100, 48000, 88200, 96000, 176400, 192000,
            352800, 384000)
  -r KHZ    sampling rate in kHz (44[.1], 48, 88[.2], 96, 176[.4], 192, 352[.8],
            384)
  -r cd     equivalent to -b 16 -r 44100 -2 (includes conversion to stereo)
  -r dvd    equivalent to -b 16 -r 48000
  -r sacd   equivalent to -b 24 -r 88200
  -r dvda   equivalent to -b 24 -r 96000
  -r bluray equivalent to -b 24 -r 96000
  -r pono   equivalent to -b 24 -r 192000
  -r dxd    equivalent to -b 24 -r 352800
  -2        convert to stereo: 2.1, 4.0, 5.0, 5.1 and 7.1 audio will be
            downmixed, using proper channel mappings; mono audio will be
            upmixed to dual-mono (stereo with two identical channels)

-------------------------------------------------------------------------------

Encoding options:

  -c CODEC  use specified CODEC: wav, aiff, caf,
            flac, wv (WavPack), wvh (WavPack Hybrid), wvl (WavPack lossy),
            lossyWAV, lossyFLAC, lossyWV, ape (Monkey's Audio), alac (m4a),
            aac (m4a), mp3, ogg / vorbis, opus.

  -C CODEC  use specified CODEC, but discard existing metadata

  -q ARG    set compression level (variable bitrate mode; try -q help for
            a list of valid values)

  -b ARG    constant or target bitrate in bits per sample (for -c wvh/wvl)
            or in kilobits per second (for -c wvh/wvl, opus, mp3, ogg/vorbis)

  -B ARG    average bitrate in kilobits per second (for -c mp3, aac (m4a),
            ogg/vorbis, opus)

  -G ARG    MP3: apply ReplayGain (album or track) if found in source file
            metadata, after decoding and BEFORE encoding (irreversible).
            Note: it is possible to specify a preamp value with an additional
            -G parameter, for instance '-G album -G -3' or '-G track -G +2'.
            Only use positive preamp values if you know what you're doing.

  -G ARG    MP3: apply peak normalization (albumpeak or trackpeak); this makes
            the tracks as loud as possible without clipping; requires
            ReplayGain metadata to be available in the source files.
            Note: it is possible to specify an arbitrary peak reference lower
            than 0dBFS with an additional -G parameter: '-G albumpeak -G -4'.

  -G GAIN   MP3: apply arbitrary GAIN (signed number from -99.99 to +99.99)

  -H HASH   compute hash of raw PCM (CRC32, MD5, SHA1, SHA256 or SHA512,
            FLAC, WavPack, Monkey's Audio and lossyFLAC, lossyWV only)
  -H ^HASH  do NOT compute HASH even if it's in caudecrc

-------------------------------------------------------------------------------

Decoding options:

  -d        decode to WAV (input: AIFF, CAF, FLAC, WavPack, Monkey's Audio)
  -t        test file integrity
  -H HASH   compute hash of raw PCM (CRC32, MD5, SHA1, SHA256 or SHA512,
            FLAC, WavPack, Monkey's Audio and lossyFLAC, lossyWV only)
  -H ^HASH  do NOT compute HASH even if it's in caudecrc

-------------------------------------------------------------------------------

ReplayGain options (mutually exclusive from -c/-d/-t/-T):

  -g        generate ReplayGain metadata
  -G ARG    MP3: compute and apply gain of type ARG (album or track)
            (no tags, works everywhere)
  -G ARG    MP3: apply peak normalization (albumpeak or trackpeak);
            this makes the tracks as loud as possible without clipping.
            Note: it is possible to specify an arbitrary peak reference lower
            than 0dBFS with an additional -G parameter: '-G albumpeak -G -4'.
  -G GAIN   MP3: apply arbitrary GAIN (signed number from -99.99 to +99.99).

-------------------------------------------------------------------------------

Information:

  -h        display this help and exit
  -V        output version information and exit

-------------------------------------------------------------------------------

caudec uses a temporary directory for processing files ($TMPDIR or /tmp by
default). If you wish to use another directory, set the CAUDECDIR environment
variable to its path (export CAUDECDIR="/some/dir"), or set CAUDECDIR in your
configuration file (~/.caudecrc or /etc/caudecrc).

To enable debugging, set the CAUDECDEBUG environment variable to 'true'
(export CAUDECDEBUG='true'). caudec will output some additional information,
as well as a log of errors that occurred while using external tools.

For more help, see the online documentation:
https://github.com/gcocatre/caudec
```