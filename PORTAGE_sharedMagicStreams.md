Up: [PortageII](PortageMachineTranslation.md)
Previous: [Performance](PORTAGE_sharedPerformanceConsiderations.md)
Next: [PortageLive](PortageLiveManual.md)

-----------------------

## Magic Streams

Almost all programs in the PORTAGE shared suite use what's known as a ''magic stream'', which transparently handles any of the following in place of a file name argument:
* standard input or output (specified as "-")
* a pipe
* a compressed (.gz, .z, .bz2 or .lzma) file

For example, suppose some program `portageii_pgm` reads an input file and writes an output file. Normally it gets run like this:
| portageii_pgm -in infile -out outfile
Now suppose the files are compressed. With magic stream, any of the following could be used:
| portageii_pgm -in infile.gz -out outfile.gz

| gzip infile.gz | portageii_pgm -in - -out - | gzip > outfile.gz

| portageii_pgm -in "zcat infile.gz|" -out outfile.gz

And what if you want to pipe two or more inputs to a PORTAGE shared program?  You can specify as many pipes as there are file arguments:
|  head `yourFile.txt` | portageii_pgm \
|   -in1 i1.gz                       \
|   -in2 "head -4 i2.txt|"           \
|   -in3 i3.txt                      \
|   -in4  -                          \
|   -out1 o1.gz                      \
|   -out2 "| head -4 > outfile.txt"  \
|   -out3 o3.txt                     \
|   -out4 -                          \
|   -out5 "/dev/null"
Here in4 will contain the first lines of `yourFile.txt` and out4 is going to be displayed on stdout.  NOTE that only one "-" as input can work correctly since every magic stream opened with "-" is attached to cin (stdin). Similarly, multiple output streams opened with "-" will yield mixed output on cout (stdout).

-----------------------

Up: [PortageII](PortageMachineTranslation.md)
Previous: [Performance](PORTAGE_sharedPerformanceConsiderations.md)
Next: [PortageLive](PortageLiveManual.md)