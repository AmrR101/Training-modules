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

To run clusterProfiler GO over-representation analysis, we will change our gene names into Ensembl IDs, since the tool works a bit easier with the Ensembl IDs. There are a few clusterProfiler functions that allow us to map between gene IDs:

```r
## clusterProfiler does not work as easily using gene names, so turning gene names 
## into Ensembl IDs using clusterProfiler::bitr and merge the IDs back with the DE results
keytypes(org.Hs.eg.db)
ids <- bitr(rownames(res_tableOE), 
            fromType = "SYMBOL", 
            toType = c("ENSEMBL", "ENTREZID"), 
            OrgDb = "org.Hs.eg.db")

## The gene names can map to more than one Ensembl ID (some genes change ID over time), 
## so we need to remove duplicate IDs prior to assessing enriched GO terms
non_duplicates <- which(duplicated(ids$SYMBOL) == FALSE)

ids <- ids[non_duplicates, ] 

## Merge the Ensembl IDs with the results     
merged_genes_ensembl <- merge(x=res_tableOE, y=ids, by.x="row.names", by.y="SYMBOL")             
                
sigOE <- subset(merged_genes_ensembl, padj < 0.05)

sigOE_genes <- as.character(sigOE$ENSEMBL)

```

We will use the Ensembl IDs for all genes as the background dataset:

```r
## Create background dataset for hypergeometric testing using all genes tested for significance in the results                 
allOE_genes <- as.character(merged_genes_ensembl$ENSEMBL)
```

Now we can perform the GO enrichment analysis:

```r
## Run GO enrichment analysis 
ego <- enrichGO(gene = sigOE_genes, 
                    universe = allOE_genes, 
                    keytype = "ENSEMBL", 
                    OrgDb = org.Hs.eg.db, 
                    ont = "BP", 
                    pAdjustMethod = "BH", 
                    qvalueCutoff = 0.05, 
                    readable = TRUE)

## Output results from GO analysis to a table
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
## To color genes by log2 fold changes, we need to extract the log2 fold changes from our results table creating a named vector
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

## Running gprofiler to identify enriched processes among significant genes

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

Over-representation analyses are only a single type of functional analysis method that is available for teasing apart the biological processes important to your condition of interest. Other types of analyses can be equally important or informative, including functional class scoring and pathway topology methods. 

# Functional class scoring tools
Functional class scoring (FCS) tools, such as [GSEA](http://software.broadinstitute.org/gsea/index.jsp), most often use the gene-level statistics or log2 fold changes for all genes from the differential expression results, then look to see whether gene sets for particular biological pathways are enriched among the large positive or negative fold changes. 

<img src="../img/gsea_theory.png" width="600">

The hypothesis of FCS methods is that although large changes in individual genes can have significant effects on pathways (and will be detected via ORA methods), weaker but coordinated changes in sets of functionally related genes (i.e., pathways) can also have significant effects.  Thus, rather than setting an arbitrary threshold to identify 'significant genes', **all genes are considered** in the analysis. The gene-level statistics from the dataset are aggregated to generate a single pathway-level statistic and statistical significance of each pathway is reported. This type of analysis can be particularly helpful if the differential expression analysis only outputs a small list of significant DE genes. 

### Gene set enrichment analysis using clusterProfiler and Pathview

Using the log2 fold changes obtained from the DESeq2 analysis for every gene, gene set enrichment analysis and pathway analysis can be performed using clusterProfiler and Pathview tools.

For gene set or pathway analysis using clusterProfiler, coordinated differential expression over gene sets is tested instead of changes of individual genes. "Gene sets are pre-defined groups of genes, which are functionally related. Commonly used gene sets include those derived from KEGG pathways, Gene Ontology terms, MSigDB, Reactome, or gene groups that share some other functional annotations, etc. Consistent perturbations over such gene sets frequently suggest mechanistic changes" [1]. 

To perform GSEA analysis of KEGG gene sets, clusterProfiler requires the genes to be identified using Entrez IDs for all genes in our results dataset.

```r
library(pathview)
library(clusterProfiler)

# Return all genes with Entrez IDs
all_results_entrez <- getBM(filters = "external_gene_name", 
                   values = rownames(res_tableOE),
                   attributes = c("entrezgene","external_gene_name"),
                   mart = human)
                   
