# vgm-converter
Script for processing SN76489 PSG based VGM files 

## Functionality
VGM files containing data streams for the SN76489 PSG are encoded for a particular clock speed.
Playing back these files on hardware (or emulators) with different clock speeds will not sound accurate
due to the fact that the clock speed of the sound chip dictates the frequencies that are output.

This script allows VGM files to be transposed for different clock speeds. Currently it supports translation
between NTSC, PAL and BBC Micro clock frequencies (4Mhz)

It also supports transposing of tuned white or period noise effects.

Further, it can re-process the VGM file so that the sound data can be output at a fixed playback interval (eg. 50Hz/60Hz/100Hz).
This is useful for using VGM as an optimized music data format on older 8-bit machines where timed interrupts have fixed intervals.
 
There are also some utility functions:
 - it can filter out unwanted channels
 - work in progress toward a raw binary output file format
 - work in progress toward dumping of a human readable logs of the VGM
 

## Usage

`vgmconverter.py <vgmfile> [-transpose <n>] [-quantize <n>] [-filter <n>] [-rawfile <filename>] [-output <filename>] [-dump] [-verbose]`


where:

`<vgmfile> is the source VGM file to be processed. Wildcards are not yet supported.`

Supports gzipped VGM or .vgz files.

### Options


`[-transpose <clocktype>, -t <clocktype>]`

Transpose the source VGM to a new frequency. For clocktype, specify 'ntsc' (3.57MHz), 'pal' (4.2MHz) or 'bbc' (4.0MHz)

`[-quantize <n>, -q <n>]`

Quantize the VGM to a specific playback update interval. For n, specify an integer Hz value

`[-filter <n>, -n <n>] `

Strip one or more output channels from the VGM. For n, specify a string of channels to filter eg. '0123' or '13' etc.

`[-rawfile <filename>, -r <filename>] `

Output a raw binary file version of the chip data within the source VGM. A default quantization of 60Hz will be applied if not specified with -q

`[-output <filename>, -o <filename>] `

Specifies the output filename for the processed VGM. It is optional as sometimes it's useful to process a VGM file only for informational purposes.

`[-dump, -d] `

Output human readable version of the VGM

`[-verbose, -v] `

Emit debug information


## Examples

Converting non-BBC Micro SN76489 VGM music to be compatible with a BBC Micro:

`vgmconverter.py myfile.vgm -t bbc -o beebfile.vgm`

Quantizing VGM music to 50Hz fixed playback rate:

`vgmconverter.py beebfile.vgm -q 50 -o beebfile50.vgm`

Transpose and quantize a VGM, outputting a raw binary file containing pure sound chip data streams:

`vgmconverter.py myfile.vgm -q 50 -t bbc -r beebfile.bin`

All of the above:

`vgmconverter.py myfile.vgm -q 50 -t bbc -o beebfile50.vgm -r beebfile.bin`

## Notes

* Processing is applied in a fixed order regardless of the command line order.
* Wildcard filenames are not yet supported
* -dump and -rawfile options are still a work in progress.

## Raw data file format

Intended as a compact data format of the VGM for memory-constrained 8-bit platforms, the binary format is structured as follows:

* SN76489 sound chip data is organised into a stream of packets - 1 packet per playback interval (50Hz = 20ms etc.)
* Each packet contains a header byte followed by upto 11 bytes of sound chip data
* The header byte indicates how many data bytes are in the packet, or 0 if no data needs to be sent to the sound chip for this interval.
* This is then repeated for the duration of the song
* The file ends with 0xFF

There is no support for looping in this format (yet).


