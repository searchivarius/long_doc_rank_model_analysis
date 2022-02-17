# This repository accompanies a paper to provide instructions (and further details) to reproduce results

*Understanding Performance of Long-Document Ranking Models through Comprehensive Evaluation and Leaderboarding*, 

Leonid Boytsov, Tianyi Lin, Fangwei Gao, Yutian Zhao, Jeffrey Huang, Eric Nyberg.

All the models except neural Model 1 (which we were not able to release due to patenting/licensing issues) 
are implemented in our [FlexNeuART framework](https://github.com/oaqa/FlexNeuART/tree/pypi2021), 
currently it is not the branch `pypi2021`, not the main branch! One has to compile and install this branch
locally rather than from `pypi`. 

*Note on dependencies:* there is currently some issue 
with their installation and one may need to run `pyhon setup.py` twice.

The framework is used out-of-the-box and provides documentations regarding indexing collections and training 
the models.

Crucially:
1. The list of models with corresponding configuration files can be [found in this folder](model_conf).
2. The list of queries for both MS MARCO (documents) and Robust04 (with five folds), which be directly used in FlexNeuART without further processing can be found [here](queries).
3. We make available our main preprocesed training set for MS MARCO (documents). 
Yet, we cannot do the same for Robust04, because obtaining Robust04 documents requires signing a 
licensing agreement with NIST.

# Data processing and model training

The model training & evaluation pipeline has the following steps:
1. Downloading and indexing data (collection specific)
2. Organizing data into sub-folders and pointing environmental variable `COLLECT_ROOT` to the root-directory, 
containing potentially multiple collection sub-directories.
3. Creating indexes:
   1. retrieval index, e.g., Lucene
   2. forward index: here we always use the field `raw_text` whose field type is `textRaw'.
4. Exporting training (we provide a sample script for MS MARCO)
5. Training the model
6. Optionally validating it on a number of query sets, unless the exported data contains a complete testing
set itself.
7. In the case of Robust04 collections, we evaluate performance in the cross-validating mode. Thus,
one trains a model for each sub-folder and then merges output runs.

# MS MARCO
## Data processing for MS MARCO

We experiment with two MS MARCO collections: v1 and v2. They have 
collection specific conversion scripts which 
can be found in respective sub-directories of the [data_convert](https://github.com/oaqa/FlexNeuART/tree/master/scripts/data_convert).
directory. 

Importantly, to generate training data and reproduce all results, one needs
to create Lucene index that combines `doc2query information`. The respective scripts
can be found [here](data_convert/msmarco/add_doc2query_docs.py).
FlexNeuART input data is organized into multiple-fields, the default searchable field is `text`
and this is the field that needs to be expanded using `doc2query`. The expansion process will
create a new text field, which can be then index using Lucene.

**Note on FlexNeuART field terminology:** although it may be worth renaming some fields in the future,
currently both the query and the document ID are encoded using the field name `DOCNO`. When
`DOCNO` is present in `AnswerFields.jsonl` file, it is a document ID, when it is present 
in `QueryFields.jsonl` it denotes a query ID. 

For MS MARCO v2, we used a more involved `dense-sparse` retrieval. 
However, because we do not train on MS MARCO v2 data and only evaluate on MS MARCO v2,
we simply provide the output (in the form of a compressed run in TREC NIST format)
for the test queries from TREC DL 2021. Such runs are also provided for 
Robust04 and MS MARCO v1 and [they are all stored in this folder of this repository](trec_runs_cached).
Note that FlexNeuART can use such runs directly as a replacement of the first-stage retriever:
it matches cached runs using query ID. 

## Data processing for MS MARCO v1 and v2

To simplify reproduction, we share our main training set. 
Assume that the root folder for all collections is the folder `collections`:
```
mkdier <some top-level-path>/collections
export COLLECT_ROOT=<some top-level-path>/collections
```
Create a directory for MS MARCO v1 data files and download the prepared training set:
```
mkdir -p $COLLECT_ROOT/msmarco_v1/derived_data
cd $COLLECT_ROOT/msmarco_v1/derived_data
wget boytsov.info/datasets/flexneuart/msmarco_long_doc/cedr_mcds_100_50_0_5_0_s0_bitext_2021-11-17.tar.bz2
tar jxvf cedr_mcds_100_50_0_5_0_s0_bitext_2021-11-17.tar.bz2
```
In addition to training data, you need to copy model files from this repository:
```
cp -r <this repo location on disk>/model_conf/main/ $COLLECT_ROOT/msmarco_v1/model_conf

```

Each model has its unique FlexNeuART code: Parameters are specified via configuration file.
The combinations used in this paper are provided [here](model_conf/README.md).
Most models train on a GPU with 11 GB memory including PARADE models with a given
values of a sliding window width and stride. However, if you substnatially decrease the window size,
you may need a bigger-memory GPU. Likewise, a full-length Transformer needs more memory.
BTW, the full-length document is truncated to have at most 1471 subword tokens.
Further increase in the input length has virtually no effect on model performance.

Assume we chose to train a basic BERT `FirstP` model, which has the code `vanilla_bert`
and a config `model_conf/config_short.json`. Then training can be executed as follows. 
Go to the directory where you installed FlexNeuART additional scripts and run the training script:
```
./train_nn/train_model.sh  \
    msmarco_v1 \
    cedr_mcds_100_50_0_5_0_s0_bitext_2021-11-17/text_raw \
    vanilla_bert \
    -json_conf model_conf/config_short.json
```

This script will run for one epoch (more does not help) and produces output (model including) in the following
directory:
```
$COLLECT_ROOT/msmarco_v1/derived_data/ir_models/vanilla_bert/0
```
It computes the value of the MAP as well, however, this is done for a test set sample.
To fully validate the trained model one needs to use the following:
1. A processed MS MARCO collection with an indexed (forward index type `textRaw`) field `text_raw`
2. A run that represents a test part of interest, e.g., [TREC DL 2019 run](trec_runs_cached/test2019).
3. A processed query file, e.g., [this one](queries/test2019) for TREC DL 2019.
4. A relevance information (QREL) file (provided except for TREC DL 2021).
5. The script `train_nn/eval_model.py`

The validation procedure for MS MARCO v2 and TREC DL 2021 is analogous, but you need to
obtain QREL file on your own: At the moment of the submission, these QREL files are not
available to people who did not participate in TREC.

## Training data for Robust04

Because Robust04 is not available for download (unless you sign an agreement), we cannot provide
a precomputed training set, but we provide the following:
1. Preprocessed [queries and qrels](queries/robust04).
2. [First-stage runs for fields `title` and `description`](trec_runs_cached/robust04).

The indexing/training pipeline for Robust04 is covered by generic FlexNeuART documentation.
One important difference though is that the data conversion uses [a generic conversion 
script](https://github.com/oaqa/FlexNeuART/blob/pypi2021/scripts/data_convert/ir_datasets/README.md),
which relies on [`ir_datases`](https://ir-datasets.com/).