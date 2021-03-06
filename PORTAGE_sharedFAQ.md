Up: [PortageII](PortageMachineTranslation.md)
Previous: [RequirementsForTrainingAMidSizeSystem](TrainingEuroparl.md)

-------------------------

## Frequently Asked Questions

### How much training data do I need?

If you have less than 1M words or 100K segment pairs, don?t even bother trying! Otherwise, the more high-quality data from the same domain/client, the better.

### How much training data do I need if I?m using Generic 1.0 or 2.0 (gc.ca) as a background model?

You can probably get away with as little as 50K segment pairs, although with such small amounts of data the Generic system might work better on its own, especially if your domain is similar to something the Government of Canada is in any way involved with, since the Generic system is trained on tens of millions of segment pairs crawled from all federal web sites.

### How often should I retrain the system?

It depends. A 5 or 10 percent increase in data size will not make a significant different in machine translation quality: we typically notice a difference when we have at least 25% or more growth in the size of the training data. However, in a professional translation setting, we know that the most recent material is often the most relevant, so retraining as frequently as possible is desirable as it makes the newest material available to the system.

So our recommendation is to retrain as often as your resources allow, prioritizing the domains that have had the most new text. Once 5K new sentences have been added to a particular translation memory, it's almost certainly worth retraining a system based on that translation memory. You might end up retraining the high-volume systems weekly, with lower volume ones maybe monthly or so.

### How can I integrate terminological data?

First answer: it doesn't usually make a difference, so don't bother.

Second answer: if you really must, append your terminological database as another bitext at the end of your training corpus. This way, the system will have seen all the words in your terminology database.

But don't try to force PORTAGE shared to use your terminology systematically. Although a human can instantly see when a potential match for a term applies or does not, a machine cannot do so easily.

### Why don?t I use Google Translate instead?

This may make sense if you just want general-purpose translation. However, if you have a sufficient amount of training data from the relevant domain/client, PORTAGE shared lets you build a custom system for that domain. Another reason not to use Google Translate: your clients may not appreciate having their data floating around in the cloud ? there could be security issues.

### Why does PORTAGE shared make dumb mistakes?

Remember, PORTAGE shared is a machine with different strengths and weaknesses than human beings. If the problem is a word that?s weirdly translated, a misspelled word, or inappropriate punctuation, search for that text in the data you trained it on! The problem often comes from there. If the problem is with grammar, e.g., poor long-distance subject-verb agreement, that?s because PORTAGE shared is not good at grammar. On the other hand, it is often surprisingly good at translating idioms ? if they were in the training data.

### I find myself correcting the same error in PORTAGE shared's output over and over ? why?

If you mean that you see errors today that you fixed in the post-edited output a month ago, try to convince your managers to add newly post-edited data to the training data more often, and to retrain PORTAGE shared more often. PORTAGE shared learns from corrected data, so after retraining it should stop making many of the mistakes you corrected earlier. If you mean that you see errors now that you fixed five minutes ago, that?s a known problem with the current version of PORTAGE shared; some day, we hope to give it the ability to learn corrections "on the fly" without requiring a complete retraining of the system.

### Can I use a system trained on data from one domain/client for translating data from another domain/client?

If you have some translations from the new domain/client, there are ways of training PORTAGE shared so that it combines the new data with data from other domains to yield reasonable performance.




-------------------------

Up: [PortageII](PortageMachineTranslation.md)
Previous: [RequirementsForTrainingAMidSizeSystem](TrainingEuroparl.md)


