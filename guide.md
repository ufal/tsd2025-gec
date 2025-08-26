# Guide

All experiments, training, and evaluation of models are designed to run inside a Docker container. 
The Docker image provides all required dependencies (Python environment, libraries, external tools such as Aspell and MorphoDiTa).

---

## Quickstart

### 1. Docker image

How to build Docker image:

```bash
IMAGE_NAME="<docker-image-name>"
IMAGE_TAG="<docker-image-tag>"
bash build.sh $IMAGE_NAME $IMAGE_TAG
```

### (2. Kubernetes example)

```bash
kubectl apply -f kubernetes/train.yaml 
```

### 3. Running training

Inside the container, experiments are launched via the main pipeline script.  
Here’s an example command that activates conda, navigates to the selected model configuration folder, and runs training:

Training:

```bash
# Replace <selected-model-config-folder> with your config folder, e.g., transformer-base-pretrain
/bin/bash -c "source ~/miniconda3/etc/profile.d/conda.sh && \
conda activate && \
cd /code/src/<selected-model-config-folder> && \
python ../pipeline/run.py --config config.json"
```

### 4. Running training

Evaluation:

```bash
/bin/bash -c "source ~/miniconda3/etc/profile.d/conda.sh && \
conda activate && \
cd /code/src/<selected-model-config-folder> && \
python ../pipeline/run.py --config config.json --eval
```


## [run.py](./code/src/pipeline/run.py) — Training, Inference, and Evaluation Launcher

This script is the main entry point for running experiments in the Czech Grammar Error Correction project.  
It supports training models, creating predictions, and evaluating results based on a configuration file.

Usage:

```
python run.py --config <config.json> [OPTIONS]

The script will start training by default.

Arguments

    --config
    Path to the configuration JSON file that defines training or evaluation parameters.

Optional flags

    --eval
    Run complete evaluation instead of training. 

        Only one eval mode can be active at a time (infer, eval_preds, multi_eval, infer_opt).
        Eval modes:

        --infer
        Run only inference on the model and save predictions to files.

        --eval_preds
        Evaluate predictions that were already saved in files.

        --multi_eval
        Evaluate multiple experiments at once (comma-separated list of config directories).

        --infer_opt
        Run inference using a checkpoint with optimizer, then save predictions to files.

```



## Model and Tokenizer Utilities - model_utils

This folder contains scripts for creating Transformer models and tokenizers for the Czech Grammar Error Correction project.

**1. create_model.py**

This script initializes and saves a BART-style Transformer model with configurable size.

Usage
```bash
python create_model.py --output-dir <output-path> [--model-size small|base|big]
```

**2. create_tokenizer.py**

This script trains a WordPiece tokenizer from a text corpus and saves it for use with Transformers.

Usage
```bash
python create_tokenizer.py --corpus-file <path-to-corpus> --output-dir <output-dir> [--vocab-size 32000] [--min-frequency 5]
```


## Dataset Preparation and Sampling Utilities - dataset_utils

This folder contains scripts for creating, filtering, and sampling datasets for Czech Grammar Error Correction experiments.

**1. create_oversampled_datasets.py**

Generates an oversampled dataset by sampling lines from multiple domain-specific datasets according to a scaling factor.

Usage
```bash
python create_oversampled_datasets.py \
  --nf <natives_formal.tsv> \
  --nwi <natives_web_informal.tsv> \
  --romani <romani.tsv> \
  --sl <second_learners.tsv> \
  -f <scaling-factor> \
  -t <total-samples> \
  -o <output.tsv>
```

**2. get_domain.py**

Filters samples in a .m2 dataset by domain using metadata.

Usage
```bash
python get_domain.py \
  --domain <domain-name> \
  --meta <meta.tsv> \
  --domain_meta <domain.meta> \
  --domain_m2 <domain.m2> \
  --output <output.m2>
```

**3. get_domain_text.py**

Filters and combines input and gold files for a specific domain, producing a domain-specific dataset.

Usage
```bash
python get_domain_text.py \
  --domain <domain-name> \
  --meta <meta.tsv> \
  --domain_meta <domain.meta> \
  --input <sentence.input> \
  --gold <sentence.gold> \
  --output <output.tsv>
```

**4. sample.py**

Generates random samples from multiple domain datasets for controlled experiments.

Defines TOTAL_SIZES and RANDOM_SEEDS for multiple sampling scenarios.

Samples proportionally from four domains (natives_formal, natives_web_informal, romani, second_learners).

Outputs shuffled samples to files named domain_sample_size_<size>_rs_<seed>.tsv.