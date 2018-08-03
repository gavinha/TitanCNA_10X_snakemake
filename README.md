# *Snakemake workflow for TITAN analysis of 10X Genomics WGS*

## Description
This workflow will run the TITAN copy number analysis for set of tumour-normal pairs, starting from the BAM files aligned using [Long Ranger](https://support.10xgenomics.com/genome-exome/software/pipelines/latest/what-is-long-ranger) software. The analysis includes haplotype-based copy number prediction and post-processing of results. It will also perform model selection at the end of the workflow to choose the optimal ploidy and clonal cluster solutions.  
Viswanathan SR*, Ha G*, Hoff A*, et al. Structural Alterations Driving Castration-Resistant Prostate Cancer Revealed by Linked-Read Genome Sequencing. *Cell* 174, 433–447.e19 (2018).

## Contact
Gavin Ha  
Fred Hutchinson Cancer Research Center  
contact: <gavinha@gmail.com> or <gha@fredhutch.org>  
Date: August 3, 2018  

## Requirements
### Software packages or libraries
 - R-3.4
   - [TitanCNA](https://github.com/gavinha/TitanCNA) (v1.15.0) or higher
   		- TitanCNA imports: GenomicRanges, GenomeInfoDb, VariantAnnotation, dplyr, data.table, foreach
   - [ichorCNA](https://github.com/broadinstitute/ichorCNA) (v0.1.0) 
   - HMMcopy
   - optparse
   - stringr
   - SNPchip
   - doMC
 - Python 3.4 
   - snakemake-3.12.0
   - PySAM-0.11.2.1
   - PyYAML-3.12
 - [bxtools](https://github.com/walaj/bxtools)


### Scripts used by the workflow
The following scripts are used by this snakemake workflow:
 - [getMoleculeCoverage.R](code/getMoleculeCoverage.R) Normalizing/correcting molecule-level coverage
 - [getPhasedHETSitesFromLLRVCF.R](code/getPhasedHETSitesFromLLRVCF.R) - Extracts phased germline heterozygous SNP sites from the Long Ranger analysis of the normal sample
 - [getTumourAlleleCountsAtHETSites.py](code/getTumourAlleleCountsAtHETSites.py) - Extracts allelic counts from the tumor sample at the germline heterozygous SNP sites
 - [titanCNA_v1.15.0_TenX.R](code/titanCNA_v1.15.0_TenX.R) - Main R script to run TitanCNA
 - [selectSolution.R](code/selectSolution.R) - R script to select optimal solution for each sample
 - [combineTITAN-ichor.R](code/combineTITAN-ichor.R) - R script to merge autosomes and chrX results, plus post-processing steps including adjusting max copy values.

## Tumour-Normal sample list
The list of tumour-normal paired samples should be defined in a YAML file. In particular, the [Long Ranger](https://support.10xgenomics.com/genome-exome/software/pipelines/latest/what-is-long-ranger) (v2.2.2) analysis directory is listed under samples.  See `config/samples.yaml` for an example.  Both fields `samples` and `pairings` must to be provided.  `pairings` key must match the tumour sample while the value must match the normal sample.
```
samples:
  tumor_sample_1:  /path/to/tumor/longranger/dir
  normal_sample_1:  /path/to/normal/longranger/dir


pairings:
  tumor_sample_1:  normal_sample_1
```


## snakefiles
1. `moleculeCoverage.snakefile`
2. `getPhasedAlleleCounts.snakefile`
3. `TitanCNA.snakefile`

Invoking the full snakemake workflow for TITAN
```
# show commands and workflow
snakemake -s TitanCNA.snakefile -np
# run the workflow locally using 5 cores
snakemake -s TitanCNA.snakefile --cores 5
# run the workflow on qsub using a maximum of 50 jobs. Broad UGER cluster parameters can be set directly in config/cluster.sh. 
snakemake -s TitanCNA.snakefile --cluster-sync "qsub -l h_vmem={params.mem},h_rt={params.runtime}" -j 50 --jobscript config/cluster.sh
```
This will also run both `moleculeCoverage.snakefile` and `getPhasedAlleleCounts.snakefile` which generate the necessary inputs for `TitanCNA.snakfile`.

`moleculeCoverage.snakefile` and `getPhasedAlleleCounts.snakefile` can also be invoked separately. If only one but not both results are needed, then you can invoke the snakefiles independently.
```
snakemake -s moleculeCoverage.snakefile --cores 5
# OR
snakemake -s getPhasedAlleleCounts.snakefile --cores 5
``` 




