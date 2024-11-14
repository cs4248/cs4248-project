# CS4248 Project
## G16

# Background
This repository contains scripts for training ensemble models and prediction. It was created for the purposes of exploring ensemble methods for translating chinese to english.

The following ensemble methods are included:
* Multi-Engine Machine Translation (MEMT)
* Mixture of Experts (MoE)
* Back translation
* Transductive Ensemble Learning (TEL)


## Setup of environments
Before running any script in this repository, ensure that the environment has been set up. The creation and activation of a virtual environment is optional, but highly recommended.

### Create virtual environment
`python -m venv .venv`

### Activate virtual environment
#### Windows OS
`.venv/Scripts/activate`

#### Linux and Mac OS
`source .venv/bin/activate`

### Install dependencies
`pip install -r requirements.txt`


## Base models
### Prerequisites
* `tatoeba.en`
* `tatoeba.zh`
* `train.zh-en.en`
* `train.zh-en.zn`
* `wmttest2022.AnnA.en`
* `wmttest2022.zh`

For **mBART**: 
The `.env.sample` has to be converted to `.env` before running the file. This is because the mBART requires an older version for the package `protobuf` while the COMET evaluation metric requires an updated version. Thus the `.env` file is a workaround solution the developers provided.

### Prediction
For **NLLB**, **mBART**, **marianMT**, **T5**: 

`python {model}.py  -text {text_path} -ft -batch {size} -out {output_path}`

<br/>

`text` - Text_Path: The path containing the untranslated Chinese text file.

`ft` - Fine Tune: Decide between using the base or fine-tuned version, only include if you want to use the fine-tuned version (Optional)

`batch` - Batch Size: Decide how many predictions to be done at a time. E.g. 10 (Optional)

`out` - Output_path: The path to output the translated English text file.

<br/>

For **Small100**:

`python small100.py -text {text_path} -batch {size}  -out {output_path}`

<br/>

`text` - Text_Path: The path containing the untranslated Chinese text file.

`batch` - Batch Size: Decide how many predictions to be done at a time. E.g. 10 (Optional)

`out` - Output_path: The path to output the translated English text file.

### Finetune
Running this command will generate the respective model.pt which can be used for fine-tuned translation.

<br/>

For **NLLB**, **mBart**, **marianMT**, **T5**: 

`python {model}.py -text {text_path} -label {label_path}`

`text` - Text_Path: The path containing the untranslated Chinese text file.

`label` - Label_Path: The path containing the ideal translated English text file.

<br/>

For **Small100**:

No fine tuning provided.

# Ensemble methods
In the following section, we assume the following prerequisites have been met.

## MEMT
### Prerequisites:
`train_memt_labels.txt` 


### Training
It will create 2 additional files `filtered_train_memt_text.txt` and `filtered_train_memt_labels.txt`, which can be deleted after the command completes.

<br/>

`python -m memt.memt_train -model_path {model_path_to_save}` 

<br/>

`model_path` - Model_path_to_save: The path to store the model trained.

### Prediction
`python -m memt.memt_pred -model_path {saved_model_path}  -text {text_to_translate} -out {Output file}`

<br/>

`model_path` - saved_model_path: The path where the trained model is saved in.

`text` - Text_to_translate: The path containing untranslated chinese text 

`out` - Output file path: The path to store the predictions

## MoE
### Training
`python -m moe.soft_gated_moe -text {text_path} -pred {pred_path1} {pred_path2} {pred_path3} {pred_path4} -label {label_path} -model {model_path} -batch {batch_size}`

<br/>

`text` - Text_path: The path containing the untranslated Chinese text file.

`pred` - Pred_path: The paths containing the expert model translated English text files. Note that the script only allows for 4 expert models, so 4 prediction files corresponding to each expert model should be provided here. The pred path must correspond with the text path.

`label` - Label_path: The path to the label file containing reference English text.

`model` - Model_path: The path to store the model trained.

`batch` - Batch Size: Decide how many predictions to be done at a time. (Optional, default is 32)


