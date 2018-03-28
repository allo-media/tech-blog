# Brace yourself data selection, industrialization is coming!

Industrialization is one of the most challenging problems for our company and definitely not only ours. In fact, the world of research is far away from productivity. And the question was asked when we thought about building language model (referred as LM in the following) automatically, especially regarding data selection. In fact, our LM was handmade thanks to heuristics built by empirical means ... Meaning that we had to experiment to find the best system with our own eyes. And this is so not compatible with automating.

What is data selection you may ask? However, first thing first.

## ASR: Automatic Speech Recognition Prelude

In ASR, all is very dependant of the language we want to recognize. If we explain it really simply, we often consider building two boxes.

- The first one is called acoustic model. It represents the way of speaking. What sounds can be put together to form a word, a sentence. In fact, we represent the language by a series of phoneme. If we watch in [Wiktionary](https://en.wiktionary.org/wiki/phoneme) it's clearer isn't it? Don't confuse with syllable through. We can take an example. The word 'through'(1 syllable) consists of three sounds, three phonemes: 'th' 'r' 'oo'.

- The second one is called language model which we want to build automatically. It represents the distribution of the words in the given language. Thanks to probabilities, the system is capable of picking the best correspondence between the list of phoneme and the words/sentences.

In order to build these boxes, we need data and the more they are relevant, the better it is! That's why we need data selection.

## How we used to do,

As we discussed above, data selection is a very important step while building a system. Like tons of people, we used the [Moore-Lewis method](http://www.aclweb.org/anthology/P10-2041) which was adapted for bilingual use (like translation) by Axelrod&co called [Domain Adaptation via Pseudo In-Domain Data Selection](https://aclanthology.info/pdf/D/D11/D11-1033.pdf). These are very effective ways to select data using two corpora (in and out domain) and comparing the cross-entropy.

this method need people to choose and validate the model. Moreover, something hits us hard. This article turned out to be eight this year and as we all know, eight years is a long period in Computer Science and in Automatic Speech Recognition. So we asked ourselves: has anyone done some work about data selection since this article? And are there any relevant work ready for a more industrialized turn?

## Searching.... Finding!

After browsing 178 articles quoting the Moore-Lewis one, a title caught our eyes: [Cynical Selection of Language Model Training Data](https://arxiv.org/pdf/1709.02279.pdf). The name was so catchy, we had to explore it. Written by Amittai Axelrod, remember we mentioned him above, we decided to give it a shot [here](https://github.com/allo-media/cynical-selection) because the abstract was full of good promises ... And was compatible with industrialization! Unlike the previous method, the algorithm ends up with a response, letting us continue our road toward automating.

## How does it work? How did we make it work?

The goal is to selection data from our out-domain corpora that can extend our in-domain data. Suppose you have a small in-domain corpora, which you are a hundred percent positive that is representative. The algorithm will take this corpus and a more generic one, where you don't know what's relevant or not. It will then select the sentences that match the specific one. The script can take arguments which are detailed in the header of the script. It's only required the two corpora data to work:

'./cynical-selection.py --task inDomainFile.txt --unadapted outDomainFile.txt'

and return you the list of those sentences along with their scores in a '.jaded' file constructed as follows:

'model score sentence score (penalty + gain) length penalty sentence gain sentence id (in the selection) sentence id (in the unadapted corpora) best word word gain sentence'

In the end, we didn't lose any performance using this method and it allows us to automatize this part, taking us one step closer to industrialization.

## To conclude

This method allows us to focus on other side of the systems, making us more productive and more serene towards the building of language model. So it's a success captain!
