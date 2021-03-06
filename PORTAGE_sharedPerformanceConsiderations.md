Up: [PortageII](PortageMachineTranslation.md)
Previous: ConfidenceEstimation
Next: [MagicStreams](PORTAGE_sharedMagicStreams.md)

-------------------------

## Memory Problems

Here are some suggestions for dealing with memory problems.

### `gen_phrase_tables` runs out of memory

* Consider reducing the `-m` parameter: 8 is worthwhile if you have enough memory, but use lower values if necessary.

* Phrase tables can be trained in smaller chunks via `gen-jpt-parallel.sh`.  Increase the `-n` parameter as high as you need to fit the process in the memory available to you.  This script produces a joint frequency phrase table.  Run `joint2cond_phrase_tables` on the result with the `-reduce-mem`.  The first step takes advantage of parallel computing resources, if available, and the second step takes about half the memory that `gen_phrase_tables` would have needed.
Warning: if you want to use the `-w` and `-prune1` parameters, you should specify `-w` to `gen-jpt-parallel.sh` before the `GPT` keyword, and `-prune1` to `joint2cond_phrase_tables`, otherwise their semantics will be distorted.

### Decoder runs out of memory

Many factors interact here.  The size of your models is significant, as is your choice of the parameters affecting the quality vs. speed trade-off, listed below.

A key variable is the length of the sentences you are trying to translate.  Decoding time and memory requirements grow significantly with sentence length.  `canoe` does not impose any limit on your sentence length, but very long sentences are likely to cause crashes if you don't have large amounts of memory.  We recommend avoiding sentences longer than 100 or so words in your dev and test sets if possible.

Note that the amount of memory required by the cube pruning decoder is less predictable than with the default decoder.  It seems to be more sensitive to long sentences than the default decoder, for reasons that we have not identified yet.

## Speed issues and the Quality vs. Speed trade-off

Several options in PORTAGE shared control settings that have a significant impact on the quality of the translations produced and the resources required (time and/or memory) to produce it.

### Long paragraphs and our tokenizer

When you use our tokenizer to perform sentence splitting (`tokenize.pl` or `utokenize.pl`), it processes a whole paragraph at a time, where the paragraph boundary is defined to be a blank line, i.e., two consecutive newline characters.  Running time is normally linear in file size, but if you have a large file without paragraph boundaries, it will all be slurped in memory.

To avoid this problem, and to improve the accuracy of sentence boundary detection, make sure you preserve paragraph boundaries of your data while you do your preprocessing.  If your data is already sentence-split, use the `-noss` option to disable sentence splitting.  In this case, the file will be processed one line at a time. If your data has one paragraph per line (instead of using a blank line to mark paragraph boundaries), use the `-paraline` option.

### Long files and sentence alignment

When you use our sentence aligner, `ssal` the two files are fully loaded into memory, as required to perform the Gale and Church dynamic programming algorithm.  Therefore, if your corpus consists of a large collection of document pairs, keep them in separate files, ideally one document per file, when you perform sentence alignment.  After tokenization, sentence splitting and alignment are performed, then it is safe to concatenate all your training data into large files.

### Long sentences and HMM word alignment models

Unlike with IBM2 models, long sentence pairs in your training data will significantly slow down HMM word alignment model training.  If you get warnings about long sentences when running `train_ibm` or `cat.sh`, you might consider redoing the preprocessing of your corpora to somehow chop up long sentences into shorter ones.

Chopping very long sentences, or maybe even deleting them, is not likely to have a significant impact on quality.

IBM1 and IBM2 models are not significantly affected by long sentence pairs because the algorithms for them all take quadratic time in sentence length (for each sentence pair), while the algorithms for HMM word alignment models take cubic time.

### Phrase table training options

`gen_phrase_tables -m` ''max'': maximum phrase length.

A higher ''max'' generally means better translation quality/BLEU score.  8 is generally a good value.  With large corpora, computing resource limits might force you to use a lower value, although in some cases, especially on large corpora, using 9 or 10 can yield further increases in BLEU scores.  

Large ''max'' values significantly increase the memory and time requirements for `gen_phrase_tables`.  They also increase memory and time requirements `canoe`, though not as much since, by default, `canoe` filters phrase tables while loading them, keeping only entries relevant for the sentences it is asked to translate.

