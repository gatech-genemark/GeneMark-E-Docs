# GeneMark-ES

> TODO: Some description

## Running GeneMark-ES

The only mandatory input to GeneMark-ES is the genomic sequence.

```bash
gmes_petap.pl --seq genome.fasta.masked --ES --soft_mask auto --cores 8
```

The commonly used optional parameters (`--soft_mask auto` and `--cores 8`) are described in the [general info](usage/general.md) section.
