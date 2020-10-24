# Annotation of a novel eukaryotic genome

This tutorial shows how to use GeneMark to predict genes in a sequence of a novel genome, for which no data than the genomic sequence itself are available.

We will cover data preparation, repeat masking, prediction itself, and how to evaluate the prediction result.

## Preparing the test data 

We will use Arabidopsis thaliana as an example genome in this tutorial. 

!> The following steps show how to prepare the example inputs and are thus irrelevant for an actual use-case of annotation of a novel genome. The tutorial itself starts with the [Downloading cross-species proteins section](examples/novel_genome?id=downloading-cross-species-proteins)

Download the genomic sequence from NCBI:

```bash
wget wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/001/735/GCF_000001735.4_TAIR10.1/GCF_000001735.4_TAIR10.1_genomic.fna.gz
gunzip GCF_000001735.4_TAIR10.1_genomic.fna.gz
mv GCF_000001735.4_TAIR10.1_genomic.fna genome.fasta
```

The genome comes with repeat-masking. Remove the masking to simulate a novel genome:

```bash
probuild --reformat_fasta --in genome.fasta --out tmp_genome.fasta --uppercase 1 --letters_per_line 60 --original
mv tmp_genome.fasta genome.fasta
```

?> The `probuild` program is included in the GeneMark-ES suite

Download annotation for result evaluation:

```bash
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/001/735/GCF_000001735.4_TAIR10.1/GCF_000001735.4_TAIR10.1_genomic.gff.gz
gunzip GCF_000001735.4_TAIR10.1_genomic.gff.gz
mv GCF_000001735.4_TAIR10.1_genomic.gff annotation.gff

```
!> Actual novel genomes usually do not have a reference annotation available. In this tutorial, we will also show how to evaluate the results when the reference annotation is not available.

