Up [PortageII](PortageMachineTranslation.md) / [Translating](PORTAGE_sharedTranslating.md)
Previous: [Decoding](PORTAGE_sharedTranslatingDecoding.md)
Next: [Postprocessing](PORTAGE_sharedTranslatingPostprocessing.md)

-------------------------

## Translating: Rescoring Nbest Lists

The quality of raw output from canoe can be improved by having it generate nbest lists and reordering them using a rescoring model.

#### Rescoring Script

Like for training a rescoring model, the `rescore.py` script can be used to run all steps in rescored translation: nbest-list generation, feature-file generation, and nbest rescoring to select the best candidates. Assuming a rescoring model, `rescore-model.rat`, has been trained using the procedure described in the [RescoringScript#RescoringScript](PORTAGE_sharedTrainingOptimizingWeights.md) section, the following command will write a final translation to the file `text_fr.rule.rat`:
|   rescore.py -trans --nbest-size 1000 -f canoe.ini.cow -msrc text_fr.rule \
|      rescore-model.rat text_fr.tok text_en*.al >& log.rat-trans

Note that the nbest list size (1000 in this example) should match the size used for training, although this is not obligatory. Also, the reference files `text_en*.al` are optional; when they are provided, rat will write the BLEU score for the final rescored translation to its standard output.


-------------------

Up [PortageII](PortageMachineTranslation.md) / [Translating](PORTAGE_sharedTranslating.md)
Previous: [Decoding](PORTAGE_sharedTranslatingDecoding.md)
Next: [Postprocessing](PORTAGE_sharedTranslatingPostprocessing.md)