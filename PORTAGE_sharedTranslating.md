Up [PortageII](PortageMachineTranslation.md)
Previous: [OptimizingWeights](PORTAGE_sharedTrainingOptimizingWeights.md)
Down: [Decoding](PORTAGE_sharedTranslatingDecoding.md)

-------------------------

'''Note:''' this section does not reflect current best practice. The whole translation pipeline is now automated in our experimental framework for translation during training, and in script `translate.pl` for use in Portage''''''Live.  Please read `tutorial.pdf` in `framework` and the PortageLiveManual.

## Translating

Source texts to be translated by PORTAGE shared must be in
[TokenizedText#TokenizedText](PORTAGE_sharedFileFormats.md) or
[MarkedUpText#MarkedUpText](PORTAGE_sharedFileFormats.md) formats; the
[TextProcessing#TextProcessing](PORTAGE_sharedTextProcessing.md) section describes the required processing steps. Translation itself consists of three main steps, of which the last two are optional: decoding, rescoring, and postprocessing.

* [Decoding](PORTAGE_sharedTranslatingDecoding.md)
* [RescoringNbestLists](PORTAGE_sharedTranslatingRescoringNbestLists.md)
* [Postprocessing](PORTAGE_sharedTranslatingPostprocessing.md)

* Detailed notes on [TheDecoderSearchAlgorithmsAndDataStructures](PORTAGE_sharedDecoderSearchAlgorithmsAndDataStructures.md)
* Notes about UsingPhrasetablesInCanoe

-------------------------

Up [PortageII](PortageMachineTranslation.md)
Previous: [OptimizingWeights](PORTAGE_sharedTrainingOptimizingWeights.md)
Down: [Decoding](PORTAGE_sharedTranslatingDecoding.md)