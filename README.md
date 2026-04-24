# Pre-processing of 10x Single-Cell RNA-seq Data using Galaxy

## 1. Introduction

Single-cell RNA sequencing (scRNA-seq) enables transcriptomic profiling at the resolution of individual cells, allowing researchers to study cellular heterogeneity and identify distinct biological populations. However, raw sequencing data contains multiple technical artifacts, including PCR duplicates, sequencing errors, and non-cell barcodes.

This project focuses on preprocessing 10x Genomics scRNA-seq data using the Galaxy platform. The aim is to convert raw FASTQ files into a high-quality gene expression matrix suitable for downstream analysis.

---

## 2. Dataset and Experimental Context

The dataset used in this analysis is derived from Peripheral Blood Mononuclear Cells (PBMCs), a standard reference dataset in single-cell studies.

| Attribute | Description |
|----------|------------|
| Dataset | PBMC |
| Platform | 10x Genomic Chromium |
| Chemistry | V3 |
| Input Files | FastQ (R1, R2) |
| Reference Genome | hg19 |
| Data Size | ~300 cells |

The sequencing data is split into two reads: R1 contains the cell barcode and UMI, while R2 contains the cDNA sequence used for gene expression analysis.

---

## 3. Conceptual Framework

The preprocessing workflow relies on three key components.

- The cell barcode uniquely identifies each cell.  
- The UMI ensures accurate transcript counting by removing duplicates.  
- The final expression matrix represents gene expression across all cells.  

These components are fundamental to ensuring that the resulting dataset reflects true biological variation rather than technical noise.

---

## 4. Computational Environment

All analysis steps were performed on the Galaxy platform using the STARsolo pipeline. This tool integrates multiple preprocessing steps into a single workflow, improving both efficiency and reproducibility.

---

## 5. Methodology

### Data Upload and Preparation

The FASTQ files (R1 and R2) and barcode whitelist were uploaded into Galaxy. Care was taken to ensure correct pairing of reads and selection of the appropriate whitelist corresponding to 10x v3 chemistry.

### Quality Assessment

The quality of raw sequencing reads was assessed using FastQC. This step provided insight into sequencing quality, GC content distribution, and duplication levels. The results indicated acceptable sequencing quality for downstream processing.

### Alignment and Quantification

The core step of the workflow involved running STARsolo. The tool extracted cell barcodes and UMIs from R1 reads and validated them against the whitelist. Reads from R2 were then aligned to the reference genome (hg19), and gene-level counts were generated.

| Parameter | Value | Purpose |
|----------|------|--------|
| CB Length | 16bp | Identify cells |
| UMI Length | 10bp | Remove duplicates |
| Whitelist | Enabled | Filter valid cells |
| Genome | hg19 | Alignment reference |

UMI deduplication was applied to collapse duplicate reads, ensuring accurate quantification.

### Expression Matrix Generation

The output consisted of three files:

- matrix.mtx  
- barcodes.tsv  
- features.tsv  

These files together form a sparse gene expression matrix representing gene counts per cell.

---

## 6. Quality Control and Filtering

Quality control was performed to remove low-quality cells and technical artifacts. Several metrics were evaluated, including gene count per cell, total UMI counts, and mitochondrial gene expression.

| Metric | Interpretation |
|-------|---------------|
| Low gene count | Empty droplets |
| High gene count | Doublets |
| High mitochondrial % | Damaged cells |

Filtering thresholds were applied to remove cells that did not meet quality criteria. This step significantly improved the reliability of the dataset.

---

## 7. Results and Observations

The preprocessing workflow successfully generated a clean gene expression matrix. Alignment statistics indicated a high proportion of mapped reads, reflecting good sequencing quality.

After filtering, the dataset exhibited reduced noise and improved consistency in gene expression across cells. The processed data is suitable for downstream analyses such as clustering and visualization.

### Visualizations

- Genes
![image alt](https://github.com/Rameensajjad/Preprocessing-of-scRNA/blob/56225248512ed44c9b875ef3ed70499f3971a59c/Genes.png) - Barcodes
![image alt](https://github.com/Rameensajjad/Preprocessing-of-scRNA/blob/e882686f00badf0b6db3a720fb6f2543c17eb23c/Barcode.png)  
- Matrix Count
![image alt](https://github.com/Rameensajjad/Preprocessing-of-scRNA/blob/e882686f00badf0b6db3a720fb6f2543c17eb23c/Matrix%20count.png)  
- Star Alignment plot
![image alt](https://github.com/Rameensajjad/Preprocessing-of-scRNA/blob/e882686f00badf0b6db3a720fb6f2543c17eb23c/star_alignment_plot.png)  
- Multiqc summary table
![image alt](https://github.com/Rameensajjad/Preprocessing-of-scRNA/blob/e882686f00badf0b6db3a720fb6f2543c17eb23c/table_scatter_plot.png)  
- Dropletutils plot
![image alt](https://github.com/Rameensajjad/Preprocessing-of-scRNA/blob/e882686f00badf0b6db3a720fb6f2543c17eb23c/dropletutils%20plot.png)  

---

## 8. Limitations

The use of a subsampled dataset limits biological diversity. Additionally, the workflow does not include batch correction or ambient RNA removal, which may be necessary for larger datasets.

---
