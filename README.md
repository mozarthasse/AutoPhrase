# AutoPhrase: Automated Phrase Mining from Massive Text Corpora

## Publications

Please cite the following two papers if you are using our tools. Thanks!

*   Jingbo Shang, Jialu Liu, Meng Jiang, Xiang Ren, Clare R Voss, Jiawei Han, "**[Automated Phrase Mining from Massive Text Corpora](https://arxiv.org/abs/1702.04457)**", accepted by IEEE Transactions on Knowledge and Data Engineering, Feb. 2018.

*   Jialu Liu\*, Jingbo Shang\*, Chi Wang, Xiang Ren and Jiawei Han, "**[Mining Quality Phrases from Massive Text Corpora](http://hanj.cs.illinois.edu/pdf/sigmod15_jliu.pdf)**”, Proc. of 2015 ACM SIGMOD Int. Conf. on Management of Data (SIGMOD'15), Melbourne, Australia, May 2015. (\* equally contributed, [slides](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/sigmod15SegPhrase.pdf))

## Recent Changes 

### 2018.03.04
*   Fix a few bugs during the pre-processing and post-processing, i.e., ```Tokeninzer.java```. Previously, when the corpus contains characters like ```/```, the results could be wrong or errors may occur.
*   When the phrasal segmentation is serving new text, for the phrases (every token is seen in the traning corpus) provided in the knowledge base (```wiki_quality.txt```), the score is set as ```1.0```. Previously, it was kind of infinite.

### 2017.10.23
*   Support extremely large corpus (e.g., 100GB or more). Please comment out the ```// define LARGE``` in the beginning of ```src/utils/parameters.h``` before you run AutoPhrase on such a large corpus.
*   Quality phrases (every token is seen in the raw corpus) provided in the knowledge base will be incorporated during the phrasal segmentation, even their frequencies are smaller than ```MIN_SUP```.
*   Stopwords will be treated as low quality single-word phrases.
*   Model files are saved separately. Please check the variable ```MODEL``` in both ```auto_phrase.sh``` and ```phrasal_segmentation.sh```.
*   The end of line is also a separator for sentence splitting.

## New Features
(compared to SegPhrase)

*   **Minimized Human Effort**. We develop a robust positive-only distant training method to estimate the phrase quality by leveraging exsiting general knowledge bases.
*   **Support Multiple Languages: English, Spanish, and Chinese**. The language
    in the input will be automatically detected.
*   **High Accuracy**. We propose a POS-guided phrasal segmentation model incorporating POS tags when POS tagger is available. Meanwhile, the new framework is able to extract single-word quality phrases.
*   **High Efficiency**. A better indexing and an almost lock-free parallelization are implemented, which lead to both running time speedup and memory saving.

## Related GitHub Repositories

