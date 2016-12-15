## Synopsis

The aim of this work was to create an efficient software for driving large scale experiments on Learning To Rank algorithms.
It is based on Terrier SE (http://terrier.org) and currently uses 3 open-source libraries of LTR algorithm:

- RankLib (https://sourceforge.net/p/lemur/wiki/RankLib/)
- QuickRank (https://github.com/hpclab/quickrank)
- JForests (https://github.com/yasserg/jforests/blob/master/README.md)

The currently implemented algorithms are:

- mart [1]
- ranknet [2]
- rankboost [3]
- adarank [4]
- coordinateascent [5]
- lambdamart [6]
- listnet [7]
- randomforest [8]
- linesearch [9]
- gradientboostingbinaryclassifier [10]


## The software

Starting from runs generated with classical search-engines we perform reranking according to the specified algorithms to conduct large scale experiments on the efficiency of LTR algorithm

In order to make our software as efficient as possible we build it in two main module:

- *Features Extraction* is responsable for pre-computing LETOR features from a Terrier generated index and save them in efficient byte tables on disk
- *LTR Algorithms Execution* is responsable for the execution of LTR algorithms and saving of the reranked results 

The code is compiled using Java 8.

## How to use

In this guide it is assumed that Terrier is installed on your machine. RankLib, QuickRank and JForests are included in the software but you can use different versions modifying the properties file.
We refer to the official sites for installation guides and we give here a brief step-by-step guide for running our software.

For this software we assume that you have Terrier SE working. We refer to the "Installing and Running Terrier" guide (http://terrier.org/docs/v4.1/quickstart.html).

We assume that the operating system is Linux or Mac/OSX and that the collection is stored in the directory `/data/collection/`. 

Topics file is in the standard TREC format while qrel file has the format:

> query-number    0    document-id    relevance

Run files must also be in the standard format:

> query-number    Q0    document-id    rank    score    run-name


Once you have defined the corpus of documents you want to index you have to setup Terrier for using TREC test collections

1. Go to Terrier folder

 `cd terrier-core-4.1`

2. Setup Terrier for using a TREC test collection

 `./bin/trec_setup.sh /data/collection/`

3. Index the collection

 `./bin/trec_terrier.sh -i`

With Terrier's default settings, the resulting index will be created in the var/index folder within the Terrier installation folder.

**IMPORTANT:** To use our software you have to know the path to the folder where the index is created along with the path to the terrier.properties file used for indexing (It will be used to extract fields informations and topic processing informations)


## Features Extraction

Here we explain the configuration to use our first module to precompute LETOR features on disk. The full working of the software is controlled by java properties file.


We give a brief explanation of the configuration properties. We refer to the well-commented example of properties file for further explanation.

General configuration:

`terrier_home` Path to Terrier installation folder

`index` Path to the generated index

`terrier_properties` Path to Terrier properties file

`runsFolder` Path to the folder that contains the runs for which we want to computer letor features

`pool` Path to qrel file

`topics` Path to topics file

`logger_level` Define the logging level

Output configurations:

- `table_path` Where to save the generated tables containing the computed letor

- `pointers_path` Where to save the generated files containing positional informations (hashmap qid/byte_offset)


`features` Define which features to compute. Every used feature need to be defined with `classname` and `datatype`.

The syntax is as follow:

`terrier.<feature-name>`				For features computed with Terrier.

`features.<feature-name>`				For features defined in this software.

`features.<feature-name>-<field_name>`	For features defined in this software computed on a single field.

`features.<feature-name>.class`				For features classname.

`features.<feature-name>.datatype`				For features datatype ( java.lang.Integer | java.lang.Double)



Once in the jar directory run with

`java -jar Picello1.jar path_to_properties_file`

If the path to properties file is not passed the dafault location will be used (resources/config.properties)

## Picello2

Here we explain how to use module2 to perform re-ranking starting from the files generated by module1 execution. The full working of the software is controlled by java properties file.


We give a brief explanation of the configuration properties. We refer to the well-commented example of properties file for further explanation.

General configuration:

`runsFolder` Path to the folder that contains the runs for which we want to compute letor features (same as Picello1)

`tables_path` Path to the tables generated by Picello1

`pointers_path` Path to the pointers file generated by Picello1

Output configurations:

`out_path` Output directory. There will be two folders: `out_path/runs` contains re-ranked versions of the run files in runsFolder, `out_path_info` contains informations on the generated Letor files

Learning To Rank configurations:

`train` Specify topics used as training set (e.g. `351,367,368,388,..`)

`validation` Specify topics used as validation set 

`test` Specify topics used as test set

`used_features` Specify which of the precomputed features to use for this execution

`algorithms` LTR algorithms to lunch in this execution (The syntax is as follow: `algorithms=<algo-name>,<algo-name>,...,<algo-name>`)
Possible values are:

- mart
- ranknet
- rankboost
- adarank
- coordinateascent
- lambdamart
- listnet
- randomforest
- linesearch
- gradientboostingbinaryclassifier

For configuration options specific to each algorithm refer to libraries sites. 
The syntax is as follow:

`<algorithm>.<parameter>=`						For options common to every library.

`<algorithm>.<ltr-library-name>.<parameter>=`		For options specific to a single library.

`<algorithm>.libraries` Specify with which libraries execute this algorithm (e.g. `mart.libraries=ranklib,quickrank`)

Once in the jar directory run with

`java -jar Picello2.jar path_to_properties_file`

If the path to properties file is not passed the dafault location will be used (resources/config.properties)

Results will be written in `out_path/runs` following this name notation: `<run_name>_<algorithm_name>_<library_name>`.


## Contributors

We welcome comments and contribution!

For any question write to `paolopicelloit@gmail.com`

## License

A short snippet describing the license (MIT, Apache, etc.)

## References

[1] J.H. Friedman. Greedy function approximation: A gradient boosting machine. Technical Report, IMS Reitz Lecture, Stanford, 1999; see also Annals of Statistics, 2001.

[2] C.J.C. Burges, T. Shaked, E. Renshaw, A. Lazier, M. Deeds, N. Hamilton and G. Hullender. Learning to rank using gradient descent. In Proc. of ICML, pages 89-96, 2005.

[3] Y. Freund, R. Iyer, R. Schapire, and Y. Singer. An efficient boosting algorithm for combining preferences. The Journal of Machine Learning Research, 4: 933-969, 2003.

[4] J. Xu and H. Li. AdaRank: a boosting algorithm for information retrieval. In Proc. of SIGIR, pages 391-398, 2007.

[5] D. Metzler and W.B. Croft. Linear feature-based models for information retrieval. Information Retrieval, 10(3): 257-274, 2007.

[6] Q. Wu, C.J.C. Burges, K. Svore and J. Gao. Adapting Boosting for Information Retrieval Measures. Journal of Information Retrieval, 2007.

[7] Z. Cao, T. Qin, T.Y. Liu, M. Tsai and H. Li. Learning to Rank: From Pairwise Approach to Listwise Approach. ICML 2007. 

[8] L. Breiman. Random Forests. Machine Learning 45 (1): 5–32, 2001.

[9] D. G. Luenberger. Linear and nonlinear programming. Addison Wesley, 1984.

[10] Y. Ganjisaffar, R. Caruana, C. Lopes, Bagging Gradient-Boosted Trees for High Precision, Low Variance Ranking Models, in SIGIR 2011, Beijing, China.

[11] Capannini, G., Lucchese, C., Nardini, F. M., Orlando, S., Perego, R., and Tonellotto, N. Quality versus efficiency in document scoring with learning-to-rank models. Information Processing & Management, 2016.
