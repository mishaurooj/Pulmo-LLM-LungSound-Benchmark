# Pulmo-LLM: Lung Sound Report Generation Benchmark

## Project title

**Benchmark of a General-Purpose LLM versus a Domain-Adapted Pulmonology LLM for Lung Sound Report Generation**

## Research question

Does fine-tuning Qwen with pulmonology-specific data improve the quality and efficiency of lung sound report generation compared with the original base model?

## Project summary

This project builds a complete lung sound analysis and pulmonology report generation pipeline. It integrates multiple respiratory datasets, processes lung sound recordings, generates classifier-based evidence, creates structured report-generation data, fine-tunes a Qwen language model using QLoRA, and compares the original base model against the fine-tuned Pulmo-LLM.

The final benchmark evaluates whether domain adaptation improves generated clinical report quality, structure, label consistency, efficiency, and deployment readiness.

## Datasets used

This work uses five datasets:

1. **ICBHI 2017 Respiratory Sound Database**  
   Lung sound recordings with respiratory cycle annotations.

2. **Mendeley Lung Sounds Dataset**  
   Lung sound recordings with disease and sound labels.

3. **HLS-CMDS Dataset**  
   Lung sound recordings grouped into HS, LS, and Mix categories.

4. **MIMIC-IV Clinical Database Demo 2.2**  
   Structured clinical tables used for pulmonology-oriented text and metadata analysis.

5. **PulmoDatasets**  
   Pulmonary disease-related tabular/text datasets used for pulmonary relevance and text-token auditing.

## Main workflow

```text
START
  ↓
1. Dataset audit and integration
   (5 datasets, audio + text, file inventory, metadata, rows, words, tokens)
  ↓
2. Lung sound Dataset A generation
   (ICBHI + Mendeley + HLS-CMDS converted into unified classifier-ready CSV files)
  ↓
3. Classifier evidence creation
   (normal/abnormal prediction, lung sound class, disease-related acoustic pattern)
  ↓
4. LLM field creation
   (llm_input + llm_report generated from classifier evidence)
  ↓
5. Qwen QLoRA fine-tuning
   (base Qwen model adapted to pulmonology report generation)
  ↓
6. Base vs fine-tuned report generation
   (same records passed through original Qwen and Pulmo-LLM)
  ↓
7. Evaluation
   (word count, token count, report structure, label agreement, efficiency, dataset-wise metrics)
  ↓
8. Final outputs
   (CSV files, generated reports, metrics, graphs, confusion matrices, final report)
END
```

## Hardware used

Recommended hardware:

```text
GPU: NVIDIA GPU with 8 GB VRAM minimum, 12 GB or more recommended
RAM: 16 GB minimum, 32 GB recommended
Storage: 80 GB free minimum
OS: Windows 10/11
Python: 3.10 recommended
```

For QLoRA fine-tuning, a CUDA-capable GPU is strongly recommended.

## Software requirements

Install Anaconda or Miniconda first.

Recommended Python version:

```text
Python 3.10
```

Core libraries:

```text
numpy
pandas
scikit-learn
matplotlib
seaborn
librosa
soundfile
joblib
tqdm
torch
transformers
datasets
peft
trl
accelerate
bitsandbytes
sentencepiece
protobuf
openpyxl
```

Optional libraries:

```text
pyreadr
hf_xet
huggingface_hub
```

## Anaconda setup

Create the environment:

```bat
conda create -n pulmo-llm python=3.10 -y
conda activate pulmo-llm
```

Install PyTorch with CUDA. For CUDA 12.1:

```bat
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
```

Install the remaining libraries:

```bat
pip install numpy pandas scikit-learn matplotlib seaborn librosa soundfile joblib tqdm openpyxl
pip install transformers datasets accelerate peft trl bitsandbytes sentencepiece protobuf huggingface_hub hf_xet
pip install pyreadr
```

Check GPU availability:

