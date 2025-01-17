---
author: "Åsa Björklund  &  Paulo Czarnewski"
date: 'January 22, 2021'
output:
  html_document:
    self_contained: true
    highlight: tango
    df_print: paged
    toc: yes
    toc_float:
      collapsed: false
      smooth_scroll: true
    toc_depth: 3
    keep_md: yes
    fig_caption: true
  html_notebook:
    self_contained: true
    highlight: tango
    df_print: paged
    toc: yes
    toc_float:
      collapsed: false
      smooth_scroll: true
    toc_depth: 3
editor_options: 
  chunk_output_type: console
---


<style>
h1, .h1, h2, .h2, h3, .h3, h4, .h4 { margin-top: 50px }
p.caption {font-size: 0.9em;font-style: italic;color: grey;margin-right: 10%;margin-left: 10%;text-align: justify}
</style>

# Differential gene expression

In this tutorial we will cover about Differetial gene expression, which comprises an extensive range of topics and methods. In single cell, differential expresison can have multiple functionalities such as of identifying marker genes for cell populations, as well as differentially regulated genes across conditions (healthy vs control). We will also exercise on how to account the batch information in your test.

We can first load the data from the clustering session. Moreover, we can already decide which clustering resolution to use. First let's define using the `louvain` clustering to identifying differentially expressed genes.  


```r
suppressPackageStartupMessages({
    library(scater)
    library(scran)
    # library(venn)
    library(cowplot)
    library(ggplot2)
    # library(rafalib)
    library(pheatmap)
    library(igraph)
    library(dplyr)
})

sce <- readRDS("data/results/covid_qc_dr_int_cl.rds")
```

## Cell marker genes
***

Let us first compute a ranking for the highly differential genes in each cluster. There are many different tests and parameters to be chosen that can be used to refine your results. When looking for marker genes, we want genes that are positivelly expressed in a cell type and possibly not expressed in the others.


```r
# Compute differentiall expression
markers_genes <- scran::findMarkers(x = sce, groups = as.character(sce$louvain_SNNk15), 
    lfc = 0.5, pval.type = "all", direction = "up")

# List of dataFrames with the results for each cluster
markers_genes
```

```
## List of length 9
## names(9): 1 2 3 4 5 6 7 8 9
```

```r
# Visualizing the expression of one
markers_genes[["1"]]
```

```
## DataFrame with 18121 rows and 10 columns
##                         p.value                  FDR              logFC.2
##                       <numeric>            <numeric>            <numeric>
## PF4        5.42236942598342e-16 9.82587563682454e-12     3.01259842765971
## PPBP       5.00056541348096e-10 4.53076229288442e-06     2.74257205744216
## GNG11      5.06161511405363e-09 3.05738424939219e-05     1.73041831456037
## OST4       3.07547152464266e-08 0.000139326548745124     1.87877915781507
## NRGN       1.85074354590057e-07 0.000670746475905283     2.08202120232519
## ...                         ...                  ...                  ...
## AL592183.1                    1                    1  -0.0187820319786617
## AC007325.4                    1                    1  -0.0103004003928936
## AL354822.1                    1                    1 -0.00115246909162764
## AC004556.1                    1                    1  -0.0203955520005165
## AC233755.1                    1                    1                    0
##                         logFC.3              logFC.4              logFC.5
##                       <numeric>            <numeric>            <numeric>
## PF4            3.49645473674068     3.71650582803699     3.97082330250393
## PPBP           3.25285030340934      3.5358030592694      4.0585250617998
## GNG11          1.83623668138815     1.88469241020645     2.05995995533478
## OST4           1.88123537370156     2.10997397036826     2.22804490374774
## NRGN           2.43702690676073     2.57392457310674     3.43286114568247
## ...                         ...                  ...                  ...
## AL592183.1  -0.0834303779606505   -0.132234591840394  -0.0992384326208183
## AC007325.4 -0.00659670894011874   -0.020049794508921 -0.00325537366633247
## AL354822.1 -0.00207784348641178 -0.00307332511360672  -0.0148900779361155
## AC004556.1  -0.0863727572375302   -0.151851034301112  -0.0649677107625443
## AC233755.1                    0                    0                    0
##                          logFC.6              logFC.7              logFC.8
##                        <numeric>            <numeric>            <numeric>
## PF4             3.75882391271893     3.92452797172521     3.93989492544756
## PPBP            3.42402456902586     3.94767644870726      3.9178936327251
## GNG11           1.89687239074213     2.05368301018222     2.05703647611279
## OST4            2.14540449652277      1.9307661625303     2.25580705714899
## NRGN             2.7537850162175     3.34180837857761     3.37772776639445
## ...                          ...                  ...                  ...
## AL592183.1   -0.0839305968400599   -0.161401303237351   -0.199465267512809
## AC007325.4 -0.000770947272540883 -0.00493851468672034 -0.00136015783179628
## AL354822.1                     0 -0.00489866295805064  -0.0158253670990768
## AC004556.1    -0.100088330539916  -0.0714877934768116  -0.0425330546944955
## AC233755.1                     0                    0 -0.00406988649560987
##                         logFC.9
##                       <numeric>
## PF4            3.93296778606565
## PPBP           4.00643150853551
## GNG11          2.07571196413914
## OST4           1.85064380320819
## NRGN           3.39901107279735
## ...                         ...
## AL592183.1   -0.136006712007594
## AC007325.4 -0.00474466901986934
## AL354822.1 -0.00520532662403466
## AC004556.1  -0.0876779685867214
## AC233755.1                    0
```

We can now select the top 25 up regulated genes for plotting.


