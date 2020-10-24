# GeneMark-ES

GeneMark-ES [[1](https://academic.oup.com/nar/article/33/20/6494/1082033), [2](https://genome.cshlp.org/content/18/12/1979)] is an unsupervised **self-training** algorithm that identifies protein-coding genes in eukaryotic genomes.

## Running GeneMark-ES

The only mandatory input to GeneMark-ES is the genomic sequence.

```bash
gmes_petap.pl --seq genome.fasta.masked --ES --soft_mask auto --cores 8
```

The commonly used optional parameters (`--soft_mask auto` and `--cores 8`) are described in the [general info](usage/general.md) section.
