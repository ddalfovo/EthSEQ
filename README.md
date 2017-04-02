# EthSEQ: ethnicity annotations from whole exome sequencing data

Whole exome sequencing (WES) is widely utilized both in translational cancer genomics studies and in the setting of precision medicine. Stratification of individual’s ethnicity is fundamental for the correct interpretation of personal genomic variation impact. We implemented EthSEQ to provide reliable and rapid ethnicity annotation from whole exome sequencing individual’s data. EthSEQ can be integrated into any WES based processing pipeline and exploits multi-core capabilities.

EthSEQ requires genotype data at SNPs positions for a set of individuals with known ethnicity (the reference model) and either a list of BAM files or genotype data (in VCF format) of individuals with unknown ethnicity. EthSEQ annotates the ethnicity of each individual using an automated procedure and returns detailed information
about individual’s inferred ethnicity, including aggregated visual reports. 

***

## Perform ethnicity analysis with individuals genotype data from VCF file

Analysis of 6 individuals from 1,000 Genome Project using a reference model built from 1,000 Genome Project individual's genotype data. Genotype data for 10,000 SNPs included in Agilent Sure Select v2 captured regions are provided in input to EthSEQ in VCF format while reference model is provided in GDS format and describes genotype data for 1,000 Genome Project individuls for the same SNPs set. 

```{r}
library(EthSEQ)

## Run the analysis
ethseq.Analysis(
  target.vcf = system.file("extdata", "Samples_SS2_10000SNPs.vcf",
	package="EthSEQ"),
  out.dir = "/tmp/EthSEQ_Analysis/",
  model.gds = system.file("extdata", "Reference_SS2_10000SNPs.gds",
	package="EthSEQ"),
  verbose=TRUE,
  composite.model.call.rate = 1)

## Load and display computed ethnicity annotations
ethseq.annotations = read.delim("/tmp/EthSEQ_Analysis/Report.txt",
	sep="\t",as.is=TRUE,header=TRUE)
head(ethseq.annotations)

## Delete analysis folder
unlink("/tmp/EthSEQ_Analysis/",recursive=TRUE)
```

## Perform ethnicity analysis using pre-computed reference model

Analysis of 6 individuals from 1,000 Genome Project using a reference model built from 1,000 Genome Project individual's genotype data. Genotype data for 123,292 SNPs included in Agilent Sure Select v2 captured regions are provided in input to EthSEQ in VCF format while reference model selected among the set of pre-computed reference model. Reference model SS2.Light refers to the reference model built from 800 individuals from 1,000 Genome Project and considering 123,292 SNPs included in Agilent Sure Select v2 captured regions. Note that a reference model version called SS2 considering gentoype data for more than 2,000 individuals from 1,000 Genome Project is also available.

```
library(EthSEQ)

## Download genotype data in VCF format
dir.create("/tmp/EthSEQ_Data/")
download.file("https://github.com/aromanel/EthSEQ_Data/raw/master/Sample_SS2.vcf",
  destfile = "/tmp/EthSEQ_Data/Sample_SS2.vcf")

## Run the analysis
ethseq.Analysis(
  target.vcf = "/tmp/EthSEQ_Data/Sample_SS2.vcf",
  out.dir = "/tmp/EthSEQ_Analysis/",
  model.available = "SS2.Light",
  model.folder = "/tmp/EthSEQ_Data/",
  verbose=TRUE,
  composite.model.call.rate = 1)

## Delete analysis folder
unlink("/tmp/EthSEQ_Analysis/",recursive=TRUE)
unlink("/tmp/EthSEQ_Data/",recursive=TRUE)
```

## Perform ethnicity analysis from BAM files list

Analysis of individual NA07357 from 1,000 Genome Project using a reference model built from 1,000 Genome Project individual's genotype data. Genotype data for 10,000 SNPs included in Agilent Sure Select v2 captured regions are provided in input to EthSEQ with a BAM file. reference model is provided in GDS format and describes genotype data for 1,000 Genome Project individuls for the same SNPs set. Note than the BAM given in input to EthSEQ is a toy BAM file containing only reads overlapping the positions of the 10,000 SNPs considered in the analysis.