```r
# Colect the top 25 genes for each cluster and put the into a single table
top25 <- lapply(names(markers_genes), function(x) {
    temp <- markers_genes[[x]][1:25, 1:2]
    temp$gene <- rownames(markers_genes[[x]])[1:25]
    temp$cluster <- x
    return(temp)
})
top25 <- as_tibble(do.call(rbind, top25))
top25$p.value[top25$p.value == 0] <- 9.99999999999999e-301
top25
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["p.value"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["FDR"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["gene"],"name":[3],"type":["chr"],"align":["left"]},{"label":["cluster"],"name":[4],"type":["chr"],"align":["left"]}],"data":[{"1":"5.422369e-16","2":"9.825876e-12","3":"PF4","4":"1"},{"1":"5.000565e-10","2":"4.530762e-06","3":"PPBP","4":"1"},{"1":"5.061615e-09","2":"3.057384e-05","3":"GNG11","4":"1"},{"1":"3.075472e-08","2":"1.393265e-04","3":"OST4","4":"1"},{"1":"1.850744e-07","2":"6.707465e-04","3":"NRGN","4":"1"},{"1":"2.279237e-07","2":"6.883675e-04","3":"HIST1H2AC","4":"1"},{"1":"3.516811e-06","2":"9.104019e-03","3":"CAVIN2","4":"1"},{"1":"4.051409e-06","2":"9.176949e-03","3":"HIST1H3H","4":"1"},{"1":"4.958262e-06","2":"9.983184e-03","3":"TSC22D1","4":"1"},{"1":"6.368110e-06","2":"1.073362e-02","3":"MYL9","4":"1"},{"1":"6.515632e-06","2":"1.073362e-02","3":"GP9","4":"1"},{"1":"8.437806e-06","2":"1.274179e-02","3":"PDLIM1","4":"1"},{"1":"1.569497e-05","2":"2.142025e-02","3":"CA2","4":"1"},{"1":"1.654895e-05","2":"2.142025e-02","3":"TREML1","4":"1"},{"1":"2.555446e-05","2":"3.087149e-02","3":"ACRBP","4":"1"},{"1":"8.904342e-05","2":"1.008472e-01","3":"TUBB1","4":"1"},{"1":"1.138405e-04","2":"1.213473e-01","3":"CMTM5","4":"1"},{"1":"1.300170e-04","2":"1.308910e-01","3":"MTURN","4":"1"},{"1":"1.907936e-04","2":"1.819669e-01","3":"CLEC1B","4":"1"},{"1":"2.854641e-04","2":"2.586447e-01","3":"SNCA","4":"1"},{"1":"3.293002e-04","2":"2.841547e-01","3":"RGS18","4":"1"},{"1":"6.652366e-04","2":"5.479433e-01","3":"RGS10","4":"1"},{"1":"8.003119e-04","2":"6.305414e-01","3":"TAGLN2","4":"1"},{"1":"8.939151e-04","2":"6.749432e-01","3":"HIST1H2BJ","4":"1"},{"1":"1.208210e-03","2":"8.757588e-01","3":"HIST1H4H","4":"1"},{"1":"2.129243e-78","2":"3.858402e-74","3":"S100A8","4":"2"},{"1":"5.534858e-60","2":"5.014858e-56","3":"S100A9","4":"2"},{"1":"1.736683e-43","2":"1.049015e-39","3":"RETN","4":"2"},{"1":"4.274269e-25","2":"1.936351e-21","3":"S100A12","4":"2"},{"1":"2.376393e-18","2":"8.612524e-15","3":"PLBD1","4":"2"},{"1":"9.806479e-17","2":"2.961720e-13","3":"RNASE2","4":"2"},{"1":"1.174210e-16","2":"3.039695e-13","3":"HP","4":"2"},{"1":"2.637057e-15","2":"5.973263e-12","3":"LGALS1","4":"2"},{"1":"6.126059e-15","2":"1.233448e-11","3":"CTSD","4":"2"},{"1":"4.000951e-10","2":"7.250123e-07","3":"SERPINB1","4":"2"},{"1":"1.453341e-07","2":"2.394182e-04","3":"MCEMP1","4":"2"},{"1":"7.235005e-07","2":"1.092546e-03","3":"SELL","4":"2"},{"1":"2.197835e-06","2":"3.063613e-03","3":"GCA","4":"2"},{"1":"1.543123e-04","2":"1.997353e-01","3":"PGD","4":"2"},{"1":"5.225407e-03","2":"1.000000e+00","3":"HMGB2","4":"2"},{"1":"1.294755e-02","2":"1.000000e+00","3":"IL1R2","4":"2"},{"1":"2.239959e-02","2":"1.000000e+00","3":"RFLNB","4":"2"},{"1":"2.408958e-02","2":"1.000000e+00","3":"SLC2A3","4":"2"},{"1":"5.529934e-02","2":"1.000000e+00","3":"ALOX5AP","4":"2"},{"1":"9.047229e-02","2":"1.000000e+00","3":"PKM","4":"2"},{"1":"1.010072e-01","2":"1.000000e+00","3":"H2AFJ","4":"2"},{"1":"1.632067e-01","2":"1.000000e+00","3":"GAPDH","4":"2"},{"1":"3.885454e-01","2":"1.000000e+00","3":"FPR1","4":"2"},{"1":"3.957235e-01","2":"1.000000e+00","3":"VCAN","4":"2"},{"1":"3.985862e-01","2":"1.000000e+00","3":"FOLR3","4":"2"},{"1":"4.149050e-21","2":"7.518494e-17","3":"IL1B","4":"3"},{"1":"3.719406e-08","2":"3.369968e-04","3":"G0S2","4":"3"},{"1":"3.372107e-06","2":"2.036865e-02","3":"SGK1","4":"3"},{"1":"1.153657e-05","2":"5.226354e-02","3":"DUSP6","4":"3"},{"1":"3.384554e-05","2":"1.226630e-01","3":"ZFP36L1","4":"3"},{"1":"1.800960e-03","2":"1.000000e+00","3":"MPEG1","4":"3"},{"1":"1.116056e-02","2":"1.000000e+00","3":"IER3","4":"3"},{"1":"2.043576e-02","2":"1.000000e+00","3":"ZFP36","4":"3"},{"1":"4.182234e-02","2":"1.000000e+00","3":"ISG15","4":"3"},{"1":"1.243646e-01","2":"1.000000e+00","3":"MARCKS","4":"3"},{"1":"1.388834e-01","2":"1.000000e+00","3":"TNFAIP2","4":"3"},{"1":"1.655125e-01","2":"1.000000e+00","3":"APOBEC3A","4":"3"},{"1":"3.304999e-01","2":"1.000000e+00","3":"DUSP1","4":"3"},{"1":"3.973590e-01","2":"1.000000e+00","3":"ICAM1","4":"3"},{"1":"4.383213e-01","2":"1.000000e+00","3":"IRF2BP2","4":"3"},{"1":"5.408402e-01","2":"1.000000e+00","3":"CSF3R","4":"3"},{"1":"5.675248e-01","2":"1.000000e+00","3":"CCR1","4":"3"},{"1":"6.147987e-01","2":"1.000000e+00","3":"PIM3","4":"3"},{"1":"6.963147e-01","2":"1.000000e+00","3":"SOD2","4":"3"},{"1":"7.134342e-01","2":"1.000000e+00","3":"BACH1","4":"3"},{"1":"7.365638e-01","2":"1.000000e+00","3":"PLEK","4":"3"},{"1":"7.373486e-01","2":"1.000000e+00","3":"NFKBIA","4":"3"},{"1":"8.411561e-01","2":"1.000000e+00","3":"C15orf48","4":"3"},{"1":"8.898440e-01","2":"1.000000e+00","3":"KLF4","4":"3"},{"1":"8.943655e-01","2":"1.000000e+00","3":"IFNGR2","4":"3"},{"1":"4.327178e-51","2":"7.841279e-47","3":"LST1","4":"4"},{"1":"1.368360e-49","2":"1.239802e-45","3":"CDKN1C","4":"4"},{"1":"1.867456e-38","2":"1.128006e-34","3":"FCGR3A","4":"4"},{"1":"1.668024e-28","2":"7.556567e-25","3":"SMIM25","4":"4"},{"1":"1.472182e-25","2":"5.335481e-22","3":"MS4A7","4":"4"},{"1":"4.680560e-23","2":"1.413607e-19","3":"COTL1","4":"4"},{"1":"8.119674e-18","2":"2.101952e-14","3":"LRRC25","4":"4"},{"1":"1.778374e-16","2":"4.028239e-13","3":"AIF1","4":"4"},{"1":"1.736216e-15","2":"3.495774e-12","3":"RHOC","4":"4"},{"1":"6.239874e-15","2":"1.130728e-11","3":"TCF7L2","4":"4"},{"1":"1.347946e-14","2":"2.220558e-11","3":"PECAM1","4":"4"},{"1":"3.195998e-14","2":"4.826224e-11","3":"CTSC","4":"4"},{"1":"4.421539e-14","2":"6.163285e-11","3":"CSF1R","4":"4"},{"1":"1.761910e-12","2":"2.280540e-09","3":"DRAP1","4":"4"},{"1":"2.283304e-10","2":"2.758383e-07","3":"SERPINA1","4":"4"},{"1":"2.463345e-09","2":"2.789893e-06","3":"HES4","4":"4"},{"1":"6.538811e-09","2":"6.969987e-06","3":"MBD2","4":"4"},{"1":"7.803572e-08","2":"7.856030e-05","3":"RRAS","4":"4"},{"1":"1.259586e-07","2":"1.201314e-04","3":"IFITM3","4":"4"},{"1":"1.516004e-07","2":"1.373576e-04","3":"RNASET2","4":"4"},{"1":"2.065131e-07","2":"1.782011e-04","3":"UNC119","4":"4"},{"1":"2.829595e-06","2":"2.330686e-03","3":"WASF2","4":"4"},{"1":"4.787957e-06","2":"3.772286e-03","3":"SPI1","4":"4"},{"1":"6.692747e-06","2":"5.053303e-03","3":"CALM2","4":"4"},{"1":"7.302486e-06","2":"5.293134e-03","3":"PSAP","4":"4"},{"1":"9.716529e-67","2":"1.760732e-62","3":"PRF1","4":"5"},{"1":"1.220754e-63","2":"1.106064e-59","3":"KLRF1","4":"5"},{"1":"3.318422e-55","2":"2.004437e-51","3":"GZMB","4":"5"},{"1":"1.294979e-46","2":"5.866578e-43","3":"CD247","4":"5"},{"1":"1.590798e-44","2":"5.765369e-41","3":"GNLY","4":"5"},{"1":"1.319247e-37","2":"3.984347e-34","3":"FGFBP2","4":"5"},{"1":"2.042586e-35","2":"5.287673e-32","3":"SPON2","4":"5"},{"1":"1.239602e-25","2":"2.807854e-22","3":"KLRD1","4":"5"},{"1":"9.930547e-25","2":"1.999460e-21","3":"TRDC","4":"5"},{"1":"5.445810e-22","2":"9.868352e-19","3":"CTSW","4":"5"},{"1":"6.396126e-16","2":"1.053675e-12","3":"CLIC3","4":"5"},{"1":"2.528944e-14","2":"3.818917e-11","3":"MYOM2","4":"5"},{"1":"1.387712e-12","2":"1.934364e-09","3":"NKG7","4":"5"},{"1":"4.041152e-09","2":"5.230694e-06","3":"CD7","4":"5"},{"1":"4.906542e-06","2":"5.927430e-03","3":"KLRB1","4":"5"},{"1":"1.885248e-03","2":"1.000000e+00","3":"IL2RB","4":"5"},{"1":"2.059972e-03","2":"1.000000e+00","3":"HOPX","4":"5"},{"1":"9.330114e-03","2":"1.000000e+00","3":"APMAP","4":"5"},{"1":"4.213115e-02","2":"1.000000e+00","3":"ABHD17A","4":"5"},{"1":"1.254590e-01","2":"1.000000e+00","3":"CST7","4":"5"},{"1":"1.351332e-01","2":"1.000000e+00","3":"GNPTAB","4":"5"},{"1":"2.426434e-01","2":"1.000000e+00","3":"CEP78","4":"5"},{"1":"4.396810e-01","2":"1.000000e+00","3":"SYNGR1","4":"5"},{"1":"4.726800e-01","2":"1.000000e+00","3":"PRSS23","4":"5"},{"1":"5.385764e-01","2":"1.000000e+00","3":"S1PR5","4":"5"},{"1":"2.199664e-01","2":"1.000000e+00","3":"HLA-DRB5","4":"6"},{"1":"3.664346e-01","2":"1.000000e+00","3":"FCER1A","4":"6"},{"1":"8.559414e-01","2":"1.000000e+00","3":"CLEC10A","4":"6"},{"1":"9.985435e-01","2":"1.000000e+00","3":"HNRNPH1","4":"6"},{"1":"9.996354e-01","2":"1.000000e+00","3":"CD1C","4":"6"},{"1":"9.999220e-01","2":"1.000000e+00","3":"ENHO","4":"6"},{"1":"9.999993e-01","2":"1.000000e+00","3":"HLA-DQA1","4":"6"},{"1":"9.999994e-01","2":"1.000000e+00","3":"HLA-DPA1","4":"6"},{"1":"9.999997e-01","2":"1.000000e+00","3":"CBX6","4":"6"},{"1":"9.999997e-01","2":"1.000000e+00","3":"KCNK6","4":"6"},{"1":"9.999999e-01","2":"1.000000e+00","3":"PPP1R14B","4":"6"},{"1":"9.999999e-01","2":"1.000000e+00","3":"USF2","4":"6"},{"1":"1.000000e+00","2":"1.000000e+00","3":"C1orf56","4":"6"},{"1":"1.000000e+00","2":"1.000000e+00","3":"MAT2A","4":"6"},{"1":"1.000000e+00","2":"1.000000e+00","3":"CDC37","4":"6"},{"1":"1.000000e+00","2":"1.000000e+00","3":"GSN","4":"6"},{"1":"1.000000e+00","2":"1.000000e+00","3":"HNRNPU","4":"6"},{"1":"1.000000e+00","2":"1.000000e+00","3":"ARF5","4":"6"},{"1":"1.000000e+00","2":"1.000000e+00","3":"KTN1","4":"6"},{"1":"1.000000e+00","2":"1.000000e+00","3":"HNRNPM","4":"6"},{"1":"1.000000e+00","2":"1.000000e+00","3":"PPIA","4":"6"},{"1":"1.000000e+00","2":"1.000000e+00","3":"TUBA1B","4":"6"},{"1":"1.000000e+00","2":"1.000000e+00","3":"LMNA","4":"6"},{"1":"1.000000e+00","2":"1.000000e+00","3":"SLC25A3","4":"6"},{"1":"1.000000e+00","2":"1.000000e+00","3":"C1QBP","4":"6"},{"1":"5.609294e-47","2":"1.016460e-42","3":"IL7R","4":"7"},{"1":"5.039634e-23","2":"4.566160e-19","3":"SARAF","4":"7"},{"1":"5.794823e-21","2":"3.500266e-17","3":"RCAN3","4":"7"},{"1":"4.462649e-16","2":"2.021691e-12","3":"LDHB","4":"7"},{"1":"6.527515e-15","2":"2.365702e-11","3":"PIK3IP1","4":"7"},{"1":"1.155340e-14","2":"3.489318e-11","3":"MAL","4":"7"},{"1":"5.374149e-14","2":"1.391214e-10","3":"NOSIP","4":"7"},{"1":"8.914734e-11","2":"2.019299e-07","3":"TCF7","4":"7"},{"1":"1.450188e-07","2":"2.919872e-04","3":"TPT1","4":"7"},{"1":"2.881374e-07","2":"5.221338e-04","3":"LEPROTL1","4":"7"},{"1":"7.230341e-04","2":"1.000000e+00","3":"LEF1","4":"7"},{"1":"1.296255e-02","2":"1.000000e+00","3":"AP3M2","4":"7"},{"1":"1.631319e-02","2":"1.000000e+00","3":"PRKCA","4":"7"},{"1":"1.671909e-02","2":"1.000000e+00","3":"LTB","4":"7"},{"1":"5.890820e-02","2":"1.000000e+00","3":"RPS14","4":"7"},{"1":"8.563274e-02","2":"1.000000e+00","3":"SERINC5","4":"7"},{"1":"1.180489e-01","2":"1.000000e+00","3":"RPL4","4":"7"},{"1":"3.279125e-01","2":"1.000000e+00","3":"RPS4X","4":"7"},{"1":"3.813936e-01","2":"1.000000e+00","3":"ARHGAP15","4":"7"},{"1":"3.856184e-01","2":"1.000000e+00","3":"TRABD2A","4":"7"},{"1":"4.372602e-01","2":"1.000000e+00","3":"IKZF1","4":"7"},{"1":"6.221300e-01","2":"1.000000e+00","3":"GIMAP7","4":"7"},{"1":"6.309127e-01","2":"1.000000e+00","3":"KLF2","4":"7"},{"1":"6.397415e-01","2":"1.000000e+00","3":"EEF1A1","4":"7"},{"1":"7.342347e-01","2":"1.000000e+00","3":"AQP3","4":"7"},{"1":"2.665070e-225","2":"4.829373e-221","3":"MS4A1","4":"8"},{"1":"2.530845e-122","2":"2.293072e-118","3":"TNFRSF13C","4":"8"},{"1":"7.632029e-121","2":"4.610000e-117","3":"CD79A","4":"8"},{"1":"1.204448e-120","2":"5.456449e-117","3":"IGHD","4":"8"},{"1":"2.981057e-112","2":"1.080395e-108","3":"LINC00926","4":"8"},{"1":"1.347020e-80","2":"4.068225e-77","3":"IGHM","4":"8"},{"1":"2.568423e-56","2":"6.648913e-53","3":"RALGPS2","4":"8"},{"1":"1.152455e-50","2":"2.610455e-47","3":"BANK1","4":"8"},{"1":"1.918904e-39","2":"3.863607e-36","3":"CD83","4":"8"},{"1":"1.288892e-38","2":"2.335601e-35","3":"CD22","4":"8"},{"1":"2.267397e-37","2":"3.735227e-34","3":"CD37","4":"8"},{"1":"9.766772e-37","2":"1.474864e-33","3":"NFKBID","4":"8"},{"1":"1.609933e-32","2":"2.244123e-29","3":"JUND","4":"8"},{"1":"1.244199e-27","2":"1.610438e-24","3":"TCL1A","4":"8"},{"1":"9.500198e-27","2":"1.147687e-23","3":"VPREB3","4":"8"},{"1":"1.017715e-25","2":"1.152626e-22","3":"CD79B","4":"8"},{"1":"6.471914e-25","2":"6.898680e-22","3":"P2RX5","4":"8"},{"1":"8.533669e-24","2":"8.591034e-21","3":"BLK","4":"8"},{"1":"1.589452e-18","2":"1.515919e-15","3":"TAGAP","4":"8"},{"1":"1.627050e-16","2":"1.474188e-13","3":"FCER2","4":"8"},{"1":"5.811165e-16","2":"5.014482e-13","3":"FAM129C","4":"8"},{"1":"8.688017e-15","2":"7.156162e-12","3":"AFF3","4":"8"},{"1":"6.873522e-13","2":"5.415439e-10","3":"TCF4","4":"8"},{"1":"3.923492e-12","2":"2.962400e-09","3":"EZR","4":"8"},{"1":"1.014154e-11","2":"7.350993e-09","3":"FCRL1","4":"8"},{"1":"8.339891e-24","2":"1.511272e-19","3":"CD8A","4":"9"},{"1":"5.518262e-17","2":"4.999821e-13","3":"DUSP2","4":"9"},{"1":"1.532478e-15","2":"9.256676e-12","3":"TRGC2","4":"9"},{"1":"3.116866e-12","2":"1.412018e-08","3":"LYAR","4":"9"},{"1":"3.890098e-09","2":"1.409849e-05","3":"GZMK","4":"9"},{"1":"2.428778e-07","2":"7.335315e-04","3":"KLRG1","4":"9"},{"1":"4.340522e-06","2":"1.123637e-02","3":"CD8B","4":"9"},{"1":"3.068889e-02","2":"1.000000e+00","3":"CD3D","4":"9"},{"1":"1.170862e-01","2":"1.000000e+00","3":"IL32","4":"9"},{"1":"4.964501e-01","2":"1.000000e+00","3":"TNFAIP3","4":"9"},{"1":"6.522238e-01","2":"1.000000e+00","3":"PIK3R1","4":"9"},{"1":"7.650978e-01","2":"1.000000e+00","3":"CD3G","4":"9"},{"1":"9.525077e-01","2":"1.000000e+00","3":"SRSF7","4":"9"},{"1":"9.931377e-01","2":"1.000000e+00","3":"CCL5","4":"9"},{"1":"9.986064e-01","2":"1.000000e+00","3":"TUBA4A","4":"9"},{"1":"9.996717e-01","2":"1.000000e+00","3":"RPS26","4":"9"},{"1":"9.999587e-01","2":"1.000000e+00","3":"LINC01871","4":"9"},{"1":"9.999742e-01","2":"1.000000e+00","3":"PTPRC","4":"9"},{"1":"9.999992e-01","2":"1.000000e+00","3":"CD2","4":"9"},{"1":"9.999997e-01","2":"1.000000e+00","3":"CD3E","4":"9"},{"1":"9.999998e-01","2":"1.000000e+00","3":"SUB1","4":"9"},{"1":"9.999999e-01","2":"1.000000e+00","3":"HSP90AA1","4":"9"},{"1":"1.000000e+00","2":"1.000000e+00","3":"SELENOK","4":"9"},{"1":"1.000000e+00","2":"1.000000e+00","3":"TERF2IP","4":"9"},{"1":"1.000000e+00","2":"1.000000e+00","3":"CD99","4":"9"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

We can now select the top 25 up regulated genes for plotting.


```r
par(mfrow = c(1, 5), mar = c(4, 6, 3, 1))
for (i in unique(top25$cluster)) {
    barplot(sort(setNames(-log10(top25$p.value), top25$gene)[top25$cluster == i], 
        F), horiz = T, las = 1, main = paste0(i, " vs. rest"), border = "white", 
        yaxs = "i", xlab = "-log10FC")
    abline(v = c(0, -log10(0.05)), lty = c(1, 2))
}
```

![](scater_05_dge_files/figure-html/unnamed-chunk-4-1.png)<!-- -->![](scater_05_dge_files/figure-html/unnamed-chunk-4-2.png)<!-- -->

We can visualize them as a heatmap. Here we are selecting the top 5.


```r
top5 <- as_tibble(top25) %>% group_by(cluster) %>% top_n(-5, p.value)

