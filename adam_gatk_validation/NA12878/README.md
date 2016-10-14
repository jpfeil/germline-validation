# NA12878 Germline Run

## Overview
This was an initial test to iron out bugs in the pipeline. I started with the 1000 Genomes NA12878 alignment BAM and ran it through the Toil GATK germline pipeline. I then compared our calls to the GIAB high confidence calls.

## Sample
NA12878
`ftp://ftp-trace.ncbi.nlm.nih.gov/1000genomes/ftp/phase3/data/NA12878/high_coverage_alignment/NA12878.mapped.ILLUMINA.bwa.CEU.high_coverage_pcr_free.20130906.bam`

## Baseline 
Used the GIAB integrated high confidence calls as ground truth. This set of calls integrates several sequencing platforms to attempt to correct for platform specific sequencing errors. It also includes the homozygous reference bases to estimate the false positive rate. 

Baseline
`ftp://ftp-trace.ncbi.nlm.nih.gov/giab/ftp/data/NA12878/analysis/GIAB_integration/NIST_RTG_PlatGen_merged_highconfidence_v0.2.primitives.vcf.gz.md5`

It also avoids regions of the genome that are difficult to make high confidence calls. 

Regions
`ftp://ftp-trace.ncbi.nlm.nih.gov/giab/ftp/data/NA12878/analysis/GIAB_integration/union13callableMQonlymerged_addcert_nouncert_excludesimplerep_excludesegdups_excludedecoy_excludeRepSeqSTRs_noCNVs_v2.19_2mindatasets_5minYesNoRatio_AddRTGPlatGenConf_filtNISTclustergt9_RemNISTfilt_RemPartComp_RemRep_RemPartComp_v0.2.bed.gz`

## Commands 

The baseline callset does not include Y and MT, so I removed them in the VQSR callset


`vcfallelicprimitives NA12878.vqsr.sep9VQSR.vcf > NA12878.vqsr.sep9VQSR.primitives.vcf`

`vcf-sort NA12878.vqsr.sep9VQSR.primitives.vcf > NA12878.vqsr.sep9VQSR.primitives.sorted.vcf`

`rtg bgzip NA12878.vqsr.sep9VQSR.primitives.sorted.vcf`

`vcftools V--gzvcf NA12878.vqsr.sep9VQSR.primitives.sorted.vcf.gz --out NA12878.vqsr.sep9VQSR.primitives.sorted.minusYMT.vcf.gz --not-chr Y --not-chr MT --recode --recode-INFO-all`

`rtg index -f vcf NA12878.vqsr.sep9VQSR.primitives.sorted.minusYMT.vcf.gz`

`rtg vcfeval --bed-regions=union13callableMQonlymerged_addcert_nouncert_excludesimplerep_excludesegdups_excludedecoy_excludeRepSeqSTRs_noCNVs_v2.19_2mindatasets_5minYesNoRatio_AddRTGPlatGenConf_filtNISTclustergt9_RemNISTfilt_RemPartComp_RemRep_RemPartComp_v0.2.bed -b NIST_RTG_PlatGen_merged_highconfidence_v0.2.primitives.vcf.gz -c NA12878.vqsr.sep9VQSR.primitives.sorted.minusYMT.vcf.gz -o vqsr_eval_092616_correct_baseline -t b37.sdf`

## VQSR Results

| Threshold | True-pos-baseline | True-pos-call | False-pos |  False-neg | Precision |  Sensitivity |  F-measure |
|-----------| ------------------|---------------|-----------| -----------|-----------|------------- |---------   |
|    5.000  |          3143316  |      3148977  |    12610  |   248891   |  0.9960   |    0.9266    |  0.9601    |
|     None  |          3166812  |      3172396  |    13857  |   225395   |  0.9957   |    0.9336    |  0.9636    |


## Hard filter Results

|Threshold | True-pos-baseline | True-pos-call | False-pos | False-neg | Precision | Sensitivity | F-measure |
|----------|-------------------|---------------|-----------|-----------|-----------|-------------|---------- |
|    5.000 |           3351340 |       3359990 |     16017 |     40867 |    0.9953 |      0.9880 |    0.9916 |
|     None |           3374152 |       3382820 |     16969 |     18055 |    0.9950 |      0.9947 |    0.9948 |
