# Contextual Semantic Parsing

The official pytorch implmentation of our paper [How Far are We from Effective Context Modeling ? An Exploratory Study on Semantic Parsing in Context](https://arxiv.org/pdf/2002.00652.pdf). This code contains multiple context modeling techniques on modeling context in semantic parsing. It provides `readable`, `fast` and `strong` baselines for the community.

## Task

![task](misc/task.svg)

Semantic parsing, which translates a natural language sentence into its corresponding executable logic form (e.g. Structured Query Language, SQL), relieves users from the burden of learning techniques behind the logic form. The majority of previous studies on semantic parsing assume that queries are context-independent and analyze them in isolation. However, in reality, users prefer to interact with systems in a dialogue, where users are allowed to ask context-dependent incomplete questions. That arises the task of **Semantic Parsing in Context**, which is quite challenging as there are complex contextual phenomena. 

## Model Framework

![model](misc/semantic_framework.svg)

Our backbone is the Sequence to Sequence model with a Grammar-Based Decoder, especially using the IRNet grammar (SemQL).


## Install Dependencies

### Python Environment

First of all, you should setup a python environment. This code base has been tested under python 3.x, and we officially support python 3.7.

After installing python 3.7, we strongly recommend you to use `virtualenv` (a tool to create isolated Python environments) to manage the python environment. You could use following commands to create a environment.

```bash
python -m pip install virtualenv
virtualenv venv
```

### Activate Virtual Environment
Then you should activate the environment to install the dependencies. You could achieve it via using the command as below. (Please change $ENV_FOLDER to your own virtualenv folder path, e.g. venv)

```bash
$ENV_FOLDER\Scripts\activate.bat (Windows)
source $ENV_FOLDER/bin/activate (Linux)
```

### Install Libraries

The most important requirements of our code base are as following:
- pytorch >= 1.2.0 (not tested on other versions, but 1.0.0 may work though)
- allennlp >= 0.9.0

You should install them at first.

## Prepare Dataset

You could download the two datasets [SParC](https://yale-lily.github.io/sparc) and [CoSQL](https://yale-lily.github.io/cosql). And then rename the dataset top folder as `dataset_sparc` and `dataset_cosql` respectively. An example structure for dataset `SParC` is as following:

```
|- dataset_sparc
    |- database
        |- academic
        |- activity_1
        |- ...
    |- train.json
    |- dev.json
    |- tables.json
|- models
|- predictor
|- ...
```

If you want to train models using pretrained files, please download [Glove Twitter](http://nlp.stanford.edu/data/glove.twitter.27B.zip). Unzip and rename the folder into `glove`.

## Training

![task](misc/settings.svg)

We use the command `allennlp` to train our models, and all the hyper-parameters for different settings are stored in configs listed under `train_configs` and `train_configs_bert`. The config and model architectures in our paper are as following:

| Config | Model in Paper |
| :--- | :---: |
| concat.none.jsonnet | Concat History |
| turn.none.jsonnet | Turn-level Encoder|
| none.attn.jsonnet | SQL Attention|
| none.gate.jsonnet | Gate Mechanism |
| none.token.jsonnet | Action Copy |
| none.tree.jsonnet | Tree Copy |
| concat.attn.jsonnet | Concat + Attention |
| concat.token.jsonnet | Concat + Token Copy |
| concat.tree.jsonnet | Concat + Tree Copy |
| turn.attn.jsonnet | Turn + SQL Attention|
| turn.token.jsonnet | Turn + Action Copy|
| turn.tree.jsonnet | Turn + Tree Copy|
| turn.token.attn.jsonnet | Turn + Action Copy + SQL Attention|

For example, you could run `Concat History` on `SParC` using the following script:

- Under linux:
```bash
export seed=1
export config_file=train_configs/concat.none.jsonnet
export model_file=checkpoints_sparc/sparc_concat_model
export tables_file=dataset_sparc/tables.json
export database_path=dataset_sparc/database
export dataset_path=dataset_sparc
export train_data_path=dataset_sparc/train.json
export validation_data_path=dataset_sparc/dev.json
export pretrained_file=glove/glove.twitter.27B.100d.txt
allennlp train -s $model_file $config_file \
--include-package dataset_reader.sparc_reader \
--include-package models.sparc_parser \
-o '{"model.serialization_dir":"$model_file","random_seed":"$seed","numpy_seed":"$seed","pytorch_seed":"$seed","dataset_reader.tables_file":"$tables_file","dataset_reader.database_path":"$database_path","train_data_path":"$train_data_path","validation_data_path":"$validation_data_path","model.text_embedder.tokens.pretrained_file":"$pretrained_file","model.dataset_path":"$dataset_path"}'
```
- Under Windows (`"""` is to escape the double quotation mark, equalivant to `"`):
```cmd
set seed=1
set config_file=train_configs/concat.none.jsonnet
set model_file=checkpoints_sparc/sparc_concat_model
set tables_file=dataset_sparc/tables.json
set database_path=dataset_sparc/database
set dataset_path=dataset_sparc
set train_data_path=dataset_sparc/train.json
set validation_data_path=dataset_sparc/dev.json
set pretrained_file=glove/glove.twitter.27B.100d.txt
allennlp train -s %model_file% %config_file% ^
--include-package dataset_reader.sparc_reader ^
--include-package models.sparc_parser ^
-o {"""model.serialization_dir""":"""%model_file%""","""random_seed""":"""%seed%""","""numpy_seed""":"""%seed%""","""pytorch_seed""":"""%seed%""","""dataset_reader.tables_file""":"""%tables_file%""","""dataset_reader.database_path""":"""%database_path%""","""train_data_path""":"""%train_data_path%""","""validation_data_path""":"""%validation_data_path%""","""model.text_embedder.tokens.pretrained_file""":"""%pretrained_file%""","""model.dataset_path""":"""%dataset_path%"""}
```

We provide windows batch scripts for convenience, you could modify them to linux according the above examples.

## Experimental Results

*Note: We will release pretrained weights under other settings later.*

| Dataset | BERT | Config | Best | Avg | Pretrained_Weights |
| :---: | :---: |:--- | :---: | :---: | :---: |
| SParC | No | concat.none.jsonnet | 41.8 | 40.0 | [model.tar.gz](https://github.com/microsoft/ContextualSP/releases/download/sparc.concat/model.tar.gz)|
| SParC | Yes | concat.none.jsonnet | 52.6 | 51.0 |  [model.tar.gz](https://github.com/microsoft/ContextualSP/releases/download/sparc.bert.concat/model.tar.gz)|
| CoSQL | No | concat.none.jsonnet | 33.5 | 32.4 | [model.tar.gz](https://github.com/microsoft/ContextualSP/releases/download/cosql.concat/model.tar.gz)|
| CoSQL | Yes | concat.none.jsonnet | 41.0 | 40.4 | [model.tar.gz](https://github.com/microsoft/ContextualSP/releases/download/cosql.bert.concat/model.tar.gz)|

## Predict

You could predict SQLs using trained model checkpoint file using the following command:

- Under Linux
```bash
export model_file=checkpoints_sparc/sparc_concat_model
export validation_file=dataset_sparc/dev.json
export validation_out_file=dataset_sparc/dev.jsonl
export prediction_out_file=predict.jsonl
python postprocess.py --valid_file $validation_file --valid_out_file $validation_out_file
allennlp predict \
--include-package dataset_reader.sparc_reader \
--include-package models.sparc_parser \
--include-package predictor.sparc_predictor \
--predictor sparc \
--dataset-reader-choice validation \
--batch-size 1 \
--cuda-device 0 \
--output-file $model_file/$prediction_out_file \
$model_file/model.tar.gz $validation_out_file
```
- Under Windows
```cmd
set model_file=checkpoints_sparc/sparc_concat_model
set validation_file=dataset_sparc/dev.json
set validation_out_file=dataset_sparc/dev.jsonl
set prediction_out_file=predict.jsonl
python postprocess.py --valid_file %validation_file% --valid_out_file %validation_out_file%
allennlp predict ^
--include-package dataset_reader.sparc_reader ^
--include-package models.sparc_parser ^
--include-package predictor.sparc_predictor ^
--predictor sparc ^
--dataset-reader-choice validation ^
--batch-size 1 ^
--cuda-device 0 ^
--output-file %model_file%/%prediction_out_file% ^
%model_file%/model.tar.gz %validation_out_file
```

## Demo

You could also host a demo page using the following command:

- Under Linux
```bash
export model_file=checkpoints_sparc/sparc_concat_model
python -m allennlp.service.server_simple \
    --archive-path $model_file/model.tar.gz \
    --predictor sparc \
    --include-package predictor.sparc_predictor \
    --include-package dataset_reader.sparc_reader \
    --include-package models.sparc_parser \
    --title "Contextual Semantic Parsing Demo" \
    --field-name question \
    --field-name database_id
```
- Under Windows
```cmd
set model_file=checkpoints_sparc/sparc_concat_model
python -m allennlp.service.server_simple ^
    --archive-path %model_file%/model.tar.gz ^
    --predictor sparc ^
    --include-package predictor.sparc_predictor ^
    --include-package dataset_reader.sparc_reader ^
    --include-package models.sparc_parser ^
    --title "Contextual Semantic Parsing Demo" ^
    --field-name question ^
    --field-name database_id
```

The question field accepts an interaction of questions splitted by `;`. See the demo page below (only accepts database_id appeared in `database` folder):

![demo](misc/demo.png)


## Acknowledgement

We will thank the following repos which are very helpful to us.

- [allennlp](https://github.com/allenai/allennlp)
- [IRNet](https://github.com/microsoft/IRNet)
- [spider-schema-gnn](https://github.com/benbogin/spider-schema-gnn)
- [sparc official github repo](https://github.com/taoyds/sparc)

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
