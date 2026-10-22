# blastoid-scenic-grn

Gene regulatory network (GRN) inference from single-cell RNA-seq of human blastoids using pySCENIC. The analysis identifies transcription factor regulons, scores their activity in each cell with AUCell, and visualizes cell populations in regulon-activity space with UMAP.


The most active regulons were found to be PPARG and GATA3 inside bCTB (cytotrophoblasts) and POU5F1 (or OCT-4) inside bEPI (epithelial)

![UMAP of cells by regulon activity](DATA/Pyscenic%20UMAP%20plots%20screenshot.png)

## Overview

The full analysis is in a single notebook, `01_pyscenic_grn_analysis.ipynb`. Plots and outputs are saved in the notebook, so you can read the results on GitHub without running anything.

1. **GRN inference.** GRNBoost2 (arboreto) infers TF-to-target adjacencies from the day 21 blastoid expression matrix, restricted to a list of human TFs.
2. **Regulon prediction.** Adjacencies are grouped into modules, then pruned with cisTarget motif enrichment (pySCENIC `prune2df`) using the hg38 10 kb up / 10 kb down motif rankings database and the v10 motif-to-TF annotations. Enriched motifs and regulons are saved to `DATA/motifs.csv` and `DATA/regulons.p`.
3. **Regulon activity.** AUCell scores each regulon in each cell. The AUC matrix is shown as a clustermap, then embedded with PCA, a nearest-neighbor graph, and UMAP (scanpy), colored by cell type and by the three regulons with the highest mean activity.
4. **QC and clustering (scanpy).** Basic filtering, normalization, highly variable gene selection, and Louvain clustering of the expression data, with diagnostic plots.

### QC parameters

| Step | Setting |
|---|---|
| Minimum genes per cell | 200 |
| Minimum cells per gene | 3 |
| Maximum genes per cell | fewer than 4,000 |
| Normalization | 10,000 counts per cell, then log1p |
| Highly variable genes | min_mean 0.0125, max_mean 3, min_disp 0.5 |
| Neighbors / PCs | 15 neighbors, 30 PCs |
| Louvain resolution | 0.4 |

## Limitations

GRN inference (GRNBoost2, pruning, AUCell) was run on the unfiltered expression matrix. The QC and clustering steps in step 4 were run separately on the same data, and their filtered outputs were not fed back into pySCENIC. Results should be read with that in mind. 

## Data

Large input files are not stored in this repository. Download them and place them in the folders below, which should sit next to the notebook.

```
.
├── 01_pyscenic_grn_analysis.ipynb
├── environment.yml
├── DATA/          # created by the notebook (loom files, regulons.p, motifs.csv)
├── DATABASES/
│   └── hg38_10kbp_up_10kbp_down_full_tx_v10_clust.genes_vs_motifs.rankings.feather
└── RESOURCES/
    ├── Blastoid.1day21.data.csv
    ├── Blastoid.1.cell_types.csv
    ├── allTFs_hg38.txt
    └── motifs-v10nr_clust-nr.hgnc-m0.001-o0.0.tbl
```

- **Single-cell expression data and cell type labels:** human blastoid scRNA-seq, sample GSM7677371 in GEO series GSE226794 (Karvas et al., 2023). GEO provides the processed data as MTX/TSV files in GSE226794_RAW.tar; the notebook reads a CSV version of the day 21 matrix, RESOURCES/Blastoid.1day21.data.csv
- **Cell Type Labels:** `RESOURCES/Blastoid.1.cell_types.csv`
- **Motif rankings database:** `hg38_10kbp_up_10kbp_down_full_tx_v10_clust.genes_vs_motifs.rankings.feather` from the [cisTarget resources](https://resources.aertslab.org/cistarget/databases/homo_sapiens/hg38/refseq_r80/mc_v10_clust/gene_based/). 
- **Motif-to-TF annotations:** `motifs-v10nr_clust-nr.hgnc-m0.001-o0.0.tbl` from the [motif2tf folder](https://resources.aertslab.org/cistarget/motif2tf).
- **Human TF list:** `allTFs_hg38.txt`, from the TF lists provided on the [cisTarget resources site](https://resources.aertslab.org/cistarget/).

## Setup

The notebook was run on Windows with Python 3.8.

```
conda env create -f environment.yml
conda activate pyscenic_env
jupyter notebook
```

Open `01_pyscenic_grn_analysis.ipynb` from the project folder. The first cell sets the working directory to the project root, so the `DATA`, `DATABASES`, and `RESOURCES` folders are found automatically.

pySCENIC is sensitive to package versions, so use the pinned versions in `environment.yml`. The environment file was exported on Windows and may need small adjustments on other platforms.

Do not use "Run All" the first time. GRNBoost2 takes a long time on a matrix this size, and the pruning step overwrites `DATA/regulons.p` and `DATA/motifs.csv`. The saved outputs in the notebook show what each step produced.

## Tools

pySCENIC, arboreto (GRNBoost2), ctxcore, scanpy, anndata, loompy, umap-learn, python-igraph, louvain, pandas, numpy, seaborn, matplotlib.

## References
- Karvas RM, et al. 3D-cultured blastoids model human embryogenesis from pre-implantation to early gastrulation stages. Cell Stem Cell 30(9), 1148-1165.e7 (2023). doi:10.1016/j.stem.2023.08.005
- Aibar S, et al. SCENIC: single-cell regulatory network inference and clustering. *Nature Methods* 14, 1083-1086 (2017).
- Van de Sande B, et al. A scalable SCENIC workflow for single-cell gene regulatory network analysis. *Nature Protocols* 15, 2247-2276 (2020).