merged_all_results_entrez <- merge(data.frame(res_tableOE), all_results_entrez, by.x="row.names", by.y="external_gene_name") 
```

When performing our analysis, we need to remove the NA values and duplicates (due to gene ID conversion) prior to the analysis:

```r
# Remove any NA values
all_results_gsea <- subset(merged_all_results_entrez, entrezgene != "NA")

# Remove any duplicates
all_results_gsea <- all_results_gsea[which(duplicated(all_results_gsea$Row.names) == F), ]
```
Finally, extract and name the fold changes:

```r
# Extract the ordered foldchanges
foldchanges <- all_results_gsea$log2FoldChange

names(foldchanges) <- all_results_gsea$entrezgene
```

Order the foldchanges in decreasing order:

```r
foldchanges <- sort(foldchanges, T)

head(foldchanges)
```

Perform the GSEA using KEGG gene sets:

```r
# GSEA using gene sets from KEGG pathways
gseaKEGG <- gseKEGG(geneList = foldchanges,
              organism = "hsa",
              nPerm = 1000,
              minGSSize = 120,
              pvalueCutoff = 0.05,
              verbose = FALSE)
              
gseaKEGG_results <- gseaKEGG@result
```

View the enriched pathways:

```r
gseaKEGG_results

write.csv(gseaKEGG_results, "results/gseaOE.csv", quote=F)
```

Use Pathview to integrate the pathway data from clusterProfiler into pathway images:

```r
pathview(gene.data = foldchanges,
              pathway.id = "hsa03040",
              species = "hsa",
              limit = list(gene = 2,
              cpd = 1))
```

>**NOTE:** Printing out Pathview images for all significant pathways can be easily performed as follows:
> ```r
> library(purrr)
> get_kegg_plots <- function(x) {
>    pathview(gene.data = foldchanges, pathway.id = gsea_results$ID[x], species = "mmu", 
>        limit = list(gene = 2, cpd = 1))
> }
>
> map(1:length(gsea_results$ID), get_kegg_plots)
> ```

Instead of exploring enrichment of KEGG gene sets, we can also explore the enrichment of BP Gene Ontology terms: 

```r
# GSEA using gene sets associated with BP Gene Ontology terms
gseaGO <- gseGO(geneList = foldchanges, 
              OrgDb = org.Hs.eg.db, 
              ont = 'BP', 
              nPerm = 1000, 
              minGSSize = 100, 
              maxGSSize = 500, 
              pvalueCutoff = 0.05,
              verbose = FALSE) 

gseaGO_results <- gseaGO@result
gseaplot(gseaGO, geneSetID = 'GO:0007423')
```
There are other gene sets available for GSEA analysis in clusterProfiler (Disease Ontology, Reactome pathways, etc.). In addition, it is possible to supply your own gene set GMT file, such as a GMT for MSigDB using special clusterProfiler functions as shown below:

```r
biocLite("GSEABase")
library(GSEABase)

# Load in GMT file of gene sets (we downloaded from the Broad Institute [website](http://software.broadinstitute.org/gsea/msigdb/collections.jsp) for MSigDB)

c2 <- read.gmt("/data/c2.cp.v6.0.entrez.gmt.txt")

msig <- GSEA(foldchanges, TERM2GENE=c2, verbose=FALSE)

msig_df <- data.frame(msig)
```

### Gene set enrichment analysis using GAGE and Pathview

Gene set enrichment analysis and pathway analysis can be performed using [GAGE (Generally Applicable Gene-set Enrichment for Pathway Analysis)](http://bioconductor.org/packages/release/bioc/html/gage.html) and [Pathview](http://bioconductor.org/packages/release/bioc/html/pathview.html) tools. "GAGE assumes a gene set comes from a different distribution than the background and uses two-sample t-test to account for the gene set specific variance as well as the background variance. The two-sample t-test used by GAGE identifies gene sets with modest but consistent changes in gene expression level."[[2](http://bmcbioinformatics.biomedcentral.com/articles/10.1186/1471-2105-10-161)]

#### Exploring enrichment of KEGG pathways
To get started with GAGE and Pathview analysis, we need to load multiple libraries:

```r
# Install packages if this is your first time using them

