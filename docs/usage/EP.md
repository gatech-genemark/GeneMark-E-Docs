# GeneMark-EP+

GeneMark-EP+ [[1](https://academic.oup.com/nargab/article/2/2/lqaa026/5836691)] is a semi-supervised **self-training** algorithm that identifies protein coding genes in eukaryotic genomes. Compared to fully unsupervised GeneMark-ES, GeneMark-EP+ uses homologous protein sequences of any evolutionary distance to improve both training and predictions.

## Running GeneMark-EP+ with automatic protein hint generation

In this mode, ProtHint is automatically run to generate hints from supplied proteins. The hints are subsequently used by GeneMark-EP+ during model training and gene prediction steps.

```bash
gmes_petap.pl --seq genome.fasta.masked --EP --dbep proteins.fasta --soft_mask auto --cores 8
```

?> See the [preparing protein evidence](usage/preparing_proteins.md) page for more details on how to prepare the `proteins.fasta` input.

The commonly used optional parameters (`--soft_mask auto` and `--cores 8`) are described in the [general info](usage/general.md) section.

## Running ProtHint outside of GeneMark-EP+

If you want to have more control over the way protein hints are generated from protein input, you can execute ProtHint separately and plug its output into GeneMark-EP+.

### ProtHint

ProtHint is distributed as part of the GeneMark-ES suite (inside `ProtHint` folder); it can also be obtained separately from https://github.com/gatech-genemark/ProtHint.

To run ProtHint, use the following command:

```bash
prothint.py genome.fasta.masked proteins.fasta  --workdir output_folder
```

* ProtHint runs GeneMark-ES to generate gene seeds. If you already have a GeneMark-ES output, you can supply it to ProtHint with `--geneSeeds ES/genemark.gtf` option.
* Number of parallel CPU threads can be controlled by `--threads N` option. Contrary to GeneMark-E\*, ProtHint uses all available threads by default.
* To display a full list of options offered by ProtHint, run `prothint.py --help`

### Using ProtHint output in GeneMark-EP+

ProtHint generates two main outputs:

* `prothint.gff` contains a set of all reported hints
* `evidence.gff` contains a high-confidence subset of `prothint.gff`. Hints in this output pass stringent filtering criteria and generally have > 95% specificity.

?> ProtHint also creates `prothint_augustus.gff` output which can be directly used by [BRAKER](https://github.com/Gaius-Augustus/BRAKER) and [AUGUSTUS](https://github.com/Gaius-Augustus/Augustus).

These outputs are used by GeneMark-EP+ in the following way:

```bash
gmes_petap.pl --seq genome.fasta.masked  --EP prothint.gff --evidence evidence.gff  --soft_mask auto --cores 8
```

All hints in `prothint.gff` are used during model training. The high-confidence hints in `evidence.gff` are enforced in the final prediction. 

?> You can omit the `--evidence evidence.gff` option to prevent the enforcement of high-confidence hints. In such a case, the algorithm only uses protein hints to improve parameter estimation in training, as in [GeneMark-ET](usage/et.md).