### Prediction
`python -m moe.soft_gated_moe -text {text_path} -pred {pred_path1} {pred_path2} {pred_path3} {pred_path4} -out {output_path} -model {model_path} -batch {batch_size}`

<br/>

`text` - Text_path: The path containing the untranslated Chinese text file.

`pred` - Pred_path: The paths containing the expert model translated English text files. Note that the script only allows for 4 expert models, so 4 prediction files corresponding to each expert model should be provided here. The pred path must correspond with the text path.

`out` - Output_path: The path to output the translated English text file.

`model` - Model_path: The path to the model to use for predictions.

`batch` - Batch Size: Decide how many predictions to be done at a time. (Optional, default is 32)



## Back translation
Steps:
1. Generate reverse translations for the N models to be ensembled together.
2. Generate cosine similarities for the N models to be ensembled together.
3. Make the final prediction using the N models’ English translations as well as cosine similarities.

### Generate reverse translation
`python -m back_translate.reverse_translate -text {text_path} -out {output_path} -batch {batch_size}`

<br/>

`text` - Text_path: The path containing the untranslated Chinese text file.

`out` - Output_path: The path to output the translated English text file.

`batch` - Batch Size: Decide how many predictions to be done at a time. (Optional, default is 32)

### Generate cosine similarity
`python -m back_translate.cosine_similarity -text {text_path} -pred {pred_path} -out {output_path}`

<br/>

`text` - Text_path: The path containing the untranslated Chinese text file.

`pred` - Pred_path: The path to the backtranslated chinese sentences text file.

`out` - Output_path: The path to save the similarity score file.

### Prediction
`python -m back_translate.back_translate_model -pred {pred_path1} {pred_path2} … {pred_pathN} -sim {sim_path1} {sim_path2} … {sim_pathN} -out {output_path}`

<br/>

`pred` - Pred_path: The paths to the English translated text files for each model. Ensure that the number of paths included matches the number of sim file paths included.

`sim` - Sim_path: The paths to the similarity score files. Ensure that the order of sim paths is the same as the pred paths. i.e. first pred path and first sim path must be from the same model.

`out` - Output_path: The path to save the back translation ensemble predictions.


## TEL
### Training
Running this command will generate the meta model.pt which can then be used to predict.

<br/>

`python -m transductive_ensemble.transductive_ensemble -text {text_path} -label {label_path1} {label_path2} …{label_path_N}`

<br/>

`text` - Text_Path: The path containing the untranslated Chinese text file.

`label` - Label_Paths: The paths containing the predicted English text files from the various base learners.

E.g.  -label /nllb_predictions/pred_tatoeba.en /mbart_predictions/pred_tatoeba.en /t5_predictions/pred_tatoeba.en



### Prediction

Requires the meta model to be trained before predicting.

<br/>

`python -m transductive_ensemble.transductive_ensemble -text {text_path} -batch {size} -out {output_path}`

<br/>

`text` - Text_Path: The path containing the untranslated Chinese text file.

`batch` - Batch Size: Decide how many predictions to be done at a time. E.g. 10 (Optional)

`out` - Output_path: The path to output the translated English text file.

## Evaluation 
`python eval.py -metric {comet/bleu} -pred {pred_path} -label {label_path} -src {source_path}`

<br/>

`metric` - Metric to use: Evaluate via COMET or BLEU score, only accepting bleu or comet.

`pred` - Pred_Path: The path containing the predicted English text file.

`label` - Label_Path: The path containing the ideal translated English text file.

`src` - Source_path: The path containing the untranslated Chinese text file. (Only required for COMET)

## Best BLEU
`python best_bleu.py -perm -label {label_path} -pred {pred_path1} {pred_path2} … {pred_pathN}`

<br/>

`perm` - Perm: Flag variable to toggle between permutation generation and single file best BLEU. Include the tag to enable permutation mode. Exclude to disable permutation mode. Optional.

`pred` - Pred_Paths: The paths containing the predicted English text files for each model to be included in the permutation. The order of the pred paths is important as the result will be in a dictionary format where the inclusion of the character “n” corresponds to the presence of the nth model in the list of pred paths provided.

`label` - Label_Path: The path containing the ideal translated English text file.
