# Data README — XLSR Pipeline / CFPP2000 Segments

---

## 1. General Information

**Dataset title:** 30-second audio segments extracted from CFPP2000 — Anita Musso (11th arrondissement, Paris)  
**Derived from:** CFPP2000 (*Corpus de Français Parlé Parisien des années 2000*)  
**Original corpus URL:** http://cfpp2000.univ-paris3.fr  
**Source recording URL:** http://cfpp2000.univ-paris3.fr/data/public/11eme/Anita_MUSSO_F_46_11e/Anita_MUSSO_F_46_11e.wav  
**Corpus creators:** Branca-Rosoff, S., Fleury, S., Lefeuvre, F. & Pires, M. — Université Sorbonne Nouvelle – Paris 3  
**Segmentation by:** Zoé de Vries (ENS Paris-Saclay / Université Paris-Cité, 2025–2026)

---

## 2. Methodological Information

**How segments were created:**  
The source recording (`Anita_MUSSO_F_46_11e.wav`) was downloaded from the CFPP2000 public archive. It was resampled to 16 000 Hz mono using `librosa`. The resampled signal was then split into 50 consecutive non-overlapping segments of exactly 30 seconds (480 000 samples at 16 kHz). Any trailing audio shorter than 30 seconds was discarded. This procedure is fully automated in cell 0 of `xlsr_pipeline.ipynb`.

**Speaker metadata (from CFPP2000):**  
- Speaker ID: Anita_MUSSO  
- Sex: F  
- Age: 46  
- Arrondissement: 11th (Paris)  
- Interview type: sociolinguistic interview on urban life in Paris  
- Corpus section: public data (freely downloadable without registration)

**Audio format:**

| Property | Value |
|----------|-------|
| Format | WAV (PCM) |
| Sample rate | 16 000 Hz (resampled from source) |
| Channels | 1 (mono) |
| Bit depth | 16-bit |
| Segment duration | 30 seconds |
| Number of segments | 50 |

---

## 3. File Overview

The `data/segments/` folder is **not versioned** in this repository. It is generated automatically by running cell 0 of `xlsr_pipeline.ipynb`, which downloads the source WAV and performs the segmentation.

Expected file structure after running cell 0:

```
data/segments/
├── segment_01.wav
├── segment_02.wav
├── ...
└── segment_50.wav
```

---

## 4. Provenance and Licence

**Source corpus licence:** CC BY-NC-SA 4.0  
Full licence text: https://creativecommons.org/licenses/by-nc-sa/4.0/

The CFPP2000 corpus is publicly accessible for research and educational use under the above licence. The segments in `data/segments/` are derivative works of the CFPP2000 corpus and are therefore also distributed under CC BY-NC-SA 4.0. **They may not be used for commercial purposes.**

The pipeline code (`xlsr_pipeline.ipynb`, `requirements.txt`) is licensed separately under the MIT licence (see `LICENSE` at the root of this repository).

**Reference for the corpus:**

> Branca-Rosoff, S., Fleury, S., Lefeuvre, F. & Pires, M. (2012). *Discours sur la ville. Présentation du Corpus de Français Parlé Parisien des années 2000 (CFPP2000)*. Université Sorbonne Nouvelle – Paris 3. http://cfpp2000.univ-paris3.fr