source("http://bioconductor.org/biocLite.R") 
biocLite('gage', 'pathview', 'gageData', 'biomaRt', 'org.Hs.eg.db', 'DOSE', 'SPIA') 

# Loading the packages needed for GAGE and Pathview analysis

library(gage)
library(gageData)
library(biomaRt)
library(org.Hs.eg.db)
```

To determine whether pathways in our dataset are enriched, we need to first obtain the gene sets to test:

```r
# Create datasets with KEGG gene sets to test

kegg_human <- kegg.gsets(species = "human", id.type = "kegg")
names(kegg_human)

kegg.gs <- kegg_human$kg.sets[kegg_human$sigmet.idx]
head(kegg.gs)
```

Now that we have our pathways to test, we need to bring in our own data. We will use the log2 fold changes output by DESeq2, `res_tableKD_sorted`, to determine whether particular pathways are enriched. GAGE requires the genes have Entrez IDs, so we will also convert our gene names into Entrez IDs prior to analysis. *If you do not have this file, you can download it using this [link](https://github.com/hbc/NGS_Data_Analysis_Summer2016/raw/master/sessionIII/data/res_tableKD.txt).*

> A useful tutorial for using GAGE and Pathview is available from Stephen Turner on R-bloggers: [http://www.r-bloggers.com/tutorial-rna-seq-differential-expression-pathway-analysis-with-sailfish-deseq2-gage-and-pathview/](http://www.r-bloggers.com/tutorial-rna-seq-differential-expression-pathway-analysis-with-sailfish-deseq2-gage-and-pathview/)

```r
#Set up

## Turn DESeq2 results into a dataframe

DEG <- data.frame(res_tableKD_sorted)

## Tool expects entrez IDs, so use Biomart to acquire IDs for the gene names

### Create database to search

mart <- useDataset("hsapiens_gene_ensembl", 
                   useMart('ENSEMBL_MART_ENSEMBL', 
                           host =  'www.ensembl.org')) 
                           
###List entrez IDs for each gene name

entrez <- getBM(filters= "external_gene_name", 
                attributes= c("external_gene_name", "entrezgene"),
                values= row.names(DEG),
                mart= mart)

## Merge results dataset, DEG, with Entrez IDs dataset

entrez_results <- merge(entrez, DEG, by.x = "external_gene_name", by.y ="row.names")
head(entrez_results, n=15)

entrez_results <- subset(entrez_results, entrezgene != "NA")
head(entrez_results, n=15)

## Extract only the log2FC values

foldchanges <- entrez_results$log2FoldChange
head(foldchanges)

## Name foldchanges vector with corresponding Entrez IDs as needed by GAGE tool

names(foldchanges) <- entrez_results$entrezgene
head(foldchanges)
```
Now the data is ready to run for GAGE analysis. The [GAGE vignette](https://www.bioconductor.org/packages/devel/bioc/vignettes/gage/inst/doc/gage.pdf) provides detailed information on running the analysis. Note that you can run the analysis and look for pathways with genes statistically only up- or down-regulated. Alternatively, you can explore statistically perturbed pathways, which are enriched in genes that may be either up- or down- regulated. For KEGG pathways, looking at both types of pathways could be useful.

```r
# Run GAGE

keggres = gage(foldchanges, gsets=kegg.gs, same.dir=T)
names(keggres)
head(keggres$greater) #Pathways that are up-regulated
head(keggres$less) #Pathways that are down-regulated

# Explore genes that are up-regulated

sel_up <- keggres$greater[, "q.val"] < 0.05 & !is.na(keggres$greater[, "q.val"])
path_ids_up <- rownames(keggres$greater)[sel_up]
path_ids_up

# Get the pathway IDs for the significantly up-regulated pathways

keggresids = substr(path_ids_up, start=1, stop=8)
keggresids
```
Now that we have the IDs for the pathways that are significantly up-regulated in our dataset, we can visualize these pathways and the genes identified from our dataset causing these pathways to be enriched using [Pathview](https://www.bioconductor.org/packages/devel/bioc/vignettes/pathview/inst/doc/pathview.pdf). 

```r
# Run Pathview

## Use Pathview to view significant up-regulated pathways

