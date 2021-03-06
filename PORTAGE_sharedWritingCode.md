Up: [PortageII](PortageMachineTranslation.md) / [ForProgrammers](PORTAGE_sharedProgrammerReference.md)
Next: [UsingMake](PORTAGE_sharedMakeNotes.md)

-------------------------

## Writing Code

If you are going to modify the code, it is strongly recommended to you setup a local revision control repository (Git, SVN, CVS, or whichever you prefer).

From this point, we assume you are working from a fresh copy of the code, up-to-date with respect to your own repository.

### Compiling

To compile the code, go into the `PortageII/src` directory and type:
|   make -j 5
|   make -j 5 install

The default location where ''make install'' installs things is `PortageII/bin`.

If you want to override this default, you can specify the `INSTALL_DIR` variable on the ''make'' command line:
|   make INSTALL_DIR=/path/to/install/dir install

When you type ''make install'' or the above command line from `PortageII/src`, ''make'' will put everything where it belongs.  Subdirectories `lib` and `bin` will be created in the install directory if they don't already exist.  To use the programs in a custom `INSTALL_DIR`, however, you will have to update your environment variables accordingly.

See [MakeNotes](PORTAGE_sharedMakeNotes.md) for other useful options, such as turning on debugging or doing parallel compiling.

### Where to put your new code

Once you've installed, the next step is to decide where to put your new code. Try to fit it in with one of the existing modules:

* adaptation - model adaptation
* build - tools for making and installing
* canoe - decoding
* confidence - confidence estimation
* distortion - lexicalized distortion models
* eval - automatic evaluation of MT output
* lm - language models
* logging - logging facility for debugging
* nn - neural network models
* preprocessing - markup, tokenization, and linguistically-motivated text processing
* rescoring - nbest list reordering
* textutils - basic text processing utilities
* tm - translation models
* tp_''''models - tightly packed models
* tpt - library tightly packed data structures
* truecasing - truecasing software
* utils - basic tools for other modules to use
* word_''''align - word alignment models

If your stuff is too distinct from any of these, create a new module by creating a new subdirectory under `src`. You will probably need a Makefile - see the examples in other subdirectories, which rely on the canned rules in `src/build/Makefile.incl`. You don't absolutely need to go this route, but if you don't you will have to supply actions for the targets in the top-level Makefile in `src`: `all`, `clean`, `new`, `install`, `new_install`, `doxy`, `docs`, and `export`.

### Let us know about your changes

When you have made changes to the PORTAGE shared code, please let us (the NRC)
know by sending us e-mail at `PORTAGE_shared@nrc-cnrc.gc.ca` so that we can arrange to
incorporate your changes into the core and make it available to other
PORTAGE shared licensees, if your PORTAGE shared license agreement has clauses to that effect.


-------------------------

Up: [PortageII](PortageMachineTranslation.md) / [ForProgrammers](PORTAGE_sharedProgrammerReference.md)
Next: [UsingMake](PORTAGE_sharedMakeNotes.md)
