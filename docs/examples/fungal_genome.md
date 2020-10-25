# Annotation of a novel fungal genome

This tutorial shows how to use GeneMark to predict genes in a sequence of a novel **fungal** genome, for which no data than the genomic sequence itself are available. Many steps are identical to general instructions shown in the [tutorial on annotation of a novel genomes](/examples/novel_genome.md). We will highlight important differences.

## Preparing the test data 

We will use Schizosaccharomyces pombe as an example genome in this tutorial. 

!> The following steps show how to prepare the example inputs and are thus irrelevant for an actual use-case of annotation of a novel genome. The tutorial itself starts with the [Downloading cross-species proteins section](examples/fungal_genome?id=downloading-cross-species-proteins)

Download the genomic sequence from NCBI:

```bash
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/002/945/GCF_000002945.1_ASM294v2/GCF_000002945.1_ASM294v2_genomic.fna.gz
gunzip GCF_000002945.1_ASM294v2_genomic.fna.gz
mv GCF_000002945.1_ASM294v2_genomic.fna genome.fasta
```

Clean the fasta headers to only contain contig ID:

```bash
sed -i -E 's/(>[^ ]+) .+/\1/' genome.fasta
```

The genome comes with repeat-masking. Remove the masking to simulate a novel genome:

```bash
probuild --reformat_fasta --in genome.fasta --out tmp_genome.fasta --uppercase 1 --letters_per_line 60 --original
mv tmp_genome.fasta genome.fasta
```
?> The `probuild` program is included in the GeneMark-ES suite

Download annotation for result evaluation:

```bash
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/002/945/GCF_000002945.1_ASM294v2/GCF_000002945.1_ASM294v2_genomic.gff.gz
gunzip GCF_000002945.1_ASM294v2_genomic.gff.gz 
mv GCF_000002945.1_ASM294v2_genomic.gff annotation.gff
```
!> Actual novel genomes usually do not have a reference annotation available. In this tutorial, we will also show how to evaluate the results when the reference annotation is not available.

