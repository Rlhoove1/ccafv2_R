# ccAFv2: Cell cycle classifier for R and Seurat
[![standard-readme compliant](https://img.shields.io/badge/readme%20style-standard-brightgreen.svg?style=flat-square)](https://github.com/RichardLitt/standard-readme)

This repository is for the R package for the cell cycle classifier ccAFv2. The input for the ccAFv2 classifier is single cell or nuclei, or spatial RNA-seq data.  The features of this classifier are that it classifies six cell cycle states (G1, Late G1, S, S/G2, G2/M, and M/Early G1) and a quiescent-like G0 state, and it incorporates a tunable parameter to filter out less certain classifications. This package is implemented in R so that it can be used in [Seurat](https://satijalab.org/seurat/) analysis workflows. We provide examples of how to install, run ccAFv2 on seurat objects (both sc/snRNA-seq and ST-RNA-seq), plot and use results, and regress out cell cycle effects if that is desired.

## Table of Contents

- [Install](#install)
- [Classifying single cell or nuclei RNA-seq](#classifying-single-cell-or-nuclei-rna-seq)
	- [Cell cycle classification](#cell-cycle-classification)
	- [Cell cycle likelihoods](#cell-cycle-classification)
    - [Plotting cell cycle states](#plotting-cell-cycle-states)
	- [Applying thresholds](#applying-thresholds)
	- [Cell cycle regression](#cell-cycle-regression)
- [Usage spatial RNA-seq](#usage)
	- [Cell cycle classification](#cell-cycle-classification)
	- [Cell cycle likelihoods](#cell-cycle-classification)
    - [Plotting cell cycle states](#plotting-cell-cycle-states)
	- [Applying thresholds](#applying-thresholds)
- [Maintainers](#maintainers)
- [Contributing](#contributing)

## Install

The installation of ccAv2 in R requires the [devtools](https://cran.r-project.org/web/packages/devtools/readme/README.html) package be install first. The devtools packace can be accomplished using the command:

```r
install.packages('devtools')
```

Once the devtools package is installed it can then be used to install the ccAFv2 R package using the following command:

```r
devtools::install_github('plaisier-lab/ccafv2_R/ccAFv2')
```

## Classifying single cell or nuclei RNA-seq

### Input for classification

It is expected that the input for the ccAFv2 classifier will be a Seurat object that has been thorougly quality controlled. We provide an example of our quality control pipeline in can be found [here](https://github.com/plaisier-lab/ccAFv2/blob/main/scripts/02_scQC_2024.R). Is is preferred that the data in the Seurat object be SCTransformed, however, the standard approach for normalization only applies to the highly variable genes. This can exclude genes needed for the
accurate classification of the cell cycle. For this reason the ccAFv2 PredictCellCycle function used to classify cell cycle states runs the SCTransform function again parameterized so that it will retain all genes captured in the dataset.

### Cell cycle classification

Classification is as easy as two lines that can be added to any Seurat workflow. First the library must be loaded and then the PredictCellCycle function is run:

```r
library(ccAFv2)
seurat_obj = PredictCellCycle(seurat_obj)

```
When the classifier is running it should look something like this:

```r
Running ccAFv2:
 Redoing SCTransform to ensure maximum overlap with classifier genes...
 Total possible marker genes for this classifier: 861
  Marker genes present in this dataset: 861
  Missing marker genes in this dataset: 0
 Predicting cell cycle state probabilities...
93/93 [==============================] - 1s 4ms/step
93/93 [==============================] - 1s 4ms/step
 Choosing cell cycle state...
 Adding probabilitities and predictions to metadata
Done
```

It is important to look at how many marker genes were present in the dataset. We found that when less than 689 marker genes (or 80%) were found in the dataset that this led significantly less accurate predictions. Using the default 'do_sctransform' paramter setting of TRUE should yeild the largest possible overlap with the marker genes. And some of the later values for the timing and 93/93 may differ for your dataset, which is perfectly fine.

There are several options that can be passed to the PredictCellCycle function:
```r
PredictCellCycle(seurat_obj, 
                 cutoff=0.5, 
                 do_sctransform=TRUE,
                 assay='SCT',
                 species='human',
                 gene_id='ensembl',
                 spatial = FALSE) 
```
- **seurat_obj**: a seurat object must be supplied to classify, no default
- **cutoff**: the value used to threchold the likelihoods, default is 0.5
- **do_sctransform**: whether to do SCTransform before classifying, default is TRUE
- **assay**: which seurat_obj assay to use for classification, helpful if data is prenormalized, default is 'SCT'
- **species**: from which species did the samples originate, either 'human' or 'mouse', defaults to 'human'
- **gene_id**: what type of gene ID is used, either 'ensembl' or 'symbol', defaults to 'ensembl'
- **spatial**: whether the data is spatial, defaults to FALSE


### Cell cycle classification results

The results of the cell cycle classification is stored in the seurat object metadata. The likelihoods for each cell cycle state can be found with the labels of each cell cycle state ('Neural.G0', 'G1', 'Late.G1', 'S', 'S.G2', 'G2.M', and 'M.Early.G1') and the classification for each cell can be found int the 'ccAFv2'. Here is the first 10 rows of the U5-hNSC predictions:

```
                 orig.ident       nCount_RNA nFeature_RNA percent.mito
AAACATACTAACCG-1 SeuratProject    3147       1466         0.05274865
AAACATTGAGTTCG-1 SeuratProject    7621       2654         0.02611206
AAACATTGCACTGA-1 SeuratProject    7297       2616         0.02686035
AAACATTGCTCAGA-1 SeuratProject    3426       1568         0.02597782
AAACATTGGTTTCT-1 SeuratProject    2384       1269         0.01971477
AAACCGTGTAACGC-1 SeuratProject   11846       3463         0.02439642
AAACGCACCTTCTA-1 SeuratProject    3021       1249         0.01886792
AAACGCTGGTATGC-1 SeuratProject    5937       2127         0.02846555
AAACGCTGTGCTGA-1 SeuratProject    8361       2793         0.03564167
AAACGGCTGTCTAG-1 SeuratProject    4591       1605         0.04530603
                 nCount_SCT nFeature_SCT G1           G2.M         Late.G1
AAACATACTAACCG-1 5436       1541         4.102312e-02 5.279975e-05 2.521812e-04
AAACATTGAGTTCG-1 6562       2650         1.091572e-06 2.126902e-08 1.087603e-07
AAACATTGCACTGA-1 6533       2616         4.089213e-06 1.517231e-04 6.863667e-07
AAACATTGCTCAGA-1 5416       1608         9.184604e-03 2.376643e-01 1.454412e-02
AAACATTGGTTTCT-1 5437       1480         6.363088e-01 1.654975e-04 9.073097e-02
AAACCGTGTAACGC-1 6860       3082         5.167215e-15 9.999999e-01 1.610888e-19
AAACGCACCTTCTA-1 5537       1308         4.707390e-06 3.709571e-08 6.463646e-05
AAACGCTGGTATGC-1 5954       2127         2.741130e-01 2.113478e-05 7.243306e-01
AAACGCTGTGCTGA-1 6666       2780         1.320201e-04 2.646190e-07 1.700663e-06
AAACGGCTGTCTAG-1 5568       1608         3.885373e-01 3.646265e-03 2.107647e-02
                 M.Early.G1   Neural.G0    S            S.G2         ccAFv2
AAACATACTAACCG-1 4.220059e-05 9.585268e-01 5.953405e-05 4.332590e-05 Neural G0
AAACATTGAGTTCG-1 8.592179e-10 9.998107e-01 1.868801e-04 1.347080e-06 Neural G0
AAACATTGCACTGA-1 4.317539e-09 1.362584e-06 6.708771e-07 9.998415e-01 S/G2
AAACATTGCTCAGA-1 6.309561e-01 5.861283e-02 4.330526e-03 4.470748e-02 M/Early G1
AAACATTGGTTTCT-1 2.112151e-05 1.050980e-03 1.899571e-01 8.176555e-02 G1
AAACCGTGTAACGC-1 1.919447e-18 3.083377e-16 6.074685e-23 1.652127e-15 G2/M
AAACGCACCTTCTA-1 4.957183e-07 9.401034e-07 9.998928e-01 3.640788e-05 S
AAACGCTGGTATGC-1 7.537001e-06 3.954049e-04 9.016516e-04 2.307067e-04 Late G1
AAACGCTGTGCTGA-1 1.102674e-08 9.998627e-01 2.419535e-06 1.108445e-06 Neural G0
AAACGGCTGTCTAG-1 7.390769e-04 5.405837e-01 5.436572e-03 3.998061e-02 Neural G

```

### Plotting cell cycle states

We provide plotting functions that colorize the cell cycle states in the way used in our manuscripts. We strongly suggest using these functions when plotting if possible.

#### Plotting a UMAP with cell cycle states

Plotting cells using ther first two dimensions from a dimensionality reduction method (e.g., PCA, tSNE, or UMAP) is a common way to represent single cell or nuclei RNA-seq data. We have an overloaded DimPlot function that colorizes the cells based on their called cell cycle state. The function accepts all the parameters that DimPlot can accept, exceptr for group.by and cols. Here is how the plotting function should be run:

```r
DimPlot.ccAFv2(seurat_obj)
```

#### Plotting the impact of varying likelihood thresholds

Plotting cells using ther first two dimensions from a dimensionality reduction method (e.g., PCA, tSNE, or UMAP) is a common way to represent single cell or nuclei RNA-seq data. We have an overloaded DimPlot function that colorizes the cells based on their called cell cycle state. The function accepts all the parameters that DimPlot can accept, exceptr for group.by and cols. Here is how the plotting function should be run:

```r
DimPlot.ccAFv2(seurat_obj)
```


### Cell cycle regression

- [open-source-template](https://github.com/davidbgk/open-source-template/) - A README template to encourage open-source contributions.

## Maintainers

[@plaisier-lab](https://github.com/plaisier-lab).

## Contributing

Feel free to dive in! [Open an issue](https://github.com/plaisier-lab/ccAFv2_R/issues/new) or submit PRs.