`gen_phrase_tables` or `joint2cond_phrase_tables -prune1` ''n'': prune the phrase table so that each language1 phrase has at most ''n'' language2 translations.

This pruning is based on joint frequencies, and is done before smoothing.  A setting of 100 is sometimes useful, especially for noisy training corpora where a lot of spurious phrase pairs might be observed.

With the decoder's ''L'' parameter set to 30 (see below), ''n''=100 or 200 doesn't normally affect the decoder output much.  These two parameters differ in that the decoder's ''L'' parameter picks the ''L'' best candidate translations for a source according to all phrase tables and their combined log-linear weights, whereas the ''n'' parameter here prunes one phrase table without consideration for what other phrase tables might contain, and looking at raw counts only, not smoothed probabilities. 

### Language model training options

PORTAGE shared does not provide language model training software, but how you train your language models is known to have a significant impact on translation quality.  Larger language models will cost you in memory and speed, but that usually pays off in quality.  It is almost always worth using 4-grams LM's, and sometimes also 5-grams, depending on how much target-language text is available.  You can make your LM smaller by pruning small counts, usually without much impact on quality.  Having an extra language model trained on a large monolingual corpus is also usually worthwhile, despite its cost.  The LM deserves a lot of resources; it is better to skimp on other parameters (see above and below) before skimping on LM parameters.

To significantly speed up loading LM's in `canoe`, we recommend converting them to our binary format using `arpalm2binlm`.  This program requires enough memory to hold the whole LM in memory, but the binary files load 10 to 100 times faster than the text ones, with otherwise identical results.

### Decoder options

# Stack size (''S''; set via the `[stack]` setting in `canoe.ini`). Each stack corresponds to a number of words covered in the source sentence. The stack size is the maximum number of decoder states considered in a stack. ''S'' can be decreased to speed the system up, or increased to expand the translation search space.  With cube pruning (using `[cube-pruning]` in the `canoe.ini`), values between 300 and 10000 are reasonable.  Without cube pruning, reasonable values might be between 100 and 1000.
# Beam threshold (''T''; `[beam-threshold]`). In a given stack, a decoder state is only kept if its probability is no worse than the probability of the best state times ''T''. ''T'' must be less than one; the smaller it is, the more decoder states are kept. ''T'' can be increased to speed the system up, or decreased to expand the translation search space.  Reasonable values are around 0.001 to 0.00001.
# Translation table limit (''L''; `[ttable-limit]`). This is the maximum number of target phrases that may be considered for a given source phrase. Traditionally, we have set ''L'' to 30. This parameter can be decreased to speed the system up.  Increasing it is not typically worthwhile, but we have seen cases where 40 or 50 gave a BLEU gain.
# Distortion limit (''DL''; `[distortion-limit]`). The maximum number of words within which rearrangement may take place. This parameter can be decreased to speed the system up.  The typical value we use is 7, but 4 or 5 is often enough.  Some research groups have also experimented with monotonic decoding (''DL''=0), or quasi-monotonic decoding (''DL''=0 together with `[dist-phrase-swap]`) for certain purposes.

The cube pruning decoder (activated by `[cube-pruning]`) speeds up translation by a factor of up to 10 for comparable output quality, so we always use it, and we recommend its use as well.

We've done some experiments on the quality/speed tradeoff for French-English translation (in both directions), and we found that very fast translation can be obtained, at a loss of only 1 BLEU point in some cases, with cube pruning, S as low as 100 or 300, T as high as 0.1 or 0.03, L down to 5, and DL at 4 or 5.  Whether these settings will also work acceptably for you will depend on your language pair, your data and your application.  In this experiment, "very fast" meant about 10 sentences per second, while the slower, maximum quality system took 1 to 5 seconds per sentence.

### Rescoring features

Rescoring is fairly expensive and does not always yield improvements in quality/BLEU score.  The cost of computing various rescoring features varies widely by feature, so your choice of features will also significantly affect total running time and memory requirements.

-------------------------

Up: [PortageII](PortageMachineTranslation.md)
Previous: ConfidenceEstimation
Next: [MagicStreams](PORTAGE_sharedMagicStreams.md)
