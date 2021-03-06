Up: [PortageII](PortageMachineTranslation.md)
Previous: [PortageLive](PortageLiveManual.md)
Next: [ForProgrammers](PORTAGE_sharedProgrammerReference.md)

-----------------------

## Portage''''Live Customization using Plugins

Portage''''Live supports many customizations to handle situations that are specific to a given system correctly. These customizations are enabled through seven plugins that can be installed with each deployed system.

The default plugins have changed significantly in versions 2.2 and 3.0 of PORTAGE shared. `doc/PortageAPIComparison.pdf` contains a comparison chart documenting the recent improvements to the plugins. If you have custom plugins, even if provided by the NRC, we strongly recommend you review the new default plugins and consider using them instead. The 3.0 plugins roll in most custom processing we've provided in response to client requests in the past and should work well for most situations.

### Installing custom plugins

The default version of these plugins already does some processing. If you want to turn on or off a particular aspect, you need to 
# make a copy of the plugin into the trained system's `plugins` directory: `framework/plugins/` during training, `/opt/PortageII/models/`''context''`/plugins/` in a deployed system. The default plugins are found in PORTAGE shared's bin/ directory.
# edit the copy to change what processing it actually performs.

### The main plugins

Here are the four main plugins, in the order Portage''''Live calls them:

# `preprocess_plugin` does some automated data cleanup and normalization. By default, it also splits slash-separated words, such as "before/after". To turn off '''slash separation''', edit the plugin and comment out the line that reads "`FIX_SLASHES=1`".
# `predecode_plugin` is used to insert rule-based translation instructions, e.g., for translating numbers and applying fixed-term lists. For English-French and French-English system, '''rule-based translation of numbers''' is done by default. To turn that behaviour off, comment out the line that read "`MARK_NUMBERS_ENFR=1`". See [FixedTerms#FixedTerms](PortageLiveCustomizationPlugins.md) below for instructions on turning on '''fixed-term rules'''.
# `postdecode_plugin` is a place-holder for future use; don't use it.
# `postprocess_plugin` is used to make changes to the final output, just before returning it to the user. By default, it inserts '''non-breaking spaces''' where appropriate in French output (disable this feature by commenting out "`ADD_FR_NBSP=1`"). It can optionally fix the '''formatting of numbers in French''', as an alternative to the rule-based option mentioned above: uncomment the line "`fix-en-fr-numbers.pl |`" to enable this. It can also insert '''non-breaking hyphens''' in selected patterns, such as in short hyphenated codes: this is more complicated because it depends on how your application represents these, but the default plugin includes commented-out sample code for several options we're aware of.

### Tokenization plugins

The PORTAGE shared tokenizer and detokenizer supports French, English, Spanish, and Danish. For other languages, three additional plugins required:
# `sentsplit_plugin` is used to perform sentence splitting on a source-language paragraph. It supports Arabic and Chinese by default.
# `tokenize_plugin` is used to perform tokenization of source language text. It works on Chinese out of the box. It supports Arabic with some additional installation work; ask us if you need this.
# `detokenize_plugin` is used to perform detokenization of target language output.

### Fixed Terms

Starting with version 2.2, PORTAGE shared can handle a list of fixed terms, which are used to bypass the translation system for fixed expressions that should not be translated statistically.

The purpose of the fixed-terms module is to handle specific brand names, proper names and other terms that accept no variations and must be systematically translated exactly the same way.

Limitations:
* No morphological processing of any kind is done. This is not intended for terms with morphological variants, although source-language variants can be entered as separate fixed terms.
* A given term can only have one translation.
* No disambiguation will be performed: if the given term appears in text to translate, it will get its fixed translation even when that might not have been appropriate. Therefore, make sure not to enter text that can sometimes be used in a different way.
* The fixed-terms list is lower-cased, and it is applied to text to translate after lowercasing. Therefore, there is no way to distinguish between "Rice" and "rice" or between "US" and "us". Therefore, make sure to add enough context to make your fixed terms unambiguous, e.g., "Mr. Rice" instead of "Rice".

