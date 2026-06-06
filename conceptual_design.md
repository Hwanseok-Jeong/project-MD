# Multi-Omic Single-Cell Dissection of Shared Neuro-Immune Circuitries in Meniere’s Disease and Vestibular Migraine

---

## 1. Context & Rationale

Meniere’s Disease (MD) and Vestibular Migraine (VM) are debilitating episodic disorders sharing overlapping clinical phenotypes, including episodic vertigo, tinnitus, and sensorineural hearing loss. Differential diagnosis remains challenging due to the absence of robust molecular biomarkers.

Recent genome-wide association studies (GWAS) suggest a shared genetic architecture (\(r_g \approx 0.23\)), supporting a shift from localized inner-ear pathology to systemic neuro-inflammatory disease models.

The primary cellular components implicated in this process are **Peripheral Blood Mononuclear Cells (PBMCs)**—specifically **T-cells** (adaptive immune regulation) and **Macrophages/Monocytes** (innate inflammatory activation). These cells contribute to systemic inflammatory signaling via cytokines such as _TNF-α_ and _CXCL8_, potentially affecting neurovascular and inner-ear homeostasis.

However, the upstream **gene regulatory networks (GRNs)** and transcriptional control mechanisms underlying this shared pathology remain incompletely characterized.

---

## 2. Project Intention & Goal

This project develops an automated multi-omic analysis framework using joint single-cell profiling (**scRNA-seq + scATAC-seq from matched cells**).

- **Primary Objective:** Identify shared transcription factor (TF) programs regulating pathological gene expression in T-cells and Macrophages across MD and VM.
- **Secondary Objective:** Evaluate generative multi-omic models and leverage genome-scale foundation models as in-silico references for regulatory state comparison.

---

## 3. Computational Environment & Architectural Constraints

- **Infrastructure:** Linux HPC environment (high storage capacity) and local constrained compute environment (~20GB quota).
- **Workflow Engine:** Nextflow with Conda-based reproducibility (`-profile conda`) for environment isolation without root privileges.
- **Programming Stack:**
  - Python: `scvi-tools`, `PyTorch`, `Scanpy`
  - R: `Seurat`, `Signac`, `Cicero`, `limma-voom`

---

## 4. End-to-End Workflow Blueprint (4-Stage Pipeline)

```
[Stage 1: Data Ingestion & QC]
├── GSE269114 (scATAC-seq)
├── GSE269117 (scRNA-seq)
└── Cell filtering, barcode QC, fragment QC (Nextflow)
            ▼
[Stage 2: Multi-Omic Representation Learning]
├── MultiVI (scvi-tools)
├── Baseline: PCA / MOFA
└── Evaluation: KNN/KNC/CPD
            ▼
[Stage 3: In-Silico Regulatory Baseline Comparison]
├── scBasset (DNA → chromatin accessibility)
├── Enformer (DNA → regulatory signal baseline)
└── Compare patient ATAC-seq signals vs genome-derived regulatory expectations
            ▼
[Stage 4: Gene Regulatory Network Inference]
├── Subset T-cells and Macrophages
├── Cicero co-accessibility network inference
├── TF motif enrichment using JASPAR
└── Integration with scRNA-seq expression filtering
```

---

## Stage 1: Data Ingestion & QC

- Automated Nextflow pipeline for reproducible preprocessing.
- Input datasets: paired scRNA-seq and scATAC-seq (GSE269114 / GSE269117).
- Quality control includes:
  - mitochondrial RNA filtering
  - ATAC fragment distribution filtering
  - low-quality barcode removal

---

## Stage 2: Multi-Omic Representation Learning

- MultiVI is used to generate a joint latent embedding of RNA and ATAC modalities.
- Compared against:
  - PCA-based embeddings
  - MOFA latent factors
- Evaluation metrics:
  - Local neighborhood preservation (KNN consistency)
  - Global structure retention
- Cell type annotation performed using marker-based methods (e.g., ScType, CellAssign).

---

## Stage 3: In-Silico Regulatory Baseline Comparison

- scBasset models chromatin accessibility from DNA sequence in a single-cell context.
- Enformer predicts regulatory signals (including accessibility proxies) from reference genome sequences.

**Analysis strategy:**
- Compare patient-derived ATAC-seq signals against:
  - scBasset-predicted accessibility (cell-aware model)
  - Enformer-predicted regulatory baseline (genome prior model)

This provides a **multi-scale in-silico reference framework** for identifying disease-associated regulatory deviations.

---

## Stage 4: Gene Regulatory Network Inference

- T-cells and Macrophages are isolated from latent embedding space.
- Cicero is used to infer enhancer–promoter co-accessibility networks.
- TF motif enrichment performed using JASPAR database.
- Expression-aware filtering applied using matched scRNA-seq data to reduce false positives.

---

## 5. Optional Extensions

- RNA velocity / trajectory inference: scVelo, CellRank
- Cell-cell communication: LIANA, NicheNet
- RNA foundation models: scGPT, Geneformer for transcriptional state embedding and perturbation analysis