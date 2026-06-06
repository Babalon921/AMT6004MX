AMT6004MX FINAL

Direct Download of Required Features:
https://zenodo.org/records/2553414/files/acousticbrainz-mediaeval-features-train-45.tar.bz2

# AMT6004MX — Music Information Retrieval & BPM Prediction

> **University of Plymouth** · Audio & Music Technology · Programming Portfolio  
> Student: Harry Gray · October 2025

---

## Overview

A Music Information Retrieval (MIR) pipeline and Machine Learning model built in Python, trained on the **MediaEval AcousticBrainz Genre** dataset (~182,978 tracks). The project processes large-scale JSON audio feature data, cleans and converts it to CSV, visualises danceability distributions, and trains a **Random Forest Regressor** to predict BPM from rhythm-based audio features.

---

## Features

| Feature | Description |
|---|---|
| **Multi-threaded preprocessing** | 16-worker thread pool for fast JSON validation and conversion at scale |
| **JSON → CSV pipeline** | Flattens nested JSON audio features into ML-ready CSV format |
| **Data visualisation** | Danceability line plots and KDE histograms via matplotlib & Seaborn |
| **Random Forest Regressor** | BPM prediction from rhythm features (onset rate, beat count, histogram data) |
| **Hyperparameter tuning** | Two model variants (A/B) with overfitting analysis and comparison |
| **Baseline benchmarking** | DummyRegressor baseline for model validation |
| **Ethical analysis** | Documented bias considerations around western music over-representation |

---

## Results

| Metric | Baseline | Model A (Test) | Model B (Test) |
|---|---|---|---|
| MAE | 20.50 | 1.095 | **1.058** |
| RMSE | 25.03 | 4.427 | **4.392** |
| R² | 0.00 | 0.969 | **0.969** |

Model B achieves a **~94.8% improvement** in MAE over the baseline dummy regressor, with better generalisation than Model A (train/test MAE gap reduced from 0.674 → 0.192).

---

## Dataset

**MediaEval AcousticBrainz Genre** — Bogdanov et al., 2018  
DOI: [10.5281/zenodo.2553414](https://doi.org/10.5281/zenodo.2553414)

- Source file: `acousticbrainz-mediaeval-features-train-45.tar.bz2`
- ~182,978 tracks used (subset of the full 29M track collection)
- Audio features extracted using [Essentia](https://essentia.upf.edu/) (tonal, rhythm, lowlevel, metadata)
- Each track stored as a uniquely referenced JSON (e.g. `53a0bb05-3564-4e52-b28c-ffcc0369dcd8.json`)

---

## Pipeline

### 1. Preprocessing

```python
# Set your local dataset path
DATA = r"path/to/acousticbrainz-mediaeval-train"
THREADS = 16
```

- Recursively walks directories (`4a–4f`, `40–49`, `5a–5f`, `50–59`)
- Validates files by reading the first 1024 bytes
- Converts valid JSON files to flat CSVs using a fixed header scheme
- Missing values handled via `item.get(k, None)` — excluded rather than imputed

### 2. ML Features Used

```python
FEATURES = {
    "onset_rate", "beats_count", "bpm",
    "danceability", "bpm_histogram_mean",
    "bpm_histogram_first_peak_bpm",
    "bpm_histogram_second_peak_spread",
    "bpm_histogram_second_peak_weight"
}
TARGET_FEATURE = "bpm"
```

All features are drawn from Essentia's `rhythm` analysis profile, keeping feature correlation high and improving prediction accuracy.

### 3. Model

**Random Forest Regressor** (scikit-learn) — chosen for its non-linear handling of musical features, resistance to outliers, and scalability with high-dimensional data.

**Model B hyperparameters (final):**
```python
RandomForestRegressor(
    n_estimators=500,
    max_depth=15,
    min_samples_split=5,
    min_samples_leaf=3,
    max_features='sqrt',
    random_state=42,
    n_jobs=-1
)
```

---

## Installation

```bash
git clone https://github.com/Babalon921/AMT6004MX
cd AMT6004MX
pip install -r requirements.txt
```

**Dependencies:** `pandas`, `numpy`, `scikit-learn`, `matplotlib`, `seaborn`, `tqdm`

---

## Ethical Considerations

This model was trained on community-contributed data that disproportionately reflects **Western music** (rock, pop) via AllMusic and Discogs sources. Key concerns:

- **Genre bias** — BPM predictions are less accurate for non-Western music with irregular time signatures or polyrhythmic structures
- **Temporal bias** — Tempo rubato and non-standard time signatures (7/4, 11/8, 13/8) may be misclassified
- **Lack of consent** — Artists have limited control over how extracted features are used
- **Potential misuse** — BPM predictions used in copyright detection or recommendation systems could discriminate against underrepresented genres

If deployed, transparency about these limitations is strongly advised.

---

## Limitations

- JSON nesting is partially flattened — deeper nested features may be lost
- No deduplication handling in the current pipeline
- Model struggles with syncopated structures (e.g. 88 BPM tracks represented as 176 BPM)
- No anti-aliasing for tempo ambiguity (octave errors in BPM detection)

---

## References
Key sources: [AcousticBrainz](https://acousticbrainz.org) · [Essentia](https://essentia.upf.edu) · [scikit-learn RandomForestRegressor](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestRegressor.html) · Bogdanov et al. (2018)

---

*University of Plymouth · AMT6004MX · 2025*
