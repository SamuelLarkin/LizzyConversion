Up: [PortageII](PortageMachineTranslation.md)
Previous: [AnnotatedBibliography](PORTAGE_sharedAnnotatedBibliography.md)
Next: [FrequentlyAskedQuestions](PORTAGE_sharedFAQ.md)

-------------------------

'''Warning''': this page is out of date, as the statistics were gathered years ago for Portage 1.4.2. Since then, machines are much faster, but PORTAGE shared is more complex. These statistics are still a potentially useful indication, but not necessarily reliable.

##Training a mid-size system (what kind of machine do I need)
You can train a mid-size system using the corpora available from http://www.statmt.org/wmt09/translation-task.html.
More precisely, download the following:
* http://www.statmt.org/wmt09/training-parallel.tar
* http://www.statmt.org/wmt09/training-monolingual.tar
* http://www.statmt.org/wmt09/dev.tgz
* http://www.statmt.org/wmt09/additional-dev.tgz
* http://www.statmt.org/wmt09/test.tgz

Rename your file appropriately and place them under corpora/ in the framework.  Here are the size of each file.
| #Lines      #Words        #Char         shortest line   longest line   mean      sdev         filename
| L:2000      W:53088       C:317356      s:1             l:107          m:26.54   sdev:15.13   dev2006_en.raw
| L:2000      W:55088       C:357257      s:1             l:114          m:27.54   sdev:15.82   dev2006_fr.raw
| L:2000      W:52388       C:312927      s:1             l:119          m:26.19   sdev:15.07   devtest2006_en.raw
| L:2000      W:54149       C:352709      s:1             l:113          m:27.07   sdev:15.45   devtest2006_fr.raw
| L:2000      W:53627       C:320432      s:1             l:119          m:26.81   sdev:15.49   test2006_en.raw
| L:2000      W:55764       C:361702      s:1             l:116          m:27.88   sdev:16.05   test2006_fr.raw
| L:2000      W:53535       C:318972      s:1             l:112          m:26.77   sdev:15.38   test2007_en.raw
| L:2000      W:55157       C:358557      s:1             l:114          m:27.58   sdev:15.73   test2007_fr.raw
| L:2000      W:54301       C:324417      s:1             l:125          m:27.15   sdev:15.77   test2008_en.raw
| L:2000      W:56217       C:365342      s:1             l:129          m:28.11   sdev:16.66   test2008_fr.raw
| L:1658841   W:40624075    C:243409294   s:1             l:668          m:24.49   sdev:14.80   europarl-v4_en.raw.gz
| L:1428799   W:36198774    C:216718721   s:0             l:407          m:25.34   sdev:15.25   europarl-v4.fr-en_en.raw.gz
| L:1428799   W:37649967    C:249125357   s:0             l:465          m:26.35   sdev:15.96   europarl-v4.fr-en_fr.raw.gz
| L:1676435   W:42618754    C:281943473   s:1             l:465          m:25.42   sdev:15.45   europarl-v4_fr.raw.gz

| L:6212874   W:157634884   C:994586516   s:0             l:668          m:25.37   sdev:15.37   TOTAL

## Without rescoring and Lexicalized Distortion Model
### Changes to your Makefile.params
Configure you `Makefile.params` located at the root of the framework as follow:
|SRC_LANG ?= en
|TGT_LANG ?= fr
 
|TRAIN_LM      ?= europarl-v4
|TRAIN_TM      ?= europarl-v4.fr-en
|TUNE_DECODE   ?= dev2006
|TUNE_RESCORE  ?= devtest2006
|TUNE_CE       ?= test2006
|TEST_SET      ?= test2008
 
| LM_TOOLKIT = SRI
| #DO_RESCORING = 1
| DO_CE = 1
| DO_TRUECASING = 1
| ICU = 1
| #USE_LDM =

### BLEU score
You should obtain a `BLEU` similar to this:

| Human readable value: 35.28 +/- 0.96

### Resources Required
You can find out how much resources that was required by running the following command from the root of the framework:
| make summary

