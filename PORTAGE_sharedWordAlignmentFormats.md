Up: [PortageII](PortageMachineTranslation.md)
Previous: [TextFileFormats](PORTAGE_sharedFileFormats.md)
Next: [ConstructingModels](PORTAGE_sharedTrainingModels.md)

-------------------------

## Using Word Alignments

When using the framework to train PORTAGE shared systems, you will see the word alignments stored in `*.align.gz` files in directory `models/wal`. These will be stored in the `sri` format (see below) which is most common in the SMT community. We used to recalculate alignments on the fly each time we needed them, because IBM1 and IBM2 alignments are quick to generate. But with HMM and IBM4 alignments, that strategy is no longer reasonable.

Now, the `sri` format can be hard to read, so we have tools to generate word alignment in several formats, and to manipulate them.

To inspect, evaluate, or use word alignments directly, one can use the program `align-words` to output word alignments in any of the formats documented below.

The program `eval_word_alignment` can be used to evaluate word alignments against a reference alignment.  As input, it supports the `sri`, `hwa` and `green` formats shown below.

The program `word_align_tool` can be used to manipulate word alignment files.  It can convert from `sri`, `green` and `hwa` to any of the other formats shown below, as well as perform a couple transformations on alignments themselves.

The program `gen_phrase_tables` can read `GIZA++` alignment files, as an alternative to generating alignments on the fly.  This is intended to allow the use of word alignment models trained with `GIZA++` in the PORTAGE shared phrase extraction process. The `align-words` program can also be used to convert `GIZA++` alignment files to any of the formats listed below.

The program `word_align_phrasepairs` produces word alignments that are consistent with a given phrase alignment (as generated during decoding, for example).

## Word Alignment Formats

We illustrate the formats produced by `align-words` with an example.  This example is found in `test-suite/unit-testing/word_align_tool` (run `make` to generate all the output formats).

English input:

`For years the Canadian Alliance has attempted to get the former finance minister to treat municipalities with respect .`

French input:

`L' Alliance canadienne tente depuis des ann?es d' obtenir que l' ancien ministre des Finances traite les municipalit?s avec respect .`


### Format: `aachen`

 SENT: 0
 S 0 0
 S 1 1
 S 2 2
 S 3 6
 S 4 3
 S 4 5
 S 5 4
 S 6 6
 S 7 7
 S 8 8
 S 9 8
 S 10 9
 S 11 10
 S 11 11
 S 12 12
 S 13 11
 S 14 12
 S 15 14
 S 16 13
 S 17 15
 S 18 16
 S 19 17
 S 20 18


### Format: `hwa`

(Output is in a file, `aligned.NNN`.)

 0  0  (L', For)
 1  1  (Alliance, years)
 2  2  (canadienne, the)
 3  6  (tente, attempted)
 4  3  (depuis, Canadian)
 4  5  (depuis, has)
 5  4  (des, Alliance)
 6  6  (ann?es, attempted)
 7  7  (d', to)
 8  8  (obtenir, get)
 9  8  (que, get)
 10  9  (l', the)
 11  10  (ancien, former)
 11  11  (ancien, finance)
 12  12  (ministre, minister)
 13  11  (des, finance)
 14  12  (Finances, minister)
 15  14  (traite, treat)
 16  13  (les, to)
 17  15  (municipalit?s, municipalities)
 18  16  (avec, with)
 19  17  (respect, respect)
 20  18  (., .)


### Format: `matrix`

 |F|y|t|C|A|h|a|t|g|t|f|f|m|t|t|m|w|r|.|
 |o|e|h|a|l|a|t|o|e|h|o|i|i|o|r|u|i|e| |
 |r|a|e|n|l|s|t| |t|e|r|n|n| |e|n|t|s| |
 | |r| |a|i| |e| | | |m|a|i| |a|i|h|p| |
 | |s| |d|a| |m| | | |e|n|s| |t|c| |e| |
 | | | |i|n| |p| | | |r|c|t| | |i| |c| |
 | | | |a|c| |t| | | | |e|e| | |p| |t| |
 | | | |n|e| |e| | | | | |r| | |a| | | |
 | | | | | | |d| | | | | | | | |l| | | |
 | | | | | | | | | | | | | | | |i| | | |
 | | | | | | | | | | | | | | | |t| | | |
 | | | | | | | | | | | | | | | |i| | | |
 | | | | | | | | | | | | | | | |e| | | |
 | | | | | | | | | | | | | | | |s| | | |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |x| | | | | | | | | | | | | | | | | | |L'
 | |x| | | | | | | | | | | | | | | | | |Alliance
 | | |x| | | | | | | | | | | | | | | | |canadienne
 | | | | | | |x| | | | | | | | | | | | |tente
 | | | |x| |x| | | | | | | | | | | | | |depuis
 | | | | |x| | | | | | | | | | | | | | |des
 | | | | | | |x| | | | | | | | | | | | |ann?es
 | | | | | | | |x| | | | | | | | | | | |d'
 | | | | | | | | |x| | | | | | | | | | |obtenir
 | | | | | | | | |x| | | | | | | | | | |que
 | | | | | | | | | |x| | | | | | | | | |l'
 | | | | | | | | | | |x|x| | | | | | | |ancien
 | | | | | | | | | | | | |x| | | | | | |ministre
 | | | | | | | | | | | |x| | | | | | | |des
 | | | | | | | | | | | | |x| | | | | | |Finances
 | | | | | | | | | | | | | | |x| | | | |traite
 | | | | | | | | | | | | | |x| | | | | |les
 | | | | | | | | | | | | | | | |x| | | |municipalit?s
 | | | | | | | | | | | | | | | | |x| | |avec
 | | | | | | | | | | | | | | | | | |x| |respect
 | | | | | | | | | | | | | | | | | | |x|.
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+



### Format: `compact`

 0       1;2;3;7;4,6;5;7;8;9;9;10;11,12;13;12;13;15;14;16;17;18;19;


### Format: `ugly`

Warning: this format does not contain all alignment information.

`L'/For Alliance/years canadienne/the tente/attempted depuis/Canadian depuis/has des/Alliance ann?es/attempted d'/to obtenir/get que/get l'/the ancien/former ancien/finance ministre/minister des/finance Finances/minister traite/treat les/to municipalit?s/municipalities avec/with respect/respect ./. `


### Format: `green`

`0 1 2 6 3,5 4 6 7 8 8 9 10,11 12 11 12 14 13 15 16 17 18`


### Format: `sri`

`0-0 1-1 2-2 3-6 4-3 4-5 5-4 6-6 7-7 8-8 9-8 10-9 11-10 11-11 12-12 13-11 14-12 15-14 16-13 17-15 18-16 19-17 20-18 `


### Fromat: `uli`

`1 0:0:unspec 1:1:unspec 2:2:unspec 3,6:6:unspec 4:3,5:unspec 5:4:unspec 7:7:unspec 8,9:8:unspec 10:9:unspec 11,13:10,11:unspec 12,14:12:unspec 15:14:unspec 16:13:unspec 17:15:unspec 18:16:unspec 19:17:unspec 20:18:unspec`




-------------------------

Up: [PortageII](PortageMachineTranslation.md)
Previous: [TextFileFormats](PORTAGE_sharedFileFormats.md)
Next: [ConstructingModels](PORTAGE_sharedTrainingModels.md)
