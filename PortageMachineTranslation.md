Software for Statistical Machine Translation.

## PORTAGE shared 3.0

The PORTAGE shared series includes several leaps forward in NRC's SMT technology.

With Neural Network Joint Models (NNJM), version 3.0 brings deep learning to the set of tools used to improve machine translation quality, as well as NRC's improved reordering model based on sparse features, coarse LM''''''s, and other improvements. The NRC has put significant efforts in optimizing the default training parameters to improve the quality of the translations produced by PORTAGE shared, and we expect users to see and appreciate the difference.

With version 2.0, we added two important features to help translators: the transfer of markup tags from the source sentence to the machine translation output, as well as handling the increasingly common xliff file format. 2.1 and 2.2 were maintenance releases incorporating minor patches.

With version 1.0, we brought in significant improvements to translation engine that resulted in better translations, and contributed to our success at the NIST Open Machine Translation 2012 Evaluation (`OpenMT12`).

See file RELEASES for full details. See also `doc/PortageAPIComparison.pdf` for a series of tables showing the evolution of the API, the plugin architecture and the training configuration for PORTAGE shared versions 1.0, 2.0, 2.1, 2.2 and 3.0.

## User Manual

* [Background](PORTAGE_sharedOverview.md)
* [RequiredPriorKnowledge](PORTAGE_sharedKnowledgePrerequisites.md)
* [WhereToFindThings](PORTAGE_sharedWhereToFindThings.md)
* [TextProcessing](PORTAGE_sharedTextProcessing.md)
** [Extracting_corpora_from_a_TMX](TMXProcessing.md)
* File Formats
** [TextFileFormats](PORTAGE_sharedFileFormats.md)
** [WordAlignmentFormats](PORTAGE_sharedWordAlignmentFormats.md)
* Training
** [ConstructingModels](PORTAGE_sharedTrainingModels.md)
*** [LanguageModels](PORTAGE_sharedTrainingLanguageModels.md)
*** [OtherModels](PORTAGE_sharedTrainingOtherModels.md)
*** TightlyPackedTries
** [OptimizingWeights](PORTAGE_sharedTrainingOptimizingWeights.md) 
* [Translating](PORTAGE_sharedTranslating.md)
** [Decoding](PORTAGE_sharedTranslatingDecoding.md)
** [RescoringNbestLists](PORTAGE_sharedTranslatingRescoringNbestLists.md)
** [Postprocessing](PORTAGE_sharedTranslatingPostprocessing.md)
** Detailed notes on [TheDecoderSearchAlgorithmsAndDataStructures](PORTAGE_sharedDecoderSearchAlgorithmsAndDataStructures.md)
** Notes about UsingPhrasetablesInCanoe
* [Evaluation](PORTAGE_sharedEvaluation.md)
* ConfidenceEstimation
* [PerformanceConsiderations](PORTAGE_sharedPerformanceConsiderations.md)
* [MagicStreams](PORTAGE_sharedMagicStreams.md)
* PortageLiveManual
* PortageLiveCustomizationPlugins
* [ProgrammerReference](PORTAGE_sharedProgrammerReference.md)
** [WritingCode](PORTAGE_sharedWritingCode.md)
** [UsingMake](PORTAGE_sharedMakeNotes.md) (detailed notes)
** [ProgrammingGuidelines](PORTAGE_sharedProgrammingGuidelines.md)
* [AnnotatedBibliography](PORTAGE_sharedAnnotatedBibliography.md)
* [RequirementsForTrainingAMidSizeSystem](TrainingEuroparl.md)
* [FrequentlyAskedQuestions](PORTAGE_sharedFAQ.md)