```bat
python -c "import torch; print(torch.cuda.is_available()); print(torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'CPU only')"
```

## Folder paths used in this project

The scripts assume this Windows folder layout:

```text
D:\LungSound
├── ICBHI
│   ├── Dataset
│   ├── Code
│   └── Results
│
├── Mendley lung
│   ├── Audio Files
│   ├── Code
│   └── Data annotation.xlsx
│
├── HLS-CMDS
│   ├── Dataset
│   └── Results
│
└── Pulmo-LLM
    ├── Code
    ├── Datasets
    └── Results
```

If your folders are different, edit the default paths inside the scripts or pass paths using command-line arguments.

## Step 1: Audit all five datasets

This step counts files, file size, table rows, audio duration, estimated words, estimated tokens, and creates visual graphs.

Run:

```bat
cd /d D:\LungSound\Pulmo-LLM\Code
python 00_audit_5_datasets_visual.py
```

Expected outputs:

```text
D:\LungSound\Pulmo-LLM\Results\Five_Dataset_Audit_Visual
├── 01_dataset_wise_summary.csv
├── 02_file_wise_details.csv
├── 03_text_column_word_token_details.csv
├── 04_read_errors.csv
├── five_dataset_audit_report.txt
└── graphs
    ├── files_by_dataset.png
    ├── size_mb_by_dataset.png
    ├── audio_seconds_by_dataset.png
    ├── rows_by_dataset.png
    ├── words_by_dataset.png
    └── tokens_by_dataset.png
```

Use these outputs to show dataset scale in your thesis or presentation.

## Step 2: Generate Dataset A from three lung sound datasets

Dataset A combines ICBHI, Mendeley Lung, and HLS-CMDS into a unified classifier-output format.

Run:

```bat
python 03_generate_dataset_a.py --dataset all
```

Expected outputs:

```text
D:\LungSound\Pulmo-LLM\Datasets\Dataset_A
├── dataset_a_icbhi.csv
├── dataset_a_mendeley.csv
├── dataset_a_hls.csv
└── dataset_a_all.csv
```

Dataset A contains standardized fields such as:

```text
dataset
patient_id
recording_id
unit_id
binary_true
binary_pred
sound_true
sound_pred
disease_true
disease_pred
confidence scores
probability distributions
```

## Step 3: Add LLM input and report fields

This step converts classifier evidence into two fields:

```text
llm_input
llm_report
```

Run:

```bat
python 04_add_2_llm_fields.py --make-jsonl
```

Expected outputs:

```text
D:\LungSound\Pulmo-LLM\Datasets\Dataset_A
├── dataset_a_all_with_2_llm_fields.csv
└── dataset_a_llm.jsonl
```

The JSONL file is used for Qwen fine-tuning.

## Step 4: Fine-tune Qwen with QLoRA

This step trains the domain-adapted Pulmo-LLM.

Run:

```bat
python 08_pulmo_llm_all_in_one.py --mode all
```

This performs:

```text
1. Train/validation/test split
2. QLoRA fine-tuning
3. Report generation
4. Report evaluation
5. Graph generation
```

Expected outputs:

```text
D:\LungSound\Pulmo-LLM\Results\Pulmo_LLM_Qwen
├── data
│   ├── train.jsonl
│   ├── validation.jsonl
│   └── test.jsonl
├── adapter
│   ├── adapter_config.json
│   ├── adapter_model.safetensors
│   └── tokenizer files
├── generated
│   └── generated_reports.csv
├── evaluation
│   ├── per_report_metrics.csv
│   ├── summary_metrics.json
│   ├── sound_label_confusion_matrix.csv
│   └── sound_label_classification_report.txt
└── graphs
```

## Step 5: Compare base Qwen vs fine-tuned Pulmo-LLM

This is the main benchmark for the research question.

Run:

