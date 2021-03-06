Up: [PortageII](PortageMachineTranslation.md)
Previous: TightlyPackedTries
Next: [Translating](PORTAGE_sharedTranslating.md)

-------------------------

'''Important note:''' these steps are automated following current best practice in our experimental framework.  See `tutorial.pdf` in `framework` for details.

## Training: Optimizing Weights

Optimizing weights means tuning a relatively small set of parameters that control the contributions from various components like language and translation models. Translation quality tends to be very sensitive to these settings. Here are the parameter tuning steps for PORTAGE shared:

* [OptimizingDecodingWeights#OptimizingWeightsforCanoeDecoding](PORTAGE_sharedTrainingOptimizingWeights.md)
* [OptimizingRescoringWeights#OptimizingWeightsforRescoring](PORTAGE_sharedTrainingOptimizingWeights.md)
** [RescoringScript#RescoringScript](PORTAGE_sharedTrainingOptimizingWeights.md)

### Optimizing Weights for Canoe Decoding

PORTAGE shared supports a variety of weight tuning algorithms.  The old `cow.sh` has been replaced by `tune.py`, which supports MERT (using Powell's algorithm, the only method `cow.sh` supported), MIRA, lattice MIRA, PRO, SVM, and "expsb" (an experimental idea).
See Cherry and Foster (NAACL 2012) for a comparison of these methods.

See [DecoderModelTraining#DecoderModelTraining](PORTAGE_sharedOverview.md) for a detailed description of our original MERT algorithm.
Most of the new algorithms follow a similar batch-decode/batch-optimize loop, but some use lattices instead of n-best lists, and the optimizer used varies.

The `tune.py` script takes a marked-up version of the src, `dev1_ch.rules`, and a set of files `dev1_en.ref[123]` containing tokenized reference translations, as well as an input `canoe.ini` file indicating what decoder features and options are to be used, and produces `canoe.ini.cow` file with the best weights founds.

Ideally, you should use the framework to invoke `tune.py`, but here is a minimalist example using lattice MIRA, the recommended algorithm:
|   cat > canoe.ini
|   [ttable-limit] 30
|   [distortion-limit] 7
|   [ttable-multi-prob] phrases.fr2en
|   [lmodel-file] lm.en
|   [use-ftm]
|   ^D
|   canoe-escapes.pl -add dev1_fr.tok dev1_fr.rule
|   tune.py -v -f canoe.ini -o canoe.ini.cow -a lmira -c 1 -p 4
|      dev1_ch.rule dev1_en.ref1 dev1_en.ref2 dev1_en.ref3 >& log.lmira

To use a TPPT instead of `TMText` translation tables, replace the `[ttable-multi-prob]` line by this line:
|   [ttable-tppt] phrases.fr2en.tppt

Notes about using `tune.py`:

* Use `tune.py -h` for a description of the options to this command.  

* In the `canoe.ini` above, we provided `use-ftm`.  That's because other models will automatically be used if present, whereas by default the forward translation model feature is only used to select candidate phrases for each source phrase.  By specifying `use-ftm`, or a weight for this feature, we tell `tune.py` and `canoe` that we want this feature used and tuned.

* We provided the file `dev1_fr.rule` instead of `dev1_fr.tok` as the source text.  Since canoe has a markup language to encode rule-based translations, we need to escape the three special characters `<`, `>` and `\`, which is what `canoe-escapes.pl -add` does.  If you create your own rule-based translations, you should escape with a `\` any occurrence of these three characters meant to be interpreted literally.  In this case, do not use `canoe-escapes.pl`.

* This example uses a simplistic set of decoder parameters, which may not be optimal.  An example closer to recommended practice is found in `framework`.

* Tuning can take a long time, usually a day or more.  If you have suitable computing resources, you can use its parallel options to speed things up considerably.  The example above decodes 4-ways parallel, but we typically use 20 or 30 ways parallel on our computing cluster.

* To monitor its progress, look at the contents of the files `summary` and `summary.wts`, which contain the results of each iteration: including BLEU scores and the weight settings that produced it.  Note that BLEU scores do not necessarity improve monotonically.

* When `tune.py` is done, it produces a file called `canoe.ini.cow` with the weights that produced the best BLEU score, for use in the next step, decoding new text and/or rescoring. (The default is `canoe.tune`, but our framework uses `-o canoe.ini.cow`, so we did the same in the example here for clarity.)


### Optimizing Weights for Rescoring

It is sometimes beneficial to "rescore" the output from the decoder by having it produce n-best lists instead of single translation hypotheses, then using a richer log-linear model to choose the best hypothesis for each source sentence from the corresponding n-best list. 

See also:
* [RescoringModelTraining#RescoringModelTraining](PORTAGE_sharedOverview.md) -- detailed description of the algorithm used to 
learn rescoring weights,
* `doc/README.rescoring` (or `src/rescoring/README`) -- overview of the rescoring software, and
* RescoringOverview.pdf and OchLineMax.pdf (also in `doc/`) -- implementation information.

To train a rescoring model, you need:
* A source text;
* A version of the source text containing either [translation_rules#MarkedUpText,](PORTAGE_sharedFileFormats.md) or, if rules are not used, a version with the special characters escaped; 
* One or more reference translations for the source text (preferably more than one), line-aligned with the source text;
* A canoe configuration file containing learned weights (e.g., `canoe.ini.cow`, as output by `tune.py`); and
* A "rescoring model" file that specifies the desired features (typically this includes all features used for decoding, plus a number of additional features) in the syntax described by `rescore_train -h`, taken from the list given by `rescore_train -H`.

The source and reference files should be [tokenized#TokenizedText](PORTAGE_sharedFileFormats.md) and [aligned#AlignedText.](PORTAGE_sharedFileFormats.md)

#### Rescoring Script

For rescoring, we now support two tuning algorithms: our original Powell implementation (MERT), as well as n-best MIRA.  Specify `-a mira` if you want to use MIRA (recommended).

The recommended way to train a rescoring model is to use the `rescore.py` script (through the framework, preferably). This script executes all the steps described in the manual rescoring section below. It expects a rescoring model file in a special format:

* Features in the "small" set (i.e., decoder features) are specified as `FileFF:ffvals,`''i wt'', where ''i'' ranges from 1 to the size of the set, and ''wt'' is the feature's optimal weight, as found by `tune.py`.

* Additional features not in the "small" set are specified using the syntax described by `rescore_train -h`.  Any of the features listed by `rescore_train -H` may be used.  All features will be generated in parallel by `rescore.py` using `gen-features-parallel.sh`.  Expensive features will be divided into several "horizontal" blocks for parallelization, while cheap ones won't be.

Here is an example of creating a rescoring model file `rescore-model` for `rescore.py`, assuming an optimized canoe configuration file `canoe.ini.cow` output from `tune.py`. The first step is to encode features in the "small" set:
| > configtool rescore-model:ffvals canoe.ini.cow > rescore-model
|
| > cat rescore-model
| FileFF:ffvals,1 0.2
| FileFF:ffvals,2 -0.6
| FileFF:ffvals,3 1
| FileFF:ffvals,4 0.5
| FileFF:ffvals,5 0.3

In this case, the "small" set consists of 4 features. Appending 6 additional features gives the following "large" model:
| FileFF:ffvals,1 0.2
| FileFF:ffvals,2 -0.6
| FileFF:ffvals,3 1
| FileFF:ffvals,4 0.5
| FileFF:ffvals,5 0.3
| IBM1TgtGivenSrc:ibm1.en_given_fr
| IBM1SrcGivenTgt:ibm1.fr_given_en
| IBM2TgtGivenSrc:ibm2.en_given_fr 
| IBM2SrcGivenTgt:ibm2.fr_given_en 
| HMMTgtGivenSrc:hmm.tgt_given_src
| HMMSrcGivenTgt:hmm.src_given_tgt
| HMMVitTgtGivenSrc:hmm.tgt_given_src
| HMMVitSrcGivenTgt:hmm.src_given_tgt
| IBM1WTransTgtGivenSrc:ibm2.en_given_fr
| IBM1WTransSrcGivenTgt:ibm2.fr_given_en
| IBM1DeletionTgtGivenSrc:ibm1.en_given_fr#0.2
| IBM1DeletionSrcGivenTgt:ibm1.fr_given_en#0.2
| nbestWordPostLev:1#<ffval-wts>#<pfx>
| nbestWordPostTrg:1#<ffval-wts>#<pfx>
| nbestWordPostSrc:1#<ffval-wts>#<pfx>
| nbestPhrasePostSrc:1#<ffval-wts>#<pfx>
| nbestPhrasePostTrg:1#<ffval-wts>#<pfx>
| nbestNgramPost:3#1#<ffval-wts>#<pfx>
| nbestSentLenPost:1#<ffval-wts>#<pfx>
| NgramFF:big-lm.en
| LengthFF
| ParMismatch
| QuotMismatch:fe
The `IBM` features require that IBM model files `ibm2.en_given_fr`, etc. (or .gz compressed versions) exist in the current directory (a path relative to the current directory could also be used).  Similarly, the `NgramFF` feature requires that an n-gram language model file `big-lm.en` exists.  The `nbest` features require the decoder weights and the prefix of all intermediate files - `rescore.py` substitutes them in for the special tokens `<ffvals-wts>` and `<pfx>` automatically.

Assuming that the source file is called `dev2_fr.tok`, the source file containing rule-based translations is `dev2_fr.rule`, and the reference files are called `dev2_en.ref*`, `rescore.py` can be run like this:
| rescore.py -train -v --nbest-size 1000 -f canoe.ini.cow -msrc dev2_fr.rule \
|    rescore-model dev2_fr.tok dev2_en.ref* >& log.rescore
The learned weights are put into `rescore-model.rat`:
| FileFF:ffvals,1 0.01057143323
| FileFF:ffvals,2 0.1320968121
| FileFF:ffvals,3 0.00924924016
| FileFF:ffvals,4 -0.003593659261
| FileFF:ffvals,5 0.03378163235
| IBM1TgtGivenSrc:ibm1.en_given_fr 0.008430841379
| IBM1SrcGivenTgt:ibm1.fr_given_en 0.06629669666
| IBM2TgtGivenSrc:ibm2.en_given_fr -0.005373505875
| IBM2SrcGivenTgt:ibm2.fr_given_en 0.01150755864
| HMMTgtGivenSrc:hmm.tgt_given_src 0.02386927834
| HMMSrcGivenTgt:hmm.src_given_tgt 0.12385729353
| HMMVitTgtGivenSrc:hmm.tgt_given_src -0.03726194853
| HMMVitSrcGivenTgt:hmm.src_given_tgt 0.1389626143
| IBM1WTransTgtGivenSrc:ibm2.en_given_fr 0.1470945776
| IBM1WTransSrcGivenTgt:ibm2.fr_given_en 1
| IBM1DeletionTgtGivenSrc:ibm1.en_given_fr#0.2 -0.6528998613
| IBM1DeletionSrcGivenTgt:ibm1.fr_given_en#0.2 -0.2944137752
| nbestWordPostLev:1#<ffval-wts>#<pfx> 0.0276774019
| nbestWordPostTrg:1#<ffval-wts>#<pfx> 0.007647737861
| nbestWordPostSrc:1#<ffval-wts>#<pfx> -0.02277778275
| nbestPhrasePostSrc:1#<ffval-wts>#<pfx> 0.00322932424
| nbestPhrasePostTrg:1#<ffval-wts>#<pfx> -0.03512396291
| nbestNgramPost:3#1#<ffval-wts>#<pfx> -0.006613836158
| nbestSentLenPost:1#<ffval-wts>#<pfx> 0.9998786
| NgramFF:big-lm.en 0.2519059479
| LengthFF -0.0257593058
| ParMismatch 0.0007744894247
| QuotMismatch:fe -0.0718164444

Here are some detailed notes on using `rescore.py`:

* Intermediate files containing nbest lists and feature values are generated (in compressed format). These are not automatically removed, because they can be useful for training other rescoring models on the same source text. To avoid collisions, these files are placed in a directory intended to be unique to the current source file and the chosen n-best list size: it contains the `-msrc` argument if specified, otherwise the name of the source file. The intermediate files are:
** ''workdir''/1best - the single-best canoe translation 
** ''workdir''/nbest.gz - fixed-size nbest lists, one per source sentence
** ''workdir''/ffvals.gz - feature values for the "small" set, line-aligned with the nbest lists
** ''workdir''/ff* - values for individual features not in the "small" set, line-aligned with the nbest lists
** ''workdir''/pal.gz - phrase-alignment info, line-aligned with the nbest lists

* `rescore.py` requires the tokenized input text with and without rules.  The version with rules (as created by `canoe-escapes.pl -add` if you don't use rules) is passed via the `-msrc` option, while the version without rules is the first position argument, provided after all the options and before the references.  The references must contain normal tokenized text, without rules.

* When a new rescoring model is trained on the same source text, the nbest, ffvals, and pal files are not re-generated (assuming the workdir is not manually changed), nor feature files for any features that the new model has in common with the old one.  Thus, only required additional work will be done.

* `rescore.py` has 3 special tokens you can use in the model file: <src>, <ffval-wts> and <pfx>.  Leave them verbatim in your model if `rescore_train -H` says they are needed for a feature - `rescore.py` will substitute them as necessary.  See `rescore.py -h` for details. 


-------------------------

Up: [PortageII](PortageMachineTranslation.md)
Previous: TightlyPackedTries
Next: [Translating](PORTAGE_sharedTranslating.md)