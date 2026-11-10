# Lightweight On-Device Audio Deepfake Detection

**A 2.10 MB CNN for real-time voice-cloning detection on mobile phones — targeted at liveness verification in mobile-banking MFA flows.**

Voice-cloning technology can now generate synthetic speech perceptually indistinguishable from a target speaker from only a few seconds of reference audio. State-of-the-art anti-spoofing models (AASIST, RawNet2) report low Equal Error Rates in laboratory conditions but are **tens of megabytes** large — impractical for on-device execution — and degrade sharply under the codec compression and noise of real telephone calls. This repository is a reproducible implementation of a **lightweight CNN under 2 MB** that meets the on-device constraint while remaining competitive under realistic channel distortions.

---

## Headline results

| Metric | Lightweight CNN | Full CNN | Mel-GMM baseline |
|---|---|---|---|
| **Parameters** | **547,937** (2.10 MB) | 4,287,937 (16.37 MB) | — |
| **Clean EER** | **9.84 ± 1.11 %** (over 5 seeds) | 8.91 % | 19.93 % |
| **G.711 codec EER** | 13.33 % | 11.53 % | — |
| **AWGN @ 20 dB SNR EER** | **14.80 %** | 16.41 % | — |
| **CPU inference latency** | **4.962 ms** | 6.584 ms | — |
| **Majority-class baseline accuracy** | 89.68 % | 89.68 % | 89.68 % |

The lightweight model is **7.8× smaller** than the full CNN, runs inference end-to-end in **under 5 ms on a CPU**, and **outperforms the full model under noise and combined distortion** while coming within ~1 percentage point of its clean-condition EER. Both CNNs rely primarily on low-frequency spectral content (0–2 kHz), as confirmed by a band ablation study.

> **Important:** EER — not raw accuracy — is the primary metric throughout. The ASVspoof 2019 LA evaluation set has a 9 : 1 spoof-to-bonafide imbalance, so a trivial classifier that predicts "spoof" for every sample reaches 89.68 % accuracy with zero actual detection capability. Every accuracy number in this repo is reported alongside that majority-class baseline for transparency.

---

## What this repository contains

```
.
├── README.md                     ← this file
├── LICENSE                       ← MIT
├── .gitignore                    ← Python / Jupyter / Colab defaults
├── HOW_TO_PUSH.md                ← first-push instructions (ignore if cloning)
└── notebooks/
    ├── DeepFake_compiled.ipynb   ← end-to-end: data → features → GMM → CNNs
    │                               → distortion → quantization → SNR sweep
    │                               → latency → charts
    └── DeepFake_CNN_2_0.ipynb    ← polished evaluation pipeline that loads
                                    pre-trained models and reproduces the
                                    waveform-level distortion results and
                                    per-attack EER breakdown
```

### Which notebook should I run?

- **`DeepFake_compiled.ipynb`** — if you're starting from scratch and want to **train the models yourself**. Downloads ASVspoof 2019 LA via the Kaggle API, extracts Mel-spectrogram features, trains the Mel-GMM baseline and both CNNs, evaluates under distortion, measures inference latency, and generates all paper figures. Expect the full run to take several hours on a Colab T4 GPU.

- **`DeepFake_CNN_2_0.ipynb`** — if you already have the **pre-trained model checkpoints** (saved to Google Drive under `ICAST_Project/models/`) and just want to reproduce the evaluation and the charts. Faster — about 1 hour for the full waveform-level distortion sweep plus the per-attack EER breakdown.

---

## Dataset

