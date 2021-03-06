Up: [PortageII](PortageMachineTranslation.md)
Previous: [UsingPhrasetables](UsingPhrasetablesInCanoe.md)
Next: ConfidenceEstimation

-------------------------

## Evaluation

'''Evalation should be done on a corpus distinct from the one used for training and rescoring'''. The standard metric is
BLEU (WER and PER are also supported), which compares MT output to one or more ''reference'' translations produced by humans. Assuming that `test_en.out` contains raw canoe output and `test_en.ref1`, `test_en.ref2`, and `test_en.ref3` contain
lowercase tokenized reference translations, BLEU score can be computed using the `bleumain` program:
|   bleumain test_en.out test_en.ref1 test_en.ref1 test_en.ref3

BLEU score can also be calculated by the "official" NIST script
(`mteval-v11b.pl`, available at http://www.nist.gov/speech/tools/), which differs from `bleumain` in that it expects detokenized, SGML-encoded text as input. (It is also much [)](slower.md)

//Bleumain works only on tokenized text; to calculate the "official" //NIST BLEU score, use the `run-bleu` script on postprocessed output //from canoe like this:
//|   run-bleu test_fr.tok test_en.txt test_en.tcref1 test_en.tcref2 //test_en.tcref3
//where all files contain one segment per line, but have normal letter
//case and tokenization conventions. Note that `run-bleu` just //reformats its arguments and passes them to the NIST-supplied script //`mteval-v11a.pl`. This script will also work with tokenized text, //but `bleumain` is faster and doesn't try to "retokenize" its input.

Evaluation can also be performed on nbest lists, to approximate the best possible BLEU score obtainable via a rescoring operation. Given a tokenized nbest list file `text_en.nbest`, and a set of one or more tokenized reference files as above, this can be calculated using the `bestbleu` program:
|   bestbleu -n 100 -K 100 -num-sents 30 \
|      -best-trans-file-cat test_en.nbest \
|      -ref-files test_en.ref1 test_en.ref2 test_en.ref3
The `-n` argument(s) specifies the size of the nbest list(s) that `bestbleu` uses to calculate results; it must be at most the size of the actual nbest lists (given by the `-K` arg).

Another useful tool is `bleucompare`, which uses bootstrap resampling to determine whether the differences in BLEU scores among two or more translations (of the same source file) are significant.

### Variants of the BLEU formula

The are several different formulas used by different systems to calculate BLEU.  They differ in how the brevity penalty is calculated, in the use of smoothing, and in tokenization.

By default, we follow Philipp Koehn (that is, the script `multi-bleu.perl` distributed with the first WMT workshops) and IBM's original implementation in using the ''closest match'' reference length for each segment to determine the reference length.  In their implementation, from `mteval-v08.pl` through to the current `mteval-v11b.pl`, NIST has changed this definition, using the ''shortest'' reference length for each segment.  To make `rescore_train`, `bleumain` and all other programs using BLEU in PORTAGE shared use the NIST definition, you can define the environment variable `PORTAGE_NIST_STYLE_BLEU`.  Some scripts distributed with Moses also let you chose how you define the reference length.

For smoothing, NIST and Philipp Koehn treat 0 counts as such, yielding a BLEU score of 0 if there are no matching k-grams for any k.  IBM, in their `bleu-1.04.pl` script, uses a smoothing method replacing 0 counts by a value depending on k.  By default, our software replaces any 0 count by 1e-30, but programs doing BLEU calculations support a `-s` switch to override this behaviour.  See `bleumain -h` for details.

Finally, most tools will tokenize your inputs (translation and references) or apply some kind of normalization, even if they were already tokenized before.  This might change the results slightly, either because tokenizing twice makes unwanted changes, or because their tokenizer doesn't make the same decisions ours (or yours) would make.  `bleumain` assumes the translation and references are already tokenized, and so it doesn't change the tokenization.  So you should provide tokenized translations and references to `bleumain`, rather than calling it on detokenized text.


-------------------------

Up: [PortageII](PortageMachineTranslation.md)
Previous: [UsingPhrasetables](UsingPhrasetablesInCanoe.md)
Next: ConfidenceEstimation
