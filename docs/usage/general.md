# General Usage Tips

## Repeat Masking

Repeat masking is essential for accurate gene prediction with GeneMark-E\*, especially in larger (>100 Mbp) eukaryotic genomes.

> [!INFO]
> **The benefits of repeat masking are two-fold**
> * Avoiding predictions of false positive gene structures in repetitive and low-complexity regions, thus increasing prediction **Specificity**.
> * Due to the self-training nature of GeneMark-E, false prediction in repetitive regions can negatively affect model training, thus decreasing prediction **Sensitivity**.

### De novo Repeat Masking

A database of species-specific transposable elements (TEs) is unlikely to be available for a novel genome. Since the types and sequences of TEs are highly variable across species, we recommend to build a de novo repeat database with [RepeatModeler2](https://www.pnas.org/content/117/17/9451):

```
BuildDatabase -name genome genome.fasta
RepeatModeler -database genome -LTRStruct
```

The repeat library can then be used by [RepeatMasker](http://www.repeatmasker.org/) to soft-mask both simple and interspersed repeats: 

```
RepeatMasker -lib genome-families.fa -xsmall genome.fasta
```

The resulting soft-masked genome is located in `genome.fasta.masked` file.

!> **Importance of soft-masking:** The use of soft-masking (`-xsmall` option), i.e. repeat regions are represented by lower-case letter, leads to better results than hard-masking, i.e. replacing repeats by the letter **N**. Soft-masking enables GeneMark to ignore short repeats located inside protein-coding genes. The maximum length of ignored repeats can be controlled by `--soft_mask N` parameter or automatically estimated by using `--soft_mask auto` option.

## Fungal mode

All algorithms in the GeneMark-ES Suite, including ProtHint, have a special mode for running on fungal genomes. This mode triggers, for example, a special branch point model and a different set of default parameters better suited for fungal genomes. To activate the fungal mode, use `--fungus` option.

## Parallel Execution

To run GeneMark-E* in a parallel mode, specify the number of parallel CPU threads with `--cores N` parameter. By default, the program uses a single CPU thread.

## Fragmented Assemblies

During model training, GeneMark-E* uses only contigs longer than 50 kbp, to avoid training on short, noisy contigs. For fragmented assemblies, in which the majority of genome is located in contigs shorter than 50 kbp, we recommend to adjust the `--min_contig` parameter, e.g. set the minimum to 10 kbp:

```
gmes_petap.pl --min_contig 10000 ...
```
## Additional options

To display a full list of options, run the GeneMark-ES suite with no arguments:

```bash
gmes_petap.pl
```