# Model configurations

There are two sets of configurations files: 
[configurataion files for the main experiement](main) and
[configuration files for sensitivity analysis](ablations).

Below is the list of main model configuration files together with model names and FlexNeuART codes.
We have not released Neural Model 1 due to patenting/licensing limitations.

| model name                  | FlexNeuART model code       | configuration file                   |
|-----------------------------|-----------------------------|--------------------------------------|
| AvgP                        | vanilla_bert                | config_long.json                     |
| FirstP (BERT)               | vanilla_bert                | config_short.json                    |
| FirstP (Longformer)         | longformer                  | config_short_longformer.json         |
| FirstP (ELECTRA)            | vanilla_bert                | config_short_electra.json            |
|                             |                             |                                      |
| MaxP                        | bert_maxp                   | config_long.json                     |
| SumP                        | bert_sump                   | config_long.json                     |
|                             |                             |                                      |
| CEDR-DRMM                   | cedr_drmm                   | config_long_drmm.json                |
| CEDR-KNRM                   | cedr_knrm                   | config_long_knrm.json                |
| CEDR-PACRR                  | cedr_pacrr                  | config_long_cedr_pacrr.json          |
|                             |                             |                                      |
| PARADE Attn                 | parade_attn                 | config_long_parade.json              |
| PARADE Attn (ELECTRA)       | parade_attn                 | config_short_electra.json            |
| PARADE Avg                  | parade_avg                  | config_long_parade.json              |
| PARADE Max                  | parade_max                  | config_long_parade.json              |
|                             |                             |                                      |
| PARADE Transf-RAND-L2       | parade_transf_rand          | config_long_parade_rand.json         |
| PARADE Transf-PRETR-L6      | parade_transf_pretr         | config_long_parade_pretr_L6.json     |
| PARADE Transf-PRETR-Q-L6    | parade_transf_wquery_pretr  | config_long_parade_pretr_L6.json     |
|                             |                             |                                      |
| Big-BIRD                    | vanilla_bert                | config_long_bigbird.json             |
| Longformer                  | longformer                  | config_long_longformer.json          |


