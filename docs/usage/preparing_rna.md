# Preparation of RNA-Seq Evidence

Many RNA-Seq aligners (STAR, HISAT2, TopHat2) can be used to align RNA-Seq reads to genome. In this description, we will show how to align the reads with [STAR](https://github.com/alexdobin/STAR).

## Aligning RNA-Seq reads

First, generate genome indices for the assembly:

```bash
STAR --runMode genomeGenerate --genomeFastaFiles genome.fasta --genomeDir db
```

Next, align the reads using STAR's 2-pass procedure:

```bash
STAR --readFilesIn pair_1.fastq pair_2.fastq \
    --alignIntronMin 20 --alignIntronMax 500000 \
    --outSAMtype BAM SortedByCoordinate --twopassMode Basic --genomeDir db
```

* The option `--outSAMtype BAM SortedByCoordinate`  outputs alignments in BAM format which is compatible with BRAKER.
* Options `--alignIntronMin 20` and `--alignIntronMax 500000` generally work well with most eukaryotes, however, feel free adjust them to fit your genome.
* `--twopassMode Basic`: In the 2-pass mapping job, STAR will map the reads twice. In the 1st pass, the novel junctions will be detected and inserted into the genome indices. In the 2nd pass, all reads will be re-mapped using annotated (from the GTF file) and novel (detected in the 1st pass) junctions. While this procedure doubles the run-time, it significantly increases sensitivity to novel splice junctions. In the absence of annotations, this option is strongly recommended. [[1]](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4631051/)

The output is saved to `Aligned.sortedByCoord.out.bam` file. STAR also outputs all mapped splice junctions to `SJ.out.tab`.

## Preparing a gff file with introns for GeneMark-ET

To prepare a `.gff` file with introns ready to be [used by GeneMark-ET](http://localhost:3000/#/usage/et), run the following script located in the GeneMark-ES suite:

```bash
star_to_gff.pl --star SJ.out.tab --gff rna_introns.gff
```