pathview(gene.data = foldchanges, pathway.id=keggresids, species="human", kegg.dir="results/")
```
![spliceosome](../img/hsa03040.pathview.png)

#### Exploring enrichment of biological processes using GO terms in GAGE
Using the GAGE tool, we can identify significantly enriched gene ontology terms for biological process and molecular function based on the log2 fold changes for all genes. While gProfileR is an overlap statistic analysis tool which uses a threshold (adjusted p<0.05 here) to define which genes are analyzed for GO enrichment, gene set enrichment analysis tools like GAGE use a list of genes (here ranked by logFC) without using a threshold. This allows GAGE to use more information to identify enriched biological processes. The introduction to GSEA goes into more detail about the advantages of this approach: [http://www.ncbi.nlm.nih.gov/pmc/articles/PMC1239896/](http://www.ncbi.nlm.nih.gov/pmc/articles/PMC1239896/).

```r
#Acquire datasets

data(go.sets.hs)
head(names(go.sets.hs))

data(go.subs.hs)
names(go.subs.hs)
head(go.subs.hs$MF)

#Use gage to explore enriched biological processes
#Biological process 

go_bp_sets = go.sets.hs[go.subs.hs$BP]
```

> If we wanted to identify enriched molecular functions we would use the code: `go.sets.hs[go.subs.hs$MF]`


```r
# Run GAGE
go_bp_res = gage(foldchanges, gsets=go_bp_sets, same.dir=T)
class(go_bp_res)
names(go_bp_res)
head(go_bp_res$greater)
go_df_enriched <- data.frame(go_bp_res$greater)

GO_enriched_BP <- subset(go_df_enriched, q.val < 0.05)
GO_enriched_BP

write.table(GO_enriched_BP, "Mov10_GAGE_GO_BP.txt", quote=F)
```

Weijun Luo, Michael Friedman, Kerby Shedden, Kurt Hankenson, and Peter Woolf. GAGE: generally applicable
gene set enrichment for pathway analysis. BMC Bioinformatics, 2009. doi:10.1186/1471-2105-10-161.

Weijun Luo and Cory Brouwer. Pathview: an R/Bioconductor package for pathway-based data integration
and visualization. Bioinformatics, 29(14):1830-1831, 2013. doi: 10.1093/bioinformatics/btt285.


## Pathway topology tools
The last main type of functional analysis technique is pathway topology analysis. Pathway topology analysis often takes into account gene interaction information along with the fold changes and adjusted p-values from differential expression analysis to identify dysregulated pathways. Depending on the tool, pathway topology tools explore how genes interact with each other (e.g. activation, inhibition, phosphorylation, ubiquitination, etc) to determine the pathway-level statistics. Pathway topology-based methods utilize the number and type of interactions between gene product (our DE genes) and other gene products to infer gene function or pathway association. 

### SPIA
The [SPIA (Signaling Pathway Impact Analysis)](http://bioconductor.org/packages/release/bioc/html/SPIA.html) tool can be used to integrate the lists of differentially expressed genes determined by DESeq2, their fold changes, and pathway topology to identify affected pathways. The blog post from [Getting Genetics Done](http://www.gettinggeneticsdone.com/2012/03/pathway-analysis-for-high-throughput.html) provides a step-by-step procedure for using and understanding SPIA. 


```r
# Set-up

## Significant genes is a vector of fold changes where the names are ENTREZ gene IDs. The background set is a vector of all the genes represented on the platform.

## Convert ensembl to entrez ids

entrez_results <- merge(DEG, entrez, by="external_gene_name")

sig_genes <- subset(entrez_results, padj< 0.05)$log2FoldChange

names(sig_genes) <- subset(entrez_results, padj< 0.05)$entrezgene

head(sig_genes)


## Remove NA and duplicated values
sig_genes <- sig_genes[!is.na(names(sig_genes))] 

sig_genes <- sig_genes[!duplicated(names(sig_genes))]

background_genes <- entrez_results$entrezgene

background_genes <- background_genes[!duplicated(background_genes)]

```

Now that we have our background and significant genes in the appropriate format, we can run SPIA:

```r
# Run SPIA.

library(SPIA)

spia_result <- spia(de=sig_genes, all=background_genes, organism="hsa")