**Primary training & evaluation:** [ASVspoof 2019 Logical Access](https://www.asvspoof.org/index2019.html).

- **Train split**: 25,380 samples (2,580 bonafide + 22,800 spoofed, A01–A06 synthesis systems)
- **Dev split**: 24,844 samples
- **Evaluation split**: **71,237 samples** (7,355 bonafide + 63,882 spoofed, A07–A19 synthesis systems — unseen at train time)
- **Majority-class baseline**: 89.68 %

Downloaded in-notebook via Kaggle (`anishsarkar22/asvpoof-2019-dataset-la` mirror). A direct Edinburgh DataShare fallback is included but currently returns 403 Forbidden — use the Kaggle route.

**Secondary evaluation (robustness):** ASVspoof 2021 LA — contains real telephony codec transmissions across 7 codecs and 4 transmission conditions, providing an ecologically valid benchmark for the telephone-channel claim. Download and evaluation steps are documented in `DeepFake_compiled.ipynb`.

---

## Model architectures

**Lightweight CNN** (547,937 params · 2.10 MB)

```
Input (1, 64, 128) Mel spectrogram, dB-scaled, min-max normalised
 ↓
Conv2d(1 → 16, 3×3) → BatchNorm → ReLU → MaxPool(2,2)
Conv2d(16 → 32, 3×3) → BatchNorm → ReLU → MaxPool(2,2)
Conv2d(32 → 64, 3×3) → BatchNorm → ReLU → MaxPool(2,2)
 ↓
Flatten → Dropout(0.3) → Linear(8192 → 64) → ReLU → Linear(64 → 1) → Sigmoid
```

**Full CNN** (4,287,937 params · 16.37 MB)

```
Input (1, 64, 128)
 ↓
Conv2d(1 → 32)   → BatchNorm → ReLU → MaxPool(2,2)
Conv2d(32 → 64)  → BatchNorm → ReLU → MaxPool(2,2)
Conv2d(64 → 128) → BatchNorm → ReLU → MaxPool(2,2)
 ↓
Flatten → Dropout(0.3) → Linear(16384 → 256) → ReLU → Dropout(0.2)
 → Linear(256 → 1) → Sigmoid
```

**Mel-GMM baseline** — two GMMs (bonafide, spoof) fit on flattened Mel spectrograms after standard scaling; decision by log-likelihood ratio.

**INT8 quantised Lightweight CNN** — dynamic quantisation of the two Linear layers via `torch.quantization.quantize_dynamic`; same accuracy, further size and CPU-latency reduction.

---

## Experimental methodology

The paper's central methodological contribution: **all channel distortions are applied at the waveform level BEFORE feature extraction.** Previous work typically applied distortion directly to spectrograms, which produces physically impossible signals — a real G.711 codec compresses the time-domain waveform, not a Mel bin.

Four distortion conditions are tested:

| Condition | What it simulates |
|---|---|
| **Clean** | Uncorrupted audio (baseline) |
| **G.711 codec** | Telephony-grade μ-law compression at 8 kHz effective sampling |
| **AWGN @ 20 dB SNR** | Thermal / environmental noise typical of mobile microphones |
| **Combined** | G.711 codec followed by AWGN — telephone call with background noise |

Plus a systematic **SNR sweep from 0 to 30 dB AWGN** and the **ASVspoof 2021 LA real-telephony benchmark**.

---

## Requirements

- Python 3.9+ (the notebooks were developed and tested on Python 3.13 in Google Colab)
- PyTorch ≥ 2.0 with CUDA for training (CPU is fine for evaluation / latency measurement)
- `librosa`, `soundfile`, `numpy`, `pandas`, `scikit-learn`, `matplotlib`
- `kaggle` CLI (for downloading ASVspoof 2019 LA)

Both notebooks run end-to-end on **Google Colab** without a local environment. Mounting Google Drive is used for persisting features, model checkpoints, results CSVs, and charts across sessions.

---

## Reproducing the results

### Option A — from scratch in Colab

1. Open `notebooks/DeepFake_compiled.ipynb` in Google Colab (File → Open notebook → GitHub tab → paste this repo URL).
2. Upload your personal `kaggle.json` when prompted in cell 1 (Settings → API → *Create new API token* on your Kaggle account).
3. Run all cells top-to-bottom. The first run takes several hours on a T4; subsequent runs on the saved checkpoints are much faster.

### Option B — evaluation only

1. Run `DeepFake_compiled.ipynb` once to generate and save the model checkpoints to your Google Drive.
2. For every subsequent experiment, run `DeepFake_CNN_2_0.ipynb`. It loads the checkpoints from Drive and reproduces the waveform-level distortion results, the per-attack EER breakdown, and the charts.

### Random seeds

All results in the paper are reported as the **mean ± standard deviation across 5 random seeds** (42, 123, 456, 789, 1001). The seeding function (`set_all_seeds`) is at the top of both notebooks and covers `random`, NumPy, PyTorch (CPU + CUDA), and `torch.backends.cudnn`.

---

## License

MIT — see [`LICENSE`](LICENSE). You are free to use, modify and redistribute this code with attribution.

---

## Contact

Questions about the code, reproducibility, or extensions: [isaiahanim20@gmail.com](mailto:isaiahanim20@gmail.com).
