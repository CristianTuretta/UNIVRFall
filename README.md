# UNIVRFall: IMU-based pre-impact, impact, and post-impact fall detection dataset

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.18346755.svg)](https://doi.org/10.5281/zenodo.18346755)
[![Paper](https://img.shields.io/badge/Paper-PerCom%20Workshops%202026-blue)](https://doi.org/10.1109/PerComWorkshops68308.2026.11585385)
[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

**UNIVRFall** is an IMU dataset for **pre-impact, impact, and post-impact fall detection** in daily life and in work settings. Most fall datasets focus on elderly care. This one covers working-age adults and includes **elevation-related falls** (ladders, scaffolds, falls from height), which are a leading cause of workplace deaths. It also includes **42.6 hours of real construction-site recordings**.

The structure follows [KFall](https://doi.org/10.3389/fnagi.2021.692865), so the two datasets can be combined and compared directly. Falls are rare (2.22% of lab data, 0.16% overall) and short (150–1200 ms), and the target hardware is a microcontroller. That makes the dataset a good benchmark for **AutoML**: neural architecture search, hyperparameter tuning, and model compression for edge AI.

> 📦 **The data is hosted on Zenodo:** <https://doi.org/10.5281/zenodo.18346755>
> This repository holds the documentation, the citation information, and usage examples.

---

## Contents

- [Highlights](#highlights)
- [Download](#download)
- [Data collection](#data-collection)
- [Activity and fall taxonomy](#activity-and-fall-taxonomy)
- [Dataset structure](#dataset-structure)
- [File formats](#file-formats)
- [Preprocessing](#preprocessing)
- [Baseline results](#baseline-results)
- [Citation](#citation)
- [License](#license)
- [Acknowledgments](#acknowledgments)

---

## Highlights

| Characteristic | Value |
|---|---|
| Participants | 39 (29 laboratory + 10 construction workers) |
| Laboratory participants | 29 (24 M / 5 F), aged 20–49 (mean 23.6 ± 6.2) |
| Activities of daily living (ADLs) | 24 (23 in the lab + 1 on-site) |
| Fall types | 21, 6 of them elevation-specific |
| Total recording time | ~46.05 h |
| Lab recordings | 3.41 h (ADLs 2.35 h, falls 1.06 h) |
| Construction-site recordings | 42.64 h (no falls) |
| Annotated fall events | 573 |
| Fall duration (onset → impact) | 150–1200 ms (mean 476.70 ± 183.91 ms) |
| Class imbalance | 2.22% falling (lab only), 0.16% (entire dataset) |
| Sensor | 6-axis IMU (accelerometer + gyroscope) at 100 Hz, lower back (L1–L2) |
| Ground truth | Frame-level labels from video synchronized at 100 fps |

Compared with other public fall datasets:

| Dataset | Subjects | ADLs | Falls | Work tasks | Worksite data |
|---|:-:|:-:|:-:|:-:|:-:|
| KFall | 32 | 21 | 15 | No | No |
| SisFall | 38 | 19 | 15 | No | No |
| MobiAct | 66 | 9 | 4 | No | No |
| FARSEEING | 15 | Real | Real | No | Yes |
| **UNIVRFall** | **29 (+10)** | **24** | **21** | **Yes** | **Yes** |

---

## Download

The dataset is a single archive, `UniVrFall_Dataset.zip` (~675 MB, 1,252 CSV files plus 29 annotation files).

```bash
# Download from Zenodo
curl -L -o UniVrFall_Dataset.zip \
  "https://zenodo.org/records/18346755/files/UniVrFall_Dataset.zip?download=1"

# Verify integrity (expected MD5: d14190e98749bc583f69f1df3e5bafc0)
md5sum UniVrFall_Dataset.zip        # Linux
md5 UniVrFall_Dataset.zip           # macOS

unzip UniVrFall_Dataset.zip
```

---

## Data collection

### Wearable device

The sensor board sits in the back panel of a **smart safety jacket** with an airbag system, which the fall detection algorithm can trigger before ground impact.

| Feature | Specification |
|---|---|
| Microcontroller | STM32F722RET6 (ARM Cortex-M7, up to 216 MHz, 512 KB Flash, 256 KB SRAM) |
| Accelerometer | LIS3DH tri-axis MEMS, ±16 g, 1 mg resolution |
| Gyroscope | LSM6DS3 tri-axis, ±2000 dps, 0.07 dps resolution |
| Sampling frequency | 100 Hz (both sensors) |
| Placement | Lower back (vertebrae L1–L2) |
| Synchronization | 3 front LEDs (Red → Yellow → Green) and a 100 fps camera |
| Storage / connectivity | 32 GB microSD, Bluetooth 5.0 (remote control) |
| Battery life | > 8 h of continuous recording |

**Coordinate system:** the x-axis points downward, the y-axis points to the right, and the z-axis is perpendicular to the board.

### Laboratory protocol (29 participants)

Each participant completed a 45-minute session: orientation (5 min), ADLs (20 min), fall simulations (15 min), and debriefing (5 min). The 23 lab ADLs and 21 fall types were performed in random order. Each activity lasted 20–30 s, and each fall type was performed once onto 15 cm foam padding. A 100 fps camera recorded every trial for annotation.

### Construction-site recordings (10 workers)

Ten workers wore the jacket during normal shifts (0.8 to 11.5 h per session, **Task 88**). The recordings cover scaffold climbing, material transport, heavy lifting, and operating equipment. No falls were induced. Only sensor data was recorded (no video or audio), and worker identities are anonymized.

### Synchronization and annotation

At the start of each session, the camera and the sensor logger start together. The jacket LEDs then run a fixed sequence (RED 500 ms → YELLOW 250 ms → GREEN 250 ms). The switch to GREEN is the reference point for aligning video and sensor data.

A trained annotator went through the video and sensor data frame by frame with the [Python Video Annotator](https://pythonvideoannotator.readthedocs.io/) and marked two frames for each fall:

1. **Fall onset**: the participant enters unrecoverable free fall (loss of postural control with downward acceleration).
2. **Ground impact**: the torso touches the ground.

These two frames split each fall trial into the **pre-fall, falling, impact, and post-fall** phases.

---

## Activity and fall taxonomy

Task IDs follow the KFall numbering. **Bold** rows are additions specific to this dataset.

<details>
<summary><b>Activities of daily living (24)</b></summary>

| ID | Activity | Duration (s) |
|:-:|---|:-:|
| 01 | Stand for 30 seconds | 9–44 |
| 02 | Stand, slowly bend, tie shoe lace, get up | 7–22 |
| 03 | Pick up an object from the floor | 3–15 |
| 04 | Gently jump (try to reach an object) | 3–12 |
| 05 | Stand, sit to ground, wait, get up with normal speed | 7–23 |
| 06 | Walk normally with turn | 14–26 |
| 07 | Walk quickly with turn | 10–21 |
| 08 | Jog normally with turn | 9–13 |
| 09 | Jog quickly with turn | 3–27 |
| 10 | Stumble with obstacle while walking | 4–16 |
| 11 | Sit on a chair for 30 seconds | 18–37 |
| 12 | Walk downstairs normally | 7–33 |
| 13 | Sit down to chair normally, get up from chair normally | 8–14 |
| 14 | Sit down to chair quickly, get up from chair quickly | 4–11 |
| 15 | Sit a moment, trying to get up, collapse into chair | 6–12 |
| 16 | Walk downstairs quickly | 5–12 |
| 17 | Lie on the floor for 30 seconds | 26–40 |
| 18 | Sit, lie down to floor normally, get up normally | 10–22 |
| 19 | Sit, lie down to floor quickly, get up quickly | 8–14 |
| 35 | Walk upstairs normally | 9–19 |
| 36 | Walk upstairs quickly | 5–14 |
| **43** | **Climb up and climb down the stairs** *(work-specific, lab)* | 10–19 |
| **44** | **Walk slowly and jump over the obstacle** *(work-specific, lab)* | 5–10 |
| **88** | **On-field construction-site activities without falling** *(real worksite)* | 0.8–11.5 h |

</details>

<details>
<summary><b>Fall types (21)</b></summary>

| ID | Fall | Category | Duration (s) |
|:-:|---|---|:-:|
| 20 | Forward fall when trying to sit down | Sitting transitions | 2–7 |
| 21 | Backward fall when trying to sit down | Sitting transitions | 2–5 |
| 22 | Lateral fall when trying to sit down | Sitting transitions | 2–5 |
| 23 | Forward fall when trying to get up | Sitting transitions | 2–6 |
| 24 | Lateral fall when trying to get up | Sitting transitions | 3–7 |
| 25 | Forward fall while sitting, caused by fainting | Fainting | 2–4 |
| 26 | Lateral fall while sitting, caused by fainting | Fainting | 2–6 |
| 27 | Backward fall while sitting, caused by fainting | Fainting | 2–5 |
| 28 | Vertical (forward) fall while walking caused by fainting | Fainting | 3–6 |
| 29 | Fall while walking, using hands to dampen fall, caused by fainting | Fainting | 3–6 |
| 30 | Forward fall while walking caused by a trip | Falls while moving | 3–6 |
| 31 | Forward fall while jogging caused by a trip | Falls while moving | 2–5 |
| 32 | Forward fall while walking caused by a slip | Falls while moving | 3–6 |
| 33 | Lateral fall while walking caused by a slip | Falls while moving | 3–6 |
| 34 | Backward fall while walking caused by a slip | Falls while moving | 2–5 |
| **37** | **Backward fall while slowly moving back** | **Elevation** | 3–8 |
| **38** | **Backward fall while quickly moving back** | **Elevation** | 2–4 |
| **39** | **Forward fall from height** | **Elevation** | 3–4 |
| **40** | **Backward fall from height** | **Elevation** | 2–6 |
| **41** | **Backward fall while climbing up the ladder** | **Elevation** | 4–12 |
| **42** | **Backward fall while climbing down the ladder** | **Elevation** | 3–7 |

Elevation falls (tasks 39–42) have longer pre-impact phases (700–1000 ms) than ground-level falls (300–600 ms). That gives a protective system more time to react.

</details>

---

## Dataset structure

```
UniVrFall_Dataset/
├── laboratory/
│   ├── sensors_data/                 # 29 subjects (SA09 … SA37), 1,234 trials
│   │   ├── SA09/
│   │   │   ├── S09T01R01.csv         # Subject 09, Task 01, Repetition 01
│   │   │   ├── S09T02R01.csv
│   │   │   └── ...
│   │   ├── ...
│   │   └── SA37/
│   └── labels_data/                  # Fall annotations, one file per subject
│       ├── SA09_label.xlsx
│       ├── ...
│       └── SA37_label.xlsx
└── OnField/                          # Construction site, 10 sessions, 18 recordings
    ├── 20241106/
    │   ├── 20241106_999.csv          # <session date>_<ID>.csv
    │   └── 20241106_1000.csv
    ├── 20241223/
    ├── ...
    └── 20250218/
```

- **Laboratory files:** `S[ID]T[XX]R[ZZ].csv`, where `ID` is the subject number, `XX` the task ID (see the [taxonomy](#activity-and-fall-taxonomy)), and `ZZ` the trial repetition.
- **Annotation files:** `SA[ID]_label.xlsx`, one per subject.
- **On-field files:** `<YYYYMMDD>_<ID>.csv`, grouped by recording session date.

---

## File formats

### Laboratory sensor data (`.csv`, 11 columns, 100 Hz)

| # | Column | Raw unit | Description |
|:-:|---|---|---|
| 1 | `TimeStamp(s)` | ms | Sample timestamp |
| 2 | `FrameCounter` | integer | Sequential frame number (matches the annotation frames) |
| 3 | `AccX` | mg | Acceleration, x-axis |
| 4 | `AccY` | mg | Acceleration, y-axis |
| 5 | `AccZ` | mg | Acceleration, z-axis |
| 6 | `GyrX` | mdps | Angular velocity, x-axis (roll) |
| 7 | `GyrY` | mdps | Angular velocity, y-axis (pitch) |
| 8 | `GyrZ` | mdps | Angular velocity, z-axis (yaw) |
| 9 | `EulerX` | degrees | Roll orientation angle |
| 10 | `EulerY` | degrees | Pitch orientation angle |
| 11 | `EulerZ` | degrees | Yaw orientation angle |

> **Units.** The raw files store acceleration in **mg** and angular velocity in **mdps**, as the sensor outputs them. The paper reports values after conversion to **m/s²** and **rad/s** (`acc * 9.80665 / 1000`, `gyr * π / 180 / 1000`), which is the conversion used to align the data with KFall.

### On-field sensor data (`.csv`)

The construction-site files come straight from the jacket logger and use different column names. The relevant columns are:

| Column | Description |
|---|---|
| `timestamp` | Sample timestamp (ms) |
| `type` | Record type. Keep rows where `type == 1` (IMU samples) |
| `accX`, `accY`, `accZ` | Acceleration (mg) |
| `gyrX`, `gyrY`, `gyrZ` | Angular velocity (mdps) |
| `roll`, `pitch`, `yaw` | Orientation angles (degrees) |

The remaining columns (`id`, `reserved1-3`, `vIn`, `vBatt`, `hall`, `inputs`, `outputs`, `aiOut0`, `aiOut1`) are device and status fields. Fall detection does not need them.

### Annotations (`.xlsx`)

| # | Column | Type | Description |
|:-:|---|---|---|
| 1 | `Task Code (Task ID)` | string | Fall code and numeric task ID, e.g. `F01 (20)` for fall type 20 |
| 2 | `Description` | string | Description of the fall |
| 3 | `Trial ID` | integer | Trial repetition number |
| 4 | `Fall_onset_frame` | integer | Frame where the fall starts |
| 5 | `Fall_impact_frame` | integer | Frame of ground contact |

The frame numbers refer to the `FrameCounter` column of the matching sensor file.

---

## Preprocessing

The usual pipeline, as used in the papers below:

1. **Unit conversion and alignment with KFall:** convert to m/s² and rad/s, then rotate the axes to match KFall.
2. **Low-pass filter:** Butterworth filter with a 5 Hz cutoff on each IMU channel.
3. **Phase labeling** from the annotation frames: *activity*, *falling* (onset → impact, minus the last 150 ms before impact, which leaves a pre-impact window for airbag deployment), *impact*, and *post-fall*.
4. **Windowing:** fixed-size windows with overlap, for example 300–400 ms with 50% overlap.

### Minimal example (Python)

```python
import numpy as np
import pandas as pd
from scipy.signal import butter, filtfilt

FS = 100  # Hz


def low_pass_filter(data, cutoff=5, fs=FS, order=4):
    b, a = butter(order, cutoff / (0.5 * fs), btype="low")
    return filtfilt(b, a, data, axis=0)


def windowing(x, y, window_ms=300, overlap=50, fs=FS):
    n = int(window_ms / 1000 * fs)
    step = int(n * (1 - overlap / 100))
    xs, ys = [], []
    for start in range(0, len(x) - n + 1, step):
        xs.append(x[start:start + n])
        ys.append(y[start])
    return np.asarray(xs), np.asarray(ys)


# Laboratory trial: subject 21, task 30 (forward fall while walking, trip)
df = pd.read_csv("UniVrFall_Dataset/laboratory/sensors_data/SA21/S21T30R01.csv",
                 index_col="FrameCounter")
labels = pd.read_excel("UniVrFall_Dataset/laboratory/labels_data/SA21_label.xlsx").ffill()

cols = ["AccX", "AccY", "AccZ", "GyrX", "GyrY", "GyrZ"]
x = df[cols].to_numpy(dtype=float)
x[:, :3] *= 9.80665 / 1000          # mg   -> m/s^2
x[:, 3:] *= np.pi / 180 / 1000      # mdps -> rad/s
x = low_pass_filter(x)

# Label the falling phase from the annotated onset and impact frames
row = labels[labels["Task Code (Task ID)"].str.contains(r"\(30\)")].iloc[0]
onset, impact = int(row["Fall_onset_frame"]), int(row["Fall_impact_frame"])
y = np.where((df.index >= onset) & (df.index < impact), "Falling", "Activity")

X, Y = windowing(x, y, window_ms=300, overlap=50)
print(X.shape, np.unique(Y, return_counts=True))
```

For on-field data, keep `type == 1` rows and use the `accX … gyrZ` columns. Every on-field window is labeled `Activity`.

---

## Baseline results

The dataset was used together with KFall to train a **lightweight CNN** for pre-impact fall detection [[DATE 2025](https://doi.org/10.23919/DATE64628.2025.10993022)]. Inputs were 400 ms windows with 50% overlap, evaluated with **5-fold subject-independent cross-validation**.

**KFall and UNIVRFall combined:**

| Model | Accuracy | Precision | Recall | F1-score |
|---|:-:|:-:|:-:|:-:|
| MLP | 96.45% | 60.23% | 54.63% | 54.25% |
| LSTM | 97.60% | 85.97% | 75.74% | 79.81% |
| ConvLSTM2D | 97.10% | 85.57% | 65.36% | 70.75% |
| **CNN** | **98.28%** | **90.40%** | **83.95%** | **86.69%** |

**Cross-dataset generalization (F1-score):**

| Train ↓ / Test → | UNIVRFall | KFall |
|---|:-:|:-:|
| UNIVRFall | 87.73% | 83.71% |
| KFall | 85.22% | 93.23% |

**On-device (STM32F722RET6):** 67 KB Flash, 17 KB RAM, 4 ms inference per window.

---

## Citation

If you use this dataset, please cite **both** of the following papers:

```bibtex
@inproceedings{turetta2025lightweight,
  title={A Lightweight CNN for Real-Time Pre-Impact Fall Detection},
  author={Turetta, Cristian and Ali, Muhammad Toqeer and Demrozi, Florenc and Pravadelli, Graziano},
  booktitle={2025 Design, Automation \& Test in Europe Conference (DATE)},
  year={2025},
  organization={IEEE}
}

@inproceedings{ali2026imu_fall_dataset,
  title     = {IMU-based pre-impact, impact, and post-impact fall detection dataset},
  author    = {Ali, Muhammad Toqeer and Turetta, Cristian and Demrozi, Florenc and Pravadelli, Graziano},
  booktitle = {Proceedings of the 22nd International Workshop on Context and Activity Modeling and Recognition with AI (CoMoRe-AI 2026), co-located with the 24th IEEE International Conference on Pervasive Computing and Communications (PerCom 2026 Workshops)},
  year      = {2026},
  address   = {Pisa, Italy},
  month     = mar,
  note      = {Accepted for publication}
}
```

Papers:
- *A Lightweight CNN for Real-Time Pre-Impact Fall Detection*, DATE 2025: [10.23919/DATE64628.2025.10993022](https://doi.org/10.23919/DATE64628.2025.10993022)
- *IMU-based pre-impact, impact, and post-impact fall detection dataset*, CoMoRe-AI @ PerCom 2026: [10.1109/PerComWorkshops68308.2026.11585385](https://doi.org/10.1109/PerComWorkshops68308.2026.11585385)

Dataset: [10.5281/zenodo.18346755](https://doi.org/10.5281/zenodo.18346755)

---

## License

The dataset and this documentation are released under the [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/) license. See [LICENSE](LICENSE).

## Authors

- **Muhammad Toqeer Ali**, University of Verona ([ORCID](https://orcid.org/0009-0003-4368-211X))
- **Cristian Turetta**, Wenzhou Business College ([ORCID](https://orcid.org/0000-0002-8018-0472))
- **Florenc Demrozi**, University of Stavanger ([ORCID](https://orcid.org/0000-0002-5422-9826))
- **Graziano Pravadelli**, University of Verona ([ORCID](https://orcid.org/0000-0002-7833-1673))

## Acknowledgments

- **Protechto s.r.l.** for providing the smart safety jacket and sensor system
- The **29 laboratory participants** and the **10 construction workers** who contributed data
- The **University of Verona** for supporting this research
- Funding projects: PREPARE, UNISCO, CollaborICE (MICS Extended Partnership)
