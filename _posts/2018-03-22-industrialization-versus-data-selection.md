# Brace yourself data selection, industrialization is coming!

Industrialization is one of the most challenging problems for a start-up like ours. In fact, the research world has little to no knowledge when it comes to productivity. And this matter struck us when we thought about building language models (referred as LM in the following) massively, especially regarding data selection.

Historically, our LMs were crafted one by one with love, with a nice cup of human intervention in between. Meaning that we had to experiment to find the best system empirically. And this is so not compatible with automation.

What is data selection you may ask? However, first thing first.



## ASR: Automatic Speech Recognition Prelude

In ASR, we usually consider building two independent modules which will be mixed together later on. Each module in itself is very dependent of the language we want to recognize.


 - The first one is called acoustic model. It represents the way of speaking. What sounds can be put together to form a word, a sentence. In fact, we represent the language by a serie of phonemes. If we look in [Wiktionary](https://en.wiktionary.org/wiki/phoneme) it's clearer isn't it? Don't confuse with syllable through. We can take an example. The word 'through'(1 syllable) consists of three sounds, three phonemes: 'th' 'r' 'oo'. So the goal is to model the sounds as a sequence of phonemes.

 - The second one is the language model, which we want to build automatically. It models the distribution of words for a given language. Thanks to those probabilities, the LM helps in picking the best correspondence between a sequence of phonemes and the words/sentences.


In order to build these modules, we need data and the more we have and the more they are relevant, the better! That's why we need data selection: we need a mean to retrieve relevant data adapted to the context of the recognition. In fact a lawyer and a baker don't speak the same language: they don't use the same lexicon. Data selection is picking the good data that match the domain within millions of examples through the usage of various automatic algorithms.



## How we used to do

As we discussed above, data selection is a very important step while building a system. Like many, we used the [Moore-Lewis](http://www.aclweb.org/anthology/P10-2041) method which was also adapted for bilingual use (like translation) by Axelrod et al. in [Domain Adaptation via Pseudo In-Domain Data Selection](https://aclanthology.info/pdf/D/D11/D11-1033.pdf). These are very effective ways to select data using two corpora (in and out domain) by comparing cross-entropies. In-domain meaning that the corpus have specific data, that are relevant with the context, the domain of recognition as explained before with the lawyer/baker thing. Whereas out-of-domain is just a pool of random data, meaning there is relevant and no-relevant data in it! Then about cross-entropy, it's a measure that help choosing well-matched data for the desired output. Thanks to some relevant segment, we compare each segment in the pool of data to retrieve the closest ones to the initial data.

![example of data selection](/assets/DataSelection.jpg)

So, using the cross-entropy to select, it's not really scalable because the algorithm can't decide when to stop on its own and he has an annoying tendency to promote very short to short sentences meaning our corpus isn't really relevant with conversations. Moreover, something hit us hard. [This paper](http://www.aclweb.org/anthology/P10-2041) turned out to be eight this year and we have never looked for another method before. So we asked ourselves: has any new work been done in data selection since this paper? And is there any relevant work ready for a more industrialized turn?



## Searching.... Finding!

After browsing 178 papers quoting the Moore-Lewis one, a title caught our eyes: [Cynical Selection of Language Model Training Data](https://arxiv.org/pdf/1709.02279.pdf). The name was so catchy, we had to explore it. Written by Amittai Axelrod (remember we mentioned him above), we decided to give it a shot [here](https://github.com/allo-media/cynical-selection) because the paper was full of good promises ... And seemed compatible with industrialization! Unlike the previous methods, the algorithm stops by itself when it has the (supposed) optimal selection, letting us continue our road toward automating.



## How does it work? How did we make it work?

The goal is to select data from our out-of-domain corpora that can extend our in-domain data. Suppose you have a small in-domain corpora, which you are a hundred percent positive that is representative. The algorithm will take this corpus and a more generic one, where you don't know what's relevant or not. It will then select the sentences that match the specific one using an implementation of the Alexrod's paper cited above. The script can take arguments which are detailed in the header of the script. It only requires the two corpora to work:

`./cynical-selection.py --task inDomainFile.txt --unadapted outDomainFile.txt`

and returns you a list of sentences along with their scores in a '.jaded' file constructed as follows:

`model score sentence score (penalty + gain) length penalty sentence gain sentence id (in the selection) sentence id (in the unadapted corpora) best word word gain sentence`
for example: 

`2.659289425334946       2.659289425334946       5.71042701737487        -3.0511375920399235     1       1       vous    -0.12597986190092164    merci à vous tous`

`5.318578850669892       2.659289425334946       5.71042701737487        -3.0511375920399235     2       26978   vous    -0.12597986190092164    et vous avez maintenant`

`7.9778682760048385      2.659289425334946       5.71042701737487        -3.0511375920399235     3       26979   vous    -0.12597986190092164    puisque vous avez des`

In the end, we didn't lose any performance using this method, we even gained accuracy most of the time. But the important part is that it allowed us to automatize this treatment, taking us one step closer to industrialization.



## To conclude

This method allows us to focus on other parts of our systems, making us more productive and more serene towards the building of language model. So it's a success captain!
