Up: PortageMachineTranslation!_PortageII / PORTAGE_sharedTrainingModels!_Models
Previous: PORTAGE_sharedTrainingOtherModels!_TrainingOtherModels
Next: PORTAGE_sharedTrainingOptimizingWeights!_OptimizingWeights

----

!! Tightly Packed Tries

!!! Overview

Tightly Packed Tries (TPT's) use memory mapped IO to access a compact representation of models directly on disk.  Load time is negligible, access time is reasonable with a cost the first time a block is read from disk.  Overall performance is comparable to regular model files, but standard disk caching performed by the operating system allows a second instance of the decoder to access the models much faster, since it will not have to re-read what previous instances have already read, as long as the information is still in cache.  Furthermore, if multiple processes are using the same models simultaneously on the same machine (on a multi-core server, for example), the models will be stored in shared memory, so that any part read by one process is immediately available to the other.

This technology is especially well suited for use in a translation server.  When the phrase tables, language models and lexicalized distortion models (if used) are all stored in their tightly-packed variants, the translation server can reasonably be configured to process a single sentence at a time, never requiring the server process to remain up between requests.  This way, no initial load time is required.  The first few requests after a long silent period, or after server startup, will be slower, as the models start being loaded into main memory.  Afterwards, the server will coast at a reliably fast speed.

Memory requirements on the server are fairly small in this mode of operation since the tightly-packed models are quite compact.
The construction of these models requires a good amount of memory, as the whole model has to be stored in memory during the conversion process from the text representations, but their use does not require much memory.  On a multi-core system serving many request in parallel, the memory footprint of each process is further reduced because the memory is shared between all processes using the same models.

Such a server setting is exactly what is implemented in the PortageLiveManual!PortageLive system, which is now run-time server and  demonstration platform we use for PORTAGE shared.

!!! File structure

Tightly-packed models are stored on disk as a directory containing several files that are each opened using memory mapped IO.  The files should be kept together and, unlike all other file formats in PORTAGE shared, cannot be gzipped, because they are opened using memory mapped IO.  However, they are already very compact and usually take less space than the gzipped text files they represent.

!!! Further information

* Presentation on TPT's by Uli Germann:
Memory_mapping_Tightly_Packed_Tries_and_Suffix_Arrays.pdf.
* PORTAGE_sharedAnnotatedBibliography!_Germann_et_al_2009#Germannetal2009, SETQA-NLP paper on TPT's.
* PORTAGE_sharedAnnotatedBibliography!_Paul_et_al2009#Pauletal2009, MT Summit XII technology showcase paper about ''PortageLiveManual!PortageLive.
* Building PORTAGE_sharedTrainingLanguageModels!_TPLM_s#TheTPLMformat with `arpalm2tplm.sh`.
* Building PORTAGE_sharedTrainingOtherModels!_TPPT_s#TheTPPTformat with `textpt2tppt.sh`.
* Building PORTAGE_sharedTrainingOtherModels!_TPLDM_s#LexicalizedDistortionModels with `textldm2tpldm.sh`.

----

Up: PortageMachineTranslation!_PortageII / PORTAGE_sharedTrainingModels!_Models
Previous: PORTAGE_sharedTrainingOtherModels!_TrainingOtherModels
Next: PORTAGE_sharedTrainingOptimizingWeights!_OptimizingWeights