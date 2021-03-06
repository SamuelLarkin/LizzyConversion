Up: [PortageII](PortageMachineTranslation.md) / [ForProgrammers](PORTAGE_sharedProgrammerReference.md)
Previous: [UsingMake](PORTAGE_sharedMakeNotes.md)
Next: [AnnotatedBibliography](PORTAGE_sharedAnnotatedBibliography.md)

-------------------------

## Programming Guidelines

The guidelines presented here represent conventions we would like all programmers to follow.  These are not hard-and-fast rules, but please adhere to these as much as possible when you contribute to Portage or PORTAGE shared.  If you wish to add to these guidelines, feel free to discuss it with the other project members, and add them if there is a consensus.

* [The_PORTAGE_environment_variable#ThePORTAGEenvironmentvariable](PORTAGE_sharedProgrammingGuidelines.md)
* [C_plus_plus_conventions#Cconventions](PORTAGE_sharedProgrammingGuidelines.md)
* [Script_conventions#Scriptconventions](PORTAGE_sharedProgrammingGuidelines.md)
* [Documentation_conventions#Documentationconventions](PORTAGE_sharedProgrammingGuidelines.md)
* [Formatting_conventions#Formattingconventions](PORTAGE_sharedProgrammingGuidelines.md)
* [Unit_testing#Unittesting](PORTAGE_sharedProgrammingGuidelines.md)
* [Regression_testing#Regressiontesting](PORTAGE_sharedProgrammingGuidelines.md)
* [Git_diff#Gitdiff](PORTAGE_sharedProgrammingGuidelines.md)
* [Feature_and_model_naming_conventions#Featureandmodelnamingconventions](PORTAGE_sharedProgrammingGuidelines.md)
* [Help_messages#Helpmessages](PORTAGE_sharedProgrammingGuidelines.md)


### The PORTAGE environment variable

Programs and scripts should NOT hardcode `/path/to/PortageII`. The best strategy is to assume that locations are implicitly available via standard environment variables like `PATH`. E.g., if your perl script calls a program `translate-perfectly`, don't call it like
`/path/to/PortageII/bin/translate-perfectly` or
`$PATH/translate-perfectly`, just say `translate-perfectly`. For special-purpose things like references to corpora and data files, assume that the environment variable `PORTAGE` points to the project root directory. Apart from standard external libraries, code should not have external dependencies (e.g., to something in a user's home directory).

### C++ conventions
* C++ programs should use the `Portage` namespace, and include `portage_defs.h` which defines this and a few other things. Note that this namespace does a ''using std'', so you don't need explicit qualifications on things from the std library (though you must avoid duplicating std names). Please avoid ''using std'' statements in include files outside the Portage namespace.
* C++ class names should have each word capitalized, with no _ between words.  E.g., `DecoderState`.
* C++ files should be all lowercase, with _ between each word.  E.g., `decoder_state.h`.
* All libraries should begin with `portage`, e.g., the `utils` module defines the `portage_utils.a` library.
* Avoid unnecessary const references in method/function parameters.  For example, write fn(double w, Uint x, T* y, const T* z) rather than fn(const double& w, const Uint& x, T* const &y, const T* const& z).  The first version has the same effective semantics but is more efficient.
* Includes for non-Portage files should use angle braces, whereas includes for Portage files, even in different directories, should all use double quotes.
* C standard libraries should all be included using their cxyz header, not the obsolete xyz.h headers.  E.g., include cmath, cstdlib, ctype, etc.
* For floating point numbers, we prefer to use doubles for the actual calculations (local and temporary variables), but in actual models (TM, LM, etc.) we use floats to save memory, since these models can get very large.
* The use of boost libraries is encouraged when the value added is significant, but please make sure you can't do the same work easily yourself, however, because boost libraries are very expensive to compile.
* Use the `MagicStream` classes for all I/O, if possible.  These automatically handle compressed files and pipes for the user.

### Script conventions
* Script (bash, perl, etc) file names should be all lowercase with - (hyphen, not underscore) between each word, and an appropriate extension.  E.g., `run-parallel.sh`.

### Documentation conventions 
* Document your code!  Write the documentation first, then the code!
* Update the documentation for any code you modify!
* Put a Copyright header at the top of all source files, with the correct year, accurately reflecting the ownership of the code within the file.
* Put the standard file/brief description/author header at the top of all source files, with the author name and the program name accurately describing that particular file.
* Your `.h` files should follow the ''doxygen'' documentation conventions (see existing `.h` files for many examples).
* Your `.cc` files can have minimal documentation if your code is clear, but explain tricky, complicated and non-obvious things you write.

### Formatting conventions
* Avoid tab characters in your files, since they don't always display consistently.  If you must use them, set your tab width to 8 in your editor.
** Vi tip:  put "set expandtab tabstop=8" in your .vimrc file.
** Emacs tip:  put "(setq-default indent-tabs-mode nil)" and "(setq tab-width 8)" in your .emacs file.
* Usually indent with 3 spaces.
** Vi tip: put "set shiftwidth=3 autoindent" in your .vimrc file.
** Emacs tip: ''any emacs user knows?''
* Avoid accented characters in source files, since we all use different default encodings!  Thus, "Majeste" and "langagieres" should be written (incorrectly, it is true) without their accent in source code.  Exception: source files for language specific processing where the encoding is semantically significant and used consistently in code, data and/or comments, such as utokenize.pl and tokenize.pl.

### Unit testing
* We now have a unit testing framework in place.  It is very easy to use, so please create a test suite for any new class you write.  See the documented example `src/utils/tests/test_your_class.h` for details.
* If a unit test suite takes too long to run, create a new directory under `test-suite/unit-testing` and write a `go.sh` script or a `Makefile` that runs the tests and reports if they are successful.
* Some unit testing used to be done in various programs called `test*`, but we are phasing those out, please use either of the two methods described above instead.

### Regression testing
Before committing your code, use the `regress-small-voc` or `toy` test suite to make sure it works.

### Git diff
Before committing your code, do a Git diff (or equivalent command for your code repository software) to make sure you agree your changes are what you think they are.

### Feature and model naming conventions
When you need to create a complex syntax for a new type of feature or model in any program in PORTAGE shared, please follow the conventions used in existing models:
* `:` is the standard list separator.  It is used in `canoe.ini` in particular.  It is also used in rescoring models to separate the name of a feature from its arguments.
* `#` is the standard argument separator.  For example, the LM base class interprets `#`''N'' as a request to limit the order, e.g. `lm_4gram_en.gz#3` treats this LM as a 3-gram LM even if it is a 4-gram LM.  Multi-prob TM's accept the argument `#REVERSED` after the file name to mean the TM should be reversed as it is read.  This separator is also used by many rescoring feature functions; see `rescore_train -H` for details.
* `;` is the alternative argument separator, to use when `#` would be ambiguous.  For example, complex LM classes such as `DynMapLM` use it to separate the `DynMap` tag, the type of map and the filename.  `#` could not be used here because LM classes already use it for a different meaning (see above).
* Exceptionally, the `FileFF` feature separates the column argument from the filename argument with `,`, but the parser looks explicitly for a comma followed by a number at the end of the string, so as to allow maximum flexibility in the file name itself.  The comma should be avoided as a separator elsewhere.

### Help messages
Every program should produce a useful help message when invoked with `-h`.  This help message should be formatted as follows:
* Each line should be at most 79 characters wide
* The first line is a synopsis giving the program name, `[options]` if it takes any, and the arguments in all caps, with optional ones in square brackets.
* The next paragraph is a description of the program, indented with two spaces.
* Then a section labelled `Options:` lists each option, also indented two spaces, with its argument (if any) immediately after in all-caps, and then a description of what the option does.
* Example:
|program [options] ARG1 ARG2 [OPTIONAL_ARG3]
|
|  Description of what program does.
|
|Options:
|  -opt1 A1   use A1 to do x
|  -opt2      do blah
|  -opt3 XYZ  do something with XYZ


-------------------------


Up: [PortageII](PortageMachineTranslation.md) / [ForProgrammers](PORTAGE_sharedProgrammerReference.md)
Previous: [UsingMake](PORTAGE_sharedMakeNotes.md)
Next: [AnnotatedBibliography](PORTAGE_sharedAnnotatedBibliography.md)