# NGS Data Analysis Manuals

> The materials in this repository are based on educational courses I have taken<br>
> It can be used as a helpful repository with cheat-sheets for NGS studies.

<div style='justify-content: center'>
<img src="https://github.com/iliapopov17/NGS-Data-Analysis-Manual/blob/main/imgs/NGS%20analysis%20workflow.png" align='center', width="100%">
</div>

_Typical workflow of NGS data analysis_

To recreate any of the steps of this manual please install:

```bash
conda env create -f ngs-manual.yml
```

And of cource do not forget to activate the envinronment!

```bash
conda activate ngs-manual
```

## Much more to be disclosured soon:
- Reference-free approach in the comparative analysis of metagenomic data
- Whole Genome and Pangenome Analyses
- 16S Amplicon Analysis

## Genomic Variation Analysis

In the [Genomic Variation Analysis folder](2%20-%20Genomic%20Variation%20Analysis) there is a detailed guide how to conduct studies on Variant Calling using `fastqc`, `trimmomatic`, `bwa`, `samtools`, `abra2`, `bcftools`, `snpEff` & `SnpSift`

## Quality Control of raw data

In the [Quality Control folder](1%20-%20Quality%20Control) there is a detailed guide how to conduct quality control of raw data using `fastqc` and `trimmomatic`.
