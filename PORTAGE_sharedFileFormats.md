Up: [PortageII](PortageMachineTranslation.md)
Previous: TMXProcessing
Next: [WordAlignmentFormats](PORTAGE_sharedWordAlignmentFormats.md)

-------------------------

## Text File Formats

* [PlainText#PlainText](PORTAGE_sharedFileFormats.md)
* [TokenizedText#TokenizedText](PORTAGE_sharedFileFormats.md)
* [MarkedUpText#MarkedUpText](PORTAGE_sharedFileFormats.md)
* [AlignedText#AlignedText](PORTAGE_sharedFileFormats.md)

### Plain Text

This is just normal text, as in a plain text file without special control characters. It may include some simple markup required by the tokenization and alignment programs, as described in [TextProcessing.](PORTAGE_sharedTextProcessing.md) For encoding information, see [EncodingConsiderations#EncodingConsiderations](PORTAGE_sharedTextProcessing.md) (summary: use utf-8).

Example:
  On December 23rd, 2004, it was warm and rainy. The next 
  day--Christmas Eve--it was -10 c. again.

### Tokenized Text

In tokenized text, each token is delimited by whitespace and each sentence is on a single line. Example:
  On December 23rd , 2004 , it was warm and rainy . 
  The next day -- Christmas Eve -- it was -10 c. again .

### Marked Up Text

This is tokenized text in which some entities, like numbers and dates, are explicitly identified using SGML-like markup. It is used for input to the decoder. Example:
  On <DATE std="23/12/2004">December 23rd , 2004</DATE> , it was warm and rainy . 
  The next day -- Christmas Eve -- it was <NUM std="-10">-10</NUM> c. again .

Markup can also specify rule-based translations for arbitrary entities, as in this example:
   <NUM target="deux millions|2 millions" prob=".9|.1">two million</NUM>

Rules of the mark-up language:
* \, < and > must be escaped using \ to be interpreted literally (in several unambiguous circumstances, they will be interpreted literally anyway, with a warning).
* The < in the open and close tags may not be followed by a space (a stand-alone < will be interpreted literally).
* The tag may be called whatever you want, but must have a matching closing tag.
* In the open tag, "target" and "english" are synonyms and provide the translations, surrounded by " and separated by |.  Literal ", |, <, > and \ must be escaped with \.  "prob" is optional, and can be used to provide their respective probabilities.  "prob" must come last if provided, no other properties are allowed.

### Aligned Text

Aligned text is text in which sentences in two translated files are put into correspondence. This is represented by synchronizing lines, so that the ith line in an English file is the translation of the ith line in the corresponding French file. Two aligned files therefore always have the same number of lines. Note that alignment can necessitate putting more than a single sentence on a line, removing explicit sentence-boundary information.

-------------------------

Up: [PortageII](PortageMachineTranslation.md)
Previous: TMXProcessing
Next: [WordAlignmentFormats](PORTAGE_sharedWordAlignmentFormats.md)