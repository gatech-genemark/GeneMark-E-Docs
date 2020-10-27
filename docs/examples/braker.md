# Annotation of a novel eukaryotic genome with BRAKER

This tutorial shows how to use [BRAKER](usage/braker.md) guided by RNA-Seq and homologous protein sequences to annotate a novel genome.

## Preparing the test data

As in the previous example, we will be using the genome of A. thaliana. See the steps for [preparing the genomic sequence](examples/novel_genome?id=preparing-the-test-data) and [reference protein data](examples/novel_genome?id=downloading-cross-species-proteins).

### RNA-Seq

Here we will also illustrate how to prepare RNA-Seq data for BRAKER. See the [usage page for more details about RNA-Seq mapping](usage/preparing_rna).

Download a species-specific RNA-Seq library from the Sequence Read Archive with [SRA Toolkit](https://github.com/ncbi/sra-tools):

```bash
prefetch SRR934391
```

Dump reads into `.fastq` files:

```bash
fastq-dump --split-files SRR934391
```

?> For this example, we arbitrarily chose [SRR934391](https://www.ncbi.nlm.nih.gov/sra/?term=SRR934391); however, any RNA-Seq reads in `.fastq` format can be used.

We will use [STAR](https://github.com/alexdobin/STAR) to map the reads to the genome. First, generate genome indices:

```bash
STAR --runMode genomeGenerate --genomeFastaFiles genome.fasta --genomeDir db
```

Next, align the reads using STAR's 2-pass procedure:

```bash
STAR --readFilesIn SRR934391_1.fastq SRR934391_2.fastq \
    --alignIntronMin 20 --alignIntronMax 500000 \
    --outSAMtype BAM SortedByCoordinate --twopassMode Basic --genomeDir db
```

The output will be save to `Aligned.sortedByCoord.out.bam`.

## Gene Prediction

Please refer to the [BRAKER documentation](https://github.com/Gaius-Augustus/BRAKER) to get BRAKER ready.

Since we already ran ProtHint in the [GeneMark-EP+ example](examples/novel_genome?id=predicting-genes-with-genemark-ep), we can directly supply the mapped proteins to BRAKER.

```bash
braker.pl --genome ../genome.fasta.masked --hints prothint/prothint_augustus.gff --bam Aligned.sortedByCoord.out.bam --etpmode --softmasking --cores 16
```

?> The runtime of this BRAKER run is \~8 hours on an 8-CPU machine

BRAKER can also be run directly with protein sequences: just replace the `--hints prothint/prothint_augustus.gff` option with `--prot_seq proteins.fasta`.

## Prediction Evaluation

!> The prediction results shown in this tutorial may slightly differ from your own results. This difference is caused by stochasticity of repeat masking, i.e. two different run for RepeatMasker/RepeatModeler can produce slightly different masking coordinates.

Since we have a reference evaluation, we can directly evaluate how accurate the prediction is. Obviously, a reference evaluation is rarely available for a novel genome; this comparison serves to illustrate how accurate the algorithm is in comparison with GeneMark-EP+ alone.

### Evaluation Against reference annotation

To compare BRAKER predictions against the reference annotation, we will use the same procedure described in the [A. thaliana example](examples/novel_genome?id=evaluation-against-reference-annotation). The BRAKER results are located in `braker/augustus.hints.gtf`.


| Method                             | BRAKER          |
|------------------------------------|:----------------|
| Gene Sensitivity (True Positives)  | 77.1 (21,250)   |
| Gene Specificity (False Positives) | 72.6 (8,039)    |
| Exon Sensitivity (True Positives)  | 82.8 (129,937)  |
| Exon Specificity (False Positives) | 86.6 (20,059)   |

Notice the accuracy increase in comparison with [GeneMark-EP+ alone](examples/novel_genome?id=evaluation-against-reference-annotation) (which is still a core part of BRAKER).

### Annotation Report

Unlike in [GeneMark-ES suite](output/evaluation?id=prediction-report), BRAKER does not have a script to automatically generate a gene prediction report. We are working on adding such script.

### BUSCO Evaluation

Without a reference annotation, we can use [BUSCO](evaluation?id=busco-evaluation) to evaluate the completeness of our prediction.

First, we need to select the closest BUSCO lineage. To list available lineage, use:

```bash
python3 busco --list-datasets
```

`brassicales_odb10` is the closest available set.

BRAKER automatically translates predicted genes to proteins and saves them to `braker/augustus.hints.aa` file.

To run BUSCO, use:

```bash
python3 busco -m protein -i braker/augustus.hints.aa -o BRAKER -l brassicales_odb10 --cpu 8
```

Visualize the BUSCO result:

```bash
python3 generate_plot.py -wd BRAKER
```
<br>

![athal_braker_busco](busco/athal_braker.png ':size=700') 

The BUSCO result shows that 4567 out of 4596 (**99.4%**) expected BUSCO genes were completely found in the prediction. 8 genes were found partially and 21 (**0.4%**) were missing. 

## Selection of a reliable gene set

In general, most false positive predictions are not supported by external evidence. We can thus filter the set of predicted genes by protein support to obtain more reliable prediction subsets. Unlike in [GeneMark-ES suite](output/reliable_subset.md), this procedure is not yet automated in BRAKER.

As a temporary workaround, you we use the `compare_intervals_exact.pl` script (located in both BRAKER and GeneMark-ES suite) to select genes supported by any hints with the following procedure.

1. Find introns which have external support:

```bash
cd braker
compare_intervals_exact.pl --f1 augustus.hints.gtf --f2 hintsfile.gff --intron --out supportedIntrons --shared12 --original 2
```

2. Select gene IDs of these introns

```bash
grep -o -P "gene_id[^;]+;" supportedIntrons | cut -f2 -d " "  > supportedIDs
```

3. Do the same for start and stop codons. 

!> Start codons are named just `start` in the `hintsfile.gff`. We need to rename them to `start_codon` before computing the matches. The same applies for stop codons.

```bash
compare_intervals_exact.pl --f1 augustus.hints.gtf --f2 <(sed "s/\tstart\t/\tstart_codon\t/" hintsfile.gff) \
    --start --out supportedStarts --shared12 --original 1
compare_intervals_exact.pl --f1 augustus.hints.gtf --f2 <(sed "s/\tstop\t/\tstop_codon\t/" hintsfile.gff) \
    --stop --out supportedStops --shared12 --original 1
grep -o -P "gene_id[^;]+;" supportedStarts | cut -f2 -d " "  >> supportedIDs
grep -o -P "gene_id[^;]+;" supportedStops | cut -f2 -d " "  >> supportedIDs
```

4. Select all genes with these IDs:

```bash
grep -Ff supportedIDs augustus.hints.gtf  > augustus.hints.any_support.gtf
```

We can now use the [compare script again](examples/novel_genome?id=evaluation-against-reference-annotation) to compare the supported subset against reference annotation.

| GeneMark-EP+ Predictions           | All predictions | Any support            |
|------------------------------------|:----------------|:-----------------------|
| Gene Sensitivity (True Positives)  | 77.1 (21,250)   | 73.4 (20,222)          |
| Gene Specificity (False Positives) | 72.6 (8,039)    | 81.0 (4,753)           |
| Exon Sensitivity (True Positives)  | 82.8 (129,937)  | 80.0 (125,511)         |
| Exon Specificity (False Positives) | 86.6 (20,059)   | 91.5 (11,704)          |


The results are similar to the one we observed with [GeneMark-EP+](examples/novel_genome?id=selection-of-a-reliable-gene-set).
