# HypervariableFinder

A bioinformatics pipeline for automated discovery of **genomic islands** (hypervariable regions) in a set of genomes genomes. It clusters near-identical genomes using **skani**, aligns them with **minimap2**, and produces publication-ready **synteny plots** that visually highlight putative genomic islands as alignment gaps.

---

## Overview

When multiple strains of a  virus or bacteria share high average nucleotide identity (ANI) but differ at specific loci, those loci are candidate **genomic islands** — regions of elevated evolutionary plasticity often linked to host interaction, defense systems, or horizontal gene transfer.

This pipeline automates the full workflow:

1. **Pairwise ANI calculation** across all input genomes (`skani`)
2. **Genome clustering** via connected-component analysis at a user-defined ANI threshold
3. **Within-cluster alignment** to a reference backbone (`minimap2`)
4. **Synteny visualization** — gaps in alignment blocks flag putative genomic islands

---

## Dependencies

| Tool | Version Tested | Install |
|------|---------------|---------|
| Python | ≥ 3.8 | — |
| [skani](https://github.com/bluenote-1577/skani) | ≥ 0.2 | `conda install -c bioconda skani` |
| [minimap2](https://github.com/lh3/minimap2) | ≥ 2.24 | `conda install -c bioconda minimap2` |
| pandas | ≥ 1.3 | `pip install pandas` |
| matplotlib | ≥ 3.5 | `pip install matplotlib` |

> **Recommended:** use a conda environment to manage all dependencies.

```bash
conda create -n hypervariable -c bioconda skani minimap2 python=3.10
conda activate hypervariable
pip install pandas matplotlib
```

---

## Installation

```bash
git clone https://github.com/yourusername/hypervariable-finder.git
cd hypervariable-finder
chmod +x hypervariable_finder.py
```

---

## Usage

```bash
python hypervariable_finder.py -i <input_dir> -o <output_dir> [options]
```

### Arguments

| Flag | Description | Default |
|------|-------------|---------|
| `-i`, `--input` | Directory of input `.fasta` genome files | *(required)* |
| `-o`, `--output` | Directory for all output files | *(required)* |
| `-t`, `--threshold` | ANI threshold (%) for clustering | `99.0` |

### Example

```bash
python hypervariable_finder.py \
  -i ./genomes/ \
  -o ./results/ \
  -t 99.5
```

---

## Output

```
results/
├── skani_out/
│   ├── input_files_list.txt
│   └── skani_edge_list.tsv        # Raw pairwise ANI values
├── clusters/
│   ├── cluster_1/                 # FASTA files for each cluster
│   ├── cluster_2/
│   └── ...
│       └── cluster_N_alignment.paf
├── cluster_1_synteny.png          # Synteny plot per cluster
├── cluster_2_synteny.png
└── ...
```

### Synteny Plot Guide

Each PNG shows genome-wide alignment blocks for all cluster members against a reference backbone:

- **Black line** — Reference genome backbone
- **Blue blocks** — Forward-strand alignments
- **Red blocks** — Reverse-strand alignments
- **White gaps** — Putative genomic islands (regions absent from the reference)

---

## How It Works

```
Input FASTAs
     │
     ▼
[skani triangle]  ──►  All-vs-all ANI edge list
     │
     ▼
[Connected Components]  ──►  Genome clusters at ≥ ANI threshold
     │
     ▼
[minimap2 asm5]  ──►  PAF alignment per cluster
     │
     ▼
[matplotlib]  ──►  Synteny PNG (gaps = genomic islands)
```

Clustering uses a depth-first search over the ANI graph, so any pair of genomes above the threshold that share a transitive connection will end up in the same cluster.

---

## Notes & Limitations

- The first FASTA file encountered in each cluster directory is used as the reference backbone. For reproducible reference selection, sort or rename your input files accordingly.
- Alignments shorter than 500 bp are filtered as spurious.
- This pipeline is designed for **high-identity comparisons** (≥ 95% ANI). For more divergent genomes, consider lowering the threshold and inspecting results carefully.
- Very large clusters may produce complex plots — consider subsetting genomes if cluster size exceeds ~20.

---

## Citation

If you use HypervariableFinder in your research, please also cite the underlying tools:

- **skani**: Jim Shaw & Yun William Yu (2023). *Rapid and accurate metagenomic sequence comparison with skani.* Nature Methods.
- **minimap2**: Heng Li (2018). *Minimap2: pairwise alignment for nucleotide sequences.* Bioinformatics.

---

## License

MIT License. See `LICENSE` for details.
