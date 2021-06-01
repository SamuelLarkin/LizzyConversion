Up: [PortageII](PortageMachineTranslation.md) / [ForProgrammers](PORTAGE_sharedProgrammerReference.md)
Previous: [WritingCode](PORTAGE_sharedWritingCode.md)
Next: [ProgrammingGuidelines](PORTAGE_sharedProgrammingGuidelines.md)

-------

Some notes about using GNU make for the PORTAGE shared project.

* To specify an install path other than your home directory, use:
|make INSTALL_PATH=/path/to/install/dir install

* To specify additional compiler flags on the make command line, supply a value for the CF variable on the command line:
|make CF=-g <target>

* To run the commands in parallel on the multiple `CPUs` of a machine, use -j N.  Don't use "-j" alone if other users are using the same machine, because then make will run as many jobs in parallel as it can!  N = 1 + number of `CPUs` on the machine seems optimal:
|make -j 5 <target>

* To add module or directory specific options, e.g., to turn off warnings you know are not a problem or due to an external package, add these flags to `MODULE_CF` in Makefile in that directory.

* Standard targets automatically available to each subdirectory:
[table]
[tr][td] `all` [td] compile everything. [/tr]
[tr][td] `clean` [td] remove all compiled files. [/tr]
[tr][td] `new` [td] make everything from scratch, i.e., make clean then make all. [/tr]
[tr][td] `install` [td] make all and then install everything needed to run PORTAGE shared. [/tr]
[tr][td] `new_install ` [td] make clean and then make install. [/tr]
[tr][td] `export` [td] make all and then install libraries and include files. [/tr]
[tr][td] `doxy` [td] make the code documentation. [/tr]
[/table]


* To speed up builds for debugging, you can make and install one sub directory and all its dependencies (direct and indirect) by using the `OT` variable ("Overall Target") of the main Makefile.  
For example, the following command, executed in your `PortageII/src` directory, runs "`make install`" in `canoe` and the directories it depends on:
|make OT=install canoe

* To speed up incremental builds further, you can use
|make <dir>/progs
to make only the libraries `<dir>` directly depends on, and then a full build in `<dir>`. 
Furthermore,
|make <dir>/<pgm>
will do the strict minimum amount of work required to build the program `<dir>/<pgm>`.
For example:
|make tm/train_ibm
Caveat: do not specify more than one such target at a time - running "`make -j 5 tm/train_ibm canoe/canoe`" will most likely corrupt compiled files in your sandbox and require a "`make clean`" to recover.  Run these two sequentially instead: "`make tm/train_ibm canoe/canoe`" (no `-j`) or "`make -j 5 tm/train_ibm; make -j 5 canoe/canoe`".

--------

Up: [PortageII](PortageMachineTranslation.md) / [ForProgrammers](PORTAGE_sharedProgrammerReference.md)
Previous: [WritingCode](PORTAGE_sharedWritingCode.md)
Next: [ProgrammingGuidelines](PORTAGE_sharedProgrammingGuidelines.md)