#  Pre-processing of 10x Single-Cell RNA-seq Data using Galaxy

> A preprocessing pipeline converting raw FASTQ files into a filtered gene expression matrix using **STARsolo** on **Galaxy** — a faster, drop-in replacement for 10x Cell Ranger.

 **Tutorial:** [Galaxy Training Network](https://training.galaxyproject.org/training-material/topics/single-cell/tutorials/scrna-preprocessing-tenx/tutorial.html)

---

##  Table of Contents
- [Introduction](#introduction)
- [Dataset](#dataset)
- [Workflow](#workflow)
- [Methodology](#methodology)
- [Quality Control](#quality-control)
- [Results](#results)
- [Limitations](#limitations)
- [References](#references)

---

## Introduction

Single-cell RNA sequencing (scRNA-seq) measures gene expression per individual cell, revealing cellular heterogeneity and rare populations that bulk sequencing misses. Raw 10x data contains technical noise — PCR duplicates, empty droplets, and damaged cells — that must be removed before analysis.

This project uses **STARsolo** on **Galaxy** to preprocess 10x Chromium V3 PBMC data, producing a clean gene expression matrix ready for clustering and annotation. STARsolo is preferred over Cell Ranger for its ~10× speed advantage and simpler input requirements.

---

## Dataset

| Attribute | Details |
|-----------|---------|
| Sample | PBMCs — 1k Healthy Donor (10x Genomics, V3 chemistry) |
| Input | 2 lanes × R1 (barcode + UMI) + R2 (cDNA) |
| Reference | hg19 / GRCh37 + `Homo_sapiens.GRCh37.75.gtf` |
| Whitelist | `3M-february-2018.txt.gz` (~3.7M valid barcodes) |
| Cell Count | ~300 cells (subsampled from 1,000 for training) |
| Platform | Galaxy (usegalaxy.org) |

---


##  Workflow 

```
┌──────────────────────────────────────────────┐
│             Raw Input Files                  │
│    R1.fastq.gz  +  R2.fastq.gz               │
│    + 10x v3 Barcode Whitelist                │
└─────────────────────┬────────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────────┐
│       STEP 1: Quality Assessment             │
│                FastQC / MultiQC              │
│  • Per-base quality scores                   │
│  • GC content distribution                  │
│  • Adapter content and duplication levels   │
└─────────────────────┬────────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────────┐
│    STEP 2: Barcode and UMI Extraction        │
│                STARsolo (R1)                 │
│  • Extract 16 bp cell barcode               │
│  • Extract 10 bp UMI                        │
│  • Validate barcodes against whitelist      │
└─────────────────────┬────────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────────┐
│       STEP 3: Read Alignment                 │
│                STARsolo (R2)                 │
│  • Align cDNA reads to hg19                 │
│  • Gene-level read assignment               │
│  • UMI deduplication                        │
└─────────────────────┬────────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────────┐
│     STEP 4: Expression Matrix Output         │
│   matrix.mtx + barcodes.tsv + features.tsv  │
└─────────────────────┬────────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────────┐
│      STEP 5: Quality Control Filtering       │
│                DropletUtils                  │
│  • Remove empty droplets (knee plot)        │
│  • Filter doublets                          │
│  • Remove damaged cells (high MT%)          │
└─────────────────────┬────────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────────┐
│    ✅ Clean Gene Expression Matrix           │
│  Ready for clustering, UMAP, and            │
│  cell-type annotation                       │
└──────────────────────────────────────────────┘
```

## Methodology

### 1. Data Upload
FASTQ files (L001 + L002, R1 and R2), the barcode whitelist, and the GTF annotation were uploaded to Galaxy from Zenodo. The I1 lane file is **not needed** by STARsolo, unlike Cell Ranger.

### 2. RNA STARsolo

| Parameter | Value |
|-----------|-------|
| Barcode length (R1) | 16 bp |
| UMI length (R1) | 10 bp |
| Reference genome | hg19 (no built-in gene model) |
| Gene annotation | `Homo_sapiens.GRCh37.75.gtf` |
| Junction overhang | 100 bp |
| scRNA-seq type | 10X Chromium / Drop-seq |

STARsolo extracts barcodes and UMIs from R1, validates them against the whitelist (1-mismatch tolerance), aligns R2 to hg19, assigns reads to genes, and collapses identical barcode + gene + UMI combinations into single counts — removing PCR duplicates.

**Output:** 6 files — a log, a mapping quality file, a BAM file, and the 3 matrix files below.

### 3. Expression Matrix (MEX Format)

| File | Content |
|------|---------|
| `matrix.mtx` | Sparse UMI counts (genes × cells) |
| `barcodes.tsv` | Valid cell barcodes |
| `features.tsv` | Gene IDs and names |

> STARsolo detected **5,200 barcodes** total — only **272** passed quality thresholds. The raw matrix is unfiltered; DropletUtils handles final cell calling.

Compatible with Seurat (`Read10X()`) and Scanpy (`sc.read_10x_mtx()`).

### 4. MultiQC
MultiQC aggregates the STARsolo mapping log into a single QC report. For scRNA-seq, **>70% uniquely mapped reads** is acceptable. High `nNoFeature` values (reads not assigned to a gene) are expected and normal for 10x datasets.

---

## Quality Control

Applied using **DropletUtils** based on three metrics:

| Metric | Low = | High = | Action |
|--------|-------|--------|--------|
| Genes per cell | Empty droplet | Doublet | Filter extremes |
| UMI per cell | Low-quality cell | Doublet | Filter extremes |
| % Mitochondrial reads | Healthy cell | Damaged cell | Upper cutoff 10–25% |

The **knee plot** ranks barcodes by UMI count (log scale). Barcodes above the inflection point are real cells; those below are empty droplets. This threshold defines which cells are retained.

---

## Results

The pipeline produced a filtered matrix with high alignment rates and a clear knee separating real cells from empty droplets.

**Genes Distribution** — genes detected per cell

![Genes](https://github.com/Rameensajjad/Preprocessing-of-scRNA/blob/56225248512ed44c9b875ef3ed70499f3971a59c/Genes.png)

**Barcodes** — valid barcodes after whitelist filtering

![Barcodes](https://github.com/Rameensajjad/Preprocessing-of-scRNA/blob/e882686f00badf0b6db3a720fb6f2543c17eb23c/Barcode.png)

**Matrix Count** — UMI distribution across cells and genes

![Matrix Count](https://github.com/Rameensajjad/Preprocessing-of-scRNA/blob/e882686f00badf0b6db3a720fb6f2543c17eb23c/Matrix%20count.png)

**STAR Alignment Plot** — mapped vs. unmapped read proportions

![STAR Alignment](https://github.com/Rameensajjad/Preprocessing-of-scRNA/blob/e882686f00badf0b6db3a720fb6f2543c17eb23c/star_alignment_plot.png)

**MultiQC Summary** — aggregated alignment and QC metrics

![MultiQC](https://github.com/Rameensajjad/Preprocessing-of-scRNA/blob/e882686f00badf0b6db3a720fb6f2543c17eb23c/table_scatter_plot.png)

**DropletUtils Knee Plot** — real cells vs. empty droplets

![DropletUtils](https://github.com/Rameensajjad/Preprocessing-of-scRNA/blob/e882686f00badf0b6db3a720fb6f2543c17eb23c/dropletutils%20plot.png)

---

## Limitations

- **~300 cells** — subsampled; full dataset has 1,000 cells
- **chrX only** on usegalaxy.org (training mode) to reduce memory load
- No ambient RNA removal (SoupX / CellBender not applied)
- No doublet detection (Scrublet / DoubletFinder not used)
- hg19 used — hg38 offers better annotation coverage

---

## References

1. Bacon W. et al. *Pre-processing of 10X Single-Cell RNA Datasets.* GTN v4. [Link](https://training.galaxyproject.org/training-material/topics/single-cell/tutorials/scrna-preprocessing-tenx/tutorial.html)
2. Dobin A. et al. (2013). STAR aligner. *Bioinformatics*, 29(1), 15–21.
3. Lun A. et al. (2019). EmptyDrops. *Genome Biology*, 20(1), 63.
4. Andrews S. (2010). FastQC. [Link](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)
5. Ewels P. et al. (2016). MultiQC. *Bioinformatics*, 32(19), 3047–3048.

---