```
library (EthSEQ)

## Download BAM file used in the analysis
dir.create("/tmp/EthSEQ_BAMs/")
download.file(
 "https://github.com/aromanel/EthSEQ_Data/raw/master/NA07357_only10000SNPs.bam",
 destfile = "/tmp/EthSEQ_BAMs/Sample.bam")
download.file(
 "https://github.com/aromanel/EthSEQ_Data/raw/master/NA07357_only10000SNPs.bam.bai",
 destfile = "/tmp/EthSEQ_BAMs/Sample.bam.bai")

## Create BAM files list 
write("/tmp/EthSEQ_BAMs/Sample.bam","/tmp/EthSEQ_BAMs/BAMs_List.txt")

## Run the analysis
ethseq.Analysis(
  bam.list = "/tmp/EthSEQ_BAMs/BAMs_List.txt",
  out.dir = "/tmp/EthSEQ_Analysis/",
  model.gds = system.file("extdata","Reference_SS2_10000SNPs.gds",
     package="EthSEQ"),
  verbose=TRUE,
  aseq.path = "/tmp/EthSEQ_Analysis/",
  mbq=20,
  mrq=20,
  mdc=10,
  run.genotype = TRUE,
  composite.model.call.rate = 1,
  cores=1)

## Delete analysis folder
unlink("/tmp/EthSEQ_BAMs/",recursive=TRUE)
unlink("/tmp/EthSEQ_Analysis/",recursive=TRUE)
```

## Perform ethnicity analysis using multi-step refinement

Multi-step refinement Analysis of individuals from 1,000 Genome Project using a reference model built from 1,000 Genome Project individual's genotype data (set of analysed individuals and individuals used for the reference model are disjoint). Genotype data for 10,000 SNPs included in Agilent Sure Select v2 captured regions are provided in input to EthSEQ in GDS format while reference model is provided in GDS format and describes genotype data for 1,000 Genome Project reference individuls for the same SNPs set. Multi-step refinement tree is constructed as matrix. Non-empty cells in columns i contains parent nodes for non-empty cells in columns i+1. Ethnic groups in child nodes should be included in parent nodes, while siblings node ethnic groups should be disjoint. Consult EthSEQ paper supplementary material for more complicated examples.  

```
library(EthSEQ)

## Create multi-step refinement matrix
m = matrix("",ncol=2,nrow=2)
m[1,1] = "SAS|EUR|EAS"
m[2,2] = "SAS|EUR"

## Run the analysis on a toy example with only 10000 SNPs
ethseq.Analysis(
  target.gds = system.file("extdata","Target_SS2_10000SNPs.gds",
	package="EthSEQ"),
  out.dir = "/tmp/EthSEQ_Analysis/",
  model.gds = system.file("extdata","Reference_SS2_10000SNPs.gds",
	package="EthSEQ"),
  verbose=TRUE,
  composite.model.call.rate = 1,
  refinement.analysis = m)

## Delete analysis folder
unlink("/tmp/EthSEQ_Sample/",recursive=TRUE)
```

## Create a reference model from multiple VCF genotype data files

Construction of a reference model from two genotype data files in VCF format and a corresponding annotation files which described ethnicity and sex of each sample contained in the genotype data files.

```
library(EthSEQ)

### Load list of VCF files paths
vcf.files = 
  c(system.file("extdata", "VCF_Test_1.vcf", package="EthSEQ"),
    system.file("extdata", "VCF_Test_2.vcf", package="EthSEQ"))

### Load samples annotations
annot.samples = read.delim(system.file("extdata", "Annotations_Test.txt",
	package="EthSEQ"))

### Create reference model
ethseq.RM(
  vcf.fn = vcf.files,
  annotations = annot.samples,
  out.dir = "/tmp",
  model.name = "Reference.Model",
  bed.fn = NA,
  phased = FALSE,
  call.rate = 1,
  cores = 1)

## Delete example file
unlink("/tmp/Reference.Model.gds")
```

