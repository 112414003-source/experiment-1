**Injected and Leaked: Actively Inducing Side-channel Leakage Using Electromagnetic Injection and Hardware Non-Linearity**

- Website page: https://injecteave.github.io/

## Repository layout (current snapshot)

```text
.
|-- README.md
|-- Case study audio/
|   |-- Cross-wall eavesdropping/
|   |   |-- Clean/                                     (6 wav)
|   |   `-- Eavesdropped/
|   |       |-- home/                                  (6 wav)
|   |       `-- meeting room/                          (6 wav)
|   |-- Closed-loop manipulation/
|   |   |-- Clean audio/                               (3 wav)
|   |   |-- Eavesdropped audio/                        (3 wav)
|   |   |-- Enhanced eavesdropped audio/               (3 wav)
|   |   |-- Synthesized audio/                         (3 wav)
|   |   |-- Audio recorded from Alice/                 (3 wav)
|   |   `-- Audio recorded from injection/             (3 wav)
|   `-- Long distance eavesdropping/                   (4 wav, 1 mp3)
`-- Enhance Code/
    |-- README.md
    |-- requirements.txt
    |-- train.py
    |-- enhancement.py
    |-- checkpoints/
    |   `-- model.ckpt
    |-- configs/
    |   `-- train_config.yaml
    |-- data/
    |   |-- clean/
    |   |   `-- README.md
    |   |-- noise/
    |   |   `-- 001.wav ... 118.wav
    |   |-- audio_utils.py
    |   `-- nonlinear_simulation.py
    |-- sgmse/
    `-- preprocess.py
```

## Data files

### Case-study audio

- `Case study audio/Cross-wall eavesdropping/Clean/`: clean source utterances used in the cross-wall eavesdropping case studies.
- `Case study audio/Cross-wall eavesdropping/Eavesdropped/home/` and `Case study audio/Cross-wall eavesdropping/Eavesdropped/meeting room/`: eavesdropped audio recovered in the home/hotel-style and meeting-room settings.
- `Case study audio/Closed-loop manipulation/`: clean, eavesdropped, enhanced, synthesized, Alice-recorded, and injection-recorded audio for the closed-loop eavesdrop-synthesize-inject case study.
- `Case study audio/Long distance eavesdropping/`: clean reference audio and eavesdropped audio at 12 m, 18 m, 24 m, and 30 m.

### Speech-enhancement training sources

- Noise source files distributed for preprocessing: `Enhance Code/data/noise/` (118 WAV files).

### Clean Speech Dataset

Clean utterances used to construct the paired speech-enhancement data are from the
**LibriSpeech `train-clean-100` subset**, released as part of **OpenSLR SLR12**.

- Official release: <http://www.openslr.org/12/>
- Resource identifier: SLR12
- Dataset archive used: `train-clean-100.tar.gz`
- License: CC BY 4.0
- Audio in this directory is not redistributed with the artifact. Download
  `train-clean-100.tar.gz` from the official OpenSLR release before running
  preprocessing.

#### Citation

Please cite LibriSpeech as:

```bibtex
@inproceedings{panayotov2015librispeech,
  title={Librispeech: an asr corpus based on public domain audio books},
  author={Panayotov, Vassil and Chen, Guoguo and Povey, Daniel and Khudanpur, Sanjeev},
  booktitle={2015 IEEE international conference on acoustics, speech and signal processing (ICASSP)},
  pages={5206--5210},
  year={2015},
  organization={IEEE}
}
```

## Speech Enhancement Code

Run the commands below from the `Enhance Code/` directory in this artifact.

### 1. Install dependencies

```bash
cd "Enhance Code"
python -m pip install -r requirements.txt
```

Requirements:
- Python 3.8+
- CUDA 11.8+
- 16GB+ GPU memory

### 2. Prepare training data pairs

Preprocessing first mixes clean speech with a selected noise clip at a random
SNR, then applies the nonlinear channel simulation implemented in
`data/nonlinear_simulation.py` before saving the paired `clean` and `noisy` files.

```bash
python preprocess.py \
  --train_clean_dir /path/to/train_clean \
  --valid_clean_dir /path/to/valid_clean \
  --noise_dir ./data/noise \
  --output_dir ./preprocessed \
  --snr_min -15 \
  --snr_max 15 \
  --num_loaders 16 \
  --num_savers 8 \
  --gpu_batch_size 16
```

### 3. Train

Training parameters are configured in `Enhance Code/configs/train_config.yaml`,
including dataset paths, batch size, learning rate, number of epochs, STFT
settings, SDE settings, and checkpoint output directory.

```bash
python train.py \
  --config configs/train_config.yaml \
  --gpus 1 \
  --seed 42
```

### 4. Inference

```bash
python enhancement.py \
  --test_file <input_file> \
  --enhanced_dir enhanced_audio \
  --ckpt checkpoints/model.ckpt \
  --N 30 \
  --device cuda
```
