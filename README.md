# 🧬 Vertebrate Genome Assembly using HiFi, Bionano and Hi-C Data

> A step-by-step Galaxy tutorial for assembling high-quality, near-error-free, chromosome-level vertebrate genome assemblies — part of the [Vertebrate Genomes Project (VGP)](https://vertebrategenomesproject.org/).

[![Galaxy Training](https://img.shields.io/badge/Galaxy-Training%20Tutorial-blue)](https://training.galaxyproject.org/training-material/topics/assembly/tutorials/vgp_genome_assembly/tutorial.html)
[![License: CC-BY 4.0](https://img.shields.io/badge/License-CC--BY%204.0-lightgrey.svg)](http://creativecommons.org/licenses/by/4.0/)
[![PURL](https://img.shields.io/badge/PURL-GTN%3AT00039-orange)](https://gxy.io/GTN:T00039)
[![Last Updated](https://img.shields.io/badge/Last%20Updated-Jan%202026-green)]()
[![Level](https://img.shields.io/badge/Level-Intermediate-yellow)]()

---

## 📖 Table of Contents

- [Overview](#-overview)
- [Learning Objectives](#-learning-objectives)
- [Requirements](#-requirements)
- [Pipeline Overview](#-pipeline-overview)
- [Input Data](#-input-data)
- [Assembly Trajectories](#-assembly-trajectories)
- [Key Glossary](#-key-glossary)
- [Tutorial Steps](#-tutorial-steps)
  - [1. Get Data](#1-get-data)
  - [2. HiFi Preprocessing with Cutadapt](#2-hifi-preprocessing-with-cutadapt)
  - [3. Genome Profile Analysis](#3-genome-profile-analysis)
  - [4. Assembly with hifiasm](#4-assembly-with-hifiasm)
  - [5. Purging Duplications](#5-purging-duplications)
  - [6. Scaffolding with Bionano](#6-scaffolding-with-bionano)
  - [7. Hi-C Scaffolding with YaHS](#7-hi-c-scaffolding-with-yahs)
- [Quality Control Tools](#-quality-control-tools)
- [Datasets](#-datasets)
- [Authors & Contributors](#-authors--contributors)
- [Citation](#-citation)
- [License](#-license)

---

## 🔬 Overview

Advances in sequencing technologies have revolutionized *de novo* genome assembly. This tutorial uses **PacBio HiFi** reads in combination with **Bionano optical maps** and **Hi-C chromatin conformation data** to generate a high-quality genome assembly following the VGP pipeline.

Two major challenges motivate the VGP approach:

- **Repetitive content** transposable elements and tandem repeats constitute over a third of mammalian genomes and are a major source of assembly fragmentation.
- **Heterozygosity** haplotype phasing is fundamental for diploid and polyploid genome assemblies, and unresolved heterozygosity leads to false duplications.

The tutorial assembles *Saccharomyces cerevisiae* S288C as a demonstration model, using synthetic HiFi reads generated from the reference genome.

---

## 🎯 Learning Objectives

- Learn the tools necessary to perform a *de novo* assembly of a vertebrate genome
- Evaluate assembly quality using reference-free QC approaches
- Understand haplotype phasing, purging, and scaffolding concepts

**Estimated time:** ~5 hours

---

## ✅ Requirements

Before starting, you should be familiar with:

- [Introduction to Galaxy Analyses](https://training.galaxyproject.org/training-material/topics/introduction)
- [Quality Control (slides)](https://training.galaxyproject.org/training-material/topics/sequence-analysis/tutorials/quality-control/slides.html)
- [Quality Control (hands-on)](https://training.galaxyproject.org/training-material/topics/sequence-analysis/tutorials/quality-control/tutorial.html)

**Platform:** [UseGalaxy.cz](https://usegalaxy.cz/)

---

## 🗺️ Pipeline Overview

The VGP pipeline is modular, consisting of ten workflows organized around five stages:

```
1. K-mer Profiling      →  Estimate genome size, heterozygosity, repeat content
2. Contig Assembly      →  hifiasm (Solo / Hi-C-phased / Trio modes)
3. Purging (optional)   →  purge_dups to remove false duplications
4. Scaffolding          →  Bionano optical maps + Hi-C (YaHS)
5. Decontamination      →  Remove non-target / mitochondrial sequences
```

---

## 📥 Input Data

| Data type | Format | Purpose |
|-----------|--------|---------|
| PacBio HiFi reads | `fasta` / `fastqsanger.gz` | Primary assembly |
| Illumina Hi-C reads | `fastqsanger.gz` | Phasing & scaffolding |
| Bionano optical maps | `.cmap` | Hybrid scaffolding |
| Parental reads *(optional)* | `fastqsanger.gz` | Trio-based phasing |

> **Coverage recommendations:** ≥30× HiFi + ~60× Hi-C for diploids. Up to 60× HiFi for highly repetitive genomes.

---

## 🔀 Assembly Trajectories

Eight analysis trajectories are available depending on input data:

| Trajectory | Input data | Output |
|:---:|---|---|
| A | HiFi only | Pseudohaplotype primary + alternate |
| B | HiFi + Hi-C | Hi-C phased hap1 + hap2 |
| C | HiFi + Bionano | Improved contiguity primary |
| D | HiFi + Hi-C + Bionano | Best contiguity |
| E | HiFi + Parental | Properly phased maternal/paternal |
| F | HiFi + Parental + Hi-C | Better phasing resolution |
| G | HiFi + Parental + Bionano | Phased + improved contiguity |
| H | HiFi + Parental + Hi-C + Bionano | Maximum quality |

> **Note:** Hi-C data used for phasing **must** come from the same individual as the HiFi data. Hi-C from a different individual can still be used for scaffolding only.

---

## 📚 Key Glossary

| Term | Definition |
|------|-----------|
| **Contig** | A contiguous, gapless sequence inferred from assembly |
| **Scaffold** | One or more contigs separated by gap (N) sequences |
| **Primary assembly** | The more complete haploid representation; includes homozygous regions and one copy of heterozygous regions |
| **Alternate assembly** | Alternate alleles from heterozygous loci not represented in the primary |
| **Pseudohaplotype** | Long phased blocks separated by regions of ambiguous haplotype; may contain switch errors |
| **Haplotig** | A contig derived from a single haplotype |
| **Purging** | Removal of false duplications, collapsed repeats, and low-coverage regions |
| **False duplication** | One genomic region represented twice in the same assembly |
| **Haplotypic duplication** | Both alleles of a heterozygous region placed into the same assembly |
| **Phasing** | Partitioning contigs by the haplotype they originate from |
| **HiFi reads** | PacBio reads 10–25 kbp long with >Q20 (≥99%) accuracy |
| **Ultra-long reads** | Oxford Nanopore reads >100 kbp; lower accuracy but extreme length |
| **N50** | Length at which contigs of that size or longer cover ≥50% of the total assembly |
| **T2T assembly** | Telomere-to-telomere; complete gapless chromosome assembly |
| **Manual curation** | Human-guided correction of misassemblies using Hi-C contact maps |
| **Missed join** | Two adjacent genomic regions not represented contiguously in the assembly |
| **Misassembly** | Structural error where non-adjacent sequences are incorrectly placed next to each other |

---

## 🔬 Tutorial Steps

### 1. Get Data

Datasets are hosted on Zenodo. This tutorial uses synthetic HiFi reads from *S. cerevisiae* S288C (generated with HIsim at 2% mutation rate, 30× coverage).

**HiFi FASTA reads** — upload and set datatype to `fasta`:
```
https://zenodo.org/record/6098306/files/HiFi_synthetic_50x_01.fasta
https://zenodo.org/record/6098306/files/HiFi_synthetic_50x_02.fasta
https://zenodo.org/record/6098306/files/HiFi_synthetic_50x_03.fasta
```

**Hi-C reads** — upload and set datatype to `fastqsanger.gz`:
```
https://zenodo.org/record/5550653/files/SRR7126301_1.fastq.gz  →  rename: Hi-C_dataset_F
https://zenodo.org/record/5550653/files/SRR7126301_2.fastq.gz  →  rename: Hi-C_dataset_R
```

After uploading, group the three HiFi FASTA files into a **Galaxy dataset collection** named `HiFi data`. Workflows require a collection as input, not individual files.

> ⚠️ Hi-C datasets are large — expect ~15 min upload time.

> ⚠️ When setting Hi-C datatype, select `fastqsanger.gz` — **not** `fastqcssanger` or any other variant.

---

### 2. HiFi Preprocessing with Cutadapt

Due to SMRT sequencing chemistry, adapter sequences can appear at unpredictable internal positions within HiFi reads. Rather than trimming, any read containing an adapter is **discarded entirely**.

**Tool:** `Cutadapt` (v4.4+galaxy0)

| Parameter | Value |
|-----------|-------|
| Mode | Single-end |
| Adapter 1 | `ATCTCTCTCAACAACAACAACGGAGGAGGAGGAAAAGAGAGAGAT` |
| Adapter 2 | `ATCTCTCTCTTTTCCTCCTCCTCCGTTGTTGTTGTTGAGAGAGAT` |
| Max error rate | `0.1` |
| Min overlap | `35` |
| Search reverse complement | `Yes` |
| Discard trimmed reads | `Yes` |

Rename output: `HiFi_collection (trimmed)`

---

### 3. Genome Profile Analysis

K-mer analysis estimates genome properties (size, heterozygosity, repeat content, error rate) without any reference genome.

#### 3a. K-mer counting — Meryl

**Tool:** `Meryl` (v1.3+galaxy6) — run three times in sequence:

1. **Count** canonical k-mers (k=31) on `HiFi_collection (trimmed)` → rename `meryldb`
2. **Union-sum** the resulting collection → rename `Merged meryldb`
3. **Generate histogram** from merged meryldb → rename `meryldb histogram`

> K-mer size 31 is long enough that most k-mers are non-repetitive and short enough to tolerate sequencing errors. For very large (>10 Gb) or highly repetitive genomes, use a larger k.

#### 3b. Genome profiling — GenomeScope2

**Tool:** `GenomeScope` (v2.0+galaxy2)

| Parameter | Value |
|-----------|-------|
| Input | `meryldb histogram` |
| Ploidy | `2` |
| K-mer length | `31` |
| Output | Summary + model parameters file |

**Expected results for *S. cerevisiae*:**

| Property | Value |
|----------|-------|
| Haploid genome size | ~11.7 Mb |
| Heterozygosity | ~0.576% |
| Diploid coverage peak | ~50× |
| Haploid coverage peak | ~25× |
| Model fit | >93% |

---

### 4. Assembly with hifiasm

#### Assembly modes

| Mode | Input | Output | Notes |
|------|-------|--------|-------|
| **Solo** | HiFi only | Primary + Alternate contigs | Pseudohaplotype; usually requires purging |
| **Hi-C phased** | HiFi + Hi-C (same individual) | Hap1 + Hap2 contigs | Recommended when same-individual Hi-C available |
| **Trio** | HiFi + parental reads | Maternal + Paternal contigs | Gold standard; requires parental samples |

#### Hi-C phased assembly (recommended path)

**Tool:** `Hifiasm` (v0.19.8+galaxy0)

| Parameter | Value |
|-----------|-------|
| Assembly mode | `Standard` |
| Input reads | `HiFi_collection (trimmed)` |
| Hi-C R1 | `Hi-C_dataset_F` |
| Hi-C R2 | `Hi-C_dataset_R` |

Rename outputs: `Hap1 contigs graph` (tag `#hap1`) and `Hap2 contigs graph` (tag `#hap2`).

#### GFA → FASTA conversion

**Tool:** `gfastats` (v1.3.6+galaxy0)

| Parameter | Value |
|-----------|-------|
| Input | Both GFA files (multi-select) |
| Tool mode | `Genome assembly manipulation` |
| Output format | `FASTA` |
| Generate initial paths | `Yes` |

Rename: `Hap1 contigs FASTA` and `Hap2 contigs FASTA`

#### Expected assembly statistics (Hi-C phased)

| Statistic | Hap1 | Hap2 |
|-----------|------|------|
| # contigs | ~16 | ~17 |
| Total length | ~11.3 Mb | ~12.2 Mb |
| Longest contig | ~1.53 Mb | ~1.53 Mb |
| N50 | ~922 kb | ~923 kb |

---

### 5. Purging Duplications

Purging is triggered when QC reveals false duplications — indicated by a high BUSCO duplication rate or a large 2-copy k-mer peak in Merqury CN plots. It is generally **not needed** for Hi-C phased or trio assemblies, but is typically required for solo (pseudohaplotype) assemblies.

**Tool suite:** `purge_dups` (via `Purge overlaps` v1.2.6+galaxy0)

#### Step-by-step

**Step 1 — Collapse HiFi collection:**
Use `Collapse Collection` to merge `HiFi_collection (trim)` into a single file → `HiFi reads collapsed`

**Step 2 — Map reads to assembly:**
`Minimap2` with preset `asm5`, output `PAF` → `Reads mapped to contigs`

**Step 3 — Read-depth analysis:**
`Purge overlaps` in `calcuts+pbcstats` mode using the PAF file. Supply cutoffs derived from GenomeScope2:
- Transition (haploid/diploid): `38`
- Upper bound (max depth): `114`
- Ploidy: `Diploid`

**Step 4 — Self-alignment:**
Split the primary FASTA at N-blocks (`split_fa`), then run `Minimap2` with preset `self-homology` → `Self-homology map primary`

**Step 5 — Purge haplotigs:**
`Purge overlaps` in purge mode using coverage stats + self-alignment PAF

**Step 6 — Reconcile alternate assembly:**
Concatenate removed haplotigs with the original alternate assembly, then re-run purging on the combined alternate to produce the final clean alternate contig set.

---

### 6. Scaffolding with Bionano

Bionano optical maps provide structural information at hundreds of kb to Mb scale, enabling contigs to be joined across repeat regions that resist sequence-based scaffolding.

**Tool:** `Bionano Solve` (Workflow 7)

The hybrid scaffolding step aligns optical map data to the sequence assembly, introducing sized gaps between joined contigs. After scaffolding:

- Run **gfastats** to check scaffold N50 and gap count
- Run **BUSCO** to confirm gene completeness is maintained
- Run **Merqury** to confirm no QV degradation

---

### 7. Hi-C Scaffolding with YaHS

Hi-C captures chromatin interaction frequencies across the genome. Read pairs that were ligated together indicate physical proximity, providing long-range ordering and orientation signals for chromosome-scale scaffolding.

#### Pre-processing

Map Hi-C reads to the Bionano-scaffolded assembly to generate alignment files.

#### Scaffolding

**Tool:** `YaHS` — uses Hi-C contact information to order and orient scaffolds into chromosomes.

#### Evaluate with Pretext

Generate a Hi-C contact map with **PretextMap** and visualize with **PretextView** or **PretextSnapshot**.

| Pattern in contact map | Interpretation |
|------------------------|---------------|
| Bright squares on diagonal | Correctly assembled chromosomes |
| Off-diagonal signal | Possible misassembly — incorrect join |
| Abrupt signal discontinuity | Possible missed join |

Manual curation corrects misassemblies and missed joins identified in the contact map.

---

## 📊 Quality Control Tools

| Tool | What it measures |
|------|-----------------|
| **gfastats** | Contig/scaffold count, N50, NG50, total length, GC content, gap count |
| **BUSCO** | Gene completeness — expected single-copy orthologs present, duplicated, fragmented, or missing |
| **Merqury** | K-mer-based QV score, assembly completeness, phasing accuracy (spectra-cn and spectra-asm plots) |
| **GenomeScope2** | Genome size, heterozygosity, repeat content from k-mer spectra |
| **PretextMap / PretextView** | Hi-C contact map for visual misassembly detection and manual curation |

### Interpreting Merqury spectra plots

**CN plot (spectra-cn):**

| Observation | Meaning |
|-------------|---------|
| 'Read-only' peak (grey/black) at low coverage | Sequencing errors or sequences missing from assembly |
| Large 2-copy (blue) peak at diploid coverage | False duplications present → purging recommended |
| Clean 1-copy peak near haploid coverage | Well-purged assembly |

**ASM plot (spectra-asm):**

| Observation | Meaning |
|-------------|---------|
| Large shared (green) peak at diploid coverage | Homozygous regions correctly split between haplotypes ✅ |
| Asymmetric haploid peaks (one assembly >> other) | Incomplete phasing; excess content in one haplotype |
| Large assembly-specific peak at diploid coverage | False duplications — one haplotype has both allele copies |

---

## 📦 Datasets

| Dataset | Zenodo | Format |
|---------|--------|--------|
| Synthetic HiFi reads (3 files) | [zenodo.org/record/6098306](https://zenodo.org/record/6098306) | `fasta` |
| Hi-C paired reads | [zenodo.org/record/5550653](https://zenodo.org/record/5550653) | `fastqsanger.gz` |
| All tutorial datasets | [zenodo.org/record/5887339](https://zenodo.org/record/5887339) | Multiple |

---

## 👥 Authors & Contributors

**Tutorial Authors:**
Alex Ostrovsky · Cristóbal Gallardo · Anna Syme · Linelle Abueg · Brandon Pickett · Giulio Formenti · Marcella Sozzoni · Anton Nekrutenko

**Reviewers:**
Saskia Hiltemann · Björn Grüning · Delphine Lariviere · Helena Rasche · Marius van den Beek · Bérénice Batut

---

## 📄 Citation

If you use this tutorial or the VGP assembly pipeline, please cite:

> Rhie, A., McCarthy, S.A., Fedrigo, O. *et al.* Towards complete and error-free genome assemblies of all vertebrate species. *Nature* **592**, 737–746 (2021). https://doi.org/10.1038/s41586-021-03451-0

**Individual tools:**

| Tool | Citation |
|------|---------|
| hifiasm | Cheng *et al.* (2021). *Nature Methods* 18, 170–175 |
| purge_dups | Guan *et al.* (2019). *Genome Research* 30, 1060–1070 |
| Merqury | Rhie *et al.* (2020). *Genome Biology* 21, 245 |
| gfastats | Formenti *et al.* (2022). *Bioinformatics* 38(17), 4214–4216 |
| GenomeScope2 | Ranallo-Benavidez *et al.* (2020). *Nature Communications* 11, 1432 |
| YaHS | Zhou *et al.* (2023). *Bioinformatics* 39(1) |

---

## 📜 License

Tutorial content is licensed under [Creative Commons Attribution 4.0 International](http://creativecommons.org/licenses/by/4.0/).
The GTN framework is licensed under [MIT](https://github.com/galaxyproject/training-material/blob/main/LICENSE.md).

---

<div align="center">

**Part of the [Galaxy Training Network](https://training.galaxyproject.org/) · [Vertebrate Genomes Project](https://vertebrategenomesproject.org/) · [Earth BioGenome Project](https://www.earthbiogenome.org/)**

</div>
