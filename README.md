# XLSR Pipeline — French ASR + Direct Speech Translation (FR → EN)
### Applied to the CFPP2000 Corpus (Anita Musso, 11th arrondissement)

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Zdv-dev-dv/xlsr-cfpp2000-pipeline/blob/main/xlsr_pipeline.ipynb)
[![License: MIT](https://img.shields.io/badge/Code-MIT-blue.svg)](LICENSE)
[![Data: CC BY-NC-SA 4.0](https://img.shields.io/badge/Data-CC%20BY--NC--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)

---

## 1. General Information

**Project title:** XLSR Pipeline — French ASR and Direct Speech Translation applied to the CFPP2000 Corpus  
**Author:** Zoé de Vries ([zoede-vries](https://cv.hal.science/zoede-vries))  
**Institution:** ENS Paris-Saclay / Université Paris-Cité — M2 Etudes anglophones - Linguistique anglaise  
**Date created:** 2025–2026  
**Language(s):** French (audio input), English (translation output), Python (code)

**Project description:**  
This notebook implements a two-stage speech processing pipeline on a sample of spoken Parisian French:

| Stage | Model | Task |
|-------|-------|------|
| A | `jonatasgrosman/wav2vec2-large-xlsr-53-french` | Automatic Speech Recognition (ASR) — French transcription |
| B | `facebook/wav2vec2-xls-r-1b-21-to-en` | Direct Speech Translation (ST) — French audio → English text |

The pipeline processes a set of max-30-second audio segments extracted from a single speaker interview in the CFPP2000 corpus and outputs a CSV file containing each segment filename, its French transcription, and its English translation. 

**Hardware note:** Stage B (XLS-R 1B) requires a GPU with at least 14 GB VRAM. A free Colab T4 runtime is sufficient. The notebook is structured to load and release each model sequentially to stay within VRAM limits.

---

## 2. Methodological Information

**Pipeline overview:**

1. **Data preparation** (`0_data_prep` section): the source recording is downloaded directly from the CFPP2000 public archive, resampled to 16 kHz mono, and split into consecutive non-overlapping segments saved to `data/segments/`.
2. **ASR — Stage A**: each segment is passed through `wav2vec2-large-xlsr-53-french`. Long segments are chunked with a 1-second overlap before inference, and chunk transcriptions are joined. The model is then deleted and VRAM freed before Stage B.
3. **ST — Stage B**: each segment is passed through `wav2vec2-xls-r-1b-21-to-en` (a `SpeechEncoderDecoderModel` with an MBart-50 decoder). Beam search (n=4) with a repetition penalty is used to reduce degeneration on longer segments.
4. **Export**: results are written to `output/transcriptions_xlsr.csv` with columns `fichier`, `transcription_fr`, `translation_en`.

**Software dependencies:** see `requirements.txt`. Pinned to `transformers==4.44.2` for compatibility with the XLS-R speech translation model.

**Reproducibility:** running all cells in order from a clean environment will reproduce the output without any manual data preparation step. The source audio URL is hardcoded in the data preparation cell.

---

## 3. Data and File Overview

```
xlsr-cfpp2000-pipeline/
├── README.md                  ← this file
├── LICENSE                    ← MIT licence (code)
├── requirements.txt           ← pinned Python dependencies
├── xlsr_pipeline.ipynb        ← main notebook (run top to bottom)
├── data/
│   ├── README.md              ← data-specific metadata and provenance
│   └── segments/              ← auto-populated by cell 0 (gitignored)
└── output/
    └── transcriptions_xlsr.csv ← pipeline output (gitignored; regenerate by running the notebook)
```

**Note:** `data/segments/` and `output/` are listed in `.gitignore`. They are not versioned.
Audio segments are archived on Zenodo: https://doi.org/10.5281/zenodo.19479351
Download and unzip into `data/segments/` before running Stage A and B cells.
The expected output CSV (`transcriptions_xlsr.csv`) is an artefact of running the notebook and is not committed.

---

## 4. Data-Specific Information

See `data/README.md` for full provenance, structure, and licence information on the audio data.

**Source corpus:** CFPP2000 (*Corpus de Français Parlé Parisien des années 2000*)  
**Speaker used:** Anita Musso (F, 46, 11th arrondissement of Paris)  
**Source file URL:** http://cfpp2000.univ-paris3.fr/data/public/11eme/Anita_MUSSO_F_46_11e/Anita_MUSSO_F_46_11e.wav  
**Segmentation:** 50 non-overlapping segments extracted starting at t=0 — archived at https://doi.org/10.5281/zenodo.19479351
**Corpus licence:** CC BY-NC-SA 4.0 (see `data/README.md`)

**Output CSV schema:**

| Column | Type | Description |
|--------|------|-------------|
| `fichier` | string | Segment filename (e.g. `segment_01.wav`) |
| `transcription_fr` | string | French transcription produced by XLSR-53 |
| `translation_en` | string | English translation produced by XLS-R 1B |

---

## Quickstart

```bash
# 1. Clone
git clone https://github.com/Zdv-dev-dv/xlsr-cfpp2000-pipeline.git
cd xlsr-cfpp2000-pipeline

# 2. Download audio segments from Zenodo
# https://doi.org/10.5281/zenodo.19479351
# Unzip into data/segments/

# 3. Install dependencies
pip install -r requirements.txt

# 4. Launch notebook
jupyter notebook xlsr_pipeline.ipynb
```

Or click the **Open in Colab** badge above and select a T4 GPU runtime (Runtime → Change runtime type → T4 GPU).

---

## Citation

If you use or build on this pipeline, please cite the CFPP2000 corpus:

> Branca-Rosoff, S., Fleury, S., Lefeuvre, F. & Pires, M. (2012). *Discours sur la ville. Présentation du Corpus de Français Parlé Parisien des années 2000 (CFPP2000)*. Université Sorbonne Nouvelle – Paris 3. http://cfpp2000.univ-paris3.fr
