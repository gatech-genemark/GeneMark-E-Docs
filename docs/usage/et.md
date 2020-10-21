# GeneMark-ET

> TODO: Some description

## Running GeneMark-ET

The mandatory inputs to GeneMark-ET are the genomic sequence and introns aligned from RNA-Seq.

```bash
gmes_petap.pl --seq  genome.fasta.masked --ET rna_introns.gff --soft_mask auto --cores 8
```

The commonly used optional parameters (`--soft_mask auto` and `--cores 8`) are described in the [general info](usage/general.md) section.

### Adjusting the intron initialization score threshold

Parameter `--et_score N` can be used to adjust the score threshold for introns used in the initial parameter estimation. Only introns witch score > N (N = 10 by default) are used to initialize the GeneMark-ET model. The intron scores are read from the 6th column of the `gff` input.

This adjustment can be useful if:
* You have high RNA-Seq coverage and want to initialize the model with reliable introns only (by increasing the `et_score` threshold)
* You have low RNA-Seq coverage and want to add more introns into the initial model estimation (by decreasing the `et_score` threshold)

For example, to use all introns with score higher than 4 during initial parameter estimation, use:
```bash
gmes_petap.pl --seq  genome.fasta.masked --ET rna_introns.gff --soft_mask auto --cores 8 --et_score 4
```
