# ECG Anomaly Detection with Autoencoders

This repository contains a notebook exploring the use of Autoencoders trained on normal segmented ECGs for anomaly detection.

---

## Data Description

- **Data Source:**  
  12-lead recordings from the PTB Diagnostic ECG Database via PhysioNet.

- **Reference:**  
  Wagner, P., Strodthoff, N., Bousseljot, R., Samek, W., & Schaeffter, T. (2022). PTB-XL, a large publicly available electrocardiography dataset (version 1.0.3). PhysioNet. RRID:SCR_007345.  
  [https://doi.org/10.13026/kfzx-aw45](https://doi.org/10.13026/kfzx-aw45)

- **Data Access:**  
  Data was loaded directly from [the PTB-XL PhysioNet web page](https://physionet.org/content/ptb-xl/1.0.3/).

### Files Used

- `records100/` — Contains waveform ECG data sampled at 100 Hz.
- `scp_statements.csv` — Annotation scheme information.
- `ptbxl_database.csv` — Main metadata file, with one row per record identified by `ecg_id`.

#### Metadata Structure

- **Identifiers** — `ecg_id`, `patient_id`, file paths for 500 Hz (`filename_hr`) and 100 Hz (`filename_lr`) signals.
- **General Metadata** — Demographic and recording details like age, sex, device, etc.
- **ECG Statements** — Includes `scp_codes` (SCP-ECG statements), `report`, and additional clinical annotations.
- **Signal Metadata** — Quality measures (noise, drift, artifacts, extra beats, pacemaker).
- **Cross-validation Folds** — Recommended 10-fold train-test splits, preserving patient assignments.  
  - _Folds 1-8:_ Training set  
  - _Fold 9:_ Validation set  
  - _Fold 10:_ Test set (high label quality)

---

## Data Download Instructions

**Note:**  
Due to size constraints (original data files >3GB), ECG waveform data is **not included** in this repository.

To download the data for local use, run:

```sh
wget -r -N -c -np https://physionet.org/files/ptb-xl/1.0.3/
```

Alternatively, visit [PhysioNet PTB-XL data page](https://physionet.org/content/ptb-xl/1.0.3/) to download required files manually.

**Required files:**
- `records100/`  — ECG waveform data (100Hz)
- `scp_statements.csv` — Annotation schema
- `ptbxl_database.csv` — Metadata

Please **do not upload these files to GitHub** when working with this repository.

---

## ECG Labels and Interpretations: 

| Label    | Meaning                | Notes / Typical ECG Findings                                                                                                              |
| -------- | ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| **NORM** | Normal                 | No significant abnormalities. Standard sinus rhythm.                                                                                      |
| **MI**   | Myocardial Infarction  | Signs of past or acute heart attack; may show Q waves, ST-T changes.                                                                      |
| **HYP**  | Hypertrophy            | Usually left or right ventricular hypertrophy; high voltage QRS, strain patterns.                                                         |
| **CD**   | Conduction Disturbance | Includes bundle branch blocks, AV blocks, abnormal QRS propagation.                                                                       |
| **STTC** | ST-T Changes           | Repolarization abnormalities: ST elevation/depression, T wave inversion or flattening; can indicate ischemia or electrolyte disturbances. |

## Schematic of an Individual Heartbeat

 Highlighted regions indicate areas typically affected by each pathology in the dataset.
 
![ECG Example](./ECG_pathologies.png)

---

## Local Set Up to Run on VS Code

1. **Create Environment:**  
   Set up your Python environment for Jupyter Notebook.
2. **Install Required Dependencies:**  
   Use the provided `requirements.txt` file to install all the necessary Python libraries:

   ```sh
   pip install -r requirements.txt
   ```
   <br>
   This will install:

   - wfdb
   - iterative-stratification
   - scikit-multilearn
   - torch, torchvision, torchaudio
   - transformers
   - scipy, tqdm, mlflow, optuna
   - pandas, numpy, scikit-learn
   - matplotlib, seaborn, plotly
   - neurokit2

   _**Note:** You must use **Python 3.10.3 or newer**._
   
4. **VS Code Configuration:**  
   Configure your VS Code environment to work with Jupyter Notebooks.

---

## Getting Started

- Clone this repository:
  ```sh
  git clone https://github.com/paulavazq/ecg-anomaly-detection-autoencoders.git
  ```
- Set up environment and install requirements.
- Open `ECG_Anomaly_Detection_Autoencoders.ipynb` in VS Code or JupyterLab.
  
- **Important:**  
After downloading the PTB-XL dataset files to your local machine, you must ensure that your code (and notebook) points to the correct local file paths for:
- `records100/`
- `scp_statements.csv`
- `ptbxl_database.csv`

Modify the data loading sections to use the location **where you stored the downloaded files**. For example:
```python
base_path = "/your/local/path/ptb-xl-1.0.3/"
```
Replace with your actual download path if needed.

---

### License

This project is for educational purposes as part of Module 3 coursework on Advance Machine Learning.
Please cite appropriately if reused.

#### Note on Version Control

Large data files (such as `records100/`, `.csv`, `.h5`, and notebook checkpoints) are **excluded from version control** using the `.gitignore` file.  
This prevents accidental uploads of big files and unnecessary clutter in the repository.

If you add other large files to your project, update `.gitignore` accordingly!
#### Ignore macOS Finder file
.DS_Store
#### Ignore (large) data files not meant for repo
*.csv
*.h5
*.npz
records100/

---
## Main results and figures

Summary
- We trained a 1D convolutional denoising autoencoder on ECG segments extracted from normal ECG recordings.
- Both the training and validation sets contained only normal segments.
- Reconstruction error from the autoencoder was used as an anomaly score to detect pathological regions in a test set that contained both normal and pathological samples.

How detection works
- The autoencoder learns to reconstruct normal ECG segments. Segments that differ substantially from normal examples produce higher reconstruction error.
- We set a threshold equal to the 90-95th percentile of validation reconstruction errors (i.e., the value that 90-95% of validation errors do not exceed). Segments with reconstruction error above this threshold are considered "high error".
- An ECG is flagged as containing a potential abnormality if it contains at least one high-error segment.

Figures
- Example: normal ECG and its segmentation (see notebook for segmentation details)  
  ![Normal ECG segmentation example](./ECG_segmentation_plot.png)

- Example: segments extracted from a single normal ECG (all leads)  
  ![Segments extracted from one ECG (normal)](./ECG_normal_segments.png)

- Autoencoder training history (loss / reconstruction error on training and validation sets)  
  ![Training history](./training_Plot.png)

- Reconstruction examples: original vs reconstructed segments (normal and pathological examples)  
  ![Reconstruction examples](./ECG_reconstruction.png)

- Detection summary: percentage of ECGs in the test set that contain at least one high-error segment (in this graph= threshold = 90th percentile of validation errors per segment). This plot shows how reconstruction error can be used to flag abnormal ECGs.  
  ![Detection results using reconstruction error](./output_RE.png)

Notes and recommendations
- See the project notebook for full details on segmentation, model architecture, training hyperparameters, and how reconstruction error is computed.

--- 

##  Conclusions

 Normal ECGs are well reconstructed and mostly ranked in the low-error category. False positives in normals are ~20–30%, depending on the cut-off.
  - Reconstruction error alone does not clearly separate most pathologies.
  - CD and HYP are detected reasonably well (~70% of samples), while MI and SSTC are harder to detect because they are often reconstructed too well.
  - Pathological categories appear to be reconstructed well, which limits the ability of reconstruction errors to distinguish between them.

- **Why CD and HYP Are Detected Better: both pathologies show anomalities in the QRS complex**
  - **Conduction Disturbance (CD):**  
    Abnormal QRS propagation or duration is captured by the autoencoder.
  - **Hypertrophy (HYP):**  
    Hypertrophy leads to increased QRS voltage and slight morphological changes. Since the autoencoder is trained only on normal ECGs, these voltage deviations and waveform alterations stand out, producing higher reconstruction errors compared to normal ECGs.

- **Why MI and SSTC Are Harder to Detect: ST-T segments anomalities**
  - **Myocardial Infarction (MI):**  
    MI presents abrupt, localized changes like Q-wave formation, ST-segment elevation, or T-wave inversion. These patterns vary depending on infarct location and timing. Some subtle MI changes may be reconstructed too well, resulting in low errors and making detection difficult.
  - **SSTC (Supraventricular Tachycardia / subtle ST changes):**  
    SSTC or minor ST-segment abnormalities can be transient or subtle, sometimes only affecting rhythm or small waveform features. The autoencoder focuses on overall waveform structure, so these small, short-lived changes often produce insignificant reconstruction errors.

- **Model Limitations and Recommendations**
 
  - While it identifies 70% of CD and HYP samples as abnormal, further improvements are needed.  
  - This pathologies show abnormalioties in the QRS complex, usually the largest and most distinctive feature in ECGs, so the model focuses on reconstructing it well. Subtle changes in the ST-T segment can get “washed out” in the reconstruction.

  - **Segmentation Improvements:**  
   
    - Segment ECGs based on R-peak intervals (R to R) rather than fixed windows. 
    - Segment ECG into QRS and ST-T → train separate autoencoders.
    - Or Baseline correction:  "normalized" so the ST-T picks are not so different in scale to QRS complex.
    - Weight ST-T in the loss function. 
    - Adjust peak detection to account for heart rate variability. 
  
    - Fine-tuning reconstruction or model architecture to better separate subtle pathologies.
    - Use/ add new features instead of raw ECG
          - feed the network ST-T sensitive features:
          - ST deviation from baseline
          - T-wave amplitude or slope