scater::plotHeatmap(sce[, order(sce$louvain_SNNk15)], features = unique(top5$gene), 
    center = T, zlim = c(-3, 3), colour_columns_by = "louvain_SNNk15", show_colnames = F, 
    cluster_cols = F, fontsize_row = 6, color = colorRampPalette(c("purple", "black", 
        "yellow"))(90))
```

![](scater_05_dge_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

We can also plot a violin plot for each gene.


```r
scater::plotExpression(sce, features = unique(top5$gene), x = "louvain_SNNk15", ncol = 5, 
    colour_by = "louvain_SNNk15", scales = "free")
```

![](scater_05_dge_files/figure-html/unnamed-chunk-6-1.png)<!-- -->


## Differential expression across conditions
***

The second way of computing differential expression is to answer which genes are differentially expressed within a cluster. For example, in our case we have libraries comming from patients and controls and we would like to know which genes are influenced the most in a particular cell type.

For this end, we will first subset our data for the desired cell cluster, then change the cell identities to the variable of comparison (which now in our case is the "type", e.g. Covid/Ctrl).


```r
# Filter cells from that cluster
cell_selection <- sce[, sce$louvain_SNNk15 == 8]

# Compute differentiall expression
DGE_cell_selection <- findMarkers(x = cell_selection, groups = cell_selection@colData$type, 
    lfc = 0.25, pval.type = "all", direction = "any")