Convert the annotation to `.gtf` format with [GenomeTools](http://genometools.org/):

```bash
gt gff3_to_gtf annotation.gff > annotation.gtf
```


## Downloading cross-species proteins

Let us assume there is no available RNA-Seq data for our novel genome. In such case, we can run GeneMark-ES to obtain purely computational prediction or we can use GeneMark-EP+ with **publicly available cross-species protein sequences** as an external evidence to improve the computational predictions. In this tutorial we will show and compare both methods. For GeneMark-EP+, we will need to download protein sequences.

Let us assume that our novel genome is a deeply rooted plant species with no sequenced close relatives. Therefore, we will be using **all Plant proteins** from OrthoDB as a general source of protein sequences.

?> GeneMark-EP+ can utilize even very remotely homologous proteins to improve its predictions.

Download the proteins and concatenate them into a single `.fasta` file:

```bash
wget https://v100.orthodb.org/download/odb10_plants_fasta.tar.gz
tar xvf odb10_plants_fasta.tar.gz
cat plants/Rawdata/* > proteins.fasta
```

Please refer to the [Preparing protein evidence](usage/preparing_proteins.md) section for more details about reference protein preparation.

> [!DANGER]
> To simulate the absence of closely related species in this experiment, we will remove proteins of species in the Arabidopsis genus from the input protein set. No such procedure is necessary with an actual novel genome.
> ```bash
> # Remove the previously created proteins.fasta file
> rm proteins.fasta
> # Only include proteins outside of the Arabidopsis genus. 3702_0 and 81972_0 are IDs of Arabidopsis species
> ls plants/Rawdata/* | grep -v -P "3702_0.fs|81972_0.fs" | xargs cat >> proteins.fasta
> ```


## Repeat Masking

Prior to predicting predicting protein coding genes, it is important to identify and mask repetitive elements. Since there is no existing repeat library for our novel genome, we will build our own with [RepeatModeler2](https://www.pnas.org/content/117/17/9451):

```bash
BuildDatabase -name genome genome.fasta
RepeatModeler -database genome -LTRStruct
```

Use the repeat library to soft-mask repeats with [RepeatMasker](http://www.repeatmasker.org/):

```bash
RepeatMasker -lib genome-families.fa -xsmall genome.fasta
```

The masked genome will outputted to  `genome.fasta.masked` file.

?> See the [section on masking in the usage page](usage/general?id=repeat-masking) for more details about repeat masking.

## Gene Prediction

Please refer to the [installation section](installation/download.md) to get GeneMark ready.

### Predicting genes with GeneMark-ES

We will start by using GeneMark-ES which works with the genomic sequence alone:

```bash
gmes_petap.pl --seq genome.fasta.masked --ES --soft_mask auto --cores 8
```

?> This procedure takes about X minutes on an 8-CPU machine.

### Predicting genes with GeneMark-EP+


The entire procedure described in this section can be achieved by running a single command:

```bash
gmes_petap.pl --seq genome.fasta.masked --EP --dbep proteins.fasta --soft_mask auto --cores 8
```

In the remainder of this section, we will split the above command into several steps to better explain how the prediction with proteins works. Moreover, the intermediate results (such as protein hints generated by ProtHint) can be useful on their own. You can skip to [prediction evaluation](examples/novel_genome?id=prediction-evaluation) if not interested in the extended procedure.

GeneMark-EP+ first need to generate protein hints (introns, start and stop codons) from all reference proteins used on input. This task is efficiently solved by [ProtHint](https://github.com/gatech-genemark/ProtHint), included in the GeneMark-ES suite in the `ProtHint` folder.

ProtHint is using GeneMark-ES to initialize the genomic regions for protein search. Since we already have GeneMark-ES results, we can re-use it with the `--geneSeeds ES/genemark.gtf` option:

```bash
prothint.py genome.fasta.masked proteins.fa  --workdir prothint --geneSeeds ES/genemark.gtf
```

?> Protein hints generation with ProtHint is optimized for speed, processing of all the supplied plant proteins takes ~ X minutes on an 8-CPU machine.

After running protHint, we can use its results in GeneMark-EP+:

```bash
gmes_petap.pl --seq genome.fasta.masked  --EP prothint/prothint.gff --evidence prothint/evidence.gff  --soft_mask auto --cores 8
```

> [!INFO]
> In the above commands, we used two different ProtHint outputs. `prothint.gff` contains all ProtHint predictions and all of these hints are used by GeneMark-EP+ to improve model training. `evidence.gff` contains a high-confidence subset of `prothint.gff`, these hints are enforced in the final GeneMark-EP+ prediction. Hints in `evidence.gff` pass stringent filtering criteria and [generally have > 95% specificity](examples/novel_genome?id=evaluation-of-prothint-hints).

?> Runtime of GeneMark-EP+ is similar to that of GeneMark-ES, about X minutes on an 8-CPU machine.

## Prediction Evaluation

In this section we will show how to evaluate the results with or without the reference annotation.

### Evaluation Against reference annotation

Since we have a reference evaluation, we can directly evaluate how accurate the prediction is. Obviously, a reference evaluation is rarely available for a novel genome; this comparison serves to illustrate how accurate the algorithm is.

#### Comparison of GeneMark-ES and GeneMark-EP+ results

We start by [comparing the GeneMark-ES results to annotation](output/evaluation?id=comparing-against-a-reference) on gene level. 

```bash
compare_intervals_exact.pl --f1 annotation.gtf --f2  ES/genemark.gtf --gene
```

!> In a gene-level comparison, full gene structures have to match the annotation **exactly** to be counted as correctly predicted.

The output looks as follows:

```bash
27444   15310   12134   55.79   annot.gtf
29015   15310   13705   52.77   ES/genemark.gtf
```

**15,310** of genes in annotation (**55.79%** sensitivity) were exactly predicted. At the same time, **13,705** (**52.77%** specificity) predicted genes were not exactly correct.

Prediction is also often evaluated on the level of exact exon matches:

```bash
compare_intervals_exact.pl --f1 annotation.gtf --f2  ES/genemark.gtf --cds --verbose

156842  120999  35843   77.15   ../annot/annot.gtf
155699  120999  34700   77.71   genemark.gtf
```

 **120,999** of exons in annotation (**77.2%** sensitivity) were exactly predicted. At the same time, **34,700** (**77.7%** specificity) predicted exons were wrong.

The result of GeneMark-EP+, guided by cross-species proteins, can be evaluated with the same commands:

```bash
compare_intervals_exact.pl --f1 annotation.gtf --f2  EP/genemark.gtf --gene --verbose
compare_intervals_exact.pl --f1 annotation.gtf --f2  EP/genemark.gtf --cds --verbose
```

The results are summarized in the table below:

| Method                             | GeneMark-ES     | GeneMark-EP+           |
|------------------------------------|:----------------|:-----------------------|
| Gene Sensitivity (True Positives)  | 55.8 (15,310)   | 73.2  (20,085) |
| Gene Specificity (False Positives) | 52.8 (13,705)   | 67.4   (9,720) |
| Exon Sensitivity (True Positives)  | 77.2 (120,999)  | 81.6 (128,029) |
| Exon Specificity (False Positives) | 77.7 (34,700)   | 83.0  (26,253) |


About \~5,000 more annotated genes were exactly predicted by GeneMark-EP+ (17.4% percentage points increase in sensitivity). Conversely, \~4000 false positive genes are not present in the GeneMark-EP+ result (14.6% percentage points increase in specificity).

?> **Using publicly available cross-species proteins significantly improved the predictions.**

#### Evaluation of ProtHint protein hints

We can also directly evaluate how reliable the protein hints supplied by ProtHint are, especially in the high-confidence `evidence.gff` file:

```bash
compare_intervals_exact.pl --f1 annotation.gtf --f2  prothint.gtf --intron
compare_intervals_exact.pl --f1 annotation.gtf --f2  evidence.gtf --intron
```

| Method                             | All introns     | High-Confidence           |
|------------------------------------|:----------------:|:-----------------------:|
| Intron Sensitivity  |  87.9  | 84.3 |
| Intron Specificity  |  85.0  | **96.9** |

We can see that ProtHint predicted a significant portion of all annotated introns (87.9%). Furthermore, the specificity of introns predicted in the high-confidence group is high (**96.9%** of predicted introns are correct).

### Annotation Report

Without a reference annotation, the prediction has to be evaluated on its own. To visualize various summary statistics about the predicted gene set, [run the following script](output/evaluation?id=prediction-report):

```bash
./predictionReport.py genemark.gtf output/gmhmm.mod report.pdf
```

The resulting report [is saved here](output/reports/athal.pdf ':ignore').

Generally, users have some expectations about the expected number of genes, distribution of gene lengths, ratio of single-exon genes, etc. This report helps to quickly check whether these expectations were met.

### BUSCO Evaluation

> **TODO**

## Selection of a reliable gene set

In general, most false positive predictions are not supported by external evidence (see [section about reliable subset selection](/output/reliable_subset.md) for more details). We can thus filter the set of predicted genes by protein support to obtain more reliable prediction subsets:

```bash
selectSupportedSubsets.py genemark.gtf --fullSupport fullSupport.gtf --anySupport anySupport.gtf --noSupport noSupport.gtf
```

Using the same comparison script [as before](examples/novel_genome?id=comparison-of-genemark-es-and-genemark-ep-results), we can compare the accuracy of the selected subsets:

| GeneMark-EP+ Predictions           | All predictions | Any support | Full support | No support |
|------------------------------------|:----------------|:-----------------------|:------------------------|:----------------------|
| Gene Sensitivity (True Positives)  | 73.2  (20,085)  | 72.0 (19,756)          | 67.3 (18,477)           | 1.2 (329)             |
| Gene Specificity (False Positives) | 67.4   (9,720)  | 75.3 (6,476)           | 91.7 (1,680)            | 9.2 (3,244)           |
| Exon Sensitivity (True Positives)  | 81.6 (128,029)  | 80.6 (126,440)         | 68.8 (107,916)          | 1.0 (1,589)           |
| Exon Specificity (False Positives) | 83.0  (26,253)  | 88.0 (17,312)          | 97.9 (2,319)            | 15.09 (8,941)         |


The full prediction set contains **20,085** genes which exactly match the annotation. At the same time, there are **9,720** incorrectly predicted gene structures:
* By keeping genes with any external support, we can remove **3,244** false predictions at the cost of losing **329** correctly predicted genes.
* In a more conservative set where only genes with full external support are kept, **8,040** false predictions are removed at the cost of losing **1,607** correct genes.

Similar trade-offs can be observed on the exon-level. For example, (97.9%) exons in the set of fully supported genes are correct. This filtering comes at a price of losing \~16% of correctly predicted exons.