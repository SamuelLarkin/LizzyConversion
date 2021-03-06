Up [PortageII](PortageMachineTranslation.md) / [Translating](PORTAGE_sharedTranslating.md)
Previous: [DecoderAlgorithms](PORTAGE_sharedDecoderSearchAlgorithmsAndDataStructures.md)
Next: [Evaluation](PORTAGE_sharedEvaluation.md)

-------------------------

## Using phrase tables in canoe

Canoe allows the simultaneous use of two kinds of phrase tables on disk:

* multi-prob phrase tables, specified by the `-ttable-multi-prob` option; and

* TPPT phrase tables, specified by the `-ttable-tppt` option.

Each kind of phrase table contains a set of phrase pairs, and
associates one or more probabilities or scores with each pair.

### Text phrase tables (obsolete)

The old single-prob text phrase table format is obsolete, and no longer supported.  We keep the description here because the description of the multi-prob phrase table builds on the information in this subsection.

Text phrase tables are the simplest and oldest format: they associate exactly one probability with each phrase pair. They come in two
varieties: "backward" tables in which the probability is
p(src|tgt), and "forward" tables in which it is
p(tgt|src). Here src is the current source phrase and tgt is
the current target phrase.  There are two possibilities for
using many different text phrase tables together: either all
tables must be backward ones, or all tables must come in
backward/forward pairs in which each member of the pair
contains the same set of phrases. Forward and backward
probabilities are distinguished in the loglinear model, and
for the purposes of filtering low-probability pairs. Here
are the possibilities:

* using only backward tables (obsolete):
** each table is a feature in the loglinear model, and is assigned a weight 
** filtering is performed by ranking translations of each source phrase according to a weighted sum of its probabilities in each table

* using paired backward/forward tables: 
** each backward table is a feature in the loglinear model, and is assigned a weight 
** forward tables are not features in the loglinear model unless an initial set of weights for them is given with the `-weight-f` switch, or the `-use-ftm` switch is used
** filtering is performed by ranking translations of each source phrase according to a weighted sum that depends on whether forward weights are used: 
*** if so, forward weights are applied to forward probabilities by default; but the `-ttable-prune-type` switch can be used to select an alternate strategy: either backward weights with forward probabilities, or all weights with all probabilities 
*** if not, backward weights are applied to forward probabilities

Each line in a forward phrase table follows this format:
| target ||| source ||| p(target|source)
Each line in a backward phrase table follows this format:
| source ||| target ||| p(source|target)

### Multi-prob phrase tables

A problem with single-prob text phrase tables was that they were highly
redundant: specifying forward and backward probabilities for
a given set of phrases requires that the phrases be listed
twice, once in a forward table and once in a backward
table. Using two alternate probability estimation schemes
simultaneously would require listing the phrases four times,
etc. Multi-prob phrase tables get around this by allowing
many probabilities to be associated with each phrase pair in
a given set. To preserve the distinction between backward
and forward probabilities without complicating the format,
both backward and forward probabilities must always be
given. For example, if a list of six probabilities is given
after a phrase pair, the first 3 are assumed to be backward
and the last 3 forward, with the 1st and 4th forming a pair,
the 2nd and 5th forming a pair, etc. Odd numbers of
probabilities are illegal in multi-prob tables.

Loglinear weighting and filtering work as described above under 'using paired backward/forward tables'.

If multiple multi-prob phrase tables are used together, each table provides its own set of probability scores.  The weights in `-weight-t` and `-weight-f` apply to the columns from the different tables in order.  For example, if you use table A with 4 columns (2 backward, 2 forward), and table B with 2 columns, `-weight-t` has to specify three weights, which will correspond, in order, to the first column in A, the second column in A, then the first column in B.  If specified, `-weight-f` must also have three weights, in order, for the third column in A, the fourth column  in A, and the second column in B.

If one table is missing a phrase pair that is provided by another, the table where it's missing is presumed to have provided it with epsilon probability, as specified by the `-ttable-log-zero` option (by default, we set log(epsilon) = -18).

Each line in a multi-prob phrase table follows this format, where `s` is the source phrase and `t` is the target phrase:
| s ||| t ||| P_1(s|t) .. P_n(s|t) P_1(t|s) .. P_n(t|s)

### Adirectional scores ("4th column")

Multi-prob phrase tables normally have three columns, as shown above.
As documented in `canoe -h`, they can also include a "4th column" with adirectional scores:
| s ||| t ||| P_1(s|t) ... P_1(t|s) ... ||| A_1(s,t) .. A_m(s,t)

These scores are separate models whose weights are set using the `-weight-a` or `-atm` option to canoe.  Their presence does not affect how pruning is performed or how the forward and backward scores are interpreted.

PORTAGE shared does not currently include software to generate this 4th column, but the format is easy to generate if you have your own features A(s,t) you wish to use.  See [Chen_et_al2009#Chenetal2009](PORTAGE_sharedAnnotatedBibliography.md) for examples of such features.

### TPPT''''s (Tightly Packed Phrase Tables)

The final kind of phrase tables are TPPT''''s,
which are used directly on disk via memory-mapped IO for instant loading and fast access.  They contain the same contents as a multi-prob table, with ''n'' backward weights and ''n'' forward weights, for any ''n'' >= 1.
Adirectional scores and the count (and multi-count) fields are supported since PORTAGE shared 1.0.

When combining multiple TPPT''''s, the weights are assigned just like when using multiple multi-prob phrase tables.
When combining multi-prob phrase tables with TPPT''''s,
the weights for the multi-prob phrase tables come first, then those for the TPPT''''s.

-------------------------

Up [PortageII](PortageMachineTranslation.md) / [Translating](PORTAGE_sharedTranslating.md)
Previous: [DecoderAlgorithms](PORTAGE_sharedDecoderSearchAlgorithmsAndDataStructures.md)
Next: [Evaluation](PORTAGE_sharedEvaluation.md)