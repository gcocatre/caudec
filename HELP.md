```
caudec 3.1.1: multiprocess audio converter
Copyright © 2012 - 2025 Guillaume Cocatre-Zilgien
https://github.com/gcocatre/caudec

Usage: caudec [ GLOBAL PARAMETERS ] [ PROCESSING [ PARAMETERS ] ] FILES
Operate on multiple audio files at once, in parallel.
Instead of multiple files, one or more directories may be specified.
Multiple codec switches (optionally paired with a -q switch) may be specified.
Supported input files: .wav, .aiff, .caf, .flac, .wv, .ape, .m4a

-------------------------------------------------------------------------------

Global parameters:

  -n N      launch N processes concurrently (1 or more);
            by default, the number of CPU cores.

  -i        ignore unsupported files

  -s        be silent, only print errors

  -z        produce machine-parsable output; must be the first parameter on the
            command line to take effect. Run 'caudec -z' on its own to print
            a description of the syntax.

  -Z        produce human-readable output to STDOUT, and machine-parsable
            output to STDERR

-------------------------------------------------------------------------------

Applying gain after decoding and before encoding:

  -G ARG    apply ReplayGain ('album' or 'track') if found in source file
            metadata. Note: it is possible to specify a preamp value with an
            additional -G GAIN parameter (see below), for instance
            '-G album -G -3' or '-G track -G +2'.

  -G ARG    apply peak normalization ('albumpeak' or 'trackpeak'); same as
            above, except album or track peak values are used as a starting
            point instead of album or track gain values. An additional preamp
            value may be specified as well (see -G GAIN below).

  -G GAIN   apply arbitrary GAIN (signed number from -99.99 to +99.99)

-------------------------------------------------------------------------------

Resampling after decoding and before encoding:

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

Encoding parameters (mutually exclusive from all other actions):

  -c CODEC  use specified CODEC: wav, aiff, caf,
            flac, wv (WavPack), wvh (WavPack Hybrid), wvl (WavPack lossy),
            lossyFLAC, ape (Monkey's Audio), alac (m4a), aac (m4a), mp3,
            ogg / vorbis, opus.

            IMPORTANT NOTE: when encoding to lossyFLAC with a quality setting
            of 'S' or lower, the input is upscaled to 24 bit and the signal is
            scaled down by lossyWAV (volume reduction) in order to improve
            quality and compression. It is thus strongly advised to run
            'caudec -g' on the resulting files in order to compute ReplayGain
            values that will bring back the volume to a normal level.
            The 'transcaude' utility will do that automatically.

  -C CODEC  use specified CODEC, but discard existing metadata

  -q ARG    set compression level (variable bitrate mode; try -q help for
            a list of valid values)

  -b ARG    constant or target bitrate in bits per sample (for -c wvh/wvl)
            or in kilobits per second (for all lossy codecs)
  -B ARG    average bitrate in kilobits per second (for all lossy codecs)

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
            located in the source directory. If FILE is an empty string,
            no artwork will be embedded, not even the source file's
            embedded artwork.

  -f ARG    copy certain files to the output directory. ARG may be a 'all',
            a comma separated list of files, and / or file extensions preceded
            by a dot. If ARG is a list of files, they must be filenames and
            those files must be located in the source directory. If ARG is
            an empty string, no files will be copied.

  -k        keep existing destination files (don't overwrite them)
  -K        keep existing destination files if they're newer than their source


When transcoding to multiple codecs at once (multiple -c parameters),
specify a -o/O/P parameter after each -c parameter in order to set per-codec
output directories. For instance:
$ caudec -c flac -P ~/Music/FLAC -c mp3 -P ~/Music/MP3 "Artist/Album"/*.flac


Similarly, specifying -a/-f after a -c parameter will affect only that codec.
For instance:
$ caudec -c flac -P ~/Music/FLAC -f cover.jpg \
  -c mp3 -P ~/Music/MP3 -f folder.jpg "Artist/Album"/*.flac


Creating a WavPack Hybrid collection with a different root directory for lossy:
$ caudec -c wvh -P ~/Music/Lossless -L ~/Music/Lossy "Artist/Album"/*.flac
That will create a mirrored directory structure with hard linked lossy files.
~/Music/Lossless will contain the full lossless collection, which can be safely
backed up and ~/Music/Lossy will contain only the lossy part of the collection.
Little to no extra storage will be used (just extra inodes).

-------------------------------------------------------------------------------

Decoding parameters (all mutually exlusive from each other and other actions):

  -d        decode to WAV (input: AIFF, CAF, FLAC, WavPack, Monkey's Audio)
  -t        test file integrity
  -H HASH   compute hash of raw PCM (CRC32, MD5, SHA1, SHA256 or SHA512,
            FLAC, WavPack, Monkey's Audio and lossyFLAC only)
  -H ^HASH  do NOT compute HASH even if it's in caudecrc
  -H i[nt]  show internal hash, if available

-------------------------------------------------------------------------------

ReplayGain parameters (mutually exclusive from all other actions):

  -g        generate ReplayGain metadata

  -G ARG    MP3: apply ReplayGain ('album' or 'track') using MP3's gain header.
            It is the standard way for MP3 and is reversible.

  -G undo   MP3: reverse ReplayGain using MP3's gain header.
            Other codecs: remove ReplayGain metadata.

-------------------------------------------------------------------------------

Touching file times (mutually exclusive from all other actions):

  -T        touch files using metadata to reflect the music's release date
            and duration. Provide all files as arguments, not just audio files,
            although at least one audio file is required for this to work.
            Use with -l when dealing with a hard linked collection (-c wvh).
            Ex: caudec -T ~/Music/Lossless/Artist/Album/*
            Ex: caudec -T ~/Music/Lossless/Artist/Album/* -l ~/Music/Lossy

-------------------------------------------------------------------------------

Information:

  -h        display this help and exit
  -I        display channels, sample size and sampling rate
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