| Portage 1.4.2, NRC-CNRC, (c) 2004 - 2010, Her Majesty in Right of Canada
| Please run "portage_info -notice" for Copyright notices of 3rd party libraries.
|
| Running in cluster mode.
|
|      log.europarl-v4_fr-kn-5g.lm:TIME-MEM                  WALL TIME: 4m34s       CPU TIME: 4m2s          VSZ: 3.508G    RSS: 3.457G
|      log.europarl-v4_en-kn-5g.lm:TIME-MEM                  WALL TIME: 6m19s       CPU TIME: 5m53s         VSZ: 3.362G    RSS: 3.312G
|      log.europarl-v4_en-kn-5g.binlm:TIME-MEM               WALL TIME: 6m47s       CPU TIME: 5m38s         VSZ: 0.364G    RSS: 0.311G
|      log.europarl-v4_fr-kn-5g.binlm:TIME-MEM               WALL TIME: 4m27s       CPU TIME: 4m24s         VSZ: 0.404G    RSS: 0.351G
|   lm:TIME-MEM                                              WALL TIME: 22m7s       CPU TIME: 19m57s        VSZ: 3.508G    RSS: 3.457G
|      log.jpt.ibm2.europarl-v4.fr-en.en-fr:TIME-MEM         WALL TIME: 9m31s       CPU TIME: 37m39s        VSZ: 1.827G    RSS: 1.580G
|      log.jpt.hmm3.europarl-v4.fr-en.en-fr:TIME-MEM         WALL TIME: 34m1s       CPU TIME: 1h58m0s       VSZ: 5.659G    RSS: 4.951G
|      log.hmm3.europarl-v4.fr-en.en_given_fr:TIME-MEM       WALL TIME: 2h10m47s    CPU TIME: 12h30m10s     VSZ: 0.507G    RSS: 0.271G
|      log.cpt.hmm3-rf-zn.europarl-v4.fr-en.en2fr:TIME-MEM   WALL TIME: 2h26m57s    CPU TIME: 2h35m9s       VSZ: 10.141G   RSS: 8.819G
|      log.ibm1.europarl-v4.fr-en.en_given_fr:TIME-MEM       WALL TIME: 25m3s       CPU TIME: 1h35m26s      VSZ: 1.048G    RSS: 0.812G
|      log.ibm1.europarl-v4.fr-en.fr_given_en:TIME-MEM       WALL TIME: 22m24s      CPU TIME: 1h31m30s      VSZ: 1.056G    RSS: 0.812G
|      log.hmm3.europarl-v4.fr-en.fr_given_en:TIME-MEM       WALL TIME: 2h33m31s    CPU TIME: 13h59m34s     VSZ: 0.501G    RSS: 0.265G
|      log.ibm2.europarl-v4.fr-en.fr_given_en:TIME-MEM       WALL TIME: 14m32s      CPU TIME: 1h8m19s       VSZ: 0.496G    RSS: 0.261G
|      log.ibm2.europarl-v4.fr-en.en_given_fr:TIME-MEM       WALL TIME: 13m35s      CPU TIME: 1h8m15s       VSZ: 0.564G    RSS: 0.266G
|      log.cpt.ibm2-rf-zn.europarl-v4.fr-en.en2fr:TIME-MEM   WALL TIME: 16m47s      CPU TIME: 19m25s        VSZ: 2.901G    RSS: 2.531G
|   tm:TIME-MEM                                              WALL TIME: 9h27m8s     CPU TIME: 1d13h23m25s   VSZ: 10.141G   RSS: 8.819G
|      log.europarl-v4_fr-kn-3g.binlm:TIME-MEM               WALL TIME: 1m7s        CPU TIME: 1m3s          VSZ: 0.175G    RSS: 0.121G
|      log.europarl-v4_fr.map:TIME-MEM                       WALL TIME: 55s         CPU TIME: 40s           VSZ: 0.086G    RSS: 0.043G
|      log.europarl-v4_fr-kn-3g.lm:TIME-MEM                  WALL TIME: 1m31s       CPU TIME: 1m42s         VSZ: 0.746G    RSS: 0.698G
|   tc:TIME-MEM                                              WALL TIME: 3m33s       CPU TIME: 3m26s         VSZ: 0.746G    RSS: 0.698G
|      log.canoe.ini.cow:TIME-MEM                            WALL TIME: 4h26m30s    CPU TIME: 12h44m46s     VSZ: 1.659G    RSS: 1.547G
|   decode:TIME-MEM                                          WALL TIME: 4h26m30s    CPU TIME: 12h44m46s     VSZ: 1.659G    RSS: 1.547G
|      log.ce_model.cem:TIME-MEM                             WALL TIME: 12m28s      CPU TIME: 2h59m13s      VSZ: 1.031G    RSS: 0.758G
|   confidence:TIME-MEM                                      WALL TIME: 12m28s      CPU TIME: 2h59m13s      VSZ: 1.031G    RSS: 0.758G
| models:TIME-MEM                                            WALL TIME: 14h31m46s   CPU TIME: 2d5h30m46s    VSZ: 10.141G   RSS: 8.819G
|   ./log.test2008.ce:TIME-MEM                               WALL TIME: 17m55s      CPU TIME: 3h17m38s      VSZ: 1.369G    RSS: 1.064G
|   ./log.test2008.out:TIME-MEM                              WALL TIME: 17m4s       CPU TIME: 3h1m56s       VSZ: 1.287G    RSS: 1.195G
|   ./log.test2008.out.tc:TIME-MEM                           WALL TIME: 27s         CPU TIME: 4s            VSZ: 0.047G    RSS: 0.009G
| translate:TIME-MEM                                         WALL TIME: 35m26s      CPU TIME: 6h19m39s      VSZ: 1.369G    RSS: 1.195G

