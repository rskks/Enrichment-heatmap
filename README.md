---
title: "Index"
author: "Oleg"
date: '2022-06-07'
output: html_document
---
## Plotting Enrichment/Revigo results

Input files are .csv that have enrichment results after redundancy removal.

Data:

Biological Processes: ./ncbbp.csv

Molecular Functions: ./ncbmf.csv

Cellular Processes: ./ncbcc.csv

```{r}
 suppressPackageStartupMessages({
    library(curl)       # read file from google drive
    library(gplots)     # heatmap.2
    library(dendextend) # make and color dendrogram
    library(colorspace) # diverge_hcl / rainbow_hcl / heat_hcl color palettes
})
my_palette <- colorRampPalette(c("orange1", "orangered", "orangered3"))(n = 299)

# import csv with duplicate sets of data, one with NA values for missing terms, one with 0 for missing terms
# cluster matrix based on data with zeroes, and plot NA values data with it 
mf <- read.csv ('./mfncb.csv', sep=";")
mf <- mf[order(mf$GO),]
mf1 <- mf[,2:13]
mf2 <- mf[,14:25]
mfmat1 <- data.matrix(mf1)
mfmat2 <- data.matrix(mf2)
row.names(mfmat1) <- mf$GO
mfmat1[mfmat1 == 1] <- NA
F_m2 <-(apply(mfmat2, 2, function(x){log2(x/median(x, na.rm = T))}))

dimnames(F_m2)[1] <- list(mf$GO)
dend1 <- as.dendrogram(hclust(dist(F_m2)))
c_group <- 4 # number of clusters
dend1 <- color_branches(dend1, k = c_group, col = rainbow_hcl) # add color to the lines
dend1 <- color_labels(dend1, k = c_group, col = rainbow_hcl)   # add color to the labels

# reorder the dendrogram, must incl. agglo.FUN = mean
rMeans <- rowMeans(F_m2, na.rm = T)
dend1 <- reorder(dend1, rowMeans(F_m2, na.rm = T), agglo.FUN = mean)

# get the color of the leaves (labels) for heatmap.2
col_labels <- get_leaves_branches_col(dend1)
col_labels <- col_labels[order(order.dendrogram(dend1))]

# if plot the dendrogram alone:
# the size of the labels:
dend1 <- set(dend1, "labels_cex", 0.8)
par(mar = c(1,1,1,1))
plot_horiz.dendrogram(dend1) # use side = T to horiz mirror if needed
heatmap.2(scale(mfmat1), main = 'GO: Molecular Function',
          # reorderfun=function(d, w) reorder(d, w, agglo.FUN = mean),
          # order by branch mean so the deepest color is at the top
          dendrogram = "row",        # no dendrogram for columns
          Rowv = dend1,              # * use self-made dendrogram
          Colv = "NA",               # make sure the columns follow data's order
          col = my_palette,          # color pattern of the heatmap
          na.color = "ivory3",       # color for NA values 
          trace="none",              # hide trace
          density.info="none",       # hide histogram
          
          margins = c(5,15),         # margin on top(bottom) and left(right) side.
          cexRow=1, cexCol = 1,      # size of row / column labels
          srtCol=0, adjCol = c(0.5,1), # adjust the direction of row label to be horizontal
          # margin for the color key
          # ("bottom.margin", "left.margin", "top.margin", "left.margin" )
          key.par=list(mar=c(5,1,3,1)),
          RowSideColors = col_labels, # to add nice colored strips        
          colRow = col_labels         # add color to label
)
