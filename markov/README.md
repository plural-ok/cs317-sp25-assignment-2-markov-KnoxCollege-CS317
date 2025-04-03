Project 0: Markov Babbler
-----------------------------

<img src="https://upload.wikimedia.org/wikipedia/commons/6/66/Andrej_Markov.jpg" width="160" alt="Picture of Russian mathematician Andrey Markov, creator of Markov Chains"> Andrey Markov (Андре́й Ма́рков, 1856-1922), for whom [Markov Chains](https://en.wikipedia.org/wiki/Markov_chain) and Markov Processes are named.


Version 1.1


* * *

### Table of Contents

- [Introduction](#Introduction)
- [Problem Representation](#Problem-Representation)
    + [Example Learning Set](#Example-Learning-Set)
    + [Example Babbler brain for a Unigram Model](#Example-Babbler-brain-for-a-Unigram-Model)
    + [Example Babbler brain for a Bigram Model](#Example-Babbler-brain-for-a-Bigram-Model)
- [Babbling Process](#Babbling-Process)
- [Code](#Code)
- [Books](#Books)
- [Submission](#Submission)

### Introduction

Our goal is to write a program that generates random sentences that kind of sound like the authors/texts that we trained our program on. For example, we may want to train our program on Lewis Carroll's _Alice in Wonderland_, and then generate sentences that kind of sound like Lewis Carroll. 

One very simple approach (and the approach we will us here) is a [Markov Chain](https://en.wikipedia.org/wiki/Markov_chain): a state diagram that has probabilities attached to each transition between states.

Here are some articles for reference:

* [Building Markov Chains in golang](https://mb-14.github.io/tech/2018/10/24/gomarkov.html)
* [Using Markov Chains to generate Lifetime Movie titles](https://www.soliantconsulting.com/blog/title-generator-using-markov-chains/)
* [https://www.axonjournal.com.au/issues/7-1/markov-game-poems](https://www.axonjournal.com.au/issues/7-1/markov-game-poems)
* [https://sookocheff.com/post/nlp/ngram-modeling-with-markov-chains/](https://sookocheff.com/post/nlp/ngram-modeling-with-markov-chains/)

### Problem Representation
    
We will represent our Babbler's brain (our state diagram) with a dictionary.
- keys: string of n space-separated words representing our n-gram states;  
    these are our Markov states
- values: list of strings representing words that can follow this state;  
    these are our successor words
    
To transition from a given state/key:
- choose successor word from values list (random index in list)
- nextState = (remove first word from current state string) and (append successor word)

#### Example Learning Set 

Consider the following pseudo-sentences we will learn from:  [`tests/test1.txt`](tests/test1.txt).

    a b c d .
    a b c e .
    a b c d e .
    a b c x y z .
    x y z a b c !
    x y z z a b c ?

#### Example Babbler brain for a Unigram Model 

If we convert this to a unigram model (probability of each word is independent of any preceding words; it only depends on the fraction of time this word appears among all the words in the training text), we get the following state diagram:

![Markov Chain unigram for above sentences](img/test1.png)

If we represent this graph using a dictionary of `keys=states` (represented as n-gram strings) and `values=successorWords` (represented as lists of word strings):
    
    {
        '.': ['EOL', 'EOL', 'EOL', 'EOL'], 
        'a': ['b', 'b', 'b', 'b', 'b', 'b'], 
        'b': ['c', 'c', 'c', 'c', 'c', 'c'], 
        'c': ['d', 'e', 'd', 'x', '!', '?'], 
        'd': ['.', 'e'], 
        'e': ['.', '.'], 
        'x': ['y', 'y', 'y'], 
        'y': ['z', 'z', 'z'], 
        'z': ['.', 'a', 'z', 'a'], 
        '!': ['EOL'], 
        '?': ['EOL']
    }
    
Notice the repeats in our lists. We are preserving multiplicitly to preserve the original probabilities. 
When babbling, choosing uniformly at random from these lists will produce a proportionally probable successor for each key/state. (With a different representation, we could consolidate duplicates, but we would need to track overall probabilities in some other way.)  
Notice also our punctuations marks only have 'EOL' termination words in their successor lists.

Our starter an stopper states are:

    starters: ['a', 'a', 'a', 'a', 'x', 'x'] 
    stoppers: ['.', '.', '.', '.', '!', '?']
    
Given a total of 6 input sentences to train on, adding each first word to starters and each last word to stoppers, we end up with 6 starters and 6 stoppers. 

#### Example Babbler brain for a Bigram Model 

If we look at bigrams (approximates the probability of a word by using only the conditional probability of one preceding word), we get:

![Markov Chain unigram for above sentences](img/test2.png)

Represented as a dictionary of bigrams, this graph is:

    {   
        'd .': ['EOL'], 
        'a b': ['c', 'c', 'c', 'c', 'c', 'c'], 
        'b c': ['d', 'e', 'd', 'x', '!', '?'], 
        'c d': ['.', 'e'], 
        'e .': ['EOL', 'EOL'], 
        'c e': ['.'], 
        'd e': ['.'], 
        'z .': ['EOL'], 
        'c x': ['y'], 
        'x y': ['z', 'z', 'z'], 
        'y z': ['.', 'a', 'z'], 
        'c !': ['EOL'], 
        'z a': ['b', 'b'], 
        'c ?': ['EOL'], 
        'z z': ['a']
    }
        
 Our starter and stoppers states here are:

     starters: ['a b', 'a b', 'a b', 'a b', 'x y', 'x y']
     stoppers: ['d .', 'e .', 'e .', 'z .', 'c !', 'c ?']
 
 ### Babbling Process (using bigram model as example)
 
 **Step 0) Uniformly at random, choose a starter _state_ from our _starters_ and set this starter as our current _state_.**  
    
e.g. let's assume we are working with the bi-gram model above  
our _starters_ are: ['a b', 'a b', 'a b', 'a b', 'x y', 'x y']
since 'a b' is n the list 4 times and 'x y' 2 times, we are twice as likely to choose 'a b'  
the overall probabilities here are: 'a b' --> 0.67, 'x y' --> 0.33 (probability of each starter is 1/len(starters))  
let's assume 'a b' was chosen as our _state_
    
 **Step 1) Add this starter _state_ to our currently empty _sentence_ we are 'babbling'. This is now the beginning of our _sentence_.**
 
_sentence_ becomes 'a b'
    
 **Step 2) Given a current _state_, choose one of the possible _successor_ words. If the _successor_ is 'EOL', return our finished _sentence_.**
 
since our _state_ is 'a b', the possible _successor_ words are ['c', 'c', 'c', 'c', 'c', 'c']
thus the the _successor_ word 'c' will be chosen (with probability 1.0)
    
 **Step 3) Append the successor word to the sentence being babbled.**
 
_sentence_ becomes 'a b c'
    
 **Step 4) Make our new state by removing the first word in the current state.**
 
our _state_ 'a b' will become 'b' (also removing the newly-leading space)
after adding the _successor_ word, our current _state_ will become 'b c' (also add a space before the successor)
 
 **Step 5) Repeat from step 2.**

### Code

Your code will go in [`babbler.py`](babbler.py).
Read the comments in the code and complete all the empty methods (most are one-liners).

    def __init__(self, n, seed=None) # already completed with initial data structures
    def add_file(self, filename) # already completed; calls add_sentence(), so do that next
    def add_sentence(self, sentence) # your first port of call; read the comments and plan out your steps
    def get_starters(self)
    def get_stoppers(self)
    def get_successors(self, ngram)
    def get_all_ngrams(self)
    def has_successor(self, ngram)
    def get_random_successor(self, ngram)
    def babble(self)
   
You can run our Markov Babbler using: 

    python babbler.py
    
By default, it will use:  
- n-grams of size 3 
- [`tests/test1.txt`](tests/test1.txt) as input file for learning   
- and will generate 5 output sentences  
    
Optionally, you can provide up to 3 arguments (size of n-gram, input file, number of output sentences): 

    python babbler.py 2 tests/test2.txt 10

### Testing with autograder

There are unit test cases for the file [`tests/test1.txt`](tests/test1.txt) in [`test_markov.py`](test_markov.py).   
Once your basic implementation of the methods in [`babbler.py`](babbler.py) is working, you should pass these test cases:

    python test_markov.py

### Books

Download [this zipfile](https://drive.google.com/open?id=1YN238uggXVqec7-rR-qkGNunvPP5QkvO) of longer texts. Unzip the zipfile into your markov folder (if you are using a cloned/forked version of this repo, the gitignore is pre-set to ignore your new "books" folder).
Train your babbler on one or more of these texts, and produce some interesting sentences; you can use any size n-grams.
Make sure that none of your interesting sentences are identical to sentences that occur in the training text. Your program must be generating unique new sentences, not randomly picking complete sentences from the training text.

### Submission
Submit (on Google Classroom):
- the babbler.py file with your code
- your most interesting 5 sentences from the longer texts