| 686M    corpora
| 96K     models/confidence/ce_model.cem
| 40K     models/ldm
| 97M     models/lm/europarl-v4_en-kn-5g.binlm.gz
| 112M    models/lm/europarl-v4_en-kn-5g.lm.gz
| 108M    models/lm/europarl-v4_fr-kn-5g.binlm.gz
| 123M    models/lm/europarl-v4_fr-kn-5g.lm.gz
| 108M    models/tm/ibm1.europarl-v4.fr-en.en_given_fr.gz
| 106M    models/tm/ibm1.europarl-v4.fr-en.fr_given_en.gz
| 85M     models/tm/ibm2.europarl-v4.fr-en.en_given_fr.gz
| 176K    models/tm/ibm2.europarl-v4.fr-en.en_given_fr.pos.gz
| 88M     models/tm/ibm2.europarl-v4.fr-en.fr_given_en.gz
| 176K    models/tm/ibm2.europarl-v4.fr-en.fr_given_en.pos.gz
| 4.0K    models/tm/hmm3.europarl-v4.fr-en.en_given_fr.dist.gz
| 50M     models/tm/hmm3.europarl-v4.fr-en.en_given_fr.gz
| 4.0K    models/tm/hmm3.europarl-v4.fr-en.fr_given_en.dist.gz
| 52M     models/tm/hmm3.europarl-v4.fr-en.fr_given_en.gz
| 1.1G    models/tm/jpt.hmm3.europarl-v4.fr-en.en-fr.gz
| 264M    models/tm/jpt.ibm2.europarl-v4.fr-en.en-fr.gz
| 3.6G    models/tm/cpt.hmm3-rf-zn.europarl-v4.fr-en.en2fr.gz
| 676M    models/tm/cpt.ibm2-rf-zn.europarl-v4.fr-en.en2fr.gz
| 101M    models/tc
| 2.8M    translate
| 7.3G    total

From the previous table, we see that, to train a system using Europarl's data we need 10.141G or more of memory.  It took 14 hours 31 minutes and 46 seconds on our multicore machine.  It would have taken close to 2 days 5 hours 30 minutes and 46 seconds to train the same system if we only had one core.  This mid-size system requires at least 7.3G of disk space.


## With rescoring and Lexicalized Distortion Model
### Changes to your Makefile.params
Configure you `Makefile.params` located at the root of the framework as follow:
|SRC_LANG ?= en
|TGT_LANG ?= fr
 
|TRAIN_LM      ?= europarl-v4
|TRAIN_TM      ?= europarl-v4.fr-en
|TUNE_DECODE   ?= dev2006
|TUNE_RESCORE  ?= devtest2006
|TUNE_CE       ?= test2006
|TEST_SET      ?= test2008
 
| LM_TOOLKIT = SRI
| DO_RESCORING = 1
| DO_CE = 1
| DO_TRUECASING = 1
| ICU = 1
| USE_LDM = 1

### BLEU score
You should obtain a `BLEU` similar to this:

| test2008.out.bleu:Human readable value: 34.87 +/- 0.94
| test2008.rat.bleu:Human readable value: 34.82 +/- 0.94

### Resources Required
You can find out how much resources that was required by running the following command from the root of the framework:
| make summary

