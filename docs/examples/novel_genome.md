# Annotation of a novel eukaryotic genome

This tutorial shows how to use GeneMark to predict genes in a sequence of a novel genome, for which no data than the genomic sequence itself are available.

We will cover data preparation, repeat masking, prediction itself, and how to evaluate the prediction result.

## Preparing the test data 

We will use Arabidopsis thaliana as an example genome in this tutorial. 

!> The following steps show how to prepare the example inputs and are thus irrelevant for an actual use-case of annotation of a novel genome. The tutorial itself starts with the [Downloading cross-species proteins section](examples/novel_genome?id=downloading-cross-species-proteins)

Download the genomic sequence from NCBI:

```bash
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/001/735/GCF_000001735.4_TAIR10.1/GCF_000001735.4_TAIR10.1_genomic.fna.gz
gunzip GCF_000001735.4_TAIR10.1_genomic.fna.gz
mv GCF_000001735.4_TAIR10.1_genomic.fna genome.fasta
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
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/001/735/GCF_000001735.4_TAIR10.1/GCF_000001735.4_TAIR10.1_genomic.gff.gz
gunzip GCF_000001735.4_TAIR10.1_genomic.gff.gz
mv GCF_000001735.4_TAIR10.1_genomic.gff annotation.gff
```
!> Actual novel genomes usually do not have a reference annotation available. In this tutorial, we will also show how to evaluate the results when the reference annotation is not available.

