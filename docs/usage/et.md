# GeneMark-ET

GeneMark-ET [[1](https://academic.oup.com/nar/article/42/15/e119/2434516)] is a semi-supervised **self-training** algorithm that identifies protein-coding genes in eukaryotic genomes. Compared to fully unsupervised GeneMark-ES, GeneMark-ET integrates information on mapped RNA-Seq reads to improve its training.

## Running GeneMark-ET

The mandatory inputs to GeneMark-ET are the genomic sequence and introns aligned from RNA-Seq.

```bash
gmes_petap.pl --seq  genome.fasta.masked --ET rna_introns.gff --soft_mask auto --cores 8
```

?> See the [preparing RNA-Seq evidence](usage/preparing_rna.md) page for more details on how to prepare the `rna_introns` input.

The commonly used optional parameters (`--soft_mask auto` and `--cores 8`) are described in the [general info](usage/general.md) section.

### Adjusting the intron initialization score threshold

Parameter `--et_score N` can be used to adjust the score threshold for introns used in the initial parameter estimation. Only introns witch score > N (N = 10 by default) are used to initialize the GeneMark-ET model. The intron scores are read from the 6th column of the `gff` input.

This adjustment can be useful if:
* You have high RNA-Seq coverage and want to initialize the model with reliable introns only (by increasing the `et_score` threshold)
* You have low RNA-Seq coverage and want to add more introns into the initial model estimation (by decreasing the `et_score` threshold)

For example, to use all introns with a score higher than 4 during initial parameter estimation, use:

```bash
gmes_petap.pl --seq  genome.fasta.masked --ET rna_introns.gff --soft_mask auto --cores 8 --et_score 4
```