```bat
python 09_compare_base_vs_finetuned_all.py ^
  --input-csv "D:\LungSound\Pulmo-LLM\Datasets\Dataset_A\dataset_a_all_with_2_llm_fields.csv" ^
  --base-model "Qwen/Qwen2.5-3B-Instruct" ^
  --adapter-dir "D:\LungSound\Pulmo-LLM\Results\Pulmo_LLM_Qwen\adapter" ^
  --output-dir "D:\LungSound\Pulmo-LLM\Results\Pulmo_LLM_Qwen\base_vs_finetuned_all"
```

Quick test:

```bat
python 09_compare_base_vs_finetuned_all.py --limit 5
```

Expected outputs:

```text
D:\LungSound\Pulmo-LLM\Results\Pulmo_LLM_Qwen\base_vs_finetuned_all
├── base
│   └── generated_reports_txt
├── finetuned
│   └── generated_reports_txt
├── all_models_per_record_results.csv
├── dataset_wise_summary.csv
├── overall_summary.csv
├── model_weight_summary.json
└── run_summary.json
```

## Evaluation metrics

The benchmark evaluates:

```text
Report word count
Report token count
Inference time
Format validity
Required clinical sections
Sound-label agreement
Disease-label consistency
Hallucination indicators
Dataset-wise performance
Base vs fine-tuned difference
```

Core classification/report metrics include:

```text
Accuracy
Precision
Recall
F1-score
Balanced accuracy
Confusion matrix
Cohen kappa
McNemar test, if paired outputs are available
```

## Expected conclusion format

Use this wording in your thesis or report:

```text
The study compared the original Qwen base model with a pulmonology-adapted Qwen model fine-tuned on structured lung sound classifier evidence and clinician-style report outputs. The fine-tuned Pulmo-LLM was evaluated across ICBHI, Mendeley Lung, and HLS-CMDS records. Performance was assessed using report-level metrics, clinical structure validation, label consistency, token usage, inference time, and dataset-wise summaries. The benchmark determines whether domain adaptation improves the quality and efficiency of lung sound report generation compared with a general-purpose base LLM.
```

## Optional: local multimodal VLM comparison

This experiment compares CNN, Qwen2.5-VL, and LLaVA using mel-spectrogram images.

Download models once:

```bat
hf download Qwen/Qwen2.5-VL-3B-Instruct ^
  --local-dir D:\LungSound\Models\Qwen2.5-VL-3B-Instruct
```

```bat
hf download llava-hf/llava-1.5-7b-hf ^
  --local-dir D:\LungSound\Models\LLaVA-1.5-7B
```

Run:

```bat
python 14_compare_qwen_llava_melspec_lungsound_study.py ^
  --mode all ^
  --run-vlm ^
  --qwen-vl "D:\LungSound\Models\Qwen2.5-VL-3B-Instruct" ^
  --llava-vl "D:\LungSound\Models\LLaVA-1.5-7B"
```

This compares:

```text
CNN baseline
Qwen2.5-VL zero-shot
Qwen2.5-VL few-shot
LLaVA zero-shot
LLaVA few-shot
```

## Keywords

```text
pulmonology
lung sounds
respiratory sound classification
large language models
Qwen
QLoRA
clinical report generation
medical AI
ICBHI
Mendeley Lung Sounds
HLS-CMDS
```

## Important privacy and storage notes

This repository should not contain patient-level private data, copyrighted datasets, audio files, trained model weights, or API keys. The repository should contain only reproducible code, configuration files, documentation, and small sample outputs.

Large files should be stored outside GitHub, for example:

```text
D:\LungSound\Datasets
D:\LungSound\Models
D:\LungSound\Pulmo-LLM\Results
```

If model weights must be shared, use Hugging Face Hub or GitHub Releases, not normal Git commits.

## Citation placeholder

If this work is used in a thesis or publication, cite it as:

```text
Suleman et al. Pulmo-LLM: Benchmarking a General-Purpose LLM versus a Domain-Adapted Pulmonology LLM for Lung Sound Report Generation. 2026.
```
