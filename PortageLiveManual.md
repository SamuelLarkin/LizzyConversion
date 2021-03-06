Up: [PortageII](PortageMachineTranslation.md)
Previous: [MagicStreams](PORTAGE_sharedMagicStreams.md)
Next: PortageLiveCustomizationPlugins

-----------------------


## Portage''''Live User/Trainer Manual


### Overview

This manual describes how to build a live translation server using PORTAGE shared, called "Portage''''Live". The rationale for doing this on virtual machines is laid out in [Paul_et_al2009#Pauletal2009.](PORTAGE_sharedAnnotatedBibliography.md) Virtualization makes it possible to port PORTAGE shared to a variety of operating systems with little effort, and to run several instances of PORTAGE shared on the same machine. For instance, Portage Live can run concurrently on a single state-of-the-art desktop or laptop computer with other applications; it can also be used in a network for distributed translation processing.

The virtualization of PORTAGE shared was made feasible by two technological innovations for the data structures that take up the most memory in a typical machine translation system, the phrase tables and the language models (LM''''s).  First, we store them in a memory-efficient structure called a tightly-packed trie [Germann_et_al2009#Germannetal2009.](PORTAGE_sharedAnnotatedBibliography.md) For instance, an LM encoded as a tightly packed trie takes up only about a quarter of the RAM that would be occupied by an LM represented in the standard ARPA format. 
Second, we use the memory-mapped IO mechanism provided by most operating systems to access the tightly packed models.  Thus, unlike with in-memory applications, a tightly packed LM doesn't need to be completely loaded into memory, and may need only 10-15% of the RAM required by the standard representation. In practice, the decoder with tightly packed tries is ready to translate in about two seconds (whereas the in-memory decoder takes 1-2 minutes to load). As relevant pages are loaded into memory, translation is slower at first, but almost the same throughput is achieved once the relevant pages are in physical memory. Most significant, pages loaded for a previous request typically remain in memory, so the system stays highly responsive after the first query.  Tightly-packed models also provide a significant edge on multi-core machines: when multiple decoder instances run in parallel with the same models, they use shared memory to access the models, so that pages loaded for one instance are available to all others at no extra cost.

To create a Portage''''Live application, one must obtain a bilingual,
parallel corpus for the two languages of interest. The next step is to obtain the PORTAGE shared code, compile it, and install the executables. One then preprocesses the corpus and trains the models (phrase tables and LM''''s), and arranges the resulting files in a specified file layout. Then, one builds the virtual appliance; alternately, one can install the models on a physical machine. Finally, one arranges for one's application to call Portage Live (e.g., via SOAP). All these steps are described in more detail in the sections below.


### Installing PORTAGE shared 

First, you need to compile and install PORTAGE shared. The instructions for this are in the "INSTALL" file on the CD.


### Training PORTAGE shared in the Framework

Once you have installed PORTAGE shared, you must train a system using your parallel corpora, preferably using our experimental framework.  The document `framework/tutorial.pdf` gives a tutorial on training models and decoding in the current framework; read this first, and then apply the same methods on your own data.  For background information, you can also read [TextProcessing,](PORTAGE_sharedTextProcessing.md) which describes how to preprocess the bilingual and unilingual training data prior to model training; the section [TextFileFormats](PORTAGE_sharedFileFormats.md) has helpful related information about file formats. Finally, [ConstructingModels](PORTAGE_sharedTrainingModels.md) describes how to train the different kinds of models, and how to find a reasonable weight on each of these models. 


### The Runtime File Layout

Once you have trained the models, you need to create the correct file layout for building the virtual appliance - you need to strip down the list of files to just those needed and arrange them in the structure needed on the Portage''''Live server. These include: 1. the PORTAGE shared software 2. the trained models 3. the web service software.

The `PortageLive` directory on the CD contains the files and scripts you need to prepare the runtime file layout.

#### PORTAGE shared runtine software layout

If you use virtual machines to deploy your translation servers, you want to minimize the size of the software installed on each one.  Most of the PORTAGE shared software is only needed while you're training models, so the runtime software structure is much smaller.

In `PortageLive/bin`, you can run
|   make
or
|   make DO_EXTERNAL_LIBS=1
to assemble the PORTAGE shared runtime software layout.  (The second variant tells make to try to gather the shared libraries too.)  First, however, you have to have installed PORTAGE shared and configured your environment as you would to run the software, i.e., by sourcing `SETUP.bash`, as described in `INSTALL`.  
The `Makefile` in `PortageLive/bin` finds the programs where they are installed and copies them into the appropriate structure for installation on the Portage''''Live translation server.  See the `README` file there for help with issues that might arise with `.so` files (shared libraries).

This structure is created inside a directory call `rpm.build.root`.  In each of the three file layout steps described here, files are placed in a directory with that name, under which the full path to the final destination on the translation server is recreated.
This is the standard mechanism for building RPM''''s, and it is helpful as a staging area even if you are planning to copy the files on the server manually.

#### Trained models layout

Again, many of the files created while you train a system are not needed on the translation server itself, so here we assemble the minimal set of files from the framework to actually perform translation.  The `portageLive` target in the framework is intended to assist you in doing so.  From your copy of the framework directory where you trained models, run 
|   make portageLive
You will see the models being converted to the right format for the runtime translation server, and a directory structure of symbolic links.  The commands `rsync -Larzv`, `scp -r` or `cp -Lr` can be used to copy this structure while expanding the links, in order to create a full runtime layout somewhere else, while `du -hL models/portageLive` will tell you the size of the fully expanded runtime layout without actually expanding it.

Note that if you are using rescoring (with `DO_RESCORING=1` in your `Makefile.params`), the rescoring model and its associated files must be manually added to the `models/portageLive` directory structure before you continue.

In `PortageLive/models`, you will find the script `prep-file-layout.sh` which will copy the model files from your trained framework into the appropriate structure under `rpm.build.root`.  When you call
|   prep-file-layout.sh <path_to_your_trained_framework_directory>
the models will be copied into `rpm.build.root/opt/Portage/models/context`, since the web service software we provide expects to find them under `/opt/Portage/models/context` on the translation server.

The `prep-file-layout.sh` script uses `scp` to copy files from the framework, so the path can include a host name and/or user name if you trained the system on a different machine from the one where you are assembling the Portage''''Live file layout.  E.g.:
|   prep-file-layout.sh training_host:/path/to/trained/framework
|   prep-file-layout.sh training_user@training_host:/path/to/trained/framework
or simply, if the files are on the same machine:
|   prep-file-layout.sh /path/to/trained/framework


#### Web service software layout

These files include some web pages, CGI scripts, and the WSDL definition for calling PORTAGE shared via SOAP.  They must ultimately find their way in the web server structure on your translation server, typically in `/var/www`.  The WSDL file must be edited to hard-code your IP address; if you use virtual machines, our instructions (see below) show how to dynamically do so each time the machine is booted.

To use the CGI scripts supplied, you may need to modify a few configuration variables before you proceed.  Towards the top of `cgi/plive.cgi`, there is a section called "USER CONFIGURATION" which you need to edit.  See the instructions there.

In `PortageLive/www`, you can run `prep-file-layout.sh` to assemble the web service software layout we used for our demos.  As in the previous steps, the files will be copied under `rpm.build.root`, following the expected installation file structure.  You may have to modify the layout and the files to adjust to your web server settings, or to the way you intend to access Portage''''Live, or to support multiple contexts (see below).

#### Making RPM''''s

Before you build your virtual machines, you should make three RPM''''s.
An RPM is like a zip file which embeds instructions on how and where to install software on a Linux machine.
We use VM''''Ware Studio to build our 
RPM''''s, but any tool that creates them will do.  
The three file layouts described above should be packaged as RPM''''s `PortageLive-bin`, `PortageLive-models-`''context'', and `PortageLive-www`.  The keyword ''context'' should be replaced by label for each different context you train, so that you can distinguish them later.

We use VM''''Ware Studio to build an RPM from each of the three `rpm.build.root` directories created earlier.
Assuming you have VM''''Ware Studio installed, and `mkpkg` is in your path, you can go into each of `PortageLive/bin`, `PortageLive/models` and `PortageLive/www`, and do the following:
# Edit `control.spec` and adjust the descriptions and other fields if necessary.  You should pay special attention to `models/control.spec` and describe what models you have trained and are installing, replacing `context` in the Name by a label representing the actual context you are packaging.
# Run `../scripts/make-rpm.sh`.  This script increments the Release number, creates the RPM file, and gives it an appropriate name.

Having followed these instructions in the three directories, you will have the three RPM''''s
necessary to build your Portage''''Live Virtual Appliance.

Note that if you are working with a physical machine as translation server, you can skip the creation of RPM''''s 
and follow the instructions under "Installing Portage''''Live on a physical machine" below.

Caveat: with very large models, we have had difficulties packaging the models into an RPM.  In such cases, we skip the creation of `PortageLive-models-`''context'' and instead we rsync the model files directly to the VM after booting it, as described under "Installing Portage''''Live on a physical machine" below.

#### Installing multiple contexts on the same machine

Portage''''Live can be associated with multiple contexts. Each context consists of a system you trained, and therefore specifies a translation direction (e.g., English to French), a domain (e.g., parliamentary debates) and all the training parameters you have chosen. Typically, the models linked to a context are designed to work well in that context (e.g., the translation of parliamentary debates from English to French).

The instructions shown above all assume a single context is installed on a translation server (an appropriate choice for Virtual Appliances), but we provide support for installing multiple contexts on the same server:

* Multiple contexts share the same PORTAGE shared runtime software, so you only install this once.

* The models for each context should be installed under `/opt/Portage/models/`''context'', where ''context'' is a label you chose for that context.

* The web service software we provide supports multiple contexts.  The CGI interface (`plive.cgi`) automatically detects which contexts are installed in `/opt/Portage/models/`, and uses a drop box to let the user pick among them.  The SOAP interface (`PortageLiveAPI.php` and `PortageLiveAPI.wsdl`) considers "context" to be the default context, but also has methods that accept a context label as and argument.

### Installing the Portage''''Live server

#### Virtual or physical machines?

As we've suggested earlier, Portage''''Live may run either on an actual physical machine, or on a virtual machine.
The machine, physical or virtual, should be a server running Linux, with apache, mod_''''ssl, php and php-soap installed.  

To help you chose between virtual or physical machines, here are some advantages of each option:

Virtual machines:
# you can deploy Portage Live on your existing infrastructure, even your desktop machines, regardless of the OS your are running on them;
# you can tap in to your virtualization infrastructure, if you already have one; 
# you can tap in to cloud computing, deploying Portage Live on 3rd party clouds; 
# virtualization makes it possible to adjust assigned resources as the load fluctuates.

Physical machines:
# if you use a multi-core machine as your translation server, you can maximize the benefits of using shared memory between instances of the decoder, and you can pool resources together for different translation contexts;
# you don't need VM''''Ware
or RPM''''s.

#### Building the Virtual Appliance

A Virtual Machine with specialized software installed on it is often referred to as a Virtual Appliance, because it is not considered a general-purpose machine, but rather an appliance with a specific purpose.

If you choose the Virtual Appliance option, instructions showing how to create it using VM''''Ware Studio are found in `PLiveVA.pdf` in `PortageLive/va`.
The RPM''''s we created previously are installed on the virtual machine as part of its creation process.

#### Installing Portage''''Live on a physical machine

If you choose the physical machine option, you can manually copy the runtime file layout using `rsync`, `scp` or similar tools.  `rsync` is probably the easiest option, since it can reproduce the structure in a single command even if it already exists partially on the destination host:
|   rsync -arzv rpm.build.root/* root@<host>:/
or, if the destination machine is the same as the building machine:
|   rsync -arv rpm.build.root/* /
This command mimics the structure in `rpm.build.root` under `/` (the root directory) on machine <host>, using the root (i.e., administrator) account.

Copy the three `rpm.build.root` structures to their destination on your physical server.  You can copy the PORTAGE shared and web service software once - you can probably even install all the PORTAGE shared software instead of the minimal layout described above, since space will likely be less limited than on virtual machines.  Then for each context you want to make available, copy the model files to an appropriate location, as suggested in the section on multiple contexts above.


### Calling Portage''''Live from Your Application

In our experiments and demos, we have used SOAP as well as PHP and CGI scripts to make the translation results available to client applications.

#### SOAP API

The advantage of using SOAP is that it makes it feasible to call Portage''''Live directly from existing service-oriented applications.  We provide a SOAP API you can use for this task, which is part of the web service software layout described earlier.  The API is implemented in the PHP script `PortageLiveAPI.php`.  The associated WSDL, `PortageLiveAPI.wsdl`, serves both as interface definition for other languages, and contains the documentation of the API.  In the end, the PHP script calls PORTAGE shared via the `soap-translate.sh` script that is created when you train your models.

The current SOAP API includes methods `translate()`, `translateTMX()` and `translateSDLXLIFF()` for translating plain text, TMX files or SDLXLIFF files, with or without confidence estimation.  `getAllContext()` lists installed contexts while `translateFileStatus()` can be used to monitor the status of asynchronous TMX translation requests.  These methods are documented in the WSDL file itself. 

A sample PHP web page invoking this service, `soap.php`, is included with the API for the purpose of testing and illustration; it calls all the methods listed above.  (But don't use it to offer Portage''''''Live via a web page; use `plive.cgi` instead, described below.)  We have also successfully used our API from Java (using Eclipse) and C# and it can be used from any language that supports SOAP.

The SOAP API has changed significantly with version 3.0. The changes over the versions of PORTAGE shared are documented in `doc/PortageAPIComparison.pdf`. Sample PHP code that you can port to your applications's language is provided in determine-version.php to automatically detect the version of Portage''''''Live installed on a particular server.

#### CGI scripts (interactive web page)

An alternative solution uses CGI scripts to make the translation results available via a web page.  A pair of CGI scripts is installed as part of the web service software layout: `plive.cgi` (to submit translation requests) and `plive-monitor.cgi` (to monitor translation requests).  These CGI scripts work as follows: a text box is available to perform translation of short texts while the user waits, and a upload button is available to process long text or TMX files "offline": the text is submitted to the server, then the user is shown a monitoring page showing progress, and the result is available as a download at the end.

#### Using SSH

A third solution is to SSH to the translation server and call `translate.pl` (or a wrapper script) directly.  The `translate.sh` script in the framework can be used to generate a wrapper script which calls `translate.pl` with appropriate options.  See for example the `soap-translate.sh` script we create for use by the SOAP API. This solution requires the use of DSS keys so you don't have to enter a password for every request.  The software integration will likely be simpler this way if you are working in a Linux environment, but more complicated in a Windows environment.

#### REST API

We now have a prototype REST API that roughly follows the Google translate API, for integration in applications that already support the Google translate API. This REST API can be considered an alpha release, however. It is not documented or supported yet. Talk to us if you're interested in using it.

#### How to choose

The best choice will depend on what you are most familiar with and what tools are most accessible to you.  In the end, all methods are calling the same `translate.pl` script; it is just a matter of determining which way is easiest in your environment, which way satisfies your security requirements, etc.


### Extending the API

If you need to extend the function call, you can do so through our API.  You should modify `PortageLiveAPI.php` and `PortageLiveAPI.wsdl` to meet your needs.  You must be familiar with the creation of web services to do this.  Similarly, you can modify the actual call in the CGI script to suit your needs.  In all cases, you can get started by looking for the call to `translate.pl` or `soap-translate.sh`.

-----------------------

Up: [PortageII](PortageMachineTranslation.md)
Previous: [MagicStreams](PORTAGE_sharedMagicStreams.md)
Next: PortageLiveCustomizationPlugins