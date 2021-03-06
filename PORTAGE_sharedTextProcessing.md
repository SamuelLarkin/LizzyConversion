Up [PortageII](PortageMachineTranslation.md)
Previous: [WhereToFindThings](PORTAGE_sharedWhereToFindThings.md)
Down: TMXProcessing

-------------------------

'''Note:''' Except for the extraction of text from TMX files and sentence alignment, all the steps described here are automated in our experimental framework. Please read `tutorial.pdf` in `framework` for details.

## Text Processing

Text processing is necessary in order to turn raw source and target text into a format suitable for model training, and also to prepare new source text for translation. Here are the steps:

* [Preprocessing#Preprocessing](PORTAGE_sharedTextProcessing.md) raw corpora
** Alternative: [Extracting_text_from_a_TMX](TMXProcessing.md)
* [Tokenizing#Tokenizing](PORTAGE_sharedTextProcessing.md) plain text
* [Aligning#Aligning](PORTAGE_sharedTextProcessing.md) tokenized text
* [Lowercasing#Lowercasing](PORTAGE_sharedTextProcessing.md) tokenized text
* [MarkingUp#MarkingUp](PORTAGE_sharedTextProcessing.md) tokenized text
* [EncodingConsiderations#EncodingConsiderations](PORTAGE_sharedTextProcessing.md)

The first two steps must be performed in this order, but the order of later steps can vary depending on the application. The [FileFormats](PORTAGE_sharedFileFormats.md) section describes the input and output formats for all steps.

### Preprocessing

This is a cleaning step to convert raw text into a normalized
[PlainText#PlainText](PORTAGE_sharedFileFormats.md) format that later steps can handle. Preprocessing tends to be specific to each corpus, so if you need to preprocess a new corpus you will probably have to write your own program, but you can use the ones below as examples.

The first and most important preprocessing step is to clean up UTF-8 text of many source of noise that are common. We wrote a script to do so: `clean-utf8-text.pl`, which we call automatically in our [tmx_prepro](TMXProcessing.md) module and in the default `preprocess_plugin` used by Portage''''''Live.
|   clean-utf8-text.pl < in.raw > in.clean
will remove control characters and normalize spaces and hyphens for better processing by the rest of the PORTAGE shared suite.

Some older sample preprocessing scripts are still available, for extracting text from very specific formats we've dealt with in the past: `prep-hans.pl`, `prep-hans-html.pl`, `prep-and-tok-europarl.pl`. They can be used as examples to write scripts adapted to your own data, but they are very old and handle only latin-1 encoded text, while we now require utf-8.

If your corpus is stored in a translation memory of some kind, export it to the TMX format and refer to TMXProcessing for extracting [PlainText#PlainText](PORTAGE_sharedFileFormats.md) from your TMX.

If your corpus exists as a complex collection of documents in various formats, you might consider acquiring commercial alignment tools, many of which can create translation memories from a large number of document formats, and then export them in the TMX format. This is by far the easiest way to build your training corpus for PORTAGE shared.

### Tokenizing

Tokenizing converts [PlainText#PlainText](PORTAGE_sharedFileFormats.md) into 
[TokenizedText#TokenizedText](PORTAGE_sharedFileFormats.md) by splitting running text into whitespace-separated tokens, and identifying sentence boundaries. For English and French, this mostly means separating punctuation from words.
Example (English/French encoded in utf-8):
|   utokenize.pl -lang=en hans-file_en.txt hans-file_en.tok
|   utokenize.pl -lang=fr hans-file_fr.txt hans-file_fr.tok

The tokenizer knows English, French, Spanish and Danish, but it might do an acceptable job in some other European languages where tokenization is sufficiently similar, especially if separating the punctuation marks is all that is required.

Similarly, our detokenizer (`udetokenize.pl`) knows English, French, Spanish and Danish.

As mentioned above, the tokenizer can do sentence boundary identification. It has several modes to accommodate common input types: use `-ss -paraline` for one-paragraph-per-line (i.e., the newline character marks the paragraph break, as is most common); use `-ss` alone for wrapped text, where the newline is just a space and paragraphs are separated by a blank line. If your data is already split into sentences, use the `-noss` option to turn off sentence splitting and preserve all existing newlines.

### Aligning

Given sentence-split, [TokenizedText#TokenizedText](PORTAGE_sharedFileFormats.md) (possibly containing markup) in two languages, sentence alignment identifies correspondences between translated sentences, and writes the results as [AlignedText#AlignedText.](PORTAGE_sharedFileFormats.md)

Example using Moore's aligner (not supplied with PORTAGE shared):
   moore-aligner.pl dir-containing-tokenized-files

Example (English/French in any encoding) using our simple sentence aligner:
|   ssal hans_fr.tok hans_en.tok

Our simple sentence aligner, `ssal`, can run in two modes: 1) using the plain Gale+Church length-based model, or 2) using IBM word alignment models.  The first mode is fastest and works well on clean data.  Using IBM models involves working in two passes, but can increase the accuracy of the alignment.  See `ssal -H` for details on using the multi-pass approach with IBM models.

Moore's aligner also uses a two-pass method based on IBM1 word alignment models.  It tends to yield high precision, but will drop part of the input corpus.

Which aligner works best for you will depend on your data. `ssal` is written in such as way as to be extensible, and indeed, we extended it a few years ago to support IBM models (see above) and hard boundaries (see below).

If you use `ssal`, it can be beneficial to leave some matching markup in these files during preprocessing, to identify section boundaries and other important structures. `ssal` will consider identical lines (including markup) as hints for alignment. This markup should be enclosed in angle brackets, like the markup codes `<sect>`, `<time 1400>`, etc that appear in the `hans*.txt` files. With the `-f` switch, the markup is filtered out during alignment so it will not pollute your output.  You can go one step further and specify a hard markup via the `-hm` option.  The hard markup is a non-crossable boundary; the effect is equivalent to having the text on either side in different files. Hard markup can be especially useful in a hierarchical alignment strategy for large documents: align sections first, then paragraphs, then sentences.

See `ssal -h` and `ssal -H` for more information.

### Lowercasing

Applied to tokenized text, e.g., to avoid treating ''the'' and ''The'' as different words.  This is usually done in order to reduce data sparseness.

Example for utf-8 data:
|   lc-utf8.pl file.tok file.lc
or, if you have ICU:
|   utf8_casemap file.tok file.lc

### Marking Up

Various entities in the text can be encoded as [MarkedUpText#MarkedUpText.](Portage_sharedFileFormats.md) There are three uses for markup in the system:

* To encode parallel structure (section boundaries, timestamps, etc) in order to help the alignment program (ssal). This markup is typically removed by the aligner when it writes its output.

* To specify translations for invariants like numbers, dates, etc. 
During decoding, translations listed in the markup can be used to augment automatically-learned translations.

PORTAGE shared includes software to use such markup to reliably translate numbers between French and English using a rule-based approach, but they can be used for many other purposes too. Our default `predecode_plugin` for use with Portage''''''Live uses marked-up text to encode the result of rule-based translation of numbers between English and French (in either direction) and to process fixed terms, if they are used. (See PortageLiveCustomizationPlugins for details.)

### Encoding Considerations

Use utf-8. Period.

For years now, we have been using utf-8 exclusively, so many scripts in PORTAGE shared now assume it, including `translate.pl` and the Portage''''''Live API.

Use `iconv` to convert text from other encodings to utf-8.

The main exception is TMX files: the standard says they can be encoded in either utf-16 or utf-8, so we automatically accept either and convert them to utf-8 as needed.

However, it is still noteworthy that except for pre- and post-processing (such as text clean up, lowercasing, tokenizing, detokenizing), the core programs in PORTAGE shared treat the data as a byte stream, and are encoding agnostic. They care only that the space, tab and newline characters be encoded using their single-byte ASCII representation. Thus, these core programs support all iso-latin-N encodings, cp-1252, utf-8, the GB encodings for Chinese, but not utf-16/ucs-2 nor utf-32/ucs-4.


-------------------------

Up [PortageII](PortageMachineTranslation.md)
Previous: [WhereToFindThings](PORTAGE_sharedWhereToFindThings.md)
Down: TMXProcessing