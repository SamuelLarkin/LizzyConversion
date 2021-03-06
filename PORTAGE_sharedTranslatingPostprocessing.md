Up [PortageII](PortageMachineTranslation.md) / [Translating](PORTAGE_sharedTranslating.md)
Previous: [RescoringNbestLists](PORTAGE_sharedTranslatingRescoringNbestLists.md)
Next: [DecoderAlgorithms](PORTAGE_sharedDecoderSearchAlgorithmsAndDataStructures.md)

------------------

## Translating: Postprocessing

#### Truecasing

Text output from canoe is lowercase
[tokenized_text#TokenizedText.](PORTAGE_sharedFileFormats.md) The first step in making this nicer to read is truecasing, which tries to restore capitals where appropriate.
Assuming language and mapping models
`tc-lm.en` and `tc-map.en` trained as described in [TrueCaseModels#TrueCaseModels,](PORTAGE_sharedTrainingModels.md) truecasing can be carried out as follows:
|   truecase.pl -bos -encoding UTF-8 -text=text_en.out \
|      -lm=tc-lm.en -map=tc-map.en > text_en.tc

The `-bos` option forces capitalization of the first letter of each sentence.

The new truecasing method, using source sentence information, requires several other options and models.  See `truecase.pl -h`, the framework and the tutorial for details.

#### Detokenizing

The second and last step in the processing chain is detokenization, to convert truecased
[tokenized_text#TokenizedText](PORTAGE_sharedFileFormats.md) back to [plain_text#PlainText](PORTAGE_sharedFileFormats.md) (but preserving the one-sentence-per-line convention). Example for English encoded in utf-8:
|   udetokenize.pl text_en.out > text_en.txt
|   udetokenize.pl text_en.tc > text_en.tc.txt

For French, add the option `-lang=fr`.

------------------

Up [PortageII](PortageMachineTranslation.md) / [Translating](PORTAGE_sharedTranslating.md)
Previous: [RescoringNbestLists](PORTAGE_sharedTranslatingRescoringNbestLists.md)
Next: [DecoderAlgorithms](PORTAGE_sharedDecoderSearchAlgorithmsAndDataStructures.md)