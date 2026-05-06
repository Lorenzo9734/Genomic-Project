# A Bioinformatics Workflow for Identifying Rare Diseases in Children from Trio-Family Data

## Authors
Giacomo Pogliana, Lorenzo Ponzone

## Overview
This repository provides a comprehensive framework for analyzing genetic data from family trios (child, father, and mother). It is designed to filter and prioritize variants—including de novo mutations and recessive traits—to facilitate the diagnosis of rare genetic disorders.

## Prerequisites & Dependencies
Ensure the following bioinformatics tools are installed and accessible in your system's $PATH:
* **Bowtie2** (Alignment)
* **SAMtools** (BAM manipulation)
* **FastQC** (Quality control)
* **Qualimap** (BAM quality control)
* **BEDTools** (Genome coverage)
* **MultiQC** (Aggregated QC reporting)
* **FreeBayes** (Variant calling)
* **BCFtools & bgzip** (VCF manipulation and filtering)
  
## Directory Structure & Input Requirements

The script must be executed from a main directory containing the necessary reference files. Each trio must be organized into its own subdirectory.

### 1. Main Directory (Reference Files)
The following files must be present in the root folder:

* `chr20` (Bowtie2 index files)
* `chr20.fa` (Reference genome FASTA)
* `chr20_ILMN_Exome_2.0_Plus_Panel.hg38_padded.bed` (Target panel)
* `samples.txt` (Sample list for BCFtools)

### 2. Trio Subdirectories
Trios should be in folders prefixed with `trio_` (e.g., `trio_01/`). Inside, the pipeline expects paired-end FASTQ files: `*.targets_R1.fq.gz` and `*.targets_R2.fq.gz`.

> [!WARNING]
> **Important: Alphabetical Ordering**
> The script assigns roles based on the **alphabetical order** of the FASTQ files. To ensure correct role assignment, name your files so they sort as follows:
> 1. **Child**
> 2. **Father**
> 3. **Mother**

## Usage

The pipeline categorizes trios based on the clinical inheritance model provided via command-line arguments.
**Syntax:**
```bash
./Pipeline.sh [INHERITANCE_MODEL] [trio_name1] [trio_name2] ...
```
## Inheritance Flags

**Available Inheritance Flags:**
* `-AR` : Autosomal Recessive
* `-AD` : Autosomal Dominant (De Novo)
* `-ADF`: Autosomal Dominant (Inherited, Father affected)
* `-ADM`: Autosomal Dominant (Inherited, Mother affected)
  
**Example Command:**
```bash
./pipeline.sh -AR trio_01 trio_02 -AD trio_03 -ADM trio_04
```
## Pipeline Workflow

For every specified trio, the pipeline executes the following steps:

1. **Setup & Read Groups**: Identifies FASTQ pairs and assigns Read Group tags (`SM:child`, `SM:father`, `SM:mother`).
2. **Alignment**: Aligns reads to the **chr20** reference using **Bowtie2** and generates sorted BAM files via **SAMtools**.
3. **Quality Control (QC)**:
    * Runs **FastQC** for raw read quality.
    * Runs **Qualimap bamqc** using the target BED file for alignment metrics.
4. **Coverage Analysis**: Generates a bedgraph (`.bg`) track with a depth cap of 100x using **BEDTools**.
5. **MultiQC**: Aggregates all individual QC metrics into a single, interactive HTML report.
6. **Variant Calling**: Performs joint variant calling across the trio using **FreeBayes**.
7. **Filtering**: Applies clinical filters based on the chosen inheritance model, focusing on variants that:
    * Intersect the BED panel.
    * Match the expected Genotype (**GT**).
    * Have a quality score **QUAL > 20**.
      
## Outputs

Inside each `trio_*` directory, you will find:

* **BAM Files**: Sorted alignments (`child.bam`, `father.bam`, `mother.bam`).
* **QC Reports**: Individual reports and the master `[trio_name]_multiqc_report.html`.
* **Coverage**: `.bg` files for visualization in genome browsers (like IGV).
* **VCF Files**:
    * `[trio_name].vcf.gz`: The raw joint-called variants.
    * `[trio_name]_cand_[MODEL].vcf`: The final filtered candidate variants.