| Portage 1.4.2, NRC-CNRC, (c) 2004 - 2010, Her Majesty in Right of Canada
| Please run "portage_info -notice" for Copyright notices of 3rd party libraries.
|
| Running in cluster mode.
|
| Resource summary for /home/larkins/exp/uqam:
|          log.ce_model.cem:TIME-MEM                             WALL TIME: 22m21s      CPU TIME: 5h24m14s      VSZ: 1.003G    RSS: 0.698G
|       confidence:TIME-MEM                                      WALL TIME: 22m21s      CPU TIME: 5h24m14s      VSZ: 1.003G    RSS: 0.698G
|          log.canoe.ini.cow:TIME-MEM                            WALL TIME: 6h41m15s    CPU TIME: 1d1h19m3s     VSZ: 2.226G    RSS: 2.148G
|       decode:TIME-MEM                                          WALL TIME: 6h41m15s    CPU TIME: 1d1h19m3s     VSZ: 2.226G    RSS: 2.148G
|          log.ldm.counts.hmm3:TIME-MEM                          WALL TIME: 14m48s      CPU TIME: 2h19m48s      VSZ: 2.372G    RSS: 2.215G
|          log.ldm.counts.ibm2:TIME-MEM                          WALL TIME: 3m19s       CPU TIME: 35m44s        VSZ: 1.042G    RSS: 0.845G
|          log.ldm.hmm3+ibm2.en2fr:TIME-MEM                      WALL TIME: 1h28m37s    CPU TIME: 55m8s         VSZ: 32.946G   RSS: 29.570G
|       ldm:TIME-MEM                                             WALL TIME: 1h46m44s    CPU TIME: 3h50m40s      VSZ: 32.946G   RSS: 29.570G
|          log.europarl-v4_en-kn-5g.binlm:TIME-MEM               WALL TIME: 6m47s       CPU TIME: 5m38s         VSZ: 0.364G    RSS: 0.311G
|          log.europarl-v4_en-kn-5g.lm:TIME-MEM                  WALL TIME: 6m19s       CPU TIME: 5m53s         VSZ: 3.362G    RSS: 3.312G
|          log.europarl-v4_fr-kn-5g.binlm:TIME-MEM               WALL TIME: 4m27s       CPU TIME: 4m24s         VSZ: 0.404G    RSS: 0.351G
|          log.europarl-v4_fr-kn-5g.lm:TIME-MEM                  WALL TIME: 4m34s       CPU TIME: 4m2s          VSZ: 3.508G    RSS: 3.457G
|       lm:TIME-MEM                                              WALL TIME: 22m7s       CPU TIME: 19m57s        VSZ: 3.508G    RSS: 3.457G
|          log.rescore-model:TIME-MEM                            WALL TIME: 1h19m22s    CPU TIME: 5h43m7s       VSZ: 1.633G    RSS: 1.513G
|       rescore:TIME-MEM                                         WALL TIME: 1h19m22s    CPU TIME: 5h43m7s       VSZ: 1.633G    RSS: 1.513G
|          log.europarl-v4_fr-kn-3g.binlm:TIME-MEM               WALL TIME: 1m7s        CPU TIME: 1m3s          VSZ: 0.175G    RSS: 0.121G
|          log.europarl-v4_fr-kn-3g.lm:TIME-MEM                  WALL TIME: 1m31s       CPU TIME: 1m42s         VSZ: 0.746G    RSS: 0.698G
|          log.europarl-v4_fr.map:TIME-MEM                       WALL TIME: 55s         CPU TIME: 40s           VSZ: 0.086G    RSS: 0.043G
|       tc:TIME-MEM                                              WALL TIME: 3m33s       CPU TIME: 3m26s         VSZ: 0.746G    RSS: 0.698G
|          log.cpt.hmm3-rf-zn.europarl-v4.fr-en.en2fr:TIME-MEM   WALL TIME: 2h26m57s    CPU TIME: 2h35m9s       VSZ: 10.141G   RSS: 8.819G
|          log.cpt.ibm2-rf-zn.europarl-v4.fr-en.en2fr:TIME-MEM   WALL TIME: 16m47s      CPU TIME: 19m25s        VSZ: 2.901G    RSS: 2.531G
|          log.hmm3.europarl-v4.fr-en.en_given_fr:TIME-MEM       WALL TIME: 2h10m47s    CPU TIME: 12h30m10s     VSZ: 0.507G    RSS: 0.271G
|          log.hmm3.europarl-v4.fr-en.fr_given_en:TIME-MEM       WALL TIME: 2h33m31s    CPU TIME: 13h59m34s     VSZ: 0.501G    RSS: 0.265G
|          log.ibm1.europarl-v4.fr-en.en_given_fr:TIME-MEM       WALL TIME: 25m3s       CPU TIME: 1h35m26s      VSZ: 1.048G    RSS: 0.812G
|          log.ibm1.europarl-v4.fr-en.fr_given_en:TIME-MEM       WALL TIME: 22m24s      CPU TIME: 1h31m30s      VSZ: 1.056G    RSS: 0.812G
|          log.ibm2.europarl-v4.fr-en.en_given_fr:TIME-MEM       WALL TIME: 13m35s      CPU TIME: 1h8m15s       VSZ: 0.564G    RSS: 0.266G
|          log.ibm2.europarl-v4.fr-en.fr_given_en:TIME-MEM       WALL TIME: 14m32s      CPU TIME: 1h8m19s       VSZ: 0.496G    RSS: 0.261G
|          log.jpt.hmm3.europarl-v4.fr-en.en-fr:TIME-MEM         WALL TIME: 34m1s       CPU TIME: 1h58m0s       VSZ: 5.659G    RSS: 4.951G
|          log.jpt.ibm2.europarl-v4.fr-en.en-fr:TIME-MEM         WALL TIME: 9m31s       CPU TIME: 37m39s        VSZ: 1.827G    RSS: 1.580G
|       tm:TIME-MEM                                              WALL TIME: 9h27m8s     CPU TIME: 1d13h23m25s   VSZ: 10.141G   RSS: 8.819G
|    models:TIME-MEM                                             WALL TIME: 20h2m30s    CPU TIME: 3d6h3m52s     VSZ: 32.946G   RSS: 29.570G
|       log.test2008.ce:TIME-MEM                                 WALL TIME: 39m29s      CPU TIME: 6h30m43s      VSZ: 1.097G    RSS: 0.797G
|       log.test2008.rat:TIME-MEM                                WALL TIME: 1h2m17s     CPU TIME: 6h18m17s      VSZ: 1.672G    RSS: 1.552G
|       log.test2008.rat.tc:TIME-MEM                             WALL TIME: 12s         CPU TIME: 5s            VSZ: 0.048G    RSS: 0.009G
|    translate:TIME-MEM                                          WALL TIME: 1h41m58s    CPU TIME: 12h49m5s      VSZ: 1.672G    RSS: 1.552G
| TIME-MEM                                                       WALL TIME: 21h44m28s   CPU TIME: 3d18h52m57s   VSZ: 32.946G   RSS: 29.570G
|

