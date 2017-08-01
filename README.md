# VSE

**V**ariant **S**et **E**nrichment

## Description

VSE is a Perl/R command line tool to calculate the enrichment of an associated variant set (AVS) for an array of genomic regions.

## Installation

### Environment

* [Perl](https://www.perl.org) (5.18 or higher)
* [R](https://cran.r-project.org) (3.1.1 or higher)
* [bedtools](http://bedtools.readthedocs.io/en/latest/) (2.2.4 or higher) and must be globally executable

#### Perl modules

* File::Basename

#### R packages

* ggplot2
* reshape
* car

### Download VSE

Download VSE from this repo via `git clone`.
Alternatively, you can just download and run `VSE.pl`. No other installation is required if you have the required programs already installed.
The directory structure must be intact, however. i.e., `lib` and `data` directories must reside in the same directory as `VSE.pl`.

## Usage

### Example

```
perl VSE.pl -f example.SNPs/NHGRI-BCa.bed \
    -l example.SNPs/ld_BCa.bed \
    -s run1 -d example.beds \
    -v
```

### Parameters

| Parameter | Description |
|-----------|-------------|
| `-f` | BED file containing tagSNPs. Columns: `chr`, `start`, `end`, and `SNP name`. |
| `-l` | BED File containing LD SNPs. Columns: `chr`, `start`, `end`, `LDSNPid`, `tagSNPid`, `other optional columns`. The LD SNPs must be calculated based on EUR population of 1000 Genome Project Phase III genotype data (Release 2013/05). It's important to use the updated genotype information for calculating LD because the newer releases are more accurate than the older ones because of inclusion of more individuals. However, for older studies, VSE will soon support 1000 Genome Project Phase I data as well. |
| `-d` | Genomic ranges of interest. **Either** path to directory containing BED files of genomic regions of interest (the name of the file is used as labels for the regions) **or** path to a BED file (the tallies will be printed on screen and the enrichment analysis will not be run). |

### Options

| Option | Description |
|--------|-------------|
| `-s` | Output directory suffix. Output files will be saved in `suffix.output` directory. |
| `-r` | `R^2` value; default 0.8. |
| `-p [all/AVS/MRV/xml/R]` | Modular run; default `all`. |
| `-A` | Suffix for existing `AVS`/`MRV` files. Only functional when `-p` is `xml`. |
| `-h` | Help |
| `-v` | Verbose |

## Output

VSE produces multiple output files in `suffix.output` directory.

| File | Description |
|------|-------------|
| `suffix.density.pdf` | Density of overlapping tallies from the null |
| `suffix.VSE.stat.txt` | Statistics table |
| `suffix.final_boxplot.pdf` | Visualizes the null distribution and enrichment of `AVS` |
| `suffix.matrix.pdf` | Binary representation of overlapping between each locus and annotation. Overlapping is defined as at least one SNP (associated or linked) is within the annotation. |
| `suffix.VSE.txt` | Matrix of all overlapping tallies by `AVS` and `MRVS`. The first column is the `AVS` tally and the rest are `MRVS`. |

## Running VSE

VSE can be run in different parts using `-p` parameter. For the first run, `-p all` or no `-p` is recommended.
However, once you create the `AVS` and `MRVS` for a set of SNPs, you can use `-p xml` and `-p R` for checking the enrichment of the SNPs over new genomic ranges.
Below are examples of certain situations and command lines that to be used:

### Running for the first time for a set of variants

```shell
perl VSE.pl -f tagSNPs.bed -l LDSNPs.bed -d /path/histone_marks/ -s run1 -v
```

### Running the same set of SNPs but for a different batch of genomic regions

```shell
perl VSE.pl -f tagSNPs.bed -l LDSNPs.bed -d /path/TF_binding/ -p xml -A run1 -s run2 -v
```

This command line will use the `AVS` and `MRVS` outputted from run1 and will produce new matrix file in `run2.output` directory. Then you can run `-p R` to compute enrichment and generate the plots.

```shell
perl VSE.pl -f tagSNPs.bed -l LDSNPs.bed -d /path/TF_binding/ -p R -A run1 -s run2 -v
```

### Running for just one genomic region file

```shell
perl VSE.pl -f tagSNPs.bed -l LDSNPs.bed -d /path/POL2_binding.bed -s run-pol2 -v
```

This will output the overlapping tallies for `AVS` and `MRVS` for POL2 on the screen. You can copy this line to any other `*.VSE.txt` file from other experiments.
For example, you can add the line to `run2.output/run2.VSE.txt` from the `TF_binding` analysis (run2). You can then run `-p R` to redo the enrichment analysis, now including POL2.

```shell
perl VSE.pl -f tagSNPs.bed -l LDSNPs.bed -d /path/TF_binding/ -p R -A run2 -s run2_with_pol2 -v
```

## Factors to consider

1. VSE is sensitive to the number of tagSNPs. From our trial and error tests, too low number of tagSNPs (below 15) provide imprecise result.
1. The quality of ChIP-seq data is very important. We recommend users to confirm the quality of the ChIP-seq data and to only use data that are of good quality to avoid false enrichment. There are tools like [ChIPQC](http://bioconductor.org/packages/release/bioc/html/ChIPQC.html) or [Chillin](https://doi.org/10.1186/s12859-016-1274-4) for quality control of ChIP-seq data.
1. Make sure that you use the same `R^2` cutoff that you used to determine your LD SNPs. Also, the LD SNPs must be calculated using 1000 Genome Project Phase III (May 2013) release.
