# Annotation of a novel eukaryotic genome with BRAKER

This tutorial shows how to use [BRAKER](usage/braker.md) to annotate a novel genome.

## Preparing the test data

As in the previous example, we will be using the genome of A. thaliana. See the steps for [preparing the genomic sequence](examples/novel_genome?id=preparing-the-test-data) and [reference protein data](examples/novel_genome?id=downloading-cross-species-proteins).

### RNA-Seq

Here we will also illustrate how to prepare RNA-Seq data and use them together with proteins in BRAKER.

> **TODO** RNA-Seq preparation steps from a custom library.

## Gene Prediction

Please refer to the [BRAKER description](usage/braker.md) to get BRAKER ready.


## Prediction Evaluation

Since we have a reference evaluation, we can directly evaluate how accurate the prediction is. Obviously, a reference evaluation is rarely available for a novel genome; this comparison serves to illustrate how accurate the algorithm is in comparison with GeneMark-EP+ alone.

### Evaluation Against reference annotation

To compare BRAKER predictions against the reference annotation, we will use the same procedure described in the [A. thaliana example](examples/novel_genome?id=evaluation-against-reference-annotation).

> **TODO** Table with BRAKER results

### Annotation Report

Without a reference annotation, the prediction has to be evaluated on its own. [Similar to GeneMark](output/evaluation?id=prediction-report), BRAKER also offers a script to visualize the annotation results:

```bash
./predictionReport.py braker.gtf  report.pdf
```

!> TODO: slight modificiations are needed to make the script compatible with BRAKER gtf.

Generally, users have some expectations about the number of genes, distribution of gene lengths, ratio of single-exon genes, etc. This report helps to quickly check whether these expectations were met.

### BUSCO Evaluation

Without a reference annotation, we can use [BUSCO](evaluation?id=busco-evaluation) to evaluate the completeness of our prediction.

First, we need to select the closest BUSCO lineage. To list available lineage, use:

```bash
python3 busco --list-datasets
```

`ascomycota_odb10` is the closest available set.

BRAKER automatically translates predicted genes to proteins and saves to `XX` file.

To run BUSCO, use:

```bash
python3 busco -m protein -i XX -o BRAKER -l ascomycota_odb10 --cpu 8
```

Visualize the BUSCO result:

```bash
python3 generate_plot.py -wd BRAKER
```
<br>

![athal_braker_busco](busco/athal_braker.png ':size=700') 

!> **TODO** COMMENT on how different the result is compared to GM

## Selection of a reliable gene set

In general, most false positive predictions are not supported by external evidence (see [section about reliable subset selection](/output/reliable_subset.md) for more details). [Similar to GeneMark](output/reliable_subset.md), BRAKER contains a script to filter predicted gene by external evidence support to obtain more reliable predictions.

>! TODO: Slight modification to the GeneMark selection script to make it compatible with BRAKER