Software for Statistical Machine Translation.

!! PORTAGE shared 3.0

The PORTAGE shared series includes several leaps forward in NRC's SMT technology.

With Neural Network Joint Models (NNJM), version 3.0 brings deep learning to the set of tools used to improve machine translation quality, as well as NRC's improved reordering model based on sparse features, coarse LM''''''s, and other improvements. The NRC has put significant efforts in optimizing the default training parameters to improve the quality of the translations produced by PORTAGE shared, and we expect users to see and appreciate the difference.

With version 2.0, we added two important features to help translators: the transfer of markup tags from the source sentence to the machine translation output, as well as handling the increasingly common xliff file format. 2.1 and 2.2 were maintenance releases incorporating minor patches.

With version 1.0, we brought in significant improvements to translation engine that resulted in better translations, and contributed to our success at the NIST Open Machine Translation 2012 Evaluation (`OpenMT12`).

See file RELEASES for full details. See also `doc/PortageAPIComparison.pdf` for a series of tables showing the evolution of the API, the plugin architecture and the training configuration for PORTAGE shared versions 1.0, 2.0, 2.1, 2.2 and 3.0.

!! User Manual

* PORTAGE_sharedOverview!_Background
* PORTAGE_sharedKnowledgePrerequisites!_RequiredPriorKnowledge
* PORTAGE_sharedWhereToFindThings!_WhereToFindThings
* PORTAGE_sharedTextProcessing!_TextProcessing
** TMXProcessing!_Extracting_corpora_from_a_TMX
* File Formats
** PORTAGE_sharedFileFormats!_TextFileFormats
** PORTAGE_sharedWordAlignmentFormats!_WordAlignmentFormats
* Training
** PORTAGE_sharedTrainingModels!_ConstructingModels
*** PORTAGE_sharedTrainingLanguageModels!_LanguageModels
*** PORTAGE_sharedTrainingOtherModels!_OtherModels
*** TightlyPackedTries
** PORTAGE_sharedTrainingOptimizingWeights!_OptimizingWeights 
* PORTAGE_sharedTranslating!_Translating
** PORTAGE_sharedTranslatingDecoding!_Decoding
** PORTAGE_sharedTranslatingRescoringNbestLists!_RescoringNbestLists
** PORTAGE_sharedTranslatingPostprocessing!_Postprocessing
** Detailed notes on PORTAGE_sharedDecoderSearchAlgorithmsAndDataStructures!_TheDecoderSearchAlgorithmsAndDataStructures
** Notes about UsingPhrasetablesInCanoe
* PORTAGE_sharedEvaluation!_Evaluation
* ConfidenceEstimation
* PORTAGE_sharedPerformanceConsiderations!_PerformanceConsiderations
* PORTAGE_sharedMagicStreams!_MagicStreams
* PortageLiveManual
* PortageLiveCustomizationPlugins
* PORTAGE_sharedProgrammerReference!_ProgrammerReference
** PORTAGE_sharedWritingCode!_WritingCode
** PORTAGE_sharedMakeNotes!_UsingMake (detailed notes)
** PORTAGE_sharedProgrammingGuidelines!_ProgrammingGuidelines
* PORTAGE_sharedAnnotatedBibliography!_AnnotatedBibliography
* TrainingEuroparl!RequirementsForTrainingAMidSizeSystem
* PORTAGE_sharedFAQ!FrequentlyAskedQuestions
