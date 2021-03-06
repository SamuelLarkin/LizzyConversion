Up: [PortageII](PortageMachineTranslation.md)
/ [TextProcessing](PORTAGE_sharedTextProcessing.md)
Next: [TextFileFormats](PORTAGE_sharedFileFormats.md)

-------------------------

## The tmx-prepro module

The steps described here, as well as the creation of train, dev and test files needed for training SMT systems using PORTAGE shared, are automated in the `tmx-prepro` module that you can find in the PORTAGE shared distribution.

This page of the user manual can help you understand what that module does.

## The TMX file format

This section describes how to process a TMX file to extract corpora in a format usable by PORTAGE shared for training or testing.  TMX stands for Translation Memory Xchange format; it is the standard format for exporting data from a translation memory: see [GALAIndustryStandards.](https://www.gala-global.org/resources/industry-standards.md) Along with a fair amount of mark-up information, a TMX file contains sentences in one language (the source language) in parallel with their translations in one or more other language(s) (the target language(s)).

By contrast, PORTAGE shared requires all source-language sentences to be in one file and all target-language sentences to be in another.  Only one target language is allowed.  The two files must be line-aligned, with one  sentence or segment per line, preferably in UTF-8 (UTF-16 is not supported).

Once you've extracted a parallel corpus from your TMX file, see [TextProcessing](PORTAGE_sharedTextProcessing.md) for the rest of the processing pipeline.

## Extracting your parallel corpora

### Convert your TMX to UTF-8

If your TMX in is UTF-16 encoding, first convert it to UTF-8 so that our tools can read it:
| iconv -c -f UTF-16 -t UTF-8 < TransMem.tmx.utf16 > TransMem.tmx

### Perform extraction
To extract your corpus (in UTF-8 encoding) from your TMX, say `TransMem.tmx`, which contains segments in, say, EN-US & FR-CA, do the following:

| tmx2lfl.pl -verbose -output=output_prefix -txt=.txt TransMem.tmx
where:
# -output is your output file prefix;
# -txt, if specified, triggers outputting a text-only parallel corpus;
# `TransMem.tmx` is your TMX file or files.

If you used the "-txt" option as shown above, you will get these five files as your output:
| output_prefix.EN-US
| output_prefix.EN-US.txt
| output_prefix.FR-CA
| output_prefix.FR-CA.txt
| output_prefix.id

If you didn't use the "-txt" option, you will only get the three files without the ".txt" extension as output.

Here, the sentences in the two "EN-US" files are the sentences in the "EN-US" language found in the TMX file, and similarly for the "FR-CA" files. These files are line aligned, so that the n-th line in `output_prefix.EN-CA.txt` is a translation of the n-th line in `output_prefix.FR-CA.txt` (or vice versa). The only difference between the "*.txt" files and their counterparts without the ".txt" extension is that the former have been stripped of almost all markup. "`output_prefix.id`" records the `IDs` for documents from which the sentences were extracted, as identified by the `Txt::Document` property of each translation unit in the TMX.

## Extracting Specific Language Pairs
### Find the src/tgt language identifiers

If you have segments for more than two languages in your TMX, you will need to specify which two you want to extract.

To find the language identifiers:

If the input TMX file is in UTF-16, then do: 
| iconv -f UTF-16 -t UTF-8 TransMem.tmx | grep -m10 'xml:lang' | sort | uniq
Or, if it is in UTF-8 or ascii, do: 
| grep -m10 'xml:lang' | sort | uniq
You should get two or more lines like these (note down the two language tags you want to keep, you will need them to extract your corpus from your tmx): 
| <tuv xml:lang="EN-US">
| <tuv xml:lang="FR-CA">

Performing the extraction is the same as previously but you will need to specify which source and target identifiers to use.  (For this program, there is no special semantics to "source" and "target", you can arbitrarily assign either language to either option.)

| tmx2lfl.pl -verbose -output=output_prefix -txt=.txt -src=EN-US -tgt=FR-CA TransMem.tmx
where:
# -output is your output file prefix;
# -src specifies your source language identifier;
# -tgt specifies your target language identifier;
# -txt, if specified, triggers outputting a text-only parallel corpus;
# `TransMem.tmx` is your TMX file or files.

The output is as described above.

## Dealing with messy output

The TMX format is not particularly strict with the actual contents of the translation units it contains.  The application that was used to create the TMX can store anything it wants, including XML code, rich-text format or whatever they use for their internal representation of text.  If the output is not clean text, you'll have to write post-processing scripts yourself to clean it up.

## What if my TMX has problems?
### TMX Specification

The TMX specification can be found here:
* TMX Specification: https://www.gala-global.org/tmx-14b
* Official TMX page: https://www.gala-global.org/resources/industry-standards

|   TMX is XML-compliant.  [...]  However, a "valid" TMX file must
|   conform to the TMX DTD, and any suspicious TMX file should be
|   verified against the TMX DTD using a validating XML parser.
The TMX DTD can be found here: [tmx14.dtd.](https://www.gala-global.org/sites/default/files/uploads/pdfs/tmx14%20%281%29.dtd.md)

### XML Specification

Following the XML specification, your TMX should not contain non-printing characters.
[XML_Valid_Char](http://www.w3.org/TR/1998/REC-xml-19980210.html#NT-Char.md)

|   Legal characters are tab, carriage return, line feed, and the
|   legal graphic characters of Unicode and ISO/IEC 10646.
|   Character Range   /* any Unicode character, excluding the
|   surrogate blocks, FFFE, and FFFF. */
|   [2] Char ::= #x9 | #xA | #xD | [#x20-#xD7FF] | [#xE000-#xFFFD] |
|   [#x10000-#x10FFFF]

### Validating your TMX

You should validate your TMX files.  At the time of writing, the current DTD for TMX can be found at:
[tmx14.dtd](http://www.lisa.org/fileadmin/standards/tmx1.4/tmx14.dtd.txt.md)

Obtain tmx14.dtd:
| curl -o tmx14.dtd -k https://www.gala-global.org/sites/default/files/uploads/pdfs/tmx14%20%281%29.dtd

(If you don't have `curl`, try `wget -O tmx14.dtd http...` instead.)

Validate that your TMX file is "well-formed":
| xmllint --noout --valid TransMem.tmx
If xmllint returns any warnings, fix them before retrying tmx2lfl.pl.

// Explains perl's xml.
// http://perl-xml.sourceforge.net/faq/#invalid_char_num

### Cleaning a TMX with `tmx-prepro`

In the past, we have encountered problems with TMX files having embedded ASCII control characters and various problems such as the ones described above. Our `tmx-prepro` module automates replacing ASCII control characters by legal XML characters before extracting text from TMX files. It also automates cleaning up the output text using `clean-utf8-text.pl`, to get results that are more usable.


-------------------------

Up: [PortageII](PortageMachineTranslation.md)
/ [TextProcessing](PORTAGE_sharedTextProcessing.md)
Next: [TextFileFormats](PORTAGE_sharedFileFormats.md)