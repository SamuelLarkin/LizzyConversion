Up: [PortageII](PortageMachineTranslation.md) / [Models](PORTAGE_sharedTrainingModels.md)
Previous: [LanguageModels](PORTAGE_sharedTrainingLanguageModels.md)
Next: TightlyPackedTries

-------------------------

'''Note:''' this section of the user manual presents all the different models you can use, but not how best to use them. The steps required to train PORTAGE shared following our current recommendations are automated in our experimental framework.  See `tutorial.pdf` in `framework` for details.

## Training: Constructing Other Models

* [TranslationModels#TranslationModels](PORTAGE_sharedTrainingOtherModels.md)
** [Phrase_tables_based_on_IBM2_word_alignment_models#PhrasetablesbasedonIBM2wordalignmentmodels](PORTAGE_sharedTrainingOtherModels.md)
** [Phrase_tables_based_on_HMM_word_alignment_models#PhrasetablesbasedonHMMwordalignmentmodels](PORTAGE_sharedTrainingOtherModels.md)
** [Phrase_tables_based_on_IBM4_word_alignment_models#PhrasetablesbasedonIBM4wordalignmentmodels](PORTAGE_sharedTrainingOtherModels.md)
** [Merged_phrase_tables#Mergedphrasetables](PORTAGE_sharedTrainingOtherModels.md)
** [TheTPPT_format#TheTPPTformat](PORTAGE_sharedTrainingOtherModels.md)
* [LexicalizedDistortionModels#LexicalizedDistortionModels](PORTAGE_sharedTrainingOtherModels.md)
* [HierarchicalLexicalizedDistortionModels#HierarchicalLexicalizedDistortionModels](PORTAGE_sharedTrainingOtherModels.md)
* [''']('''New.md) [SparseModels#SparseModels](PORTAGE_sharedTrainingOtherModels.md)
* [''']('''New.md) [NeuralNetworkJointModels#NeuralNetworkJointModels](PORTAGE_sharedTrainingOtherModels.md)
* [TruecasingModels#TruecasingModels](PORTAGE_sharedTrainingOtherModels.md)
* [Adding_a_lexicon_to_a_System#AddingALexiconToASystem](PORTAGE_sharedTrainingOtherModels.md)

### Translation Models

#### Phrase tables based on IBM2 word alignment models

Training translation models is done in two steps: first train IBM models
in both directions, then generate phrase tables using the models. Assuming
the current directory contains a set of files named `file*_fr.al` and
`file*_en.al` containing [AlignedText#AlignedText,](PORTAGE_sharedFileFormats.md) the following is a basic training process:
|   train_ibm -v  -s ibm1.fr_given_en.gz ibm2.fr_given_en.gz file* \
|      >& log.train_ibm.fr_given_en
|   train_ibm -vr -s ibm1.en_given_fr.gz ibm2.en_given_fr.gz file* \
|      >& log.train_ibm.en_given_fr
|   gen_phrase_tables -v -w1 -m8 -1 en -2 fr -multipr both
|      ibm2.fr_given_en.gz ibm2.en_given_fr.gz file* >& log.gen_phrase_tables

Notes:
# This example assumes that `file*` produces a listing in which each English file comes ahead of its French counterpart. When adapting it for other language pairs, you need to substitute the language whose abbreviation comes lexicographically first for English, and the one whose abbrev comes 2nd for French. Caveat trainor!
# When generating phrase tables with a big training corpus and you get the message 'aborted', try using the option `-m#`, with # between 4 and 7, instead of `-m8`. The parameter m represents the maximum phrase length (0 for no limit).

Output files are:
* `ibm1.{fr_given_en,en_given_fr}` - IBM model 1 ttables
* `ibm2.{fr_given_en,en_given_fr}` - IBM model 2 ttables
* `ibm2.{fr_given_en,en_given_fr}.pos` - IBM model 2 position params
* `phrases.{fr2en,en2fr}` - phrase tables in multi-prob text format

The `log.*` files contain information about the training process.

Markup in the text files is not allowed at this point.  The input should be non-marked-up, tokenized, sentence-aligned text.

#### Phrase tables based on HMM word alignment models

Starting with version 1.3, we also support using HMM word alignment models to produce phrase tables.  The `-hmm` and `-mimic` switches to `train_ibm` enable training HMM word alignment models.  `gen_phrase_tables` will automatically treat its input models as HMM if they were trained as such.  
The recommended practice is to use phrase tables trained using IBM2 models combined with another set trained using HMM models, and let `cow.sh` (see next section) tune their respective weights.

The following is a basic procedure for training a phrase table using HMM word alignment models:
|   train_ibm -v  -hmm hmm.fr_given_en.gz file* \
|      >& log.hmm.fr_given_en
|   train_ibm -vr -hmm hmm.en_given_fr.gz file* \
|      >& log.hmm.en_given_fr
|   gen_phrase_tables -v -w1 -m8 -1 en -2 fr -multipr both
|      hmm.fr_given_en.gz hmm.en_given_fr.gz file* \
|      >& log.gen_phrase_tables.hmm

Notes:
# HMM word alignment training is much slower than IBM2: HMM training takes cubic time for each training sentence pair, whereas IBM1 and IBM2 take quadratic time.  We therefore recommend the use of `cat.sh`, which parallelizes `train_ibm`, and `gen-jpt-parallel.sh`, which parallelizes `gen_phrase_tables`.
# We implemented a lot of variants on the HMM word alignment models.  Top level variants are selected via the `-mimic` switch, and a lot of switches further affect the model.  See `train_ibm -h` for details.
# For more information on training HMM word alignment models, including parallelized training and combining them with IBM2 models, follow the example in directory `framework`.

#### Phrase tables based on IBM4 word alignment models

In PORTAGE shared, we support easy integration of IBM4 word alignments produced by an external package, such as `MGiza++`.  Our experiments show that we can get best results by combining counts from IBM4, HMM and IBM2 alignments, and that this approach gives better results than using only the best alignments, or only the best two of these.  The three alignment techniques probably make different errors, so that combining phrase pairs extracted from the three exploits the strengths of each.

To simplify the integration of both different alignment models and alignment software, we have modified the pipeline described above to save the aligned corpus, and run phrase phrase extraction (`gen_phrase_tables`) using the saved alignment file instead of re-calculating the alignment using the models.  We do this for all methods, ours included.  And we use the standard SRI word alignment format, for easiest interoperability with other packages.  As usual, see the framework and its tutorial for more details.

#### Merged phrase tables

There are many ways to merge the phrase tables produced using the HMM, IBM2 and IBM4 models.  You can tally the counts together and then estimate probabilities using the merged counts.  You can also use indicator features to keep track of which alignment proposed which phrase pairs.  You can keep the relative frequency feature from each original alignment and combine with a global relative frequency feature and a global lexical smoothing feature.  You can also perform a linear combination of phrase tables that come from different corpora.  All these options are supported by the software, and some of them are automated in our framework.  See the framework and tutorial for details, as well as the help message from `joint2cond_phrase_tables`, `joint2multi_cpt`, and `train_tm_mixture`,.

In our framework, the default is to merge the counts from the HMM and IBM2 CPT''''s, which are generated using `gen-jpt-parallel.sh`, and then estimate the model parameters using `joint2cond_phrase_tables`.

#### The TPPT format

You can convert your multi-prob phrase tables into the TPPT (Tightly Packed Phrase Table) format using this command:

|   textpt2tppt.sh phrases.fr2en.gz

This will create directory `phrases.fr2en.tppt` which will contain several files that constitute the TPPT and must be kept together.  The TPPT can be used in canoe using the `[ttable-tppt]` option.  See UsingPhrasetablesInCanoe for details.

### Lexicalized Distortion Models

Moses style lexicalized distortion models are trained using `dmcount` and `dmestm`.  `dmcount` counts reordering events over a parallel training corpus, while `dmestm` estimates the model parameters from these counts.

The full documentation can be found by invoking each program with `-h`, as usual.

`dmcount` uses the same word alignment models as `gen_phrase_tables` and in fact repeats the same alignment task, so the inputs are similar (and we assume the same file names as above):
|   dmcount -v -m8 ibm2.en_given_fr ibm2.fr_given_en file* \
|      | gzip > dmcounts.fr2en.gz 2> log.dmcounts.fr2en.gz

`dmestm` uses these counts to perform MAP-smoothed estimates of the model parameters (the amount of smoothing is controlled by the -w* switches - higher values for these give smoother models).

|   dmestm -v -s -g ldm.fr2en.bkoff dmcounts.fr2en.gz | gzip \
|      > ldm.fr2en.gz 2> log.ldm.fr2en.gz

The file `ldm.fr2en.gz` is the trained lexicalized distortion model, which can be used in `canoe` via the `-distortion-model` option.

Your LDM can be converted to the TPLDM format with this command:
|   textldm2tpldm.sh ldm.fr2en.gz tpldm.fr2en

### Hierarchical Lexicalized Distortion Models

HLDM''''s are built with the same software as
LDM''''s, but with the additional `-hier` switch given to `dmcount`.
HLDM''''s give much more interesting results, and should be considered for regular use.  You should at least test them with your data, to see if they help.


### Sparse Models

With PORTAGE shared 3.0, we introduce sparse models, including the original sparse features of [Hopkins_andMay2011#HopkinsandMay2011](PORTAGE_sharedAnnotatedBibliography.md) as adapted for Portage by [Cherry_andFoster2012#CherryandFoster2012revisited,](PORTAGE_sharedAnnotatedBibliography.md)
as well as the Discriminative Reodering model of [Cherry2013#Cherry2013,](PORTAGE_sharedAnnotatedBibliography.md) which helps PORTAGE shared make better choices in word ordering during the decoding process.

Sparse features contribute a significant improvement to the quality of the translations produced by PORTAGE shared and should always be used. See `tutorial.pdf` in `framework` for instructions on how to train them.

### Neural Network Joint Models

In the last couple years, deep learning and neural networks have been shaking the world of natural language processing, including SMT. [Devlin_et_al2014#Devlinetal2014](PORTAGE_sharedAnnotatedBibliography.md) presents the first successful attempt at improving SMT output using neural networks. The NRC has reproduced the results in the Research version of Portage (see [Foster2016_addeddum_toDevlin#Foster2016addendumtoDevlin)](PORTAGE_sharedAnnotatedBibliography.md) and is continuing leading-edge research in the uses of deep learning to improve SMT.

The main model that has come out of this work so far is the Neural Network Joint Model (NNJM), which uses a deep learning approach to improve how target words are chosen to translate source text, taking into account up to 11 source words and 3 target words all at the same time.

With PORTAGE shared 3.0, we include the NNJM decoder feature, which allows users to incorporate NNJM''''''s trained at the NRC in their systems. We also include pre-trained NNJM''''''s for English-French translation in either direction as part of our updated Generic Model 2.0. We plan to release the training software for NNJM''''''s in a future release of PORTAGE shared.

### Truecasing Models

Truecasing is the process of restoring normal letter case conventions to lowercase target text produced by canoe. It is performed by treating normal text as an HMM state sequence to be recovered from observed lowercase text. The two steps in model training are thus to estimate state-transition probabilities from normal text (this is in fact a regular LM), and to estimate output probabilities from normal text mapped to lowercase.

Assuming that `corp-tc.en.gz` is a compressed truecased tokenized corpus, we train the map file using `compile_truecase_map`, and we train the LM file by invoking the LM toolkit directly.  In this example, we'll use SRILM, but IRSTLM or any toolkit would do.

| zcat corp-tc.en.gz | lc-utf8.pl | gzip > corp-lc.en.gz
| compile_truecase_map corp-tc.en.gz corp-lc.en.gz \
|    | gzip  > corp-tc.en.map.gz
| ngram-count -interpolate -kndiscount -order 3 \
|    -text corp-tc.en.gz -lm corp-tc.lm.gz
| arpalm2binlm corp-tc.lm.gz corp-tc.binlm.gz

The resulting truecasing model is the file pair `corp-tc.en.map.gz/corp-tc.binlm.gz` (fastest) or the file pair `corp-tc.en.map.gz/corp-tc.lm.gz`.

#### Truecasing Models using source information

We have improved our truecasing system with version 1.4.3.  We now transfer casing information from the source text to the output, and we handle the beginning of the sentence better.  To do so, several more models are trained, beside the casemap and LM described above, and these two models are created in a slightly different way.  See `truecase.pl -h`, the framework and the tutorial for details.

### Adding A Lexicon To A System

There are three ways to add a bitext lexicon to a system.
# Add your bilingual lexicon to the end of the corpus used for training phrase tables, then train phrase tables on this new corpus.  This simply means concatenating the parallel lexicon at the end of your training corpus, making sure you preserve line-alignment in the process.  In effect, you pretend that each entry in your lexicon is a sentence, aligned with its translation.  We have found this method to work fairly well in most situations.
# Manually create a joint frequency phrase table (jpt) from your parallel lexicon. Unlike method 1, this guarantees that only the lexicon pairs you explicitly specify will result in phrase pairs. For example, in method 1, if the lexicon contains 'potato / pomme de terre', you might end up with 'potato ||| terre' in the phrase table; this won't happen with method 2 unless you explicitly allow it. There are several options for converting the jpt into a conditional table (cpt) used by `canoe`: a) convert it by itself using `joint2cond_phrase_tables`; b) convert it along with other jpts using `joint2cond_phrase_tables`, adding counts before creating a single global cpt; and c) convert it using `joint2multi_cpt`, smoothing its counts with those from other jpts before inserting them as distinct columns in a single global cpt. Method a is not a good idea unless the lexicon is very large and complete; method b is crude but stable; method c is the most powerful, but involves parameters that require tuning for optimal performance.
# Apply your lexicon as rules in your source text that you want to translate.  A rule has the following form: <NPPP english="target phrase" prob="1.0">source phrase</NPPP>.  If you want canoe to apply rules strictly, make sure your `canoe.ini` doesn't have `[bypass-marked]`.  If you want to allow canoe to also consider phrases from phrase tables then include `[bypass-marked]` in your `canoe.ini`.  For this approach to work well, you have to be highly confident that your rules are accurate and unambiguous.  In particular, you should avoid using this method for words that have a given sense in your lexicon, but can also be general language words with a different sense or different translations.


-------------------------

Up: [PortageII](PortageMachineTranslation.md) / [Models](PORTAGE_sharedTrainingModels.md)
Previous: [LanguageModels](PORTAGE_sharedTrainingLanguageModels.md)
Next: TightlyPackedTries