top5_cell_selection <- lapply(names(DGE_cell_selection), function(x) {
    temp <- DGE_cell_selection[[x]][1:5, 1:2]
    temp$gene <- rownames(DGE_cell_selection[[x]])[1:5]
    temp$cluster <- x
    return(temp)
})
top5_cell_selection <- as_tibble(do.call(rbind, top5_cell_selection))
top5_cell_selection
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["p.value"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["FDR"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["gene"],"name":[3],"type":["chr"],"align":["left"]},{"label":["cluster"],"name":[4],"type":["chr"],"align":["left"]}],"data":[{"1":"4.207009e-100","2":"7.623520e-96","3":"RPS4Y1","4":"Control"},{"1":"3.953036e-58","2":"3.581648e-54","3":"XIST","4":"Control"},{"1":"1.235929e-40","2":"7.465424e-37","3":"RPS26","4":"Control"},{"1":"1.718179e-34","2":"7.783781e-31","3":"XAF1","4":"Control"},{"1":"1.374424e-28","2":"4.981188e-25","3":"IFI44L","4":"Control"},{"1":"4.207009e-100","2":"7.623520e-96","3":"RPS4Y1","4":"Covid"},{"1":"3.953036e-58","2":"3.581648e-54","3":"XIST","4":"Covid"},{"1":"1.235929e-40","2":"7.465424e-37","3":"RPS26","4":"Covid"},{"1":"1.718179e-34","2":"7.783781e-31","3":"XAF1","4":"Covid"},{"1":"1.374424e-28","2":"4.981188e-25","3":"IFI44L","4":"Covid"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

We can now plot the expression across the "type".


```r
scater::plotExpression(cell_selection, features = unique(top5_cell_selection$gene), 
    x = "type", ncol = 5, colour_by = "type")
```

![](scater_05_dge_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

#DGE_ALL6.2:


```r
plotlist <- list()
for (i in unique(top5_cell_selection$gene)) {
    plotlist[[i]] <- plotReducedDim(sce, dimred = "UMAP_on_MNN", colour_by = i, by_exprs_values = "logcounts") + 
        ggtitle(label = i) + theme(plot.title = element_text(size = 20))
}
plot_grid(ncol = 3, plotlist = plotlist)
```

![](scater_05_dge_files/figure-html/unnamed-chunk-9-1.png)<!-- -->


## Gene Set Analysis
***

Hypergeometric enrichment test

Having a defined list of differentially expressed genes, you can now look for their combined function using hypergeometric test:


```r
# Load additional packages
library(enrichR)

# Check available databases to perform enrichment (then choose one)
enrichR::listEnrichrDbs()
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":["geneCoverage"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["genesPerTerm"],"name":[2],"type":["dbl"],"align":["right"]},{"label":["libraryName"],"name":[3],"type":["chr"],"align":["left"]},{"label":["link"],"name":[4],"type":["chr"],"align":["left"]},{"label":["numTerms"],"name":[5],"type":["dbl"],"align":["right"]}],"data":[{"1":"13362","2":"275","3":"Genome_Browser_PWMs","4":"http://hgdownload.cse.ucsc.edu/goldenPath/hg18/database/","5":"615"},{"1":"27884","2":"1284","3":"TRANSFAC_and_JASPAR_PWMs","4":"http://jaspar.genereg.net/html/DOWNLOAD/","5":"326"},{"1":"6002","2":"77","3":"Transcription_Factor_PPIs","4":"","5":"290"},{"1":"47172","2":"1370","3":"ChEA_2013","4":"http://amp.pharm.mssm.edu/lib/cheadownload.jsp","5":"353"},{"1":"47107","2":"509","3":"Drug_Perturbations_from_GEO_2014","4":"http://www.ncbi.nlm.nih.gov/geo/","5":"701"},{"1":"21493","2":"3713","3":"ENCODE_TF_ChIP-seq_2014","4":"http://genome.ucsc.edu/ENCODE/downloads.html","5":"498"},{"1":"1295","2":"18","3":"BioCarta_2013","4":"https://cgap.nci.nih.gov/Pathways/BioCarta_Pathways","5":"249"},{"1":"3185","2":"73","3":"Reactome_2013","4":"http://www.reactome.org/download/index.html","5":"78"},{"1":"2854","2":"34","3":"WikiPathways_2013","4":"http://www.wikipathways.org/index.php/Download_Pathways","5":"199"},{"1":"15057","2":"300","3":"Disease_Signatures_from_GEO_up_2014","4":"http://www.ncbi.nlm.nih.gov/geo/","5":"142"},{"1":"4128","2":"48","3":"KEGG_2013","4":"http://www.kegg.jp/kegg/download/","5":"200"},{"1":"34061","2":"641","3":"TF-LOF_Expression_from_GEO","4":"http://www.ncbi.nlm.nih.gov/geo/","5":"269"},{"1":"7504","2":"155","3":"TargetScan_microRNA","4":"http://www.targetscan.org/cgi-bin/targetscan/data_download.cgi?db=vert_61","5":"222"},{"1":"16399","2":"247","3":"PPI_Hub_Proteins","4":"http://amp.pharm.mssm.edu/X2K","5":"385"},{"1":"12753","2":"57","3":"GO_Molecular_Function_2015","4":"http://www.geneontology.org/GO.downloads.annotations.shtml","5":"1136"},{"1":"23726","2":"127","3":"GeneSigDB","4":"http://genesigdb.org/genesigdb/downloadall.jsp","5":"2139"},{"1":"32740","2":"85","3":"Chromosome_Location","4":"http://software.broadinstitute.org/gsea/msigdb/index.jsp","5":"386"},{"1":"13373","2":"258","3":"Human_Gene_Atlas","4":"http://biogps.org/downloads/","5":"84"},{"1":"19270","2":"388","3":"Mouse_Gene_Atlas","4":"http://biogps.org/downloads/","5":"96"},{"1":"13236","2":"82","3":"GO_Cellular_Component_2015","4":"http://www.geneontology.org/GO.downloads.annotations.shtml","5":"641"},{"1":"14264","2":"58","3":"GO_Biological_Process_2015","4":"http://www.geneontology.org/GO.downloads.annotations.shtml","5":"5192"},{"1":"3096","2":"31","3":"Human_Phenotype_Ontology","4":"http://www.human-phenotype-ontology.org/","5":"1779"},{"1":"22288","2":"4368","3":"Epigenomics_Roadmap_HM_ChIP-seq","4":"http://www.roadmapepigenomics.org/","5":"383"},{"1":"4533","2":"37","3":"KEA_2013","4":"http://amp.pharm.mssm.edu/lib/keacommandline.jsp","5":"474"},{"1":"10231","2":"158","3":"NURSA_Human_Endogenous_Complexome","4":"https://www.nursa.org/nursa/index.jsf","5":"1796"},{"1":"2741","2":"5","3":"CORUM","4":"http://mips.helmholtz-muenchen.de/genre/proj/corum/","5":"1658"},{"1":"5655","2":"342","3":"SILAC_Phosphoproteomics","4":"http://amp.pharm.mssm.edu/lib/keacommandline.jsp","5":"84"},{"1":"10406","2":"715","3":"MGI_Mammalian_Phenotype_Level_3","4":"http://www.informatics.jax.org/","5":"71"},{"1":"10493","2":"200","3":"MGI_Mammalian_Phenotype_Level_4","4":"http://www.informatics.jax.org/","5":"476"},{"1":"11251","2":"100","3":"Old_CMAP_up","4":"http://www.broadinstitute.org/cmap/","5":"6100"},{"1":"8695","2":"100","3":"Old_CMAP_down","4":"http://www.broadinstitute.org/cmap/","5":"6100"},{"1":"1759","2":"25","3":"OMIM_Disease","4":"http://www.omim.org/downloads","5":"90"},{"1":"2178","2":"89","3":"OMIM_Expanded","4":"http://www.omim.org/downloads","5":"187"},{"1":"851","2":"15","3":"VirusMINT","4":"http://mint.bio.uniroma2.it/download.html","5":"85"},{"1":"10061","2":"106","3":"MSigDB_Computational","4":"http://www.broadinstitute.org/gsea/msigdb/collections.jsp","5":"858"},{"1":"11250","2":"166","3":"MSigDB_Oncogenic_Signatures","4":"http://www.broadinstitute.org/gsea/msigdb/collections.jsp","5":"189"},{"1":"15406","2":"300","3":"Disease_Signatures_from_GEO_down_2014","4":"http://www.ncbi.nlm.nih.gov/geo/","5":"142"},{"1":"17711","2":"300","3":"Virus_Perturbations_from_GEO_up","4":"http://www.ncbi.nlm.nih.gov/geo/","5":"323"},{"1":"17576","2":"300","3":"Virus_Perturbations_from_GEO_down","4":"http://www.ncbi.nlm.nih.gov/geo/","5":"323"},{"1":"15797","2":"176","3":"Cancer_Cell_Line_Encyclopedia","4":"https://portals.broadinstitute.org/ccle/home\\n","5":"967"},{"1":"12232","2":"343","3":"NCI-60_Cancer_Cell_Lines","4":"http://biogps.org/downloads/","5":"93"},{"1":"13572","2":"301","3":"Tissue_Protein_Expression_from_ProteomicsDB","4":"https://www.proteomicsdb.org/","5":"207"},{"1":"6454","2":"301","3":"Tissue_Protein_Expression_from_Human_Proteome_Map","4":"http://www.humanproteomemap.org/index.php","5":"30"},{"1":"3723","2":"47","3":"HMDB_Metabolites","4":"http://www.hmdb.ca/downloads","5":"3906"},{"1":"7588","2":"35","3":"Pfam_InterPro_Domains","4":"ftp://ftp.ebi.ac.uk/pub/databases/interpro/","5":"311"},{"1":"7682","2":"78","3":"GO_Biological_Process_2013","4":"http://www.geneontology.org/GO.downloads.annotations.shtml","5":"941"},{"1":"7324","2":"172","3":"GO_Cellular_Component_2013","4":"http://www.geneontology.org/GO.downloads.annotations.shtml","5":"205"},{"1":"8469","2":"122","3":"GO_Molecular_Function_2013","4":"http://www.geneontology.org/GO.downloads.annotations.shtml","5":"402"},{"1":"13121","2":"305","3":"Allen_Brain_Atlas_up","4":"http://www.brain-map.org/","5":"2192"},{"1":"26382","2":"1811","3":"ENCODE_TF_ChIP-seq_2015","4":"http://genome.ucsc.edu/ENCODE/downloads.html","5":"816"},{"1":"29065","2":"2123","3":"ENCODE_Histone_Modifications_2015","4":"http://genome.ucsc.edu/ENCODE/downloads.html","5":"412"},{"1":"280","2":"9","3":"Phosphatase_Substrates_from_DEPOD","4":"http://www.koehn.embl.de/depod/","5":"59"},{"1":"13877","2":"304","3":"Allen_Brain_Atlas_down","4":"http://www.brain-map.org/","5":"2192"},{"1":"15852","2":"912","3":"ENCODE_Histone_Modifications_2013","4":"http://genome.ucsc.edu/ENCODE/downloads.html","5":"109"},{"1":"4320","2":"129","3":"Achilles_fitness_increase","4":"http://www.broadinstitute.org/achilles","5":"216"},{"1":"4271","2":"128","3":"Achilles_fitness_decrease","4":"http://www.broadinstitute.org/achilles","5":"216"},{"1":"10496","2":"201","3":"MGI_Mammalian_Phenotype_2013","4":"http://www.informatics.jax.org/","5":"476"},{"1":"1678","2":"21","3":"BioCarta_2015","4":"https://cgap.nci.nih.gov/Pathways/BioCarta_Pathways","5":"239"},{"1":"756","2":"12","3":"HumanCyc_2015","4":"http://humancyc.org/","5":"125"},{"1":"3800","2":"48","3":"KEGG_2015","4":"http://www.kegg.jp/kegg/download/","5":"179"},{"1":"2541","2":"39","3":"NCI-Nature_2015","4":"http://pid.nci.nih.gov/","5":"209"},{"1":"1918","2":"39","3":"Panther_2015","4":"http://www.pantherdb.org/","5":"104"},{"1":"5863","2":"51","3":"WikiPathways_2015","4":"http://www.wikipathways.org/index.php/Download_Pathways","5":"404"},{"1":"6768","2":"47","3":"Reactome_2015","4":"http://www.reactome.org/download/index.html","5":"1389"},{"1":"25651","2":"807","3":"ESCAPE","4":"http://www.maayanlab.net/ESCAPE/","5":"315"},{"1":"19129","2":"1594","3":"HomoloGene","4":"http://www.ncbi.nlm.nih.gov/homologene","5":"12"},{"1":"23939","2":"293","3":"Disease_Perturbations_from_GEO_down","4":"http://www.ncbi.nlm.nih.gov/geo/","5":"839"},{"1":"23561","2":"307","3":"Disease_Perturbations_from_GEO_up","4":"http://www.ncbi.nlm.nih.gov/geo/","5":"839"},{"1":"23877","2":"302","3":"Drug_Perturbations_from_GEO_down","4":"http://www.ncbi.nlm.nih.gov/geo/","5":"906"},{"1":"15886","2":"9","3":"Genes_Associated_with_NIH_Grants","4":"https://grants.nih.gov/grants/oer.htm\\n","5":"32876"},{"1":"24350","2":"299","3":"Drug_Perturbations_from_GEO_up","4":"http://www.ncbi.nlm.nih.gov/geo/","5":"906"},{"1":"3102","2":"25","3":"KEA_2015","4":"http://amp.pharm.mssm.edu/Enrichr","5":"428"},{"1":"31132","2":"298","3":"Gene_Perturbations_from_GEO_up","4":"http://www.ncbi.nlm.nih.gov/geo/","5":"2460"},{"1":"30832","2":"302","3":"Gene_Perturbations_from_GEO_down","4":"http://www.ncbi.nlm.nih.gov/geo/","5":"2460"},{"1":"48230","2":"1429","3":"ChEA_2015","4":"http://amp.pharm.mssm.edu/Enrichr","5":"395"},{"1":"5613","2":"36","3":"dbGaP","4":"http://www.ncbi.nlm.nih.gov/gap","5":"345"},{"1":"9559","2":"73","3":"LINCS_L1000_Chem_Pert_up","4":"https://clue.io/","5":"33132"},{"1":"9448","2":"63","3":"LINCS_L1000_Chem_Pert_down","4":"https://clue.io/","5":"33132"},{"1":"16725","2":"1443","3":"GTEx_Tissue_Sample_Gene_Expression_Profiles_down","4":"http://www.gtexportal.org/","5":"2918"},{"1":"19249","2":"1443","3":"GTEx_Tissue_Sample_Gene_Expression_Profiles_up","4":"http://www.gtexportal.org/","5":"2918"},{"1":"15090","2":"282","3":"Ligand_Perturbations_from_GEO_down","4":"http://www.ncbi.nlm.nih.gov/geo/","5":"261"},{"1":"16129","2":"292","3":"Aging_Perturbations_from_GEO_down","4":"http://www.ncbi.nlm.nih.gov/geo/","5":"286"},{"1":"15309","2":"308","3":"Aging_Perturbations_from_GEO_up","4":"http://www.ncbi.nlm.nih.gov/geo/","5":"286"},{"1":"15103","2":"318","3":"Ligand_Perturbations_from_GEO_up","4":"http://www.ncbi.nlm.nih.gov/geo/","5":"261"},{"1":"15022","2":"290","3":"MCF7_Perturbations_from_GEO_down","4":"http://www.ncbi.nlm.nih.gov/geo/","5":"401"},{"1":"15676","2":"310","3":"MCF7_Perturbations_from_GEO_up","4":"http://www.ncbi.nlm.nih.gov/geo/","5":"401"},{"1":"15854","2":"279","3":"Microbe_Perturbations_from_GEO_down","4":"http://www.ncbi.nlm.nih.gov/geo/","5":"312"},{"1":"15015","2":"321","3":"Microbe_Perturbations_from_GEO_up","4":"http://www.ncbi.nlm.nih.gov/geo/","5":"312"},{"1":"3788","2":"159","3":"LINCS_L1000_Ligand_Perturbations_down","4":"https://clue.io/","5":"96"},{"1":"3357","2":"153","3":"LINCS_L1000_Ligand_Perturbations_up","4":"https://clue.io/","5":"96"},{"1":"12668","2":"300","3":"L1000_Kinase_and_GPCR_Perturbations_down","4":"https://clue.io/","5":"3644"},{"1":"12638","2":"300","3":"L1000_Kinase_and_GPCR_Perturbations_up","4":"https://clue.io/","5":"3644"},{"1":"8973","2":"64","3":"Reactome_2016","4":"http://www.reactome.org/download/index.html","5":"1530"},{"1":"7010","2":"87","3":"KEGG_2016","4":"http://www.kegg.jp/kegg/download/","5":"293"},{"1":"5966","2":"51","3":"WikiPathways_2016","4":"http://www.wikipathways.org/index.php/Download_Pathways","5":"437"},{"1":"15562","2":"887","3":"ENCODE_and_ChEA_Consensus_TFs_from_ChIP-X","4":"","5":"104"},{"1":"17850","2":"300","3":"Kinase_Perturbations_from_GEO_down","4":"http://www.ncbi.nlm.nih.gov/geo/","5":"285"},{"1":"17660","2":"300","3":"Kinase_Perturbations_from_GEO_up","4":"http://www.ncbi.nlm.nih.gov/geo/","5":"285"},{"1":"1348","2":"19","3":"BioCarta_2016","4":"http://cgap.nci.nih.gov/Pathways/BioCarta_Pathways","5":"237"},{"1":"934","2":"13","3":"HumanCyc_2016","4":"http://humancyc.org/","5":"152"},{"1":"2541","2":"39","3":"NCI-Nature_2016","4":"http://pid.nci.nih.gov/","5":"209"},{"1":"2041","2":"42","3":"Panther_2016","4":"http://www.pantherdb.org/pathway/","5":"112"},{"1":"5209","2":"300","3":"DrugMatrix","4":"https://ntp.niehs.nih.gov/drugmatrix/","5":"7876"},{"1":"49238","2":"1550","3":"ChEA_2016","4":"http://amp.pharm.mssm.edu/Enrichr","5":"645"},{"1":"2243","2":"19","3":"huMAP","4":"http://proteincomplexes.org/","5":"995"},{"1":"19586","2":"545","3":"Jensen_TISSUES","4":"http://tissues.jensenlab.org/","5":"1842"},{"1":"22440","2":"505","3":"RNA-Seq_Disease_Gene_and_Drug_Signatures_from_GEO","4":"http://www.ncbi.nlm.nih.gov/geo/","5":"1302"},{"1":"8184","2":"24","3":"MGI_Mammalian_Phenotype_2017","4":"http://www.informatics.jax.org/","5":"5231"},{"1":"18329","2":"161","3":"Jensen_COMPARTMENTS","4":"http://compartments.jensenlab.org/","5":"2283"},{"1":"15755","2":"28","3":"Jensen_DISEASES","4":"http://diseases.jensenlab.org/","5":"1811"},{"1":"10271","2":"22","3":"BioPlex_2017","4":"http://bioplex.hms.harvard.edu/","5":"3915"},{"1":"10427","2":"38","3":"GO_Cellular_Component_2017","4":"http://www.geneontology.org/","5":"636"},{"1":"10601","2":"25","3":"GO_Molecular_Function_2017","4":"http://www.geneontology.org/","5":"972"},{"1":"13822","2":"21","3":"GO_Biological_Process_2017","4":"http://www.geneontology.org/","5":"3166"},{"1":"8002","2":"143","3":"GO_Cellular_Component_2017b","4":"http://www.geneontology.org/","5":"816"},{"1":"10089","2":"45","3":"GO_Molecular_Function_2017b","4":"http://www.geneontology.org/","5":"3271"},{"1":"13247","2":"49","3":"GO_Biological_Process_2017b","4":"http://www.geneontology.org/","5":"10125"},{"1":"21809","2":"2316","3":"ARCHS4_Tissues","4":"http://amp.pharm.mssm.edu/archs4","5":"108"},{"1":"23601","2":"2395","3":"ARCHS4_Cell-lines","4":"http://amp.pharm.mssm.edu/archs4","5":"125"},{"1":"20883","2":"299","3":"ARCHS4_IDG_Coexp","4":"http://amp.pharm.mssm.edu/archs4","5":"352"},{"1":"19612","2":"299","3":"ARCHS4_Kinases_Coexp","4":"http://amp.pharm.mssm.edu/archs4","5":"498"},{"1":"25983","2":"299","3":"ARCHS4_TFs_Coexp","4":"http://amp.pharm.mssm.edu/archs4","5":"1724"},{"1":"19500","2":"137","3":"SysMyo_Muscle_Gene_Sets","4":"http://sys-myo.rhcloud.com/","5":"1135"},{"1":"14893","2":"128","3":"miRTarBase_2017","4":"http://mirtarbase.mbc.nctu.edu.tw/","5":"3240"},{"1":"17598","2":"1208","3":"TargetScan_microRNA_2017","4":"http://www.targetscan.org/","5":"683"},{"1":"5902","2":"109","3":"Enrichr_Libraries_Most_Popular_Genes","4":"http://amp.pharm.mssm.edu/Enrichr","5":"121"},{"1":"12486","2":"299","3":"Enrichr_Submissions_TF-Gene_Coocurrence","4":"http://amp.pharm.mssm.edu/Enrichr","5":"1722"},{"1":"1073","2":"100","3":"Data_Acquisition_Method_Most_Popular_Genes","4":"http://amp.pharm.mssm.edu/Enrichr","5":"12"},{"1":"19513","2":"117","3":"DSigDB","4":"http://tanlab.ucdenver.edu/DSigDB/DSigDBv1.0/","5":"4026"},{"1":"14433","2":"36","3":"GO_Biological_Process_2018","4":"http://www.geneontology.org/","5":"5103"},{"1":"8655","2":"61","3":"GO_Cellular_Component_2018","4":"http://www.geneontology.org/","5":"446"},{"1":"11459","2":"39","3":"GO_Molecular_Function_2018","4":"http://www.geneontology.org/","5":"1151"},{"1":"19741","2":"270","3":"TF_Perturbations_Followed_by_Expression","4":"http://www.ncbi.nlm.nih.gov/geo/","5":"1958"},{"1":"27360","2":"802","3":"Chromosome_Location_hg19","4":"http://hgdownload.cse.ucsc.edu/downloads.html","5":"36"},{"1":"13072","2":"26","3":"NIH_Funded_PIs_2017_Human_GeneRIF","4":"https://www.ncbi.nlm.nih.gov/pubmed/","5":"5687"},{"1":"13464","2":"45","3":"NIH_Funded_PIs_2017_Human_AutoRIF","4":"https://www.ncbi.nlm.nih.gov/pubmed/","5":"12558"},{"1":"13787","2":"200","3":"Rare_Diseases_AutoRIF_ARCHS4_Predictions","4":"https://amp.pharm.mssm.edu/geneshot/","5":"3725"},{"1":"13929","2":"200","3":"Rare_Diseases_GeneRIF_ARCHS4_Predictions","4":"https://www.ncbi.nlm.nih.gov/gene/about-generif","5":"2244"},{"1":"16964","2":"200","3":"NIH_Funded_PIs_2017_AutoRIF_ARCHS4_Predictions","4":"https://www.ncbi.nlm.nih.gov/pubmed/","5":"12558"},{"1":"17258","2":"200","3":"NIH_Funded_PIs_2017_GeneRIF_ARCHS4_Predictions","4":"https://www.ncbi.nlm.nih.gov/pubmed/","5":"5684"},{"1":"10352","2":"58","3":"Rare_Diseases_GeneRIF_Gene_Lists","4":"https://www.ncbi.nlm.nih.gov/gene/about-generif","5":"2244"},{"1":"10471","2":"76","3":"Rare_Diseases_AutoRIF_Gene_Lists","4":"https://amp.pharm.mssm.edu/geneshot/","5":"3725"},{"1":"12419","2":"491","3":"SubCell_BarCode","4":"http://www.subcellbarcode.org/","5":"104"},{"1":"19378","2":"37","3":"GWAS_Catalog_2019","4":"https://www.ebi.ac.uk/gwas","5":"1737"},{"1":"6201","2":"45","3":"WikiPathways_2019_Human","4":"https://www.wikipathways.org/","5":"472"},{"1":"4558","2":"54","3":"WikiPathways_2019_Mouse","4":"https://www.wikipathways.org/","5":"176"},{"1":"3264","2":"22","3":"TRRUST_Transcription_Factors_2019","4":"https://www.grnpedia.org/trrust/","5":"571"},{"1":"7802","2":"92","3":"KEGG_2019_Human","4":"https://www.kegg.jp/","5":"308"},{"1":"8551","2":"98","3":"KEGG_2019_Mouse","4":"https://www.kegg.jp/","5":"303"},{"1":"12444","2":"23","3":"InterPro_Domains_2019","4":"https://www.ebi.ac.uk/interpro/","5":"1071"},{"1":"9000","2":"20","3":"Pfam_Domains_2019","4":"https://pfam.xfam.org/","5":"608"},{"1":"7744","2":"363","3":"DepMap_WG_CRISPR_Screens_Broad_CellLines_2019","4":"https://depmap.org/","5":"558"},{"1":"6204","2":"387","3":"DepMap_WG_CRISPR_Screens_Sanger_CellLines_2019","4":"https://depmap.org/","5":"325"},{"1":"13420","2":"32","3":"MGI_Mammalian_Phenotype_Level_4_2019","4":"http://www.informatics.jax.org/","5":"5261"},{"1":"14148","2":"122","3":"UK_Biobank_GWAS_v1","4":"https://www.ukbiobank.ac.uk/tag/gwas/","5":"857"},{"1":"9813","2":"49","3":"BioPlanet_2019","4":"https://tripod.nih.gov/bioplanet/","5":"1510"},{"1":"1397","2":"13","3":"ClinVar_2019","4":"https://www.ncbi.nlm.nih.gov/clinvar/","5":"182"},{"1":"9116","2":"22","3":"PheWeb_2019","4":"http://pheweb.sph.umich.edu/","5":"1161"},{"1":"17464","2":"63","3":"DisGeNET","4":"https://www.disgenet.org","5":"9828"},{"1":"394","2":"73","3":"HMS_LINCS_KinomeScan","4":"http://lincs.hms.harvard.edu/kinomescan/","5":"148"},{"1":"11851","2":"586","3":"CCLE_Proteomics_2020","4":"https://portals.broadinstitute.org/ccle","5":"378"},{"1":"8189","2":"421","3":"ProteomicsDB_2020","4":"https://www.proteomicsdb.org/","5":"913"},{"1":"18704","2":"100","3":"lncHUB_lncRNA_Co-Expression","4":"https://amp.pharm.mssm.edu/lnchub/","5":"3729"},{"1":"5605","2":"39","3":"Virus-Host_PPI_P-HIPSTer_2020","4":"http://phipster.org/","5":"6715"},{"1":"5718","2":"31","3":"Elsevier_Pathway_Collection","4":"http://www.transgene.ru/disease-pathways/","5":"1721"},{"1":"14156","2":"40","3":"Table_Mining_of_CRISPR_Studies","4":"","5":"802"},{"1":"16979","2":"295","3":"COVID-19_Related_Gene_Sets","4":"https://amp.pharm.mssm.edu/covid19","5":"205"},{"1":"4383","2":"146","3":"MSigDB_Hallmark_2020","4":"https://www.gsea-msigdb.org/gsea/msigdb/collections.jsp","5":"50"},{"1":"54974","2":"483","3":"Enrichr_Users_Contributed_Lists_2020","4":"https://maayanlab.cloud/Enrichr","5":"1482"},{"1":"12118","2":"448","3":"TG_GATES_2020","4":"https://toxico.nibiohn.go.jp/english/","5":"1190"},{"1":"12361","2":"124","3":"Allen_Brain_Atlas_10x_scRNA_2021","4":"https://portal.brain-map.org/","5":"766"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

```r
# Perform enrichment
top_DGE <- DGE_cell_selection$Covid[(DGE_cell_selection$Covid$p.value < 0.01) & (abs(DGE_cell_selection$Covid[, 
    grep("logFC", colnames(DGE_cell_selection$Covid))]) > 0.25), ]

enrich_results <- enrichr(genes = rownames(top_DGE), databases = "GO_Biological_Process_2017b")[[1]]
```

```
## Uploading data to Enrichr... Done.
##   Querying GO_Biological_Process_2017b... Done.
## Parsing results... Done.
```


Some databases of interest:

* `GO_Biological_Process_2017b`
* `KEGG_2019_Human`
* `KEGG_2019_Mouse`
* `WikiPathways_2019_Human`
* `WikiPathways_2019_Mouse`

You visualize your results using a simple barplot, for example:


```r
par(mfrow = c(1, 1), mar = c(3, 25, 2, 1))
barplot(height = -log10(enrich_results$P.value)[10:1], names.arg = enrich_results$Term[10:1], 
    horiz = TRUE, las = 1, border = FALSE, cex.names = 0.6)
abline(v = c(-log10(0.05)), lty = 2)
abline(v = 0, lty = 1)
```

![](scater_05_dge_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

Gene Set Enrichment Analysis (GSEA)

Besides the enrichment using hypergeometric test, we can also perform gene set enrichment analysis (GSEA), which scores ranked genes list (usually based on fold changes) and computes permutation test to check if a particular gene set is more present in the Up-regulated genes, amongthe DOWN_regulated genes or not differentially regulated.


```r
# Create a gene rank based on the gene expression fold change
gene_rank <- setNames(DGE_cell_selection$Covid[, grep("logFC", colnames(DGE_cell_selection$Covid))], 
    casefold(rownames(DGE_cell_selection$Covid), upper = T))
```

 Once our list of genes are sorted, we can proceed with the enrichment itself. We can use the package to get gene set from the Molecular Signature Database (MSigDB) and select KEGG pathways as an example.


```r
# install.packages('msigdbr')
library(msigdbr)

# Download gene sets
msigdbgmt <- msigdbr::msigdbr("Homo sapiens")
msigdbgmt <- as.data.frame(msigdbgmt)

# List available gene sets
unique(msigdbgmt$gs_subcat)
```

```
##  [1] "MIR:MIR_Legacy"  "TFT:TFT_Legacy"  "CGP"             "TFT:GTRD"       
##  [5] ""                "CP:BIOCARTA"     "CGN"             "GO:MF"          
##  [9] "GO:BP"           "GO:CC"           "HPO"             "CP:KEGG"        
## [13] "MIR:MIRDB"       "CM"              "CP"              "CP:PID"         
## [17] "CP:REACTOME"     "CP:WIKIPATHWAYS"
```

```r
# Subset which gene set you want to use.
msigdbgmt_subset <- msigdbgmt[msigdbgmt$gs_subcat == "CP:WIKIPATHWAYS", ]
gmt <- lapply(unique(msigdbgmt_subset$gs_name), function(x) {
    msigdbgmt_subset[msigdbgmt_subset$gs_name == x, "gene_symbol"]
})
names(gmt) <- unique(paste0(msigdbgmt_subset$gs_name, "_", msigdbgmt_subset$gs_exact_source))
```

 Next, we will be using the GSEA. This will result in a table containing information for several pathways. We can then sort and filter those pathways to visualize only the top ones. You can select/filter them by either `p-value` or normalized enrichemnet score (`NES`).


```r
library(fgsea)

# Perform enrichemnt analysis
fgseaRes <- fgsea(pathways = gmt, stats = gene_rank, minSize = 15, maxSize = 500, 
    nperm = 10000)
fgseaRes <- fgseaRes[order(fgseaRes$NES, decreasing = T), ]

# Filter the results table to show only the top 10 UP or DOWN regulated processes
# (optional)
top10_UP <- fgseaRes$pathway[1:10]

# Nice summary table (shown as a plot)
dev.off()
plotGseaTable(gmt[top10_UP], gene_rank, fgseaRes, gseaParam = 0.5)
```

<style>
div.blue { background-color:#e6f0ff; border-radius: 5px; padding: 10px;}
</style>
<div class = "blue">
**Your turn**

Which KEGG pathways are upregulated in this cluster?
Which KEGG pathways are dowregulated in this cluster?
Change the pathway source to another gene set (e.g. "CP:WIKIPATHWAYS" or "CP:REACTOME" or "CP:BIOCARTA" or "GO:BP") and check the if you get simmilar results?
</div>

Finally, lets save the integrated data for further analysis.



```r
saveRDS(sce, "data/results/covid_qc_dr_int_cl_dge.rds")
```


### Session Info
***


```r
sessionInfo()
```

```
## R version 3.6.1 (2019-07-05)
## Platform: x86_64-conda_cos6-linux-gnu (64-bit)
## Running under: Ubuntu 20.04 LTS
## 
## Matrix products: default
## BLAS/LAPACK: /home/czarnewski/miniconda3/envs/scRNAseq2021/lib/libopenblasp-r0.3.10.so
## 
## locale:
##  [1] LC_CTYPE=C.UTF-8       LC_NUMERIC=C           LC_TIME=C.UTF-8       
##  [4] LC_COLLATE=C.UTF-8     LC_MONETARY=C.UTF-8    LC_MESSAGES=C.UTF-8   
##  [7] LC_PAPER=C.UTF-8       LC_NAME=C              LC_ADDRESS=C          
## [10] LC_TELEPHONE=C         LC_MEASUREMENT=C.UTF-8 LC_IDENTIFICATION=C   
## 
## attached base packages:
## [1] parallel  stats4    stats     graphics  grDevices utils     datasets 
## [8] methods   base     
## 
## other attached packages:
##  [1] fgsea_1.12.0                Rcpp_1.0.6                 
##  [3] msigdbr_7.2.1               enrichR_2.1                
##  [5] dplyr_1.0.3                 igraph_1.2.6               
##  [7] pheatmap_1.0.12             cowplot_1.1.1              
##  [9] scran_1.14.1                scater_1.14.0              
## [11] ggplot2_3.3.3               SingleCellExperiment_1.8.0 
## [13] SummarizedExperiment_1.16.0 DelayedArray_0.12.0        
## [15] BiocParallel_1.20.0         matrixStats_0.57.0         
## [17] Biobase_2.46.0              GenomicRanges_1.38.0       
## [19] GenomeInfoDb_1.22.0         IRanges_2.20.0             
## [21] S4Vectors_0.24.0            BiocGenerics_0.32.0        
## [23] RJSONIO_1.3-1.4             optparse_1.6.6             
## 
## loaded via a namespace (and not attached):
##  [1] bitops_1.0-6             RColorBrewer_1.1-2       httr_1.4.2              
##  [4] tools_3.6.1              R6_2.5.0                 irlba_2.3.3             
##  [7] vipor_0.4.5              DBI_1.1.1                colorspace_2.0-0        
## [10] withr_2.4.0              tidyselect_1.1.0         gridExtra_2.3           
## [13] curl_4.3                 compiler_3.6.1           BiocNeighbors_1.4.0     
## [16] formatR_1.7              labeling_0.4.2           scales_1.1.1            
## [19] stringr_1.4.0            digest_0.6.27            rmarkdown_2.6           
## [22] XVector_0.26.0           pkgconfig_2.0.3          htmltools_0.5.1         
## [25] limma_3.42.0             rlang_0.4.10             DelayedMatrixStats_1.8.0
## [28] generics_0.1.0           farver_2.0.3             jsonlite_1.7.2          
## [31] RCurl_1.98-1.2           magrittr_2.0.1           BiocSingular_1.2.0      
## [34] GenomeInfoDbData_1.2.2   Matrix_1.3-2             ggbeeswarm_0.6.0        
## [37] munsell_0.5.0            viridis_0.5.1            lifecycle_0.2.0         
## [40] stringi_1.5.3            yaml_2.2.1               edgeR_3.28.0            
## [43] zlibbioc_1.32.0          grid_3.6.1               dqrng_0.2.1             
## [46] crayon_1.3.4             lattice_0.20-41          locfit_1.5-9.4          
## [49] knitr_1.30               pillar_1.4.7             rjson_0.2.20            
## [52] fastmatch_1.1-0          glue_1.4.2               evaluate_0.14           
## [55] data.table_1.13.6        vctrs_0.3.6              gtable_0.3.0            
## [58] getopt_1.20.3            purrr_0.3.4              assertthat_0.2.1        
## [61] xfun_0.20                rsvd_1.0.3               viridisLite_0.3.0       
## [64] tibble_3.0.5             beeswarm_0.2.3           statmod_1.4.35          
## [67] ellipsis_0.3.1
```



















