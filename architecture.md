# Multi-Omic Single-Cell Dissection of MD and VM (Modern Python/Muon Framework)

## 1. Context & Rationale
Meniere’s Disease (MD) and Vestibular Migraine (VM) share overlapping clinical phenotypes (vertigo, tinnitus) and genetic architectures ($r_g \approx 0.23$). Rather than localized inner-ear defects, they are increasingly viewed as systemic neuro-inflammatory conditions.
The primary cellular battlefields are **PBMCs** (Peripheral Blood Mononuclear Cells)—specifically **T-cells** and **Macrophages/Monocytes**. These cells release inflammatory cytokines (e.g., *TNF-α*, *CXCL8*), yet the exact Gene Regulatory Networks (GRNs) driving this pathology remain unmapped.

## 2. Project Intention & Goal
This project modernizes the analysis of paired single-cell multi-omic data (scRNA-seq + scATAC-seq) using a state-of-the-art Python-centric framework (`Muon`, `scvi-tools`, `SCENIC+`).

*   **Primary Objective:** Identify shared epigenetic master switches (TF Motifs) triggering pathological gene expression in MD and VM T-cells and Macrophages.
*   **Secondary Objective (Benchmark):** Evaluate latent space representations (MultiVI vs. MOFA) to prove that deep generative models better encapsulate multi-omic biological manifolds.
*   **Tertiary Objective (In-silico Validation):** Use foundation sequence models (`scBasset`/`scGPT`/`Enformer`) as an *in-silico* validation layer for the physical healthy control data, providing orthogonal computational evidence supporting regulatory mechanisms behind the discovered ATAC peaks.

## 3. Methodological Paradigm Shift: Why Python/Muon over R/Seurat?
Previous pipelines heavily relied on converting objects back and forth between R (Seurat/ArchR) and Python (MOFA). This project adopts a **Unified Python Architecture**:
*   **Data Structure:** `MuData` (`.h5mu`), native to HDF5, enabling zero-memory-copy multimodal integration.
*   **Tools:** `scvi-tools` (Doublet detection & MultiVI), `Scanpy` (Clustering), and `SCENIC+` (GRN).
*   **Orchestration:** **Nextflow** handles computationally heavy processing (Doublets, VAE training) while **Jupyter Notebooks** are used for exploratory data analysis (EDA) and biological storytelling.

---

## 4. End-to-End Workflow Blueprint (The 6-Stage Modern Pipeline)

### Stage 1: Pipeline & Data Ingestion (Nextflow)
*   **Action:** Ingest GSE269114 (ATAC) and GSE269117 (RNA) count matrices into a single `MuData` object.
*   **QC & Doublet Detection:**
    *   RNA: Filter based on mitochondrial fraction -> run **SOLO** (VAEs via scvi-tools) for accurate doublet removal.
    *   ATAC: Filter by TSS enrichment -> run **AMULET** (read overlap logic) for specific ATAC doublet detection.
    *   *Why SOLO & AMULET?* To keep the pipeline native to Python (`SOLO`) while applying the strict physical law of diploid biology (`AMULET`).

### Stage 2: Deep Learning Representation & Benchmarking (Nextflow)
*   **Action:** Train joint latent spaces to map cells into a reduced biological manifold.
*   **Benchmark:**
    *   **Linear Baseline:** MOFA+ (Multi-Omics Factor Analysis).
    *   **Generative Baseline:** **MultiVI** (scvi-tools).
*   **Evaluation:** Extract latent coordinates (`X_mofa`, `X_multivi`) and perform comparative KNN stability and Global structure clustering, evaluating the representation quality of MultiVI relative to MOFA+.

### Stage 3: Clustering & Subsetting (Jupyter)
*   **Action:** Generate UMAPs, define cell populations via Leiden clustering, and annotate.
*   **Focus:** Subset the `MuData` object to retain only **T-cells and Macrophages**, the core hypothesize effectors in MD/VM neuro-inflammation.

### Stage 4: Multiomic DEG / DAR (Jupyter)
*   **Action:** Compress single-cell data into **Pseudobulk** profiles per sample.
*   **Contrast:** MD vs. HC, VM vs. HC, MD vs. VM.
*   **Outcome:** Extract lists of Differentially Expressed Genes (DEGs) and Differentially Accessible Regions (DARs/Peaks).

### Stage 5: Gene Regulatory Network (GRN) Extraction (Nextflow -> Jupyter)
*   **Action:** Deploy **SCENIC+**.
*   **Mechanism:** Links open chromatin regions (Enhancers/Peaks) $\rightarrow$ specific Transcription Factor (TF) binding motifs $\rightarrow$ downstream Target Gene expression.
*   *Note on Regulation:* SCENIC+ explicitly maps both **Positive** (Activator) and **Negative** (Repressor) regulons. Negative regulation is pathologically vital (e.g., loss of a repressor can cause runaway inflammation).

### Stage 6: In-Silico Regulatory Perturbation Analysis (Jupyter / GPU)

*   **Action:** Take the key transcription factor (TF)-associated regulatory peaks identified in Stage 5 and evaluate them using sequence-based foundation models such as **scBasset**, **scGPT**, or **Enformer**.

*   **Mechanism (In-Silico Mutagenesis):** Perform systematic sequence perturbation analysis by modifying TF binding motifs within candidate regulatory regions and quantifying the resulting changes in predicted chromatin accessibility or regulatory activity.

*   This enables assessment of the functional sensitivity of regulatory elements to sequence-level disruptions, providing orthogonal computational evidence that supports and refines gene regulatory relationships inferred from SCENIC+.

*   Rather than modeling full cellular or system-level dynamics, this step operates as a sequence-level perturbation framework, allowing targeted evaluation of motif importance within regulatory DNA.

---

## 5. Optional Extrapolations (Velocity & Communication)
*   **RNA Velocity (`scVelo` / `CellRank`):** Useful if investigating the specific trajectory of naïve T-cells transitioning into active/exhausted states across the disease timeline.
*   **Cell-Cell Communication (`LIANA` / `NicheNet`):** Highly recommended to prove the interaction between the two subset populations, demonstrating exactly which cytokine from the Macrophages binds to which receptor on the T-cells.