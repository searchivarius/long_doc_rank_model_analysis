# REPRODUCIBILITY GUIDELINES

This repository accompanies a paper to facilitate its reproducibility:

[Understanding Performance of Long-Document Ranking Models through Comprehensive Evaluation and Leaderboarding, 
2022](https://arxiv.org/abs/2207.01262). Leonid Boytsov, Tianyi Lin, Fangwei Gao, Yutian Zhao, Jeffrey Huang, Eric Nyberg.

All the models except neural Model 1 (which we were not able to release due to patenting/licensing issues) 
are implemented in our [FlexNeuART framework](https://github.com/oaqa/FlexNeuART/tree/pypi2021).
**Important notes:** 

1. Please, use a special branch `pypi2021`, rather the main branch! One has to compile and install this branch
locally rather than from `pypi`. 
2. For MS MARCO we provide already pre-processed training data for the main experiment, but not for the evalution in
the "leaderboarding" mode (when we train using three progressively improving candidate generators).

The framework is used out-of-the-box and provides documentations regarding **installation**, indexing collections, and training 
the models.

Crucially:
1. The list of models with corresponding configuration files can be [found in this folder](model_conf).
2. The list of queries for both MS MARCO (documents) and Robust04 (with five folds) can be found [here](queries). This should simplify reproduction, because conversion of documents is relatively straightforward, but creation of training/testing folds for Robust04 is more involved.
3. The list of the so-called run files, which contain top-*K* candidates generated by the first-stage retriever can be [found in this folder](trec_runs_cached).
4. We make available already pre-processed (main) training set for MS MARCO (in the CEDR format). See the subsection ["Simplified data processing for MS MARCO v1 and v2"](#simplified-data-processing-for-ms-marco-v1-and-v2) for details.

**Important note:** We cannot provided a prepared training set for Robust04, because it would contain documents, which cannot be accessed by the general public unless they [sign a licensing agreement with NIST](https://trec.nist.gov/data/cd45/index.html).

# Data processing and model training

The model training & evaluation pipeline has the following steps:
1. Downloading and indexing data (collection specific).
2. Organizing data into sub-folders and pointing environmental variable `COLLECT_ROOT` to the root-directory, 
containing (potentially multiple) collection sub-directories.
3. Creating indexes:
   1. retrieval index, e.g., Lucene
   2. forward index: here we always use the field `raw_text` whose field type is `textRaw'.
4. Exporting/preparing training data in the CEDR format (we provide exported data for MS MARCO).
5. Training the model.
6. Optionally validating it on a number of query sets, unless the exported data contains a complete testing
set itself.
7. In the case of Robust04 collections, we evaluate performance in the cross-validation mode. Thus,
one trains and validates a model for each fold and then merges output runs.

# MS MARCO
## Full data processing for MS MARCO v1 and v2

We experiment with two MS MARCO collections: v1 and v2. They have 
collection specific conversion scripts which can be found in respective sub-directories of the [data_convert](https://github.com/oaqa/FlexNeuART/tree/master/scripts/data_convert).
directory. A simplified training procedure (with precomputed training data) is described in the next sub-section.

Importantly, to generate training data and reproduce all results, one needs
to create a Lucene index that combines original lemmatized text with `doc2query` data. The respective scripts
can be found [here](https://github.com/oaqa/FlexNeuART/tree/master/scripts/data_convert/msmarco/add_doc2query_docs.py).
FlexNeuART input data is organized into multiple-fields, the default searchable field is `text`
and this is the field that needs to be expanded using `doc2query`. The expansion process 
creates a new text field, which can be then indexed using Lucene.

**Note on FlexNeuART field terminology:** although it may be worth renaming some fields in the future,
currently both the query and the document ID are encoded using the field name `DOCNO` (which sometimes causes confusion). 
The field names are the same, but the data files are different!
When `DOCNO` is present in `AnswerFields.jsonl` file, it is a document ID, when it is present 
in `QueryFields.jsonl` it denotes a query ID. 

For MS MARCO v1, candidate generator is BM25 on expanded representations.
For MS MARCO v2, we used a more involved `dense-sparse` retriever, which linearly combines BM25 with cosine similarity 
between [ANCE embeddings](https://github.com/microsoft/ANCE). 
We  provide the output of the first-stage retriever (in the form of a compressed run in TREC NIST format) for all test query sets,
but not for training sets.
Such runs are also provided for Robust04 v1 and [they are all stored in this folder of this repository](trec_runs_cached).
Note that FlexNeuART can use such runs directly as a replacement of the first-stage retriever (runs are retrieved using query IDs). 

## Simplified data processing for MS MARCO v1 and v2

To simplify reproduction, we share our main training set (in the CEDR format). 
We do not yet release data sets for training in the "leaderboarding" mode.

Assume that the root folder for all collections is the directory `<some top-level-path>/collections`:
```
mkdir <some top-level-path>/collections
export COLLECT_ROOT=<some top-level-path>/collections
```

Then, create a directory for MS MARCO v1 data files and download the prepared training set:
```
mkdir -p $COLLECT_ROOT/msmarco_v1/derived_data
cd $COLLECT_ROOT/msmarco_v1/derived_data
wget boytsov.info/datasets/flexneuart/msmarco_long_doc/cedr_mcds_100_50_0_5_0_s0_bitext_2021-11-17.tar.bz2
tar jxvf cedr_mcds_100_50_0_5_0_s0_bitext_2021-11-17.tar.bz2
```

In addition to training data, you need to copy model **configuration** files from this repository to the collection sub-directory:
```
cp -r <this repo location on disk>/model_conf/main/ $COLLECT_ROOT/msmarco_v1/model_conf

```

Each model has its unique FlexNeuART code: Parameters are specified via a configuration file.
Except for Neural Model 1, all models and their configuration files are [described here](model_conf/README.md).
Most models train on a single GPU with 11 GB memory. 
This includes PARADE models with provided values of a sliding window size and stride. 
However, if you substnatially decrease the window size, you will need a bigger-memory GPU. 
Likewise, a full-length Longformer needs more memory (we trained it on RTX 3090 24GB).
BTW, the full-length document is still truncated to have at most 1471 subword tokens.
Further increase in the input length has virtually no effect on model performance.

Assume we chose to train a basic BERT `FirstP` model, which has the code `vanilla_bert`
and a config `model_conf/config_short.json`. 
Then training can be carried out as follows: 
Go to the directory where you installed FlexNeuART additional scripts and run the training script:
```
./train_nn/train_model.sh  \
    msmarco_v1 \
    cedr_mcds_100_50_0_5_0_s0_bitext_2021-11-17/text_raw \
    vanilla_bert \
    -json_conf model_conf/config_short.json
```

This script will run for one epoch (more does not help) and produce output (model including) in the following
directory:
```
$COLLECT_ROOT/msmarco_v1/derived_data/ir_models/vanilla_bert/0
```

The script computes the value of the MAP metric, however, this is done only for a small test set sample.
To fully validate the trained model one needs the following:

1. A processed MS MARCO collection with an indexed field `text_raw` (forward index type `textRaw`). 
2. A run that represents a test part of interest, e.g., [TREC DL 2019 run](trec_runs_cached/msmarco_v1/test2019). All runs for the first-stage retriever are provided in this repository!
3. A processed query file, e.g., [this one](queries/msmarco_v1/test2019) for TREC DL 2019.
4. A relevance information (QREL) file (provided except for TREC DL 2021).
5. The script `train_nn/eval_model.py`

The validation procedure for MS MARCO v2 and TREC DL 2021 is analogous, but you need to
obtain QREL file on your own: At the moment of the submission, these QREL files are not
available to people who did not participate in TREC 2021 (so we cannot post them either).

## Robust04

Because Robust04 is not available for download (unless you sign an agreement), we cannot provide
a precomputed training set, but we provide the following:

1. Preprocessed [queries and qrels](queries/robust04).
2. [First-stage runs for fields `title` and `description`](trec_runs_cached/robust04).

The indexing/training pipeline for Robust04 is covered by generic FlexNeuART documentation.
One important difference though is that the data conversion uses [a generic conversion 
script](https://github.com/oaqa/FlexNeuART/blob/pypi2021/scripts/data_convert/ir_datasets/README.md),
which relies on [`ir_datases`](https://ir-datasets.com/), rather than a custom script as it is 
the case of MS MARCO.
