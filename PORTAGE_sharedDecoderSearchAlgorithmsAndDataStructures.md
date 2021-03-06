Up: [PortageII](PortageMachineTranslation.md) / [Translating](PORTAGE_sharedTranslating.md)
Previous: [Postprocessing](PORTAGE_sharedTranslatingPostprocessing.md)
Next: [UsingPhrasetables](UsingPhrasetablesInCanoe.md)

-------------------------
## Caveat

This page was written before the cube pruning decoder was written.  It describes the regular decoder.  The cube pruning decoder reuses most data structures and algorithms from the regular decoder, but changes how the search is performed and how the stacks are used.  See Huang and Chiang (ACL-2007) for a description of the cube pruning algorithm.

## Translating: The Decoder Search Algorithms And Data Structures 

Author: Ghada Badr

The decoder's main job is to find the translation that is most likely according to a set of weighted feature/model scores. The features or models evaluate the quality of the translation according to various criteria [here#TrainingConstructingModels]((PORTAGE_sharedTrainingModels.md) is a description of some models); the weights are learned in an optimization step as described in the
[COW#OptimizingWeightsforCanoeDecoding](PORTAGE_sharedTrainingOptimizingWeights.md)
section.

## Search Algorithms 

The search space of the possible translations is extremely large. Thus, the decoder must use heuristic search techniques to decide which portion of this huge search space to explore.  Excluding parts of the search space from the search is called pruning.  The main problem associated with any pruning technique is that of finding a tuning of the parameters that yields a suitable tradeoff between accuracy and speed. Our decoder employs a well-known pruning technique called beam search; beam search is a [heuristic](http://en.wikipedia.org/wiki/Heuristic.md) [search](http://en.wikipedia.org/wiki/Search_algorithm.md) algorithm. 


Beam search unfolds only the most promising nodes. There are two possible criteria for selecting the most promising nodes:
*Cardinality Pruning: In this case, we compare only the nodes that represent the hypotheses that cover the same ''number'' of words in the source sentence.
*Coverage Pruning: In this case, we compare only the nodes that represent the hypotheses that cover the same ''set'' of words in the source sentence.

Before we describe the details of pruning in beam search, a few words about scoring of a partial translation hypothesis (node ''n''):
The ''total score f(n)'' consists of the ''accumulated score g(n)'' and a ''remaining cost estimation h(n)'':
*''g(n)'' is the score which the different features/models assign to the partial translation hypothesis, covering the given set of source words.
*''h(n)'' is a heuristic estimate of the score which the features/models will assign to a completion of this partial hypothesis, covering all source words. This heuristic has to be optimistic, i.e. it has to give an upper bound for the probability of a completion.

''Note:'' In order to totally confuse everyone working with the decoder, the naming conventions in the source code are as follows:
*''score'' denotes the accumulated score ''g(n)''
*''futurescore'' denotes both the remaining cost estimate ''h(n)'' and the total score ''f(n)'', depending on the context.

In order to decide which the most promising nodes are (within the group of nodes under comparison) the following two parameters are used:

*Histogram parameter:  Beam search unfolds (extends) only the first ''m'' most promising nodes (hypotheses) under comparison and prunes the others, where ''m'' is a fixed number called the "beam width". 
*Threshold parameter: Beam search unfolds (extends) only the nodes (hypotheses) under comparison that have a total score within a beam around the best total score of all considered nodes (hypotheses). This beam is determined by adding a threshold to the best total score. All nodes (hypotheses) whose total score is outside this beam are pruned. 

Each group of nodes (hypotheses) under comparison should be kept in a priority queue (stack). These queues help in easily extracting (ranking) the best states in accordance with the pruning criterion (cardinality, coverage) that is used and the set of parameters (histogram, threshold) that are chosen. 

In cardinality pruning, the hypotheses that cover the same number of words in the source sentence will be stored in the same priority queue (stack).  When hypotheses are extended, the newly generated hypotheses will be kept in the corresponding priority queue according to the number of words covered in the source sentence. The total score is used as a criterion to rank the states within each priority queue. 

When using the histogram parameter, at some point we may reach a tie. A tie occurs when more than one hypothesis has the same total score and we cannot take all of the tied hypotheses because they will be outside the beam width (the histogram parameter). In this case, we have to apply some criteria to decide which to keep and which to prune. The criteria that are used are:
*prune according to total score ''f(n)'', if equal,
*prune according to accumulated score ''g(n)'' only, if equal,
*keep hypotheses that are created first.

In coverage pruning, one prunes not only among hypotheses that cover the same ''number'' of words (cardinality), but also among those that cover the same ''set'' of words (coverage) in the source sentence. So this constitutes pruning at a finer level.
Coverage pruning helps to avoid pruning that is too harsh. Consider the following example:
: : Let ''sj'' be a source word which is hard to translate (i.e. all its translations will have a low probability). Consider two hypotheses ''n1'' and ''n2'' of the same cardinality, where ''n1'' covers ''sj'' and ''n2'' does not. Then, it is likely that ''n1'' will be pruned because its total score is lower than that of ''n2''. Yet, ''sj'' will still have to be covered eventually, so it is unfair to punish ''n1'' just because it has already dealt with this difficult word, while ''n2'' has not yet dealt with it. 

Coverage pruning reduces the time needed for maintaining the queues, since the queues will be smaller and thus easier to maintain. 
In the current implementation, coverage pruning is applied after cardinality pruning; this implementation has bad worst-case runtime behaviour. It would be preferable to convert the priority queue (stack) into a heap of heaps, with each sub-heap representing a particular coverage. This would make coverage pruning more efficient at runtime, without changing the final output.

##Recombination

In order to keep the search space as small as possible it is crucial to perform a recombination of the search hypotheses. Every two hypotheses, which cannot be distinguished by any of the models (features) can be recombined. Only the hypothesis with the highest total score will represent a group of recombined hypotheses (as shown in the figure below). We extend only the best hypothesis that represents this group. Recombination considerably reduces the size of the search space and speeds up decoding.

_recombination.jpg

The hypothesis with the best total score will be used to represent the group of recombined hypotheses. Despite the reduction of its size, the search space is still so large that we have to prune over the resulting set of recombined hypotheses. When a recombined hypothesis is pruned out of the search space, the underlying group of hypotheses it represents will be pruned as well. 


The information about recombined hypotheses which survive pruning is kept in the phrase graph described below. After search, an N-best list is extracted which can contain these recombined hypotheses.

##Data Structures

Decoding is a sentence-based algorithm. For each sentence a new search graph is constructed, the so-called phrase graph (similar to word graphs in word-based translation systems or ASR). A node in the phrase graph is named a state; it represents a partial translation hypothesis. The graph is extended in a breadth-first manner. Extended states (hypotheses) at the same level (same number of words covered) will be stored in the same stack. The pruning strategies defined in the previous sections will be applied to each stack. Only states that pass the pruning criteria will be considered for further extension. The stacks will take care of both pruning and recombining states. Due to both pruning and recombination, it is possible that more states will be pushed into the stack than will be popped from the stack before it is empty. The stacks will take care of the cardinality pruning.

To apply coverage pruning, the states in the same stack (that pass the cardinality pruning) will be classified further according to the set of source words they cover. 

The main data structures that are used for the decoding process are the following:

*Decoder-State: A structure representing a state in the phrase graph. Each state will have a unique ID. It stores the partial translation associated with this state, a link to the previous state, links to all recombined translations associated with that state, the accumulated score, and estimation of the remaining cost. The initial state in the decoder algorithm contains an empty partial translation, empty links and all scores are zero. 

*Hypothesis-Stack: An abstract class that represents a hypothesis stack. The stack will take care of both pruning and recombining states. 

*Hyp-Hash and Hyp-Equiv: Two functor classes that are used to test if a newly created state can be recombined with any of the previous states according to the decoder features and the hypothesis represented by this new state.

*Worse-Score: A functor class to resolve a tie problem between two given states.

*Recomb-Hyp-Stack: A class to maintain the possibility of recombining the states. This class inherits the Hypothesis-Stack class. It uses both the Hyp-hash and Hyp-Equiv classes to test the possibility of recombination. It also maintains the set of states in a hash set to make the test for recombining a new state with the set of previous states easier.

*Histogram-Threshold-Hyp-Stack: A class that mainly maintains the stacks that represent the different levels in the phrase graph and takes care of different types of pruning and recombination strategies. It inherits the Recomb-Hyp-Stacks and uses the Worse-Score class to resolve ties. It stores the parameters for both the coverage and the cardinality pruning: the histogram sizes and the thresholds. It stores some statistics about the stack, for example, the number of pushes, the number of pops, and the number of pruned states. The class takes care of the following three main functions:

**Pushing a new state: A new hypothesis (state) is pushed onto the stack only if its total score passes cardinality threshold pruning. The core of the push function is implemented in the Recomb-Hyp-Stack class to test the possibility of recombining the new state. This function takes care of cardinality threshold pruning only. 

**Popping states for extension: popping hypotheses from a stack starts only after finishing all pushing in the stacks and stops as soon as we reach the maximum number of hypotheses defined by the cardinality histogram parameter. Before popping starts, a heap structure is built to store the states that were previously pushed in the stack. The heap is maintained according to the total scores of the states (using the functor class Worse-Score). This helps popping the states in order from best to worse. This function takes care of the cardinality histogram pruning and pops at most the number of hypotheses that is equal to the histogram size.

**Coverage Pruning: While popping, a map is used to keep track of coverage pruning. For every possible coverage, the map stores the best total score and the number of states popped so far. After popping a state, this map is used to test if this state passes coverage pruning (threshold and histogram) and should be kept for further expansion. 

The implementation of the core of the decoder search algorithm can be found in decoder.h/cc. The decoder.h file defines the Decoder-State structure and three main functions: a function to initiate the first decoder state, a function to extend the states, and the function run-Decoder. The run-Decoder function implements the decoder algorithm using the above defined data structures, while taking care of pruning and recombination. The decoder.cc file implements the previous three functions. The decoderstate.cc implements the Decoder-State structure. 

_stack.jpg

The definition/implementation of the Hypothesis-Stack structure (and the associated stack structures) can be found in hypothesisstack.h/cc. The exact hierarchical representation for the hypothesis stack class structure can be seen in the figure above.
Two important classes are also defined in phrasefinder.h/cc. These classes are used by the function run-Decoder. These two classes are:

*Phrase-Finder: An abstract class that defines finding the phrases that can be added to a partial translation. 

*Range-Phrase-Finder: A class that inherits the Phrase-Finder class and finds phrases using a set of available ranges. This is the only phrase finder that should be used by the system. All the phrases found are stored and returned in the triangular array structure.

##Implementation Tricks

The path to each state in the phrase graph is not unique due to recombination. That is why a reference counter is used with each state to determine if this state is still in use or not after pruning. The Hypothesis-Stack (and the associate classes) must correctly use the reference counter of the Decoder-State. It should increment the count when a state is added via push, decrement it when a state is popped, and delete the state if its counter reaches zero. This is the case when the state is not referenced any more by another state due to pruning.  

The hypothesis stack should have all states pushed into it before any states are popped from it. The Boolean variable pop-Started in the Hypothesis-Stack class is used for this purpose.

-------------------------

Up: [PortageII](PortageMachineTranslation.md) / [Translating](PORTAGE_sharedTranslating.md)
Previous: [Postprocessing](PORTAGE_sharedTranslatingPostprocessing.md)
Next: [UsingPhrasetables](UsingPhrasetablesInCanoe.md)




