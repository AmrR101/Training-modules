---
title: "Functional Analysis for RNA-seq"
author: "Mary Piper"
date: "Wednesday, August 30, 2017"
---

Approximate time: 105 minutes

Learning Objectives:
-------------------

*  Determine how functions are attributed to genes using Gene Ontology terms
*  Understand the theory of how functional enrichment tools yield statistically enriched functions or interactions
*  Discuss functional analysis using over-representation analysis, functional class scoring, and pathway topology methods
*  Explore functional analysis tools

# Functional analysis 

The output of RNA-Seq differential expression analysis is a list of significant differentially expressed genes (DEGs). To gain greater biological insight on the DEGs there are various analyses that can be done:

- determine whether there is enrichment of known biological functions, interactions, or pathways
- identify genes' involvement in novel pathways or networks by grouping genes together based on similar trends
- use global changes in gene expression by visualizing all genes being significantly up- or down-regulated in the context of external interaction data

Generally for any differential expression analysis, it is useful to interpret the resulting gene lists using freely available web- and R-based tools.  While tools for functional analysis span a wide variety of techniques, they can loosely be categorized into three main types: over-representation analysis, functional class scoring, and pathway topology [[1](../../resources/pathway_tools.pdf)]. 

![Pathway analysis tools](../img/pathway_analysis.png)

## Dataset