Convert the annotation to `.gtf` format with [GenomeTools](http://genometools.org/):

```bash
gt gff3_to_gtf annotation.gff > annotation.gtf
```

Add introns between CDS features (script `calc_introns_from_gtf` is a part of GeneMark-ES suite): 

```bash
calc_introns_from_gtf.pl --in annotation.gtf --out introns.gtf
cat introns.gtf >> annotation.gtf
```

## Downloading cross-species proteins

Let us assume there is no available RNA-Seq data for our novel genome. In such case, we can run GeneMark-ES to obtain purely computational prediction or we can use GeneMark-EP+ with **publicly available cross-species protein sequences** as an external evidence to improve the computational predictions. In this tutorial we will show and compare both methods. For GeneMark-EP+, we will need to download protein sequences.

Let us assume that our novel genome is a deeply rooted plant species with no sequenced close relatives. Therefore, we will be using **all Fungal proteins** from OrthoDB as a general source of protein sequences.

?> GeneMark-EP+ can utilize even remotely homologous proteins to improve its predictions.

Download the proteins and concatenate them into a single `.fasta` file:

```bash
wget https://v100.orthodb.org/download/odb10_fungi_fasta.tar.gz
tar xvf odb10_fungi_fasta.tar.gz
cat fungi/Rawdata/* > proteins.fasta
```

Please refer to the [Preparing protein evidence](usage/preparing_proteins.md) section for more details about reference protein preparation.

> [!DANGER]
> To simulate the absence of closely related species in this experiment, we will remove proteins of species in the Schizosaccharomyces genus from the input protein set. No such procedure is necessary with an actual novel genome.
> ```bash
> # Remove the previously created proteins.fasta file
> rm proteins.fasta
> # Only include proteins outside of the Schizosaccharomyces genus. 402676.1, 483514_1, and 653667.1 are IDs of Schizosaccharomyces species
> ls fungi/Rawdata/* | grep -v -P "402676.1.fs|483514.1.fs|653667.1.fs" | xargs cat >> proteins.fasta
> ```


## Repeat Masking

Contrary to other eukaryotes, repeat masking is not a critical step for annotating small, compact fungal genomes. Therefore, we can safely skip the repeat masking.

## Gene Prediction

Please refer to the [installation section](installation/download.md) to get GeneMark ready.

### Predicting genes with GeneMark-ES

We will start by using GeneMark-ES which works with the genomic sequence alone:

```bash
mkdir ES; cd ES
gmes_petap.pl --fungus --seq ../genome.fasta --ES --cores 8
```

!> Notice the `--fungus` flag

?> This procedure takes \~20 minutes on an 8-CPU machine.

### Predicting genes with GeneMark-EP+

Run GeneMark-EP+ with the following command, do not forget the `--fungus` flag:

```bash
gmes_petap.pl --fungus --seq genome.fasta --EP --dbep proteins.fasta --cores 8
```

?> The runtime of GeneMark-EP+ \~45 minutes on an 8-CPU machine, including the protein mapping of all fungal proteins.

> [!INFO]
> See the [example with A. thaliana](novel_genome?id=predicting-genes-with-genemark-ep) for more details about a GeneMark-EP+ run.


## Prediction Evaluation

!> The prediction results shown in this tutorial may slightly differ from your own results. In fungal GeneMark, this difference is caused by stochastic nature of branch point motif estimation.

In this section we will show how to evaluate the results with or without the reference annotation.

### Evaluation Against reference annotation

Since we have a reference evaluation, we can directly evaluate how accurate the prediction is. Obviously, a reference evaluation is rarely available for a novel genome; this comparison serves to illustrate how accurate the algorithm is.

#### Comparison of GeneMark-ES and GeneMark-EP+ results

To compare GeneMark-ES and GeneMark-EP+ predictions, we will use procedure described in the [A. thaliana example](examples/novel_genome?id=evaluation-against-reference-annotation).

The accuracy results are summarized in the table below:

| Method                             | GeneMark-ES     | GeneMark-EP+           |
|------------------------------------|:----------------|:-----------------------|
| Gene Sensitivity (True Positives)  | 82.6 (4,227)    | 83.7 (4,285)          |
| Gene Specificity (False Positives) | 84.9 (750)      | 84.3 (796)            |
| Exon Sensitivity (True Positives)  | 88.2 (8,979)    | 89.1 (9,070)          |
| Exon Specificity (False Positives) | 89.8 (1,016)    | 88.7 (1,155)          |


Compared to the [A. thaliana example](examples/novel_genome?id=evaluation-against-reference-annotation), the difference in accuracy between GeneMark-ES and GeneMark-EP+ is quite small. The main reason for this difference is that the prediction results of fungal GeneMark-ES are quite accurate and thus difficult to improve. Another contributing factor is a relative remoteness of reference proteins (see below).

#### Evaluation of ProtHint protein hints

Compared to the [A. thaliana example](examples/novel_genome?id=evaluation-against-reference-annotation), the sensitivity coverage of protein hints (60%) is lower. The reason for lower sensitivity is as follows: After [artificially removing proteins of species within the same genus](examples/fungal_genome?id=downloading-cross-species-proteins), the closest relatives in the reference proteins set were from a **different taxonomic class**. Note that the specificity of high-confidence hints (**98.8%**) is not negatively affected by the remoteness of input proteins.


| ProtHint                             | All introns     | High-Confidence           |
|------------------------------------|:----------------:|:-----------------------:|
| Intron Sensitivity  |  60.0  | 40.7 |
| Intron Specificity  |  75.4  | **98.8** |


### Annotation Report

Without a reference annotation, the prediction has to be evaluated on its own. To visualize various summary statistics about the predicted gene set, [run the following script](output/evaluation?id=prediction-report):

```bash
./predictionReport.py genemark.gtf output/gmhmm.mod report.pdf
```

The resulting report [is saved here](output/reports/spombe.pdf ':ignore').

!> Notice the additional statistics about branch point in this report (not present in the [report of A. thaliana](output/reports/athal.pdf ':ignore')). Branch point model is one of the reasons why the fungal predictions are very accurate even in the GeneMark-ES mode.

Generally, users have some expectations about the number of genes, distribution of gene lengths, ratio of single-exon genes, etc. This report helps to quickly check whether these expectations were met.

### BUSCO Evaluation

Without a reference annotation, we can use [BUSCO](evaluation?id=busco-evaluation) to evaluate the completeness of our prediction.

First, we need to select the closest BUSCO lineage. To list available lineage, use:

```bash
python3 busco --list-datasets
```

`ascomycota_odb10` is the closest available set.

Next, we need to translate the predicted genes to proteins:

```bash
get_sequence_from_GTF.pl genemark.gtf genome.fasta
```

To run BUSCO, use:

```bash
python3 busco -m protein -i prot_seq.faa -o EP_plus -l ascomycota_odb10 --cpu 8
```

Visualize the BUSCO result:

```bash
python3 generate_plot.py -wd EP_plus
```
<br>

![S_pombe_busco](busco/spombe.png ':size=700') 

!> Surprisingly, the BUSCO evaluation shows many missing genes (301 out of 1706, 17.6%). BUSCO evaluation of the reference annotation shows a similar proportion of missing genes (16.9%). Therefore, the low BUSCO score might be related to specific properties of the genome itself rather than caused by a low-quality prediction. 


## Selection of a reliable gene set

In general, most false positive predictions are not supported by external evidence (see [section about reliable subset selection](/output/reliable_subset.md) for more details). We can thus filter the set of predicted genes by protein support to obtain more reliable prediction subsets:

```bash
selectSupportedSubsets.py genemark.gtf --fullSupport fullSupport.gtf --anySupport anySupport.gtf --noSupport noSupport.gtf
```

Using the same comparison script [as before](examples/novel_genome?id=comparison-of-genemark-es-and-genemark-ep-results), we can compare the accuracy of the selected subsets:

| GeneMark-EP+ Predictions           | All predictions | Any support            | Full support            | No support            |
|------------------------------------|:----------------|:-----------------------|:------------------------|:----------------------|
| Gene Sensitivity (True Positives)  | 83.7 (4,285)    | 50.9 (2,604)           | 25.80 (1,321)           | 32.8 (1,681)          |
| Gene Specificity (False Positives) | 84.3 (796)      | 87.9 (360)             | 92.25 (111)             | 79.4 (436)            |
| Exon Sensitivity (True Positives)  | 89.1 (9,070)    | 64.6 (6,573)           | 31.85 (3,243)           | 24.5 (2,497)          |
| Exon Specificity (False Positives) | 88.7 (1,155)    | 92.9 (503)             | 96.43 (120)             | 79.3 (652)            |


The full prediction set contains **4,285** genes which exactly match the annotation. At the same time, there are **796** incorrectly predicted gene structures:
* By keeping genes with any external support, we can remove **436** false predictions at the cost of losing **1,681** correctly predicted genes.
* In a more conservative set where only genes with full external support are kept, **685** false predictions are removed at the cost of losing **2,965** correct genes.

!> Notice the big difference between this filtering and the one shown in [A. thaliana example](examples/novel_genome?id=selection-of-a-reliable-gene-set). Because the input proteins are quite remote (closest species is from a diferent taxonomic class), many correct predictions (\~40%) do not have any external support.


# Annotation of an intron-sparse fungal genome

> **This section is still a work in progress**

## Preparing the test data 

Same (give links to gtf and annot), also no repeat masking

## Gene Prediction

Same steps.

At the first glance, the predictions look...

We need to take a look at the number of predicted introns:

| Method                             | GeneMark-ES     | GeneMark-EP+           |
|------------------------------------|:----------------|:-----------------------|
| Gene Sensitivity (True Positives)  | 76.7 (4,589)    | 81.20 ()          |
| Gene Specificity (False Positives) | 83.9 (879)      | 74.02 ()          |
| Exon Sensitivity (True Positives)  | 74.2 (4,679)    |  80.67 ()          |
| Exon Specificity (False Positives) | 80.0 (1,169)    |  64.26 ()          |


Both algorithms struggle with the low number of introns. TODO: More discussion

| Method                               | GeneMark-ES     | GeneMark-EP+           |
|--------------------------------------|:----------------|:-----------------------|
| Intron Sensitivity (True Positives)  | 14.6 (41)       | 73.93 (207)            |
| Intron Specificity (False Positives) | 10.8 (339)      | 15.24 (1151)           |

Caused by the low number of introns in the genome.

ES: Learns not BP motif (not enough data)
EP learns a strong BP motif, overpredicts because there is in fact not many.

