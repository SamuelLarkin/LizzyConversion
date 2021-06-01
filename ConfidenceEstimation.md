Up: [PortageII](PortageMachineTranslation.md)
Previous: [Evaluation](PORTAGE_sharedEvaluation.md)
Next: [Performance](PORTAGE_sharedPerformanceConsiderations.md)

---------

PORTAGE shared includes a sentence-level confidence estimation (CE) layer.  The purpose of CE in this context is to produce a value -- typically a real-valued number between 0 and 1 -- that reflects the translation system's belief that it has done a Good Job. This number can then be used for various things, for example filtering out lower-quality or dubious machine translations, prioritizing post-editing effort, etc.

The user manual for the CE module has not been converted to the format of this manual.  It can be found in a plain text file in `src/confidence/README` (distributions with source code) and `doc/confidence/README` (all distributions).

---------

Up: [PortageII](PortageMachineTranslation.md)
Previous: [Evaluation](PORTAGE_sharedEvaluation.md)
Next: [Performance](PORTAGE_sharedPerformanceConsiderations.md)