To interpret the results of our functional analysis, it is necessary to understand our dataset. We will be using the output from the differential expression analysis of a real RNA-Seq dataset that is part of a larger study described in [Kenny PJ et al, Cell Rep 2014](http://www.ncbi.nlm.nih.gov/pubmed/25464849). 

The goal of the study was to investigate the interactions between various genes involved in Fragile X syndrome, a disease in which there is aberrant production of the FMRP protein that results in cognitive impairment and autistic-like features.. 

> **FMRP** is “most commonly found in the brain and is essential for normal cognitive development and female reproductive function. Mutations of this gene can lead to fragile X syndrome, mental retardation, premature ovarian failure, autism, Parkinson's disease, developmental delays and other cognitive deficits.” - from [wikipedia](https://en.wikipedia.org/wiki/FMR1)

> **MOV10**, is a putative RNA helicase that is also associated with **FMRP** in the context of the microRNA pathway. 

**The hypothesis tested by [the paper](http://www.ncbi.nlm.nih.gov/pubmed/25464849) is that FMRP and MOV10 associate and regulate the translation of a subset of RNAs.**

<img src="../img/mov10-model.png" width="400">

The data we will be working with is the differential expression results for samples overexpressing the MOV10 gene versus control samples. **Based on the authors' hypothesis, we may expect the enrichment of processes / pathways related to *translation, splicing, and the regulation of mRNAs*.**

Let's open RStudio and create a new project directory for our "Functional Analysis" lesson. 

1. Open RStudio
2. Go to the `File` menu and select `New Project`.
3. In the `New Project` window, choose `New Directory`. Then, choose `Empty Project`. Name your new directory `Functional_analysis` and then "Create the project as subdirectory of:" the Desktop (or location of your choice).
4. Click on `Create Project`.
5. After your project is completed, if the project does not automatically open in RStudio, then go to the `File` menu, select `Open Project`, and choose `Functional_analysis.Rproj`.
6. When RStudio opens, you will see three panels in the window.
7. Go to the `File` menu and select `New File`, and select `R Script`. The RStudio interface should now look like the screenshot below.
8. Let's create `data` and `results` directories within your working directory by clicking on `New Folder` within the `Files` tab. 
9. Download the **differential expression results** to the `data` directory by right clicking on [this link](https://github.com/hbctraining/Training-modules/blob/master/DGE-functional-analysis/data/Mov10oe_DE_results.csv?raw=true)

If you right click on the link, and "Save link as..". Choose `~/Desktop/Functional_analysis/data` as the destination of the file. You should now see the `Mov10oe_DE_results.csv` file appear in your working directory. 

## Over-representation analysis
The first main category of functional analysis tools is the over-representation analysis, which explores whether there is enrichment of known biological functions in a particular set of genes (e.g. significant DE genes). There are a plethora of functional enrichment tools that perform some type of over-representation analysis by querying databases containing information about gene function and interactions. **Querying these databases for gene function requires the use of a _consistent vocabulary_ to describe gene function.** One of the most widely-used vocabularies is the **Gene Ontology (GO)**. This vocabulary was established by the Gene Ontology project, and the words in the vocabulary are referred to as GO terms. 

### Gene Ontology project

"The Gene Ontology project is a collaborative effort to address the need for consistent descriptions of gene products across databases" [[2](geneontology.org/page/documentation)]. The [Gene Ontology Consortium](http://geneontology.org/page/go-consortium-contributors-list) maintains the GO terms, and these GO terms are incorporated into gene annotations in many of the popular repositories for animal, plant, and microbial genomes. 

Tools that investigate **enrichment of biological functions or interactions** can query these databases for GO terms associated with a list of genes to determine whether any GO terms associated with particular functions or interactions are enriched in the gene set. Therefore, to best use and interpret the results from these functional analysis tools, it is helpful to have a good understanding of the GO terms themselves.

### GO terms

#### GO Ontologies

To describe the roles of genes and gene products, GO terms are organized into three independent controlled vocabularies (ontologies) in a species-independent manner: 

- **Biological process:** refers to the biological role involving the gene or gene product, and could include "transcription", "signal transduction", and "apoptosis". A biological process generally involves a chemical or physical change of the starting material or input.
- **Molecular function:** represents the biochemical activity of the gene product, such activities could include "ligand", "GTPase", and "transporter". 
- **Cellular component:** refers to the location in the cell of the gene product. Cellular components could include "nucleus", "lysosome", and "plasma membrane".

Each GO term has a term name (e.g. DNA repair) and a unique term accession number (GO:0005125), and a single gene product can be associated with many GO terms, since a single gene product "may function in several processes, contain domains that carry out diverse molecular functions, and participate in multiple alternative interactions with other proteins, organelles or locations in the cell" [[3](go.pdf)]. 

#### GO term hierarchy

Some gene products are well-researched, with vast quantities of data available regarding their biological processes and functions. However, other gene products have very little data available about their roles in the cell. 

For example, the protein, "p53", would contain a wealth of information on it's roles in the cell, whereas another protein might only be known as a "membrane-bound protein" with no other information available. 

The GO ontologies were developed to describe and query biological knowledge with differing levels of information available. To do this, GO ontologies are loosely hierarchical, ranging from general, 'parent', terms to more specific, 'child' terms. The GO ontologies are "loosely" hierarchical since 'child' terms can have multiple 'parent' terms.

Some genes with less information may only be associated with general 'parent' terms or no terms at all, while other genes with a lot of information be associated with many terms.

![Nature Reviews Cancer 7, 23-34 (January 2007)](../img/go_heirarchy.jpg)

[Tips for working with GO terms](http://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1003343)

### Hypergeometric testing

In a set of genes, the frequency of GO terms can be determined, and the comparison of frequencies between a gene list & a “background” set will inform us about the over- or under-representation of the GO terms. This type of testing can inform us about over- or under-representation of other entities such as *particular motifs or pathways* too.

![go_frequencies](../img/go_freq.png)

To determine whether GO terms (or motifs and pathways) are over- or under-represented, you can determine the **probability of having the observed proportion of genes associated with a specific GO term in your gene list based on the proportion of genes associated with the same GO term in the background set**. The background dataset can be all genes in genome for your organism or you can select your own background to use.

For example, let's suppose there are 13,000 total genes in the honeybee genome and 85 genes are associated with the GO term "DNA repair". In your gene list, there are 50 genes associated with "DNA repair" out of 1,000 genes in gene list. 

By comparing the ratios, 85/13,000 in "background" dataset and 50/1,000 in your gene list, it's evident that the GO term "DNA repair" is over-represented in your dataset.

To determine whether a GO term or pathway is *significantly* over- or under-represented, tools often perform **hypergeometric testing**. Using our honeybee example, the hypergeometric distribution is a probability distribution that describes the probability of 50 genes (k) being associated with "DNA repair", for all genes in our gene list (n=1,000), from a population of all of the genes in entire genome (N=13,000) which contains 85 genes (K) associated with "DNA repair" [[4](https://en.wikipedia.org/wiki/Hypergeometric_distribution)].

The calculation of probability of k successes follows the formula:

![hypergeo](../img/hypergeo.png) 

## clusterProfiler
[clusterProfiler](http://bioconductor.org/packages/release/bioc/html/clusterProfiler.html) performs over-representation analysis on GO terms associated with a list of genes. The tool takes as input a significant gene list and a background gene list and performs statistical enrichment analysis using hypergeometric testing. The basic arguments allow the user to select the appropriate organism and GO ontology (BP, CC, MF) to test. 

### Running clusterProfiler

We first need to load the libraries and have a dataframe of all of the DE results:

```r
# Load libraries
library(clusterProfiler)
library(DOSE)
library(org.Hs.eg.db)
library(biomaRt)

# Create dataframe of all results

all_OE <- data.frame(res_tableOE) 
```

To run clusterProfiler GO over-represenation analysis, we will change our gene names into Ensembl IDs, since the tool works a bit easier with the Ensembl IDs. There are a few clusterProfiler functions that allow us to map between gene IDs:

```r
# clusterProfiler does not work as easily using gene names, so turning gene names into Ensembl IDs using clusterProfiler::bitr and merge the IDs back with the DE results

keytypes(org.Hs.eg.db)
ids <- bitr(rownames(all_OE), fromType = "SYMBOL", toType = "ENSEMBL", OrgDb = "org.Hs.eg.db")

# The gene names can map to more than one Ensembl ID (some genes change ID over time), so we need to remove duplicate IDs prior to assessing enriched GO terms

ids <- ids[which(duplicated(ids$SYMBOL) == F), ] 

# Merge the Ensembl IDs with the results
        
merged_genes_ensembl <- merge(all_OE, ids, by.x="row.names", by.y="SYMBOL")             
                
sigOE <- subset(merged_genes_ensembl, padj < 0.05)

sigOE_genes <- as.character(sigOE$ENSEMBL)

```

We will use the Ensembl IDs for all genes as the background dataset:

```r
# Create background dataset for hypergeometric testing using all genes tested for significance in the results
                   
allOE_genes <- as.character(merged_genes_ensembl$ENSEMBL)
```

Now we can perform the GO enrichment analysis:

```r
# Run GO enrichment analysis 
ego <- enrichGO(gene = sigOE_genes, 
                    universe = allOE_genes, 
                    keytype = "ENSEMBL", 
                    OrgDb = org.Hs.eg.db, 
                    ont = "BP", 
                    pAdjustMethod = "BH", 
                    qvalueCutoff = 0.05, 
                    readable = TRUE)

# Output results from GO analysis to a table
cluster_summary <- data.frame(ego)

write.csv(cluster_summary, "results/clusterProfiler_Mov10oe.csv")
```

![cluster_summary](../img/cluster_summary.png)

### Visualizing clusterProfiler results
clusterProfiler has a variety of options for viewing the over-represented GO terms. We will explore the dotplot, enrichment plot, and the category netplot.

The dotplot shows the number of genes associated with the first 50 terms (size) and the p-adjusted values for these terms (color). 

```r
dotplot(ego, showCategory=50)
```

<img src="../img/mov10oe_dotplot.png" width="600">

The enrichment GO plot below shows the relationship between the top 50 most significantly enriched GO terms, by grouping similar terms together. The color represents the p-values relative to the other displayed terms (brighter red is more significant) and the size of the terms represents the number of genes that are significant from our list.

```r
enrichMap(ego, n=50, vertex.label.font=6)
```

**To save the figure,** click on the `Export` button in the RStudio `Plots` tab and `Save as PDF...`. In the pop-up window, change the `PDF size` to `24 x 32` to give a figure of appropriate size for the text labels.

<img src="../img/mov10oe_enrichmap.png" width="800">

Finally, the category netplot shows the relationships between the genes associated with the top five most significant GO terms and the fold changes of the significant genes associated with these terms (color). The size of the GO terms reflects the pvalues of the terms, with the more significant terms being larger. This plot is particularly useful for hypothesis generation in identifying genes that may be important to several of the most affected processes. 

```r
# To color genes by log2 fold changes, we need to extract the log2 fold changes from our results table creating a named vector
OE_foldchanges <- sigOE$log2FoldChange

names(OE_foldchanges) <- sigOE$Row.names

cnetplot(ego, categorySize="pvalue", showCategory = 5, foldChange=OE_foldchanges, vertex.label.font=6)
```

**Again, to save the figure,** click on the `Export` button in the RStudio `Plots` tab and `Save as PDF...`. Change the `PDF size` to `24 x 32` to give a figure of appropriate size for the text labels.

<img src="../img/mov10oe_cnetplot.png" width="800">

If you are interested in significant processes that are **not** among the top five, you can subset your `ego` dataset to only display these processes:

```r
ego2 <- ego
ego2@result <- ego@result[c(1,3,4,8,9),]
cnetplot(ego2, categorySize="pvalue", foldChange=OE_foldchanges, showCategory = 5)
```

<img src="../img/mov10oe_cnetplot2.png" width="800">

### gProfiler

[gProfileR](http://biit.cs.ut.ee/gprofiler/index.cgi) is a tool for the interpretation of large gene lists which can be run using a web interface or through R. The core tool takes a gene list as input and performs statistical enrichment analysis using hypergeometric testing similar to clusterProfiler. Multiple sources of functional evidence are considered, including Gene Ontology terms, biological pathways, regulatory motifs of transcription factors and microRNAs, human disease annotations and protein-protein interactions. The user selects the organism and the sources of evidence to test. There are also additional parameters to change various thresholds and tweak the stringency to the desired level. 

The GO terms output by gprofileR are generally quite similar to those output by clusterProfiler, but there are small differences due to the different algorithms used by the programs.

![gprofiler](../img/gProfiler.png)

You can use gProfiler for a wide selection of organisms, and the tool accepts your gene list as input. If your gene list is ordered (e.g. by padj. values), then gProfiler will take the order of the genes into account when outputting enriched terms or pathways.

In addition, a large number (70%) of the functional annotations of GO terms are determined using _in silico_ methods to infer function from electronic annotation (IEA). While these annotations can offer valuable information, the information is of lower confidence than experimental and computational studies, and these functional annotations can be easily filtered out. 

The color codes in the gProfiler output represent the quality of the evidence for the functional annotation. For example, weaker evidence is depicted in blue, while strong evidence generated by direct experiment is shown with red or orange. Similar coloring is used for pathway information, with well-researched pathway information shown in black, opposed to lighter colors. Grey coloring suggests an unknown gene product or annotation. For more information, please see the [gProfiler paper](http://www.ncbi.nlm.nih.gov/pmc/articles/PMC1933153/).

Also, due to the hierarchical structure of GO terms, you may return many terms that seem redundant since they are child and parent terms. gProfiler allows for 'hierarchical filtering', returning only the best term per parent term.

We encourage you to explore gProfiler online, for today's class we will be demonstrating how to run it using the R package.

#### Running gProfiler

We can run gProfileR relatively easily from R, by loading the library and running the  `gprofiler` function.

```r
### Functional analysis of MOV10 Overexpression using gProfileR (some of these are defaults; check help pages) 

library(gProfileR)

# Running gprofiler to identify enriched processes among significant genes

gprofiler_results_oe <- gprofiler(query = sigOE_genes, 
                                  organism = "hsapiens",
                                  ordered_query = F, 
                                  exclude_iea = F, 
                                  max_p_value = 0.05, 
                                  max_set_size = 0,
                                  correction_method = "fdr",
                                  hier_filtering = "none", 
                                  domain_size = "annotated",
                                  custom_bg = all_genes)

```

Let's save the gProfiler results to file:

```r
## Order the results by p-adjusted value and write results to file

gprofiler_results_oe_reordered <- gprofiler_results_oe[, c("term.id", "domain", "term.name", "p.value", "overlap.size", "term.size", "intersection")]

gprofiler_results_oe_reordered <- gprofiler_results_oe_reordered[order(gprofiler_results_oe_reordered$p.value), ]

gprofiler_results_oe_GOs <- gprofiler_results_oe_reordered[grep('GO:', gprofiler_results_oe_reordered$term.id)]

write.csv(gprofiler_results_oe_GOs, 
            "results/gprofiler_MOV10_oe.csv")
```

Now, extract only those lines in the gProfiler results with GO term accession numbers for downstream analyses:

```r
## Extract only GO IDs for downstream analysis

GOs_oe <- gprofiler_results_oe_GOs$term.id

write(GOs_oe, "results/GOs_oe.txt", ncol = 1)
```

### REVIGO

[REVIGO](http://revigo.irb.hr/) is a web-based tool that can take our list of GO terms, collapse redundant terms by semantic similarity, and summarize them graphically. 

![REVIGO_input](../img/revigo_input.png)

Open `GOs_oe.txt` and copy and paste the GO ids into the REVIGO search box, and submit.

![REVIGO_output](../img/revigo_output.png)

***gProfiler and REVIGO are great tools to validate experimental results and to make hypotheses. These tools suggest pathways that may be involved with your condition of interest; you should NOT use these tools to make conclusions about the pathways involved in your experimental process.***

J. Reimand, T. Arak, P. Adler, L. Kolberg, S. Reisberg, H. Peterson, J. Vilo. g:Profiler -- a web server for functional interpretation of gene lists (2016 update). Nucleic Acids Research 2016; doi: 10.1093/nar/gkw199

Supek F, Bošnjak M, Škunca N, Šmuc T. REVIGO summarizes and visualizes long lists of Gene Ontology terms. PLoS ONE 2011. doi:10.1371/journal.pone.0021800

## [Other functional analysis methods](https://github.com/hbctraining/Training-modules/blob/master/DGE-functional-analysis/lessons/02_functional_analysis_other_methods.md)

Over-representation analyses are only a single type of functional analysis method that is available for teasing apart the biological processes important to your condition of interest. Other types of analyses can be equally important or informative, including functional class scoring and pathway topology methods. Functional class scoring methods most often take as input the foldchanges for all genes, then look to see whether gene sets for particular biological processes are enriched among the high or low fold changes. This type of analysis can be particularly helpful if differential expression analysis only output a small list of significant DE genes. Finally, pathway topology analysis often takes into account both fold changes and adjusted p-values to identify dysregulated pathways and outputs whether pathways are inhibited/activated. We have [materials](https://github.com/hbctraining/Training-modules/blob/master/DGE-functional-analysis/lessons/02_functional_analysis_other_methods.md) to lead you through these other types of functional analyses, and we encourage you to take the time to work through them.

![Pathway analysis tools](../img/pathway_analysis.png)

## Resources for functional analysis

* g:Profiler - http://biit.cs.ut.ee/gprofiler/index.cgi 
* DAVID - http://david.abcc.ncifcrf.gov/tools.jsp 
* clusterProfiler - http://bioconductor.org/packages/release/bioc/html/clusterProfiler.html
* GeneMANIA - http://www.genemania.org/
* GenePattern -  http://www.broadinstitute.org/cancer/software/genepattern/ (need to register)
* WebGestalt - http://bioinfo.vanderbilt.edu/webgestalt/ (need to register)
* AmiGO - http://amigo.geneontology.org/amigo
* ReviGO (visualizing GO analysis, input is GO terms) - http://revigo.irb.hr/ 
* WGCNA - http://www.genetics.ucla.edu/labs/horvath/CoexpressionNetwork
* GSEA - http://software.broadinstitute.org/gsea/index.jsp
* SPIA - https://www.bioconductor.org/packages/release/bioc/html/SPIA.html
* GAGE/Pathview - http://www.bioconductor.org/packages/release/bioc/html/gage.html

***
*This lesson has been developed by members of the teaching team at the [Harvard Chan Bioinformatics Core (HBC)](http://bioinformatics.sph.harvard.edu/). These are open access materials distributed under the terms of the [Creative Commons Attribution license](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0), which permits unrestricted use, distribution, and reproduction in any medium, provided the original author and source are credited.*