*   [SegPhrase](https://github.com/shangjingbo1226/SegPhrase)
*	[SegPhrase-MultiLingual](https://github.com/remenberl/SegPhrase-MultiLingual)

## Requirements

Linux or MacOS with g++ and Java installed.

Ubuntu:

* g++ 4.8 `$ sudo apt-get install g++-4.8`
* Java 8 `$ sudo apt-get install openjdk-8-jdk`
* curl `$ sudo apt-get install curl`
* ant `$ sudo apt-get install ant`

MacOS:

*   g++ 6 `$ brew install gcc6`
*   Java 8 `$ brew update; brew tap caskroom/cask; brew install Caskroom/cask/java`

## Default Run

The default run will download an English corpus from the server of our data
mining group and run AutoPhrase to get 3 ranked lists of phrases as well as 2 segmentation model files under the
```MODEL``` (i.e., ```models/DBLP```) directory. 

When using other languages, put the corresponding text corpus in the file ```data/DBLP.txt```.

#### Phrase Mining Step
```
$ ./auto_phrase.sh
```

Output files:

* ```AutoPhrase.txt```: the unified ranked list for both single-word phrases and multi-word phrases. 
* ```AutoPhrase_multi-words.txt```: the sub-ranked list for multi-word phrases only. 
* ```AutoPhrase_single-word.txt```: the sub-ranked list for single-word phrases only.
* ```segmentation.model```: AutoPhrase's segmentation model (saved for later use).
* ```token_mapping.txt```: the token mapping file for the tokenizer (saved for later use).

You can change ```RAW_TRAIN``` to your own corpus and you may also want change ```MODEL``` to a different name.

#### Phrasal Segmentation

We also provide an auxiliary function to highlight the phrases in context based on our phrasal segmentation model. There are two thresholds you can tune in the top of the script. The model can also handle unknown tokens (i.e., tokens which are not occurred in the phrase mining step's corpus).

In the beginning, you need to specify AutoPhrase's segmentation model, i.e., ```MODEL```. The default value is set to be consistent with ```auto_phrase.sh```.

```
$ ./phrasal_segmentation.sh
```

The segmentation results will be put under the ```MODEL``` directory as well (i.e., ```model/DBLP/segmentation.txt```). The highlighted phrases will be enclosed by the phrase tags (e.g., ```<phrase>data mining</phrase>```).

## Incorporate Domain-Specific Knowledge Bases

If domain-specific knowledge bases are available, such as MeSH terms, there are two ways to incorporate them.
* (**recommended**) Append your known quality phrases to the file ```data/EN/wiki_quality.txt```.
* Replace the file ```data/EN/wiki_quality.txt``` by your known quality phrases.

## Handle Other Languages

#### Tokenizer and POS tagger

In fact, our tokenizer supports many different languages, including Arabics (AR), German (DE), English (EN), Spanish (ES), French (FR), Italian (IT), Japanese (JA), Portuguese (PT), Russian (RU), and Chinese (CN). If the language detection is wrong, you can also manually specify the language by modify the ```TOKENIZER``` command in the bash script ```auto_phrase.sh``` using the two-letter code for that language. For example, the following one forces the language to be English.
```
TOKENIZER="-cp .:tools/tokenizer/lib/*:tools/tokenizer/resources/:tools/tokenizer/build/ Tokenizer -l EN"
```

We also provide a default tokenizer together with a dummy POS tagger in the ```tools/tokenizer```. Build it from ```tools/tokenizer```:

`ant clean-build`

It uses the StandardTokenizer in Lucene, and always assign a tag ```UNKNOWN``` to each token.
To enable this feature, please add the ```-l OTHER"``` to the ```TOKENIZER``` command in the bash script ```auto_phrase.sh```.
```
TOKENIZER="-cp .:tools/tokenizer/lib/*:tools/tokenizer/resources/:tools/tokenizer/build/ Tokenizer -l OTHER"
```

If you want to incorporate your own tokenizer and/or POS tagger, please create a new class extending SpecialTagger in the ```tools/tokenizer```. You may refer to StandardTagger as an example.

#### stopwords.txt

You may try to search online or create your own list.

#### wiki_all.txt and wiki_quality.txt

Meanwhile, you have to add two lists of quality phrases in the ```data/OTHER/wiki_quality.txt``` and ```data/OTHER/wiki_all.txt```. 
The quality of phrases in wiki_quality should be very confident, while wiki_all, as its superset, could be a little noisy. For more details, please refer to the [tools/wiki_enities](https://github.com/shangjingbo1226/AutoPhrase/tree/master/tools/wiki_entities).

## Docker

###Default Run

```
sudo docker run -v $PWD/results:/autophrase/results -it \
    -e FIRST_RUN=1 -e ENABLE_POS_TAGGING=1 \
    -e MIN_SUP=30 -e THREAD=10 \
    remenberl/autophrase

./autophrase.sh
```

The results will be available in the results folder.

###User Specified Input
Assuming the path to input file is ./data/input.txt.
```
sudo docker run -v $PWD/data:/autophrase/data -v $PWD/results:/autophrase/results -it \
    -e RAW_TRAIN=data/input.txt \
    -e FIRST_RUN=1 -e ENABLE_POS_TAGGING=1 \
    -e MIN_SUP=30 -e THREAD=10 \
    remenberl/autophrase

./autophrase.sh
```
