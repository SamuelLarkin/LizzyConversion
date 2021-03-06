Up [PortageII](PortageMachineTranslation.md)
Previous: [Background](PORTAGE_sharedOverview.md)
Next: [WhereToFindThings](PORTAGE_sharedWhereToFindThings.md)

-------------------------

Here is a list of topics you will need to know before you get started.  These are tools we expect you will use when you work with PORTAGE shared.  This list is not exhaustive, but covers a lot of the important things.

Key topics:
* Basic Linux tools: (e)grep, sort, uniq, tar, g(un)zip, zcat, tail, head, less, cat, diff, file.
* Navigating and managing the filesystem from the command line: cd, ls, mkdir, mv, cp, chmod, ln -s, df, du.
* The bash shell - interactive use, pipes, IO redirection, defining and using environment variables, writing simple scripts.
* Job control: running jobs in the background, and these commands: ctrl-z, bg, fg, disown, jobs, top, ps, kill.
* Make: PORTAGE shared software is built and run using make; you might not need to write makefiles, but you will need to use them and maybe modify existing ones.
* Using putty to connect to a Linux machine from a Windows one, if your primary desktop is Windows.
* Using rsync or winscp or some other tool to transfer files from Windows to Linux and back.
* A command line interface (CLI) editor: one of vi, emacs, nano or any other non GUI editor.  You may sometimes be able to work with a GUI editor, but you should not rely on always having that option, so you should know at least one CLI editor.  Nano is the simplest of these and seems to be included with recent Linux distributions.  Vi and emacs are powerful but not very friendly.  However, vi and emacs are the only editors that are guaranteed to always be present on any system, so you should probably know the basics of one of those two.

Advanced topics:
* Installing a software package from a "tarball": tar, gzip, configure, make.  PORTAGE shared itself is installed using make and g++ and some bash scripts you might have to customize.  Packages that PORTAGE shared depends on are also installed similarly.  (For example, if you learn to install nano, you can get away with ignoring vi and emacs altogether.)
* find: locate files on the file system.  This command is not intuitive to use, but very powerful and sometimes very useful. 
* Writing bash shell scripts with some control structures such as if, while, for.  Such scripts can be used to manage running a number of different experiments without having to manually launch each one, for example.
* Sed and maybe Perl: these languages are not strictly required, but you may need them as your project advances.  For example, you might use them to perform custom pre-processing or post-processing for corpora or translation memories that have unusual characteristics.

-------------------------

Up [PortageII](PortageMachineTranslation.md)
Previous: [Background](PORTAGE_sharedOverview.md)
Next: [WhereToFindThings](PORTAGE_sharedWhereToFindThings.md)

