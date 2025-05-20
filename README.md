# Multiomics project

Multiomics analysis

** In this project Dipali Kale and Niharika Garg are working on integrating bulk transcriptomics, proteomics and phosphoproteomics data processing **

Transcriptomics, proteomics and phosphoproteomics data were obtained from the article [1], which examined LUAD tumors and their adjacent non-cancerous tissues (NATs).
Multi-omic factor analysis with MOFA2 was demonstrated in MOFA_analysis.ipynb

## 1. **Data acquisition**

* **Sources**: Supplementary Tables S3–S5 from [1] Xu JY et al
* **Files**:

  * `mrna.csv` (RNA-Seq FPKM for 100 samples)
  * `proteo.csv` (iBAQ protein abundances for 206 runs)
  * `phospho.csv` (MaxQuant phosphosite intensities for 158 runs)
  * `metadata.csv` (sample labels: Tumor vs NAT)

## 2. **Quality control (QC)**

*For each modality (mRNA, proteome, phospho), working on samples × features matrices:*

1. **Sample filtering**: remove any sample with > 30 % missing values
2. **Feature filtering**: remove any gene/protein/site with > 30 % missing values
3. **Low-variance filter**: drop the bottom 10 % of features by variance
4. **Resulting QC shapes**:

   * mRNA: 100 × 16 188 → 100 × 13 319
   * Proteome: 206 × 11 119 → 182 × 6 154
   * Phospho: 158 × 22 564 → 158 × 5 014

## 3. **Complete-case intersection**

* Identified the **72 samples** profiled in all three QC’d tables
* Subset each QC matrix to those 72 samples
* Final QC shapes:

  * mRNA: 72 × 13 319
  * Proteome: 72 × 6 154
  * Phospho: 72 × 5 014

## 4. **Normalization & imputation**

* **mRNA**: log₂(FPKM + 0.1), quantile-normalize across samples
* **Proteome & Phospho**:

  1. log₂(intensity + 1)
  2. median-center each sample
  3. replace zeros with NaN and k-NN impute (k=5)
* Final processed shapes (72 samples × same feature counts)

## 5. **Exploratory PCA & clustering**

* Concatenated the three processed matrices (or ran PCA on each separately)
* Plotted the first two PCs and hierarchical clustering
* **Color‐coded** by Tumor vs NAT to confirm separation

## 6. **Extract & interpret top loadings (PC1)**

* Took the **100 features** with highest absolute loading on PC1
* Ran ORA (Enrichr) on those top features → strong “extracellular matrix organization” signal
* Heatmapped those features across the 72 samples → clear ECM-high vs ECM-low tumor subgroup

## 7. **Multi-omic factor analysis with MOFA2**

* Loaded the three processed matrices into MOFA2’s `entry_point`
* Set 10 latent factors, trained until ELBO convergence
* Extracted:

  * **Z** (72 samples × 10 factors)
  * **W** for each view (features × 10 factors)
* Confirmed Factor 1 perfectly separates Tumor vs NAT (p ≈ 10⁻³⁹, AUC = 1.0)

## 8. **Factor interpretation**

* **Factor 1** captures the joint proliferation + ECM/remodeling program
* **Factor 2** top drivers (RNA, Prot, Phos) enrich for endocytosis, focal adhesion, phosphoinositide metabolism → a membrane/cytoskeleton remodeling axis
* All other factors (3–10) show no Tumor/NAT signal, but can be mined for immune, metabolic or other biology

## 9. **Downstream modeling & visualization**

* **Tumor vs NAT classifier**: logistic regression on Z → AUC = 1.0 with Factors 1 ± 2
* **Unsupervised subtyping**: K-means (k=2–3) in Factor 1–3 space to define molecular subgroups
* **Heatmaps**: combined top drivers from Factor 1 & 2 across all three omics

---

**Next steps to add** (on an expanded metadata):

* Incorporate **Stage**, **Grade**, **Smoking status**, **Survival** into `metadata.csv`
* Correlate each MOFA factor with those clinical variables (boxplots, Cox models)
* Deploy this pipeline as a shareable notebook or web app


Reference: [1] Xu JY, Zhang C, Wang X, et al. Integrative proteomic characterization of human lung adenocarcinoma. Cell 2020;182(1):245-261.



## Setup


 Install the virtual environment and the required packages by following commands:

    ```BASH
    pyenv local 3.11.3
    python -m venv .venv
    source .venv/bin/activate
    pip install --upgrade pip
    pip install -r requirements.txt
    ```

 
