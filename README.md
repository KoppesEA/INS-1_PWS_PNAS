# INS-1_PWS_RNA-Seq
Markdown on the useage of uploaded bash scripts for RNA-Seq analysis associated with Koppes et al. 2021 for analysis of total- and small- RNA-Seq of control and PWS INS-1 cells.  

Fastq files associated with this study can be found via GEO SuperSeries GSE190337 and Experiment BioProject PRJNA786769  
GSE190334  (total RNA-seq)  
and  
GSE190336  (small RNA-seq)


*Directories and version numbers may need to be adjusted within each script by user to run as needed*

## Part I: Custom Genome Generation
Download Rnor6 top level genome sequence and gene set annotation from Ensembl (https://useast.ensembl.org/info/data/ftp/index.html)  

    wget http://ftp.ensembl.org/pub/release-98/fasta/rattus_norvegicus/dna/Rattus_norvegicus.Rnor_6.0.dna.toplevel.fa.gz
    wget http://ftp.ensembl.org/pub/release-98/gtf/rattus_norvegicus/Rattus_norvegicus.Rnor_6.0.98.gtf.gz

Note that as of 1/20/2023 the current genome and annotation is now mRatBN7.2.108.gtf.gz

Unzip the genome fasta file and then append the mIns2::mCherry-CP and hINS::neoR transgene sequences from `mINS2tg.fa` and `hINStg.fa`

     gunzip -c Rattus_norvegicus.Rnor_6.0.dna.toplevel.fa.gz > Rattus_norvegicus.Rnor_6.0.dna.toplevel.fa
     cat Rattus_norvegicus.Rnor_6.0.dna.toplevel.fa hINStg.fa mINS2tg.fa > Rattus_norvegicus.Rnor_6.0.dna.custom.fa
     gunzip -c Rattus_norvegicus.Rnor_6.0.98.gtf.gz > Rattus_norvegicus.Rnor_6.0.98.gtf
     cat Rattus_norvegicus.Rnor_6.0.98.gtf CustomAnnot.gtf > Rattus_norvegicus.Rnor_6.0.98.custom.gtf
     
Generate STAR genomic indexes from custom rat genome and annotation using `genomeGenerate` command from STAR version 2.7.0e with options specified in batch script `STARgenomeGen_custom.sh`  

Generate RSEM-STAR genomic indexes from custom rat genome and annotation using `rsem-prepare-reference` command with RSEM version 1.3.1 with options specified in batch script `RSEM_RnorV98customprep.sh`  

Generate Bowtie2 indexes from Rnor6.0.99 (without custom annotation) using the `bowtie2-build` command with bowtie2 version 2.3.4.2 with options specified in batch script `Bowtie_Rnor6v99_build.sh`

## Part II: Identification of differentially expressed genes  in total RNA using STAR alignment with gene feature counts by HT-Seq and comparison with DEseq2
RNA-seq QC check using `FastQC` version 0.11.5 and illumina NextSeq adapters trimmed using `TrimGalore` version 0.4.5 as specified in `Total_RNA_QCandTRIM.sh` with the sample list file `expList_2017.txt`  

Splice-aware reference-based alignment using STAR version 2.7.0e was implemented with options specified in `STAR_RnorCustom_mapping.sh` combined with the custom annotation genomic indexes generated in Part I.

Gene-level feature counts were made with HTseq version 0.11.2 using parameters specified in `HTSeq_Rnorcustom_v98.sh`.  
*Note that the settings* `--mode union` *and* `--nonqunique all` *were required to accurately quantify the overlapping bicistronic Snurf-Snrpn transcript and the multicopy Snord115 and Snord116 genes of the PWS domain.*  

The DESeq2 version 3.9 R package (R version 3.6) was used for differential expression anlaysis as implemented in R script `Deseq2_STARHTSeq_RScript.R`.  

*Significance cutoffs of padj<0.10 lenient or padj<0.05 stringent were used in the identification of differentially expressed genes to investigate further.*

## Part III: Alternate identification of differentially expressed genes in total RNA using STAR alignment with RSEM and comparison with DEseq2
Starting with custom genome annotation and pre-processed reads from PartI and PartII. `RSEM` v.1.3.1 with `STAR` was used to perform both alignment and feature counts as implemented in `RSEM_RnorV98custom_calcexp`.  

A normalized matrix of gene-level and isoform-level read-counts for the 6 samples was then generated using `RSEM_RnorV98custom_genedatamatrix.sh`.  

The read-count matrix was subsequently imported into R using `tximport` and differential expression quantified by `DESeq2` as written in the script `RSEM_RnorV98custom_Deseq2`.  

*A Significance cutoff of padj<0.10 was used to compare with results from PartII*


## Part IV: smallRNA-Seq differential expression using Bowtie2 alignment, HTseq and DEseq2
A `Bowtie2` genome index was generated based on the Rnor6.0 Ensembl V99 toplevel DNA sequence using bash script `INS1smallRNA_Bowtie2_Build`.  

Single end reads including Qiagen miRNA UMI and Illumina sequencing adapters were pre processed to extract UMI tags with `umi-tools` bash script  `INS1smallRNA_UMIextract.sh` followed by `cutadapt` to remove illumina sequencing adapters, filter quality and to size >18nt with script `INS1smallRNA_UMI_cutAdaptm18.sh`.  

Processed reads were then aligned using `Bowtie2` implemented with the script `INS1smallRNA_Bowtie2_Algin_UMIcutAdaptm18.sh` with the output .sam files were sorted by position and converted to .bam and then indexed using using the `samtools sort` function and then the samtools `index function as implemented in `INS1smallRNA_SamtoBam.sh` and `INS1smallRNA_Samtoolsindex.sh`.  

Aligned reads were then deduplicated using `UMI-tools` from the aligned reads with the script `INS1smallRNA_UMIm18dedup.sh`. Deduplicated reads were then used as the input for gene-level counts using HTSeq using the Ensembl v99 (non custom) GTF annotation file as implemented with the script `INS1smallRNA_HTSeq_UnionAllExon.sh` using the `--mode union` and `--nonunique all` ; notably as for the total-RNAseq the macrosatellite snoRNA-115 and snoRNA-116 expression was much lower when tested using `--nonunique none` than with `nonunique all` options as many of the multimapped expressed copies were discarded.  Last, differential smallRNA expression was anlayzed in DESeq2 with the following R script `INS1smallRNA_UMIm18_v99_exUnionNone_rscript.R` with a padj cutoff of P>0.10.





## Part V References and links to software used herein

### STAR
https://github.com/alexdobin/STAR  
https://pubmed.ncbi.nlm.nih.gov/23104886/  

### HTSeq
https://github.com/htseq/htseq  
https://htseq.readthedocs.io/en/master/  
https://pubmed.ncbi.nlm.nih.gov/25260700/  

### RSEM
http://deweylab.github.io/RSEM/  
https://pubmed.ncbi.nlm.nih.gov/21816040/  


### DESeq2
https://github.com/mikelove/DESeq2  
https://bioconductor.org/packages/DESeq2/  
https://pubmed.ncbi.nlm.nih.gov/25516281/      

### Trimgalore
https://www.bioinformatics.babraham.ac.uk/projects/trim_galore/  

### FastQC
https://www.bioinformatics.babraham.ac.uk/projects/fastqc/

### UMI-tools
https://github.com/CGATOxford/UMI-tools  
https://umi-tools.readthedocs.io/en/latest/QUICK_START.html  
https://pubmed.ncbi.nlm.nih.gov/28100584/  

### Cutadapt
https://cutadapt.readthedocs.io/en/stable/  
http://journal.embnet.org/index.php/embnetjournal/article/view/200  

### Samtools
https://github.com/samtools/samtools  

### Bowtie2
https://github.com/BenLangmead/bowtie2  
https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3322381/  
