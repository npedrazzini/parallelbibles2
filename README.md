# parallelbibles

Word-alignment models for Bible translations in 100+ historical and contemporary languages


## Requirements

1. Installation and dependencies: 
    
    - Download or clone the repository:

        `$ git clone https://github.com/npedrazzini/parallelbibles`

    - From the root directory (./parallelbibles), build the repository:

        `$ make`

    This will download and build [SyMGIZA++](https://github.com/emjotde/symgiza-pp) [[1]](#1) and install all the required dependencies in a [venv](https://docs.python.org/3/library/venv.html) called *parallels-venv*.

2. XML files, which can be of two formats:
 
    - OPUS (untokenized) (from https://opus.nlpl.eu/bible-uedin.php)

    - PROIEL (from https://proiel.github.io)

    This repository comes with OPUS XMLs (inside *original-xmls/opus-xmls*) and PROIEL XMLs for New Testament Greek, Old Church Slavonic and Gothic (inside *original-xmls/proiel-xmls*).

## Train word-alignment models

> This repository already comes with four [pre-trained models](https://github.com/npedrazzini/parallelbibles#pretrained-models). Check them out!

`$ ./train.sh`

This step will: 
1. convert OPUS/PROIEL XML files to GIZA-readable CSV files
2. train a word-alignment model for each target language
3. make GIZA's outputs easily readable and queryable 

You will be prompted to:
1. specify the input XML format (OPUS, PROIEL, or mixed)
2. enter the desired source language
3. enter the target languages (or have all the remaining as targets)
4. specify if you want to strip punctuation
5. specify if you want to bring everything to lowercase
6. provide a name for your model

NB: the chosen languages must be entered in their ISO 639-3 code. See [here](https://iso639-3.sil.org/code_tables/639/read) for the complete list and the [table](https://github.com/npedrazzini/parallelbibles#languages) below for the languages included in the models.

## Extract words and their translations 

`$ ./extract.sh`

This step will:
1. extract every occurrence of a word (or multiple words) in the source language and its translation in the target languages.
2. (optionally) generate scripts to run multidimensional scaling (MDS) on the dataset and Kriging (to draw lines around clusters probabilstically)

You will be prompted to enter:
1. the name of the model you want to use (e.g. 'model2-LC-NP')
2. a target word (e.g. 'when') or multiple target words separated by hyphen (e.g. 'when-while-since')
3. whether you want to generate the scripts necessary to run MDS on the dataset ('yes' or 'no')
4. whether you also want to apply Kriging to the MDS maps ('yes' or 'no')
5. whether you only want to extract words from the New Testament ('yes') or from both the Old and the New Testament ('no') <sup>[*](#myfootnote1)</sup>

The output will be a folder named as the target word (or words, hyphen-separated, if extracting multiple words at once) containing the following: 
1. **word.csv**: CSV file for each word. The file will contain one occurrence per line, its citation (Bible verse), context, and the translations in each target language <sup>[**](#myfootnote2)</sup>.

And if you chose to run MDS (with and without Kriging) it will also contain:

2. **word-MDS.R**: an R script to run MDS (and Kriging, if you chose to), generating a single PDF with one map per language. These maps are static and generated using base R. Best for distant-reading stages in the data exploration <sup>[***](#myfootnote3)</sup>.
3. **word-plotly.R**: an R script (alternative to *word-MDS.R*) generating multiple HTML files using the R package plotly. These maps are interactive and let you hover over the data points and look at the citation (Bible verse) and source word in context. Best for close-reading stages in the data exploration.
4. **word-data.txt**: the original data in TXT format and the citation (Bible verse) as index (rather than column, as in word.csv) and without the 'context' column.
5. **word-matrix.txt**: distance matrix between source word and target words.

<a name="myfootnote1">*</a> This is because many languages lack the whole or large sections of the Old Testament, which will result in your dataset having many NAs (which you may or may not want to avoid).

<a name="myfootnote2">**</a> **NB**: *NULL* will indicate that the model did not find a match for the word in the target language. *NA* will indicate that the target language did not have a Bible translations of that particular verse in the first place (e.g. some languages lack a translation for the whole Old Testament). 

<a name="myfootnote3">***</a> **NB 1**: This script is a heavy adaptation of the code by [[2]](#2). **NB 2**: The `lmap` function relies on the R package [qlcVisualize](https://rdrr.io/github/cysouw/qlcVisualize/). If you have issues installing it, simply save the two functions we need from that package by running the script *./scripts/postprocessing/lmap-boundary-functions.R* included in this repository. **NB 3**: The MDS script has been adapted so that it merges all translations with less than 10 occurrences with NULLs. The '10' threshold is arbitrary and was based on what seemed to be a common cut-off point between 'real' translations in the target language and casual correspondence between the source word and a specific lexical item in the target language. 

## Hierarchical clusters and NeighborNets

`./scripts/postprocessing/splitstree.R`: this script will perform hierarchical clustering and NeighborNet analysis of the languages based on a criterion *x* (default: NULL-constructions).

It takes as input the file *word-data.txt* described above.

The script will: 
1. Plot a simple hierarchical cluster of the languages in a parallel-word dataset. It currently shows how similar languages appear to be based on NULL-construction distributions.
2. Generate a Nexus (.nex) file for NeighborNet analysis, to be visualized with the [SplitsTree4](https://uni-tuebingen.de/en/fakultaeten/mathematisch-naturwissenschaftliche-fakultaet/fachbereiche/informatik/lehrstuehle/algorithms-in-bioinformatics/software/splitstree/) software. Similar to a traditional hierarchical cluster in many ways, a NeighborNet will simply not force a binary-tree type of classification.

# Pretrained models

> NB: *model2-LC-NP* is stored in this repo using [Git LFS](https://git-lfs.github.com). If you wish to use that model, you should have Git LFS installed, else you will only see a pointer file.

Four pretrained models currently come with this repository: 

1. *model1-UC-P*: **U**pper case and with **P**unctuation. English is source language. All other languages (both from OPUS and PROIEL; however see TODO) are targets.
2. *model2-LC-NP*: **L**ower **C**ase and **N**o **P**unctuation. English is source language. All other languages (both from OPUS and PROIEL; however see TODO) are targets.
3. *model3-UC-NP*: **U**pper **C**ase and **N**o **P**unctuation. English is source language. All other languages (both from OPUS and PROIEL; however see TODO) are targets.
4. *model4-LC-P*: **L**ower **C**ase and **N**o **P**unctuation. English is source language. All other languages (both from OPUS and PROIEL; however see TODO) are targets.

You can directly extract target words from either of these models by running `$ ./extract.sh`. You will be prompted to enter the name of the model you want to use.

# Languages
**OT** = Old Testament

**NT** = New Testament

| ISO 639-3 | Language | Language family |  OT | NT |Notes |
|---|---|---|---|---|---|
| acu | [Achuar-Shiwiar](https://www.ethnologue.com/language/acu) | Jivaroan |  N | Y  |   |
| afr | [Afrikaans](https://www.ethnologue.com/language/afr) | Indo-European > Germanic |  Y | Y  |   |
| agr | [Awajún](https://www.ethnologue.com/language/agr) | Jivaroan |  N |  Y |   |
| ake | [Akawaio](https://www.ethnologue.com/language/ake) | Cariban |  N |  Y |   |
| sqi/alb | [Albanian](https://www.ethnologue.com/language/sqi) | Indo-European|  Y |   Y|   |
| amh | [Amharic](https://www.ethnologue.com/language/amh) | Afro-Asiatic > Semitic | Y |  N |   |
| amu | [Guerrero Amuzgo](https://www.ethnologue.com/language/amu) | Otomanguean |  N | Y  |   |
| ara | [Arabic](https://www.ethnologue.com/language/ara) | Afro-Asiatic > Semitic | Y  |  Y |   |
| hye/arm | [Armenian](https://www.ethnologue.com/language/hye) | Indo-European |  Y | Y  |   |
| baq | [Basque](https://www.ethnologue.com/language/eus) | Isolate |  N |  Y |   |
| bsn | [Barasana-Eduria](https://www.ethnologue.com/language/bsn) | Tucanoan | N  |  Y |   |
| bul | [Bulgarian](https://www.ethnologue.com/language/bul) | Indo-European > Balto-Slavic |  Y | Y  |   |
| cak | [Kaqchikel](https://www.ethnologue.com/language/cak) | Mayan | N  | Y  |   |
| ceb | [Cebuano](https://www.ethnologue.com/language/ceb) | Austronesian > Malayo-Polynesian |  Y | Y  |   |
| cha | [Chamorro](https://www.ethnologue.com/language/cha) | Austronesian > Malayo-Polynesian |  Y |  Y |  OT only consists of the Psalms |
| zho/chi | [Chinese](https://www.ethnologue.com/language/zho) | Sino-Tibetan > Sinitic | Y  |  Y |   |
| chq | [Quiotepec Chinantec](https://www.ethnologue.com/language/chq) | Otomanguean | N  |  Y|   |
| chr | [Cherokee](https://www.ethnologue.com/language/chr) | Iroquoian |  N |  Y |   |
| chu | [Church Slavonic](https://www.ethnologue.com/language/chu) | Indo-European > Balto-Slavic|  N | Y  |   |
| cjp | [Cabécar](https://www.ethnologue.com/language/cjp) | Chibchan | N  | Y  |   |
| cni | [Asháninka](https://www.ethnologue.com/language/cni) | Maipurean | N  |  Y |   |
| cop | [Coptic](https://www.ethnologue.com/language/cop) | Afro-Asiatic > Egyptian | N  | Y  |   |
| crp | [Creoles and pidgins](https://www.ethnologue.com/language/hat/24) | Creole > French-based |  Y | Y  | The original XML files have the generic 'crp' code. This is however Haitian Creole (code hat) |
| cze | [Czech](https://www.ethnologue.com/language/ces) | Indo-European > Balto-Slavic| Y  | Y  |   |
| dan | [Danish](https://www.ethnologue.com/language/dan) | Indo-European > Germanic| Y  |  Y |   |
| deu | [German](https://www.ethnologue.com/language/deu) | Indo-European > Germanic|  Y |  Y |   |
| dik | [Southwestern Dinka](https://www.ethnologue.com/language/dik) | Nilo-Saharan > Nilotic |  N |  Y |   |
| dje | [Zarma](https://www.ethnologue.com/language/dje) |  Nilo-Saharan > Songhai | Y  | Y  |   |
| dop | [Lukpa](https://www.ethnologue.com/language/dop) | Niger-Congo > Atlantic-Congo | N  | Y  |   |
| epo | [Esperanto](https://www.ethnologue.com/language/epo) | Constructed |  Y |  Y |   |
| est | [Estonian](https://www.ethnologue.com/language/est) | Uralic | Y  |  Y |   |
| ewe | [Ewe](https://www.ethnologue.com/language/ewe) | Niger-Congo > Atlantic-Congo | N  |  Y |   |
| fin | [Finnish](https://www.ethnologue.com/language/fin) | Uralic | Y  | Y  |   |
| fra | [French](https://www.ethnologue.com/language/fra) | Indo-European > Italic|  Y | Y  |   |
| gbi | [Galela](https://www.ethnologue.com/language/gbi) | West Papuan |  N | Y  |   |
| gla | [Scottish Gaelic](https://www.ethnologue.com/language/gla) | Indo-European > Celtic|  N |  Y | The only text included is the Gospel of Mark  |
| glv | [Manx](https://www.ethnologue.com/language/glv) | Indo-European > Celtic|  Y |  Y |  The only text from the OT is the Book of Esther |
| got | Gothic | Indo-European > Germanic |  N |  Y |   |
| grc | [Ancient Greek (to 1453)](https://www.ethnologue.com/language/grc) | Indo-European | N  | Y  |   |
| ell/gre | [Modern Greek (1453-)](https://www.ethnologue.com/language/ell) | Indo-European | Y  | Y  |   |
| guj | [Gujarati](https://www.ethnologue.com/language/guj) | Indo-European > Indo-Iranian | N  | Y  |   |
| heb | [Hebrew](https://www.ethnologue.com/language/heb) | Afro-Asiatic > Semitic |  Y | N |   |
| hin | [Hindi](https://www.ethnologue.com/language/hin) | Indo-European > Indo-Iranian |  Y | Y  |   |
| hrv | [Croatian](https://www.ethnologue.com/language/hrv) | Indo-European > Balto-Slavic |  Y | Y  |   |
| hun | [Hungarian](https://www.ethnologue.com/language/hun) | Uralic | Y  | Y  |   |
| ind | [Indonesian](https://www.ethnologue.com/language/ind) | Austronesian > Malayo-Polynesian |  Y | Y  |   |
| isl | [Icelandic](https://www.ethnologue.com/language/isl) | Indo-European > Germanic |  Y |  Y |   |
| ita | [Italian](https://www.ethnologue.com/language/ita) | Indo-European > Italic| Y  | Y  |   |
| jak | [Jakun](https://www.ethnologue.com/language/jak) | Austronesian > Malayo-Polynesian | N  | Y  |   |
| jap | [Japanese](https://www.ethnologue.com/language/jpn) | Japonic |  Y |  Y |   |
| jiv | [Shuar](https://www.ethnologue.com/language/jiv) | Jivaroan |  N | Y  |   |
| kab | [Kabyle-Amazigh](https://www.ethnologue.com/language/kab) | Afro-Asiatic > Berber |  N | Y  |   |
| kbh | [Camsá](https://www.ethnologue.com/language/kbh) | Isolate | N  | Y |   |
| kor | [Korean](https://www.ethnologue.com/language/kor) | Koreanic | Y  | Y  |   |
| lat | [Latin](https://www.ethnologue.com/language/lat) | Indo-European > Italic|  Y | Y  |   |
| lav | [Latvian](https://www.ethnologue.com/language/lav) | Indo-European > Balto-Slavic| N  | Y  |   |
| lit | [Lithuanian](https://www.ethnologue.com/language/lit) | Indo-European > Balto-Slavic|  Y | Y  |   |
| mal | [Malayalam](https://www.ethnologue.com/language/mal) | Dravidian | Y  |  Y |   |
| mam | [Mam](https://www.ethnologue.com/language/mam) | Mayan |  N |  Y |   |
| mao | [Maori](https://www.ethnologue.com/language/mri) | Austronesian > Malayo-Polynesian |  Y |  Y |   |
| mar | [Marathi](https://www.ethnologue.com/language/mar) | Indo-European > Indo-Iranian |  Y |  Y |   |
| mya | [Burmese](https://www.ethnologue.com/language/mya) | Sino-Tibetan > Tibeto-Burman|  Y | Y  |   |
| nep | [Nepali](https://www.ethnologue.com/language/nep) | Indo-European > Indo-Iranian |  Y | Y  |   |
| nhg | [Tetelcingo Nahuatl](https://www.ethnologue.com/language/nhg) | Uto-Aztecan |  N | Y  |   |
| nld | [Dutch](https://www.ethnologue.com/language/nld) |  Indo-European > Germanic|  Y | Y  |   |
| nor | [Norwegian](https://www.ethnologue.com/language/nor) |  Indo-European > Germanic| Y  |  Y |   |
| ojb | [Northwestern Ojibwa](https://www.ethnologue.com/language/ojb) | Algic > Algonquian |  N |  Y |   |
| pck | [Paite Chin](https://www.ethnologue.com/language/pck) | Sino-Tibetan > Tibeto-Burman | Y  |  Y |   |
| pes | [Iranian Persian](https://www.ethnologue.com/language/pes) | Indo-European > Indo-Iranian |  Y |  Y |   |
| plt | [Plateau Malagasy](https://www.ethnologue.com/language/plt) | Austronesian > Malayo-Polynesian |  Y | Y  |   |
| pol | [Polish](https://www.ethnologue.com/language/pol) | Indo-European > Balto-Slavic|  Y |   Y|   |
| por | [Portuguese](https://www.ethnologue.com/language/por) | Indo-European > Italic|  Y | Y  |   |
| pot | [Potawatomi](https://www.ethnologue.com/language/pot) | Algic > Algonquian | N  | Y  |   |
| ppk | [Uma](https://www.ethnologue.com/language/ppk) | Austronesian > Malayo-Polynesian | N  |  Y |   |
| quc | [K'iche'](https://www.ethnologue.com/language/quc) | Mayan |  N | Y  |   |
| quw | [Tena Lowland Quichua](https://www.ethnologue.com/language/quw) | Quechuan | N  | Y  |   |
| rom | [Romany](https://www.ethnologue.com/language/rom) | Indo-European > Indo-Iranian |  N |  Y |   |
| ron/rum | [Romanian](https://www.ethnologue.com/language/ron) | Indo-European > Italic|  Y | Y  |   |
| rus | [Russian](https://www.ethnologue.com/language/rus) | Indo-European > Balto-Slavic|  Y | Y  |   |
| shi | [Tachelhit](https://www.ethnologue.com/language/shi) | Afro-Asiatic > Berber | N  | Y  |   |
| slk | [Slovak](https://www.ethnologue.com/language/slk) | Indo-European > Balto-Slavic| Y  |  Y |   |
| slv | [Slovenian](https://www.ethnologue.com/language/slv) | Indo-European > Balto-Slavic |  Y |  Y |   |
| sna | [Shona](https://www.ethnologue.com/language/sna) | Niger-Congo > Atlantic-Congo | Y  | Y |   |
| som | [Somali](https://www.ethnologue.com/language/som) | Afro-Asiatic > Cushitic | Y  |   Y|   |
| spa | [Spanish](https://www.ethnologue.com/language/spa) | Indo-European > Italic |  Y |  Y |   |
| srp | [Serbian](https://www.ethnologue.com/language/srp) | Indo-European > Balto-Slavic | Y  | Y  |   |
| ssw | [Swati](https://www.ethnologue.com/language/ssw) | Niger-Congo > Atlantic-Congo |  N |  Y |   |
| swe | [Swedish](https://www.ethnologue.com/language/swe) | Indo-European > Germanic |  Y | Y  |   |
| syr | [Syriac](https://www.ethnologue.com/language/syr) | Afro-Asiatic > Semitic |  N |  Y |   |
| tel | [Telugu](https://www.ethnologue.com/language/tel) | Dravidian | Y  | Y  |   |
| tgl | [Tagalog](https://www.ethnologue.com/language/tgl) | Austronesian > Malayo-Polynesian | Y  |  Y |   |
| tha | [Thai](https://www.ethnologue.com/language/tha) | Kra-Dai > Tai | Y  |  Y |   |
| tmh | [Tamashek](https://www.ethnologue.com/language/tmh) | Afro-Asiatic > Berber  | Y  | Y  |   |
| tur | [Turkish](https://www.ethnologue.com/language/tur) | Turkic |  Y | Y  |   |
| ukr | [Ukrainian](https://www.ethnologue.com/language/ukr) | Indo-European > Balto-Slavic | N  |  Y |   |
| usp | [Uspanteco](https://www.ethnologue.com/language/usp) | Mayan |  N |  Y |   |
| wal | [Wolaytta](https://www.ethnologue.com/language/wal) | Afro-Asiatic > Omotic |  N |  Y |   |
| wol | [Wolof](https://www.ethnologue.com/language/wol) | Niger-Congo > Atlantic-Congo |  N |  Y |   |
| xho | [Xhosa](https://www.ethnologue.com/language/xho) | Niger-Congo > Atlantic-Congo | Y  |  Y |   |
| zul | [Zulu](https://www.ethnologue.com/language/zul) | Niger-Congo > Atlantic-Congo |  N |  Y |   |

# TODO
1. Include the following languages: 
a. In all models: *vie*, *kan*, *djk*, *kek*, *agr*, *mal*
b. In *model4-LC-P* only: *mar*, *mya*, *nep*, *tel*
2. Fix issue with display of some non-Latin characters in PDF output (notably all Arabic!). Note that the characters display normally in R studio (i.e. it must be an issue with both base R *pdf* and *CairoPDF*).
3. Add info on how NULLs are treated in the models.
4. Add on how many NAs we have per language based on best model.

## References
<a id="1">[1]</a> 
Junczys-Dowmunt, Marcin & Arkadiusz Szał. 2012. SyMGiza++: Symmetrized Word Alignment Models for Machine Translation. In Pascal Bouvry, Mieczyslaw A. Klopotek, Franck Leprévost, Malgorzata Marciniak, Agnieszka Mykowiecka & Henryk Rybinski (eds.), *Security and Intelligent Information Systems (SIIS)* (Lecture Notes in Computer Science 7053), 379-390. Heidelberg-Berlin: Springer.

<a id="2">[2]</a> 
Wälchli, Bernhard. 2010. Similarity Semantics and Building Probabilistic Semantic Maps from Parallel Texts. *Linguistic Discovery* 8(1). 331-371. DOI:[10.1349/PS1.1537-0852.A.356](http://dx.doi.org/10.1349/PS1.1537-0852.A.356)