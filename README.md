# flow2img — PCAP-to-Image IoT Traffic Classification

This project converts raw network traffic captures (**PCAP**) into grayscale images and trains a CNN (**ResNet18**) for **multi-class IoT traffic classification**. 

## Overview
**Goal:** classify IoT network traffic into 5 classes (benign + 4 attacks) by:
1) reading PCAP packets (IP traffic),
2) splitting traffic into fixed time windows,
3) encoding each window as a 2D image (time × packet size),
4) training and evaluating a ResNet18 classifier.

## Dataset
The dataset used is **CICIoT2023 (CIC IoT Dataset 2023)** from the Canadian Institute for Cybersecurity (UNB):  
https://www.unb.ca/cic/datasets/iotdataset-2023.html

### What I downloaded and why
Because the full dataset is large, I used a **subset** for a controlled and reproducible study:
- **5 PCAP files**: benign + representative attack captures
- **3 merged CSV files**: used for lightweight checks and as a possible future baseline

### Local data placement (not uploaded to GitHub)
Place the downloaded files here:

```
E:\Project\flow2img\data\raw\
  ├─ pcap\   (your .pcap files)
  └─ csv\    (your merged .csv files)
```

> Note: `data/` is typically excluded from GitHub via `.gitignore` because it can be large.

---

## Repository Structure
Only the main research artifacts are tracked in GitHub:

```
src/                 # notebooks / code
documentation/       # report + presentation
results/             # figures, tables, outputs used in report
```

---

## Environment Setup (Anaconda)
### 1) Create a conda environment
Open **Anaconda Prompt** (or Git Bash if conda is available) and run:

```bash
conda create -n flow2img python=3.10 -y
conda activate flow2img
```

### 2) Install dependencies
Recommended installs (mix of conda + pip):

```bash
conda install -y numpy matplotlib scikit-learn pillow jupyter
pip install scapy
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu121
```

- If you do **not** have CUDA, install CPU wheels instead:
  ```bash
  pip install torch torchvision
  ```

---

## How to Run
### 1) Generate images from PCAP
Use the notebook in:
- `src/notebooks/FlowToImg.ipynb`

It will:
- read PCAPs,
- split into windows,
- generate grayscale images per class,
- save them under `data/images/...` (local).

### 2) Train the model (ResNet18)
Use the training notebook/code in:
- `src/notebooks/FlowToImg.ipynb` (training section)

Training details:
- ResNet18 transfer learning
- images resized to **224×224**
- train/val/test split: **70% / 15% / 15%**
- metrics: loss, accuracy, precision, recall, F1-score, ROC–AUC + confusion matrix

### 3) Evaluate and save results
Outputs (plots + tables) are stored in:
- `results/`

---

## Experiments
### Table 1. Experimental Configurations
| Run | MAX_IMAGES_PER_PCAP | TIME_WINDOW_SEC | IMG_SIZE | MIN_PKTS_PER_WINDOW | EPOCHS | LR   | BATCH |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 1  | 100 | 2.0 | 32 | 10 | 12 | 1e-4 | 32 |
| 2  | 100 | 2.0 | 32 | 10 | 20 | 1e-4 | 32 |
| 3  | 100 | 2.0 | 32 | 10 | 40 | 1e-4 | 32 |
| 4  | 100 | 2.0 | 32 | 10 | 40 | 3e-4 | 32 |
| 5  | 100 | 2.0 | 32 | 10 | 40 | 3e-5 | 32 |
| 6  | 100 | 2.0 | 32 | 10 | 40 | 1e-4 | 16 |
| 7  | 100 | 2.0 | 32 | 10 | 40 | 1e-4 | 64 |
| 8  | 100 | 4.0 | 32 | 10 | 40 | 3e-4 | 32 |
| 9  | 100 | 2.0 | 32 | 15 | 40 | 3e-4 | 32 |
| 10 | 100 | 2.0 | 64 | 10 | 40 | 3e-4 | 32 |
| 11 | 200 | 2.0 | 32 | 10 | 40 | 3e-4 | 32 |

### Table 2. Comparative Results of Experimental Runs
| Run | accuracy | loss  | precision | recall | f1-score | ROC-AUC |
|---:|---:|---:|---:|---:|---:|---:|
| 1  | 0.9333 | 0.1378 | 0.9339 | 0.9333 | 0.9333 | 0.9942 |
| 2  | 0.9333 | 0.1378 | 0.9339 | 0.9333 | 0.9333 | 0.9942 |
| 3  | 0.9333 | 0.1761 | 0.9361 | 0.9333 | 0.9336 | 0.9931 |
| 4  | 0.9733 | 0.2090 | 0.9765 | 0.9733 | 0.9732 | 0.9898 |
| 5  | 0.9200 | 0.2156 | 0.9375 | 0.9200 | 0.9181 | 0.9922 |
| 6  | 0.9467 | 0.1405 | 0.9467 | 0.9467 | 0.9467 | 0.9940 |
| 7  | 0.9333 | 0.1633 | 0.9339 | 0.9333 | 0.9333 | 0.9947 |
| 8  | 0.9333 | 0.3356 | 0.9371 | 0.9333 | 0.9341 | 0.9904 |
| 9  | 0.9467 | 0.2075 | 0.9467 | 0.9467 | 0.9467 | 0.9918 |
| 10 | 0.9867 | 0.0403 | 0.9875 | 0.9867 | 0.9867 | 0.9991 |
| 11 | 0.9333 | 0.0190 | 0.9935 | 0.9933 | 0.9933 | 1.0000 |

**Best overall (multi-metric):** the highest F1-score (**0.9933**) and ROC–AUC (**1.0000**) indicate very strong class separability.

---

## Documentation and Results
- Report and presentation are stored in `documentation/`
- Figures and tables used in the report are stored in `results/`

---

## References
[1] CIC (UNB), “CIC IoT Dataset 2023,” dataset page: https://www.unb.ca/cic/datasets/iotdataset-2023.html  
[2] K. He, X. Zhang, S. Ren, and J. Sun, “Deep Residual Learning for Image Recognition,” *CVPR*, 2016.  
[3] D. P. Kingma and J. Ba, “Adam: A Method for Stochastic Optimization,” *ICLR*, 2015 (arXiv:1412.6980).  
[4] P. García-Teodoro, J. Díaz-Verdejo, G. Maciá-Fernández, and E. Vázquez, “Anomaly-based Network Intrusion Detection: Techniques, Systems and Challenges,” *Computers & Security*, 2009.  
[5] A. L. Buczak and E. Guven, “A Survey of Data Mining and Machine Learning Methods for Cyber Security Intrusion Detection,” *IEEE Communications Surveys & Tutorials*, 2016.

---

## Author
**Incze Melinda Henrietta**
