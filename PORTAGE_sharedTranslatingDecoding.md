Up [PortageII](PortageMachineTranslation.md) / [Translating](PORTAGE_sharedTranslating.md)
Next: [RescoringNbestLists](PORTAGE_sharedTranslatingRescoringNbestLists.md)

-------------------------

## Translating: Decoding

The command `canoe -h` describes how to run the decoder. Here we assume you already trained your models and tuned them, with the results of `tune.py` available in model file `canoe.ini.cow`.

If you are translating French text that is already preprocessed, sentence-split and tokenized, say `test_fr.tok`, into English, this command does the decoding step:
|   canoe-escapes.pl -add text_fr.tok text_fr.rule
|   canoe -f canoe.ini < text_fr.rule > text_en.out

See UsingPhrasetablesInCanoe for more information on using multiple phrasetables in canoe.

If the source file contains
[MarkedUpText#MarkedUpText,](PORTAGE_sharedFileFormats.md) then any translations specified in markup are used instead of those provided by the translation model, unless the `-bypass-marked` switch is given to canoe, in which case the two are combined.  (If you use rules, don't use `canoe-escapes.pl`; your rule-creating software should escape any `<`, `>` and `\` meant to be interpreted literally.)

Running canoe on the untuned model will yield very bad results, so don't do it unless you're working on a new optimization technique. The tuning procedure described in
[OptimizingWeights#TrainingOptimizingWeights](PORTAGE_sharedTrainingOptimizingWeights.md) sets optimized value for all the model weights. In general, the remaining parameters such as `beam-threshold` and `ttable-limit` control the trade-off between accuracy and speed: they should be set to values that give a good compromise between quality and speed for your application.

-------------------------

Up [PortageII](PortageMachineTranslation.md) / [Translating](PORTAGE_sharedTranslating.md)
Next: [RescoringNbestLists](PORTAGE_sharedTranslatingRescoringNbestLists.md)