Convert the annotation to `.gtf` format with [GenomeTools](http://genometools.org/):

```bash
# 3 genes contain unexpected CDS phases, they need to be removed prior to the conversion
gt gff3_to_gtf <(grep -P -v "gene-DA397_mgp34|gene-DA397_mgp31|gene-ArthCp047" annotation.gff) > annotation.gtf
```

Add introns between CDS features (script `calc_introns_from_gtf` is a part of GeneMark-ES suite): 

```bash
calc_introns_from_gtf.pl --in annotation.gtf --out introns.gtf
cat introns.gtf >> annotation.gtf
```

## Downloading cross-species proteins

Let us assume there is no available RNA-Seq data for our novel genome. In such case, we can run GeneMark-ES to obtain purely computational prediction or we can use GeneMark-EP+ with **publicly available cross-species protein sequences** as an external evidence to improve the computational predictions. In this tutorial, we will show and compare both methods. For GeneMark-EP+, we will need to download protein sequences.

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

Prior to predicting protein coding genes, it is important to identify and mask repetitive elements. Since there is no existing repeat library for our novel genome, we will build our own with [RepeatModeler2](https://www.pnas.org/content/117/17/9451):

```bash
BuildDatabase -name genome genome.fasta
RepeatModeler -database genome -LTRStruct -pa 8
```

Use the repeat library to soft-mask repeats with [RepeatMasker](http://www.repeatmasker.org/):

```bash
RepeatMasker -lib genome-families.fa -xsmall genome.fasta -pa 16
```

The masked genome will be saved to  `genome.fasta.masked` file.

?> See the [section on masking in the usage page](usage/general?id=repeat-masking) for more details about repeat masking.

## Gene Prediction

Please refer to the [installation section](installation/download.md) to get GeneMark ready.

### Predicting genes with GeneMark-ES

We will start by using GeneMark-ES which works with the genomic sequence alone:

```bash
mkdir ES; cd ES
gmes_petap.pl --seq ../genome.fasta.masked --ES --soft_mask auto --cores 8
```

?> This procedure takes about X minutes on an 8-CPU machine.

### Predicting genes with GeneMark-EP+


The entire procedure described in this section can be achieved by running a single command:

```bash
gmes_petap.pl --seq genome.fasta.masked --EP --dbep proteins.fasta --soft_mask auto --cores 8
```

In the remainder of this section, we will split the above command into several steps to better explain how the prediction with proteins works. Moreover, the intermediate results (such as protein hints generated by ProtHint) can be useful on their own. You can skip to [prediction evaluation](examples/novel_genome?id=prediction-evaluation) if not interested in the extended procedure.

GeneMark-EP+ first needs to generate protein hints (introns, start and stop codons) from all reference proteins used on input. This task is efficiently solved by [ProtHint](https://github.com/gatech-genemark/ProtHint), included in the GeneMark-ES suite in the `ProtHint` folder.

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
> In the above commands, we used two different ProtHint outputs. `prothint.gff` contains all ProtHint predictions and all of these hints are used by GeneMark-EP+ to improve model training. `evidence.gff` contains a high-confidence subset of `prothint.gff`, these hints are enforced in the final GeneMark-EP+ prediction. Hints in `evidence.gff` pass stringent filtering criteria and [generally have > 95% specificity](examples/novel_genome?id=evaluation-of-prothint-protein-hints).

?> The runtime of GeneMark-EP+ is similar to that of GeneMark-ES, about X minutes on an 8-CPU machine.

## Prediction Evaluation

In this section, we will show how to evaluate the results with or without the reference annotation.

!> The prediction results shown in this tutorial may slightly differ from your own results. This difference is caused by stochasticity of repeat masking, i.e. two different run for RepeatMasker/RepeatModeler can produce slightly different masking coordinates.

### Evaluation Against reference annotation

Since we have a reference evaluation, we can directly evaluate how accurate the prediction is. Obviously, a reference evaluation is rarely available for a novel genome; this comparison serves to illustrate how accurate the algorithm is.

#### Comparison of GeneMark-ES and GeneMark-EP+ results

We start by [comparing the GeneMark-ES results to annotation](output/evaluation?id=comparing-against-a-reference) on the gene level. 

```bash
compare_intervals_exact.pl --f1 annotation.gtf --f2  ES/genemark.gtf --gene
```

!> In a gene-level comparison, full gene structures have to match the annotation **exactly** to be counted as correctly predicted.

The output looks as follows:

```bash
27559   15176   12383   55.07   annotation.gtf
27869   15176   12693   54.45   ES/genemark.gtf
```

**15,176** of genes in annotation (**55.07%** sensitivity) were exactly predicted. At the same time, **12,693** (**54.45%** specificity) predicted genes were not exactly correct.

Prediction is also often evaluated on the level of exact exon matches:

```bash
compare_intervals_exact.pl --f1 annotation.gtf --f2  ES/genemark.gtf --cds --verbose

156987  120442  36545   76.72   annotation.gtf
151891  120442  31449   79.30   ES/genemark.gtf
```

 **120,442** of exons in annotation (**76.72%** sensitivity) were exactly predicted. At the same time, **31,449** (**79.30%** specificity) predicted exons were wrong.

The result of GeneMark-EP+, guided by cross-species proteins, can be evaluated with the same commands:

```bash
compare_intervals_exact.pl --f1 annotation.gtf --f2  genemark.gtf --gene --verbose
compare_intervals_exact.pl --f1 annotation.gtf --f2  genemark.gtf --cds --verbose
```

The results are summarized in the table below:

| Method                             | GeneMark-ES     | GeneMark-EP+           |
|------------------------------------|:----------------|:-----------------------|
| Gene Sensitivity (True Positives)  | 55.1 (15,176)   | 72.3 (19,910)          |
| Gene Specificity (False Positives) | 54.5 (12,693)   | 69.5 (8,755)           |
| Exon Sensitivity (True Positives)  | 76.7 (120,442)  | 81.2 (127,452)         |
| Exon Specificity (False Positives) | 79.3 (31,449)   | 84.7 (22,956)          |


About \~5,000 more annotated genes were exactly predicted by GeneMark-EP+ (17.2 percentage points increase in sensitivity). Conversely, \~4000 false positive genes are not present in the GeneMark-EP+ result (15 percentage points increase in specificity).

?> **Using publicly available cross-species proteins significantly improved the prediction.**

#### Evaluation of ProtHint protein hints

We can also directly evaluate how reliable the protein hints supplied by ProtHint are, especially in the high-confidence `evidence.gff` file:

```bash
compare_intervals_exact.pl --f1 annotation.gtf --f2  prothint.gtf --intron --no_phase
compare_intervals_exact.pl --f1 annotation.gtf --f2  evidence.gtf --intron --no_phase
```

| ProtHint                      | All introns    | High-Confidence     |
|-------------------------------|:--------------:|:-------------------:|
| Intron Sensitivity            |  87.6          | 84.1                |                  
| Intron Specificity            |  86.6          | **97.5**            |

We can see that ProtHint predicted a significant portion of all annotated introns (87.6%). Furthermore, the specificity of introns predicted in the high-confidence group is high (**97.5%** of predicted introns are correct).

### Annotation Report

Without a reference annotation, the prediction has to be evaluated on its own. To visualize various summary statistics about the predicted gene set, [run the following script](output/evaluation?id=prediction-report):

```bash
./predictionReport.py genemark.gtf output/gmhmm.mod report.pdf
```

The resulting report [is saved here](output/reports/athal.pdf ':ignore').

Generally, users have some expectations about the number of genes, distribution of gene lengths, ratio of single-exon genes, etc. This report helps to quickly check whether these expectations were met.

### BUSCO Evaluation

Without a reference annotation, we can use [BUSCO](evaluation?id=busco-evaluation) to evaluate the completeness of our prediction.

First, we need to select the closest BUSCO lineage. To list available lineage, use:

```bash
python3 busco --list-datasets
```

`brassicales_odb10` is the closest available set.

Next, we need to translate the predicted genes to proteins:

```bash
get_sequence_from_GTF.pl genemark.gtf genome.fasta
```

To run BUSCO, use:

```bash
python3 busco -m protein -i prot_seq.faa -o EP_plus -l brassicales_odb10 --cpu 8
```

Visualize the BUSCO result:

```bash
python3 generate_plot.py -wd EP_plus
```

<br>

![A_thaliana_busco](busco/athal.png ':size=700') 

The BUSCO result shows that 4534 out of 4596 (98.7%) expected BUSCO genes were completely found in the prediction. 9 genes were found partially and 53 (1.1%) were missing.

## Selection of a reliable gene set

In general, most false positive predictions are not supported by external evidence (see [section about reliable subset selection](/output/reliable_subset.md) for more details). We can thus filter the set of predicted genes by protein support to obtain more reliable prediction subsets:

```bash
selectSupportedSubsets.py genemark.gtf --fullSupport fullSupport.gtf --anySupport anySupport.gtf --noSupport noSupport.gtf
```

Using the same comparison script [as before](examples/novel_genome?id=comparison-of-genemark-es-and-genemark-ep-results), we can compare the accuracy of the selected subsets:

| GeneMark-EP+ Predictions           | All predictions | Any support            | Full support            | No support            |
|------------------------------------|:----------------|:-----------------------|:------------------------|:----------------------|
| Gene Sensitivity (True Positives)  | 72.3 (19,910)   | 71.1 (19,596)          | 66.6 (18,357)           | 1.1  (314)          |
| Gene Specificity (False Positives) | 69.5 (8,755)    | 76.6 (6,002)           | 91.8 (1,630)            | 10.2 (2,753)          |
| Exon Sensitivity (True Positives)  | 81.2 (127,452)  | 80.2 (125,846)         | 68.5 (107,475)          | 1.0  (1,606)          |
| Exon Specificity (False Positives) | 84.7 (22,956)   | 89.3 (15,087)          | 98.0 (2,191)            | 17.0 (7,869)          |


The full prediction set contains **19,910** genes which exactly match the annotation. At the same time, there are **8,755** incorrectly predicted gene structures:
* By keeping genes with any external support, we can remove **2,753** false predictions at the cost of losing **314** correctly predicted genes.
* In a more conservative set where only genes with full external support are kept, **7,125** false predictions are removed at the cost of losing **1,553** correct genes.

Similar trade-offs can be observed on the exon level. For example, **98.0%** exons in the set of fully supported genes are correct. This filtering comes at the price of losing **\~16%** of correctly predicted exons.