| Disk usage for all models:
| 96K   models/confidence/ce_model.cem
| 4.9G  models/ldm
| 97M   models/lm/europarl-v4_en-kn-5g.binlm.gz
| 112M  models/lm/europarl-v4_en-kn-5g.lm.gz
| 108M  models/lm/europarl-v4_fr-kn-5g.binlm.gz
| 123M  models/lm/europarl-v4_fr-kn-5g.lm.gz
| 108M  models/tm/ibm1.europarl-v4.fr-en.en_given_fr.gz
| 106M  models/tm/ibm1.europarl-v4.fr-en.fr_given_en.gz
| 85M   models/tm/ibm2.europarl-v4.fr-en.en_given_fr.gz
| 176K  models/tm/ibm2.europarl-v4.fr-en.en_given_fr.pos.gz
| 88M   models/tm/ibm2.europarl-v4.fr-en.fr_given_en.gz
| 176K  models/tm/ibm2.europarl-v4.fr-en.fr_given_en.pos.gz
| 4.0K  models/tm/hmm3.europarl-v4.fr-en.en_given_fr.dist.gz
| 50M   models/tm/hmm3.europarl-v4.fr-en.en_given_fr.gz
| 4.0K  models/tm/hmm3.europarl-v4.fr-en.fr_given_en.dist.gz
| 52M   models/tm/hmm3.europarl-v4.fr-en.fr_given_en.gz
| 1.1G  models/tm/jpt.hmm3.europarl-v4.fr-en.en-fr.gz
| 264M  models/tm/jpt.ibm2.europarl-v4.fr-en.en-fr.gz
| 3.6G  models/tm/cpt.hmm3-rf-zn.europarl-v4.fr-en.en2fr.gz
| 676M  models/tm/cpt.ibm2-rf-zn.europarl-v4.fr-en.en2fr.gz
| 101M  models/tc
| 267M  translate
| 12G   total
|

--------------

Up: [PortageII](PortageMachineTranslation.md)
Previous: [AnnotatedBibliography](PORTAGE_sharedAnnotatedBibliography.md)
Next: [FrequentlyAskedQuestions](PORTAGE_sharedFAQ.md)