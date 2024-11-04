# GlossLM
A Massively Multilingual Corpus and Pretrained Model for Interlinear Glossed Text

[📄 Paper](https://arxiv.org/abs/2403.06399) | [📦 Models and data](https://huggingface.co/collections/lecslab/glosslm-66da150854209e910113dd87)


## Usage
```python
import transformers

# Your inputs
transcription = "o sey xtok rixoqiil"
translation = "O sea busca esposa."
lang = "Uspanteco"
metalang = "Spanish"
is_segmented = False

prompt = f"""Provide the glosses for the following transcription in {lang}.

Transcription in {lang}: {transcription}
Transcription segmented: {is_segmented}
Translation in {metalang}: {translation}\n
Glosses: 
"""

model = transformers.T5ForConditionalGeneration.from_pretrained("lecslab/glosslm")
tokenizer = transformers.ByT5Tokenizer.from_pretrained("google/byt5-base", use_fast=False)

inputs = tokenizer(prompt, return_tensors="pt")
outputs = tokenizer.batch_decode(model.generate(**inputs, max_length=1024), skip_special_tokens=True)
print(outputs[0]) # o sea COM-buscar E3S-esposa
```

## Background
Interlinear Glossed Text (IGT) is a common format in language documentation projects. It looks something like this:

```
o sey x  -tok    r  -ixoqiil
o sea COM-buscar E3S-esposa
“O sea busca esposa.”
```

The three lines are as follows:
- The **transcription** line is a sentence in the target language (possibly segmented into morphemes)
- The **gloss** line gives an linguistic gloss for each morpheme in the transcription
- The **translation** line is a translation into a higher-resource language

Creating consistent and large IGT datasets is time-consuming and error-prone. The goal of **automated interlinear glossing** is to aid in this task by **predicting the gloss line given the transcription and translation line**. 

## Detailed Usage
Interested in replicating or modifying our experiments? Pretraining, finetuning, and inference are handled with `run.py`, as follows:
```bash
python run.py -c some_config_file.cfg -o key1=val1 key2=val2
```

The run is defined by an INI-style *config file*. We have some examples for [pretraining](configs/pretrain_base.cfg), [finetuning](configs/finetune_base.cfg), and [inference](configs/predict_base.cfg). A config file looks something like:
```ini
[config]

mode = pretrain
exp_name = pretrain_base
pretrained_model = google/byt5-base

# Dataset
exclude_st_seg = false
use_translation = true
use_unimorph = false

# Training
max_epochs = 13
early_stopping_patience = 3
learning_rate = 5e-5
batch_size = 2

# Files
output_model_path = /projects/migi8081/glosslm/models/glosslm-pretrained-base
```

The full list of possible options is in [experiment_config.py](src/training/experiment_config.py).

If you'd like to run finetuning on a new language, you'll probably need to write your own training script. You can use `run.py` as a reference for how we implemented finetuning.
