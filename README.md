# Producing a table of DDX6 binding sites located within 3'UTRs across the transcriptome 
## Assessment specifications: 
_'You should take your own code or relevant software as a starting point, and modify or extend it so that it:_

_- is a github repository, with issues + pull requests that fully document the development process_

_- has unit testing with a good test coverage_

_- is pip installable_

_- is fully documented, e.g. with a README file and a "Read the Docs" website_

_- uses continuous integration (e.g. Github Actions) for automatic testing and documentation generation'_

## Background
### Argonaute Phosphorylation and the RNA-Induced Silencing Complex (RISC)
DDX6 is a RNA binding protein (RBP) that is recruited to the RNA-Induced Silencing Complex (RISC) following phosphorylation of the Argonaute (Ago2) protein. The RISC complex is responsible for seeking out specific messenger RNA transcripts (mRNAs) in the
cytoplasm and blocking their translation; Ago2 performs a key role in this function by guiding the RISC complex to target mRNA transcripts, or by directly cleaving mRNAs upon recognition. Ago2 is able to guide RISC to target mRNA transcripts with the help of
short RNA sequences called microRNAs (miRNAs). miRNA sequences are often complementary to 'seed sequences' located in the 3'-untranslated regions (3'UTRs) of certain mRNA transcripts. The loading of a single microRNA onto Ago2 prompts RISC assembly and a subsequent search for 'seed sequences'. In this way, gene expression can be silenced. Recruitment of DDX6 to this complex following Ago2 phosphorylation is thought to help confer greater specificity for target mRNAs. This is because although the function of DDX6 
is relatively undefined, it is hypothesised to recognise specific secondary structures in RNA; therefore, it may contribute to target specificity in RISC function by binding to mRNA 3'UTRs with specific secondary structures.

### Using Publicly-Available DDX6 eCLIP Data to Infer Targets of RISC-Mediated Translational Repression Following Ago2 Phosphorylation
We want to identify which mRNAs are likely to be targeted by RISC following Ago2 phosphorylation. One way in which this could be done is to identify targets of DDX6 binding. Fortunately, a series of enhanced crosslinking-immunoprecipitation experiments (eCLIP) experiments were carried out to identify the binding sites for most RBPs across the trancsriptome as part of the ENCODE project [1]. Another database called POSTAR3 was produced using these publicly-available data, alongside data from similar experiments unaffiliated with the ENCODE project, to generate a 'comprehensive map' detailing the binding patterns of RBPs[2].

## Data Processing
### (1) Extracting DDX6 experimental data 
Refer to ['Filtering_RBP_coords.py'](Filtering_RBP_coords.py). All experimental data collated by the authors of POSTAR3 relating to human RBPs was downloaded from their website: http://111.198.139.65/RBP.html. Under 'Bulk Download Request', click 'Request data'. You can then download 'human.txt.gz', which contains binding site/experimental data pertaining to all human RBPs. This file is not provided in this repository, as it is too large. The sqlite3 python library was used to filter this database for data exclusively relating to DDX6-based experiments; this solely comprised of the eCLIP experiments in HepG2 and K562 cells carried out as part of the ENCODE project. Binding site coordinates (chrom; start; end); strand; experimental method; sample/tissue used; accession of raw data; and confidence score were extracted for each experimentally-determined binding site. Output = ['DDX6_binding_coords.csv'](output/DDX6_binding_coords.csv).

### (2) Filtering 3'UTR DDX6 binding site coordinates 
Refer to ['Intersects.py'](intersects.py). A full list of 3'UTR coordinates across the human transcriptome was initially downloaded from the Genome Browser website (hg38 version), which contains: chrom, start, end, identifier (i.e. GENCODE ID), an unknown variable, and strand. The Bioframe library was used to identify overlapping intervals between the list of 3'UTR coordinates and the list of DDX6 binding site coordinates. This appeared to yield 3,201 DDX6 binding sites specific to transcript 3'UTRs. The 3,201 3'UTR coordinates and associated GENCODE IDs were exported (['UTR_overlapping_intervals.csv'](R_analysis/data/UTR_overlapping_intervals.csv) for further processing in RStudio to obtain gene-specific information. The 3,201 DDX6 binding sites and associated experimental information were exported separately (see step (4)- ['DDX6_overlapping_intervals.csv'](DDX6_overlapping_intervals.csv)).

### (3) Acquiring gene-specific information for 3'UTR DDX6 binding sites
Refer to ['biomart_script.r'](R_analysis/biomart_script.r). Bioconductor is an R-based open source software that is notoriously useful for bioinformatics. In this case, it was deemed to be more practical compared to using a python-based equivalent. The biomaRt package was used to obtain the Ensembl transcript ID, Ensembl gene ID, external gene name, and a gene description for each of the 3,201 3'UTR coordinates. Output = ['overlapping_intervals_gene_names.csv'](R_analysis/output/overlapping_intervals_gene_names.csv). IMPORTANT NOTE: our control gene Limk1, which we know to have transcripts bound by DDX6, is not present in this dataset. May need to go back and check coordinates- see if DDX6 binding sites directly overlap with 3'UTR, or partially overlap.

### (4) Compiling gene-specific information and relevant experimental data
Refer to ['final_script.py'](final_script.py). 'overlapping_intervals_gene_names.csv' and 'DDX6_overlapping_intervals.csv' were merged to produce ['DDX6_3UTR_binding_sites.csv'](R_analysis/data/DDX6_3UTR_binding_sites.csv). This contains all experimental information relating to the DDX6 binding sites present in transcript 3'UTRs, and details relating to the genes in which these 3'UTRs are located.

### (5) Performing gene ontology enrichment analysis
Refer to ['GO_analysis_script.r'](R_analysis/GO_analysis_script.r). The clusterProfiler package in Bioconductor was used to perform GO enrichment analysis of the full list of genes with DDX6 binding sites in their 3'UTRs. The enrichment analysis looks at the proportion of genes in your dataset associated with a given GO term, and compares this to the expected proportion of genes associated with this GO term across the genome. Such analyses were carried out separately for GO terms within the 'cellular component', 'biological process' and 'molecular function' categories. The top 20 significantly upregulated GO terms for each category can be seen below. Tables containing all significantly enriched GO terms are available for each of [Biological Process](/R_analysis/output/GO_BP_Summary.csv), [Cellular Component](R_analysis/output/GO_CC_Summary.csv) and [Molecular Function](R_analysis/output/GO_MF_Summary.csv).  

![Barplot of top 20 significantly enriched biological process GO terms](/R_analysis/output/clusterProfiler_GO-BP_ORA_simplify.png)
![Barplot of top 20 significantly enriched cellular component GO terms](/R_analysis/output/clusterProfiler_GO-CC_ORA_simplify.png)
![Barplot of top 20 significantly enriched molecular function GO terms](/R_analysis/output/clusterProfiler_GO-MF_ORA_simplify.png)

[1] https://pubmed.ncbi.nlm.nih.gov/32252787/
[2] https://academic.oup.com/nar/article/50/D1/D287/6353804?login=true
