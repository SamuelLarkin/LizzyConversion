Up PortageMachineTranslation!_PortageII
Previous: PORTAGE_sharedTrainingOptimizingWeights!_OptimizingWeights
Down: PORTAGE_sharedTranslatingDecoding!_Decoding

-------------------------

'''Note:''' this section does not reflect current best practice. The whole translation pipeline is now automated in our experimental framework for translation during training, and in script `translate.pl` for use in Portage''''''Live.  Please read `tutorial.pdf` in `framework` and the PortageLiveManual.

## Translating

Source texts to be translated by PORTAGE shared must be in
PORTAGE_sharedFileFormats!_TokenizedText#TokenizedText or
PORTAGE_sharedFileFormats!_MarkedUpText#MarkedUpText formats; the
PORTAGE_sharedTextProcessing!_TextProcessing#TextProcessing section describes the required processing steps. Translation itself consists of three main steps, of which the last two are optional: decoding, rescoring, and postprocessing.

* PORTAGE_sharedTranslatingDecoding!_Decoding
* PORTAGE_sharedTranslatingRescoringNbestLists!_RescoringNbestLists
* PORTAGE_sharedTranslatingPostprocessing!_Postprocessing

* Detailed notes on PORTAGE_sharedDecoderSearchAlgorithmsAndDataStructures!_TheDecoderSearchAlgorithmsAndDataStructures
* Notes about UsingPhrasetablesInCanoe

-------------------------

Up PortageMachineTranslation!_PortageII
Previous: PORTAGE_sharedTrainingOptimizingWeights!_OptimizingWeights
Down: PORTAGE_sharedTranslatingDecoding!_Decoding