head(spia_result, n=20)
```

SPIA outputs a table showing significantly dysregulated pathways based on over-representation and signaling perturbations accumulation. The table shows the following information: `pSize` is the number of genes on the pathway; `NDE` is the number of DE genes per pathway; `tA` is the observed total perturbation accumulation in the pathway; `pNDE` is the probability to observe at least NDE genes on the pathway using a hypergeometric model; `pPERT` is the probability to observe a total accumulation more extreme than tA only by chance; `pG` is the p-value obtained by combining pNDE and pPERT; `pGFdr` and `pGFWER` are the False Discovery Rate and respectively Bonferroni adjusted global p-values; and the Status gives the direction in which the pathway is perturbed (activated or inhibited). KEGGLINK gives a web link to the KEGG website that displays the pathway image with the differentially expressed genes highlighted in red.

We can view the significantly dysregulated pathways by viewing the over-representation and perturbations for each pathway.

```r
plotP(spia_result,threshold=0.05)
```
In this plot each pathway is a point and the coordinates are the log of pNDE (using a hypergeometric model) and the p-value from perturbations, pPERT. The oblique lines in the plot show the significance regions based on the combined evidence.

If we choose to explore the significant genes from our dataset occurring in these pathways, we can subset our SPIA results:

```r
## Look at pathway 05222 and view kegglink
subset(spia_result, ID == "05222")
```

Then, click on the KEGGLINK, we can view the genes within our dataset from these perturbed pathways:
![perturbed_pathway](../img/hsa05222.png)

Tarca AL, Kathri P and Draghici S (2013). SPIA: Signaling Pathway Impact Analysis (SPIA) using combined evidence of pathway over-representation and unusual signaling perturbations. [http://bioinformatics.oxfordjournals.org/cgi/reprint/btn577v1](http://bioinformatics.oxfordjournals.org/cgi/reprint/btn577v1).

***

## Other Tools

### GeneMANIA

[GeneMANIA](http://genemania.org/) is a tool for predicting the function of your genes. Rather than looking for enrichment, the query gene set is evaluated in the context of curated functional association data and results are displayed in the form of a network. Association data include protein and genetic interactions, pathways, co-expression, co-localization and protein domain similarity. Genes are represented as the nodes of the network and edges are formed by known association evidence. The query gene set is highlighted and so you can find other genes that are related based on the toplogy in the network. This tool is more useful for smaller gene sets (< 400 genes), as you can see in the figure below our input results in a bit of a hairball that is hard to interpret.

![genemania](../img/genemania.png)

> Use the significant gene list generated from the analysis we performed in class as input to GeneMANIA. Using only pathway and coexpression data as evidence, take a look at the network that results. Can you predict anything functionally from this set of genes? 

### Co-expression clustering

Co-expression clustering is often used to identify genes of novel pathways or networks by grouping genes together based on similar trends in expression. These tools are useful in identifying genes in a pathway, when their participation in a pathway and/or the pathway itself is unknown. These tools cluster genes with similar expression patterns to create 'modules' of co-expressed genes which often reflect functionally similar groups of genes. These 'modules' can then be compared across conditions or in a time-course experiment to identify any biologically relevant pathway or network information.

You can visualize co-expression clustering using heatmaps, which should be viewed as suggestive only; serious classification of genes needs better methods.  

The way the tools perform clustering is by taking the entire expression matrix and computing pair-wise co-expression values. A network is then generated from which we explore the topology to make inferences on gene co-regulation. The [WGCNA](http://www.genetics.ucla.edu/labs/horvath/CoexpressionNetwork ) package (in R) is one example of a more sophisticated method for co-expression clustering.

Functional class scoring methods most often take as input the foldchanges for all genes, then look to see whether gene sets for particular biological processes are enriched among the large positive or negative fold changes. This type of analysis can be particularly helpful if differential expression analysis only output a small list of significant DE genes. Finally, pathway topology analysis often takes into account both fold changes and adjusted p-values to identify dysregulated pathways and outputs whether pathways are inhibited/activated. We have [materials](https://github.com/hbctraining/Training-modules/blob/master/DGE-functional-analysis/lessons/02_functional_analysis_other_methods.md) to lead you through these other types of functional analyses, and we encourage you to take the time to work through them.

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