'''Important warning''': do '''not''' upload your whole terminology database as a fixed-terms list. The results will be disastrous. The fixed-terms system completely bypasses all statistical learning abilities of PORTAGE shared and is only appropriate for carefully selected, fully disambiguated cases.

Installation:

To enable the fixed-terms functionality:
* A directory `plugins/fixedTerms/` must also exist and be readable, writable and executable by the Apache process. This is automatically done for systems trained with PORTAGE shared 2.2 or more recent. We provide a script to retrofit systems trained with earlier versions: run
 `prep-fixedTerms-layout.sh /opt/PortageII/models/`''system-name''
for each pre-existing system to create this directory and set its permissions correctly.
* The `predecode_plugin` must be included in each system that might handle fixed terms, by copying it into its `plugins/` directory as documented above in section [Installing_custom_plugins#Installingcustomplugins:](PortageLiveCustomizationPlugins.md)
 `cp /opt/PortageII/bin/predecode_plugin /opt/PortageII/models/`''context''`/plugins/`
Make sure you use the plugin from PORTAGE shared version 3.0 or later, and decide whether you want rule-based English-French number translation when you install that plugin, since the same plugin controls both features.
* Systems trained with old versions of PORTAGE shared might have the `[bypass-marked]` parameter set in their `canoe.ini.cow` file. This must be removed or replaced by `[no-bypass-marked]`:
 `sed -i 's/bypass-marked/no-bypass-marked/' /opt/PortageII/models/`''system-name''`/canoe.ini.cow`

Once the fixed-terms functionality is enabled, you can upload a fixed-terms database through the Portage''''''Live API. This is meant to be called from a client application that should generate this file automatically from a more user-friendly database. We only give a brief description of the operations here; see the API documentation for more details.
* Use the `updateFixedTerms()` method to initialize or update the database. The fixed-terms list must be prepared in a tab-separated file where the first line contains:
 source language code<TAB>the target language code
and each subsequent line contains:
 a source term<TAB>the term's translation
e.g. (where <TAB> must be replaced by literal tab characters):
 eng<TAB>fra
 this is a pointless term<TAB>ce terme est inutile
* Use the `removeFixedTerms()` method to delete the fixed-terms list and turn off fixed-terms handling.
* Use the `getFixedTerms()` method to query the system for its current fixed-terms list.

After all this, you should see the behaviour of your system changed to take into account the fixed terms you defined.

Alternatively, you can manually compile and install the fixed-terms database on
your Portage''''''Live server:

* copy the fixed-terms list to the Portage''''''Live server into file `/opt/PortageII/models/`''system-name''`/plugins/fixedTerms/fixedTerms`

* compile the fixed-terms list into file `/opt/PortageII/models/`''system-name''`/plugins/fixedTerms/tm`:
 `cd /opt/PortageII/models/`''system-name''`/plugins/fixedTerms/`
for en->fr systems, assuming you place English in the first column, use this command:
 `fixed_term2tm.pl -source_column=1 -source=en -target=fr fixedTerms | sort --unique > tm`
for fr->en systems, assuming you place English in the first column, use this command:
 `fixed_term2tm.pl -source_column=2 -source=fr -target=en fixedTerms | sort --unique > tm`

* the example file above, compiled, will look like this if things worked correctly:
** en->fr:
 this is a pointless term ||| <FT target="ce terme est inutile">this is a pointless term</FT> ||| 1 1
** fr->en:
 ce terme est inutile ||| <FT target="this is a pointless term">ce terme est inutile</FT> ||| 1 1


-----------------------

Up: [PortageII](PortageMachineTranslation.md)
Previous: [PortageLive](PortageLiveManual.md)
Next: [ForProgrammers](PORTAGE_sharedProgrammerReference.md)