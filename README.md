# XLSR Pipeline — French ASR, Direct Speech Translation, and Automatic Evaluation
### Applied to the CFPP2000 Corpus (Anita Musso, 11th arrondissement)

[![License: MIT](https://img.shields.io/badge/Code-MIT-blue.svg)](LICENSE)
[![Data: CC BY-NC-SA 4.0](https://img.shields.io/badge/Data-CC%20BY--NC--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)

---

## 1. General Information

**Project title:** XLSR Pipeline — French ASR and Direct Speech Translation applied to the CFPP Corpus, with Automatic Evaluation  
**Author:** Zoé de Vries ([zoede-vries](https://cv.hal.science/zoede-vries))  
**Institution:** ENS Paris-Saclay / Université Paris-Cité — M2 Etudes anglophones - Linguistique anglaise  
**Date created:** 2025–2026  
**Language(s):** French (audio input), English (translation output), Python (code)

**Project description:**  
This repository implements a two-stage speech processing pipeline on a sample of spoken Parisian French, followed by per-segment automatic evaluation of the outputs.

| Stage | Model | Task |
|-------|-------|------|
| A | `jonatasgrosman/wav2vec2-large-xlsr-53-french` | Automatic Speech Recognition (ASR) — French transcription |
| B | `facebook/wav2vec2-xls-r-1b-21-to-en` | Direct Speech Translation (ST) — French audio → English text |
| C | sacrebleu + Unbabel/wmt22-comet-da | Per-segment evaluation — BLEU, chrF, TER, COMET |

The pipeline processes 50 non-overlapping audio segments extracted from a single-speaker interview in the CFPP2000 corpus. The main notebook outputs a CSV file containing each segment filename, its French transcription, and its English translation. The evaluation notebook computes four automatic metrics per segment and exports a scored Excel file.

**Hardware note:** Stage B (XLS-R 1B) requires a GPU with at least 14 GB VRAM. A free Colab T4 runtime is sufficient. The notebook is structured to load and release each model sequentially to stay within VRAM limits.

---

## 2. Methodological Information

**Pipeline overview:**

1. **Data preparation**: segments can be downloaded from the Zenodo archive (https://doi.org/10.5281/zenodo.19479351) and unzipped into `data/segments/`.
2. **ASR — Stage A**: each segment is passed through `wav2vec2-large-xlsr-53-french`. Long segments are chunked with a 1-second overlap before inference, and chunk transcriptions are joined. The model is then deleted and VRAM freed before Stage B.
3. **ST — Stage B**: each segment is passed through `wav2vec2-xls-r-1b-21-to-en` (a `SpeechEncoderDecoderModel` with an MBart-50 decoder). Beam search (n=4) with a repetition penalty is used to reduce degeneration on longer segments.
4. **Export**: results are written to `output/transcriptions_xlsr.csv` with columns `fichier`, `transcription_fr`, `translation_en`.
5. **Evaluation** (`eval/xlsr_eval.ipynb`): the output CSV is merged with `data/reference_transcription_translation.csv` on segment index. Per-segment BLEU, chrF, TER (sacrebleu) and COMET (Unbabel/wmt22-comet-da) scores are computed and exported to `output/xlsr_eval_scores.xlsx`.

**Software dependencies:** see `requirements.txt`. Pinned to `transformers==4.44.2` for compatibility with the XLS-R speech translation model.

**Reproducibility:** running all cells in order from a clean environment will reproduce the output without any manual data preparation step. The source audio URL is hardcoded in the data preparation cell. The reference data file is versioned in the repository.

---

## 3. Data and File Overview

```
xlsr-cfpp2000-pipeline/
├── README.md                                   <- this file
├── LICENSE                                     <- MIT licence (code)
├── requirements.txt                            <- pinned Python dependencies
├── xlsr_pipeline.ipynb                         <- main pipeline notebook (run top to bottom)
├── data/
│   ├── README.md                               <- data-specific metadata and provenance
│   ├── reference_transcription_translation.csv <- reference transcriptions (CFPP2000) and
│   │                                              reference translations (this study)
│   └── segments/                               <- audio segments (gitignored; download from Zenodo)
├── eval/
│   └── xlsr_eval.ipynb                         <- per-segment evaluation notebook (run after pipeline)
└── output/
    ├── transcriptions_xlsr.csv                 <- pipeline output (gitignored; regenerate by running
    │                                              xlsr_pipeline.ipynb)
    └── xlsr_eval_scores.xlsx                   <- evaluation output (gitignored; regenerate by running
                                                   eval/xlsr_eval.ipynb)
```

**Note on versioned vs. generated files:** `data/segments/` and all files under `output/` are listed in `.gitignore` and are not versioned. Audio segments are archived on Zenodo (see Section 4). Both output files are artefacts of running the notebooks and can be fully reproduced from the versioned inputs. `data/reference_transcription_translation.csv` is versioned in the repository.

---

## 4. Data-Specific Information

**Source corpus:** CFPP2000 (*Corpus de Français Parlé Parisien des années 2000*)  
**Speaker used:** Anita Musso (F, 46, 11th arrondissement of Paris)  
**Source file URL:** http://cfpp2000.univ-paris3.fr/data/public/11eme/Anita_MUSSO_F_46_11e/Anita_MUSSO_F_46_11e.wav  
**Segmentation:** 50 non-overlapping segments extracted starting at t=0 — archived at https://doi.org/10.5281/zenodo.19479351  
**Corpus licence:** CC BY-NC-SA 4.0 (see `data/README.md`)

**Reference data:** `data/reference_transcription_translation.csv`

| Column | Source | Used in metric computation |
|--------|--------|---------------------------|
| `reference_transcription` | CFPP2000 public website | No — included for alignment and inspection only |
| `reference_translation` | Produced by the repository author (2025–2026) | Yes — used as reference for BLEU, chrF, TER, COMET |

Reference translations were produced with the aim of preserving discourse-structural features of the source (dislocation structures, presentative constructions); marks of orality (filled pauses, self-repairs) were removed to obtain a clean translation surface. See the associated paper for full methodological detail.

**Pipeline output CSV schema:**

| Column | Type | Description |
|--------|------|-------------|
| `fichier` | string | Segment filename (e.g. `segment_01.wav`) |
| `transcription_fr` | string | French transcription produced by XLSR-53 |
| `translation_en` | string | English translation produced by XLS-R 1B |

**Evaluation output schema:**

| Column | Type | Description |
|--------|------|-------------|
| `fichier` | string | Segment filename |
| `transcription_fr` | string | ASR hypothesis (French) |
| `reference_transcription` | string | Reference transcription from CFPP2000 (not used in scoring) |
| `reference_translation` | string | Reference translation (used as metric reference) |
| `translation_en` | string | ST hypothesis (English) |
| `bleu_xlsr` | float | Sentence-level BLEU (effective_order=True, lowercase=True) |
| `chrf_xlsr` | float | chrF (char_order=6, word_order=0, β=2) |
| `ter_xlsr` | float | TER — error metric, lower is better |
| `comet_xlsr` | float | COMET (Unbabel/wmt22-comet-da) |

---

## Quickstart

```bash
# 1. Clone
git clone https://github.com/Zdv-dev-dv/xlsr-cfpp2000-pipeline.git
cd xlsr-cfpp2000-pipeline

# 2. Download audio segments from Zenodo
#    https://doi.org/10.5281/zenodo.19479351
#    Unzip into data/segments/

# 3. Install dependencies
pip install -r requirements.txt

# 4. Run the pipeline
jupyter notebook xlsr_pipeline.ipynb

# 5. Run the evaluation (requires output/transcriptions_xlsr.csv from step 4)
jupyter notebook eval/xlsr_eval.ipynb
```

---

## Evaluation

Per-segment automatic evaluation is implemented in `eval/xlsr_eval.ipynb`. The notebook merges `output/transcriptions_xlsr.csv` with `data/reference_transcription_translation.csv` on segment index before computing scores. Only `reference_translation` is used as a metric input; `reference_transcription` is retained in the output file for inspection only.

Metrics: sacrebleu (BLEU with `effective_order=True`, chrF, TER) and Unbabel/wmt22-comet-da (COMET). TER is an error metric (lower is better); BLEU, chrF, and COMET are quality metrics (higher is better).

---

## Citation

If you use or build on this pipeline, please cite the CFPP2000 corpus:

> Branca-Rosoff, S., Fleury, S., Lefeuvre, F. & Pires, M. (2012). *Discours sur la ville. Présentation du Corpus de Français Parlé Parisien des années 2000 (CFPP2000)*. Université Sorbonne Nouvelle – Paris 3. http://cfpp2000.univ-paris3.fr
