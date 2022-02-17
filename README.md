# This repository accompanies a paper to provide instructions (and further details) to reproduce results

*Understanding Performance of Long-Document Ranking Models through Comprehensive Evaluation and Leaderboarding*, 

Leonid Boytsov, Tianyi Lin, Fangwei Gao, Yutian Zhao, Jeffrey Huang, Eric Nyberg.

All the models except neural Model 1 (which we were not able to release due to patenting/licensing issues) 
are implemented in our [FlexNeuART framework](https://github.com/oaqa/FlexNeuART/tree/pypi2021), 
currently it is not the branch `pypi2021`, not the main branch! 
The framework is used out-of-the-box and provides documentations regarding indexing collections and training the models.

Crucially:
1. The list of models with corresponding configuration files can be [found in this folder](model_conf).
2. The list of queries for both MS MARCO (documents) and Robust04 (with five folds), which be directly used in FlexNeuART without further processing can be found [here](queries).

We will soon publish more detailed instructions as well as our main preprocesed training set for MS MARCO (documents).
We cannot do the same for Robust04, because obtaining Robust04 documents requires signing a licensing agreement with NIST.