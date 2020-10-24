# Output Evaluation

## Prediction Report

To get summary statistics about the predicted gene set and trained model, run the following script:

```bash
./predictionReport.py genemark.gtf output/gmhmm.mod report.pdf
```

!> The script is not called in the course of a standard GeneMark run because it depends on additional Python dependencies. The following [Python modules](https://docs.python.org/3/installing/index.html) are required: [`numpy`](https://numpy.org/), [`pandas`](https://pandas.pydata.org/), [`matplotlib`](https://matplotlib.org/), [`logomaker`](https://logomaker.readthedocs.io/en/latest/). All modules can be installed with pip: `pip3 install numpy pandas matplotlib logomaker`

The report is saved in the specified `report.pdf` file. The report includes:

* Number of predicted genes
    * Number of single-exon and multi-exon genes
* The average number of exons per gene
* The fraction of genes supported by external evidence (see the [output description](output/description.md))
* Number of complete and partial predictions
* Histograms of exon, gene, intron, and intergenic lengths
    * Exons are split into 4 categories: initial, terminal, terminal, and single.
* Distribution of the number of exons per gene
* Estimated start and stop motifs
* Estimated splice site motifs
* If the `--fungus` option was used during GeneMark run, the report also includes
    * Estimated branch point motif
    * Estimated branch point spacer duration

> [!TIP]
> See example reports generated for GeneMark-EP+ predictions on [**D. melanogaster**](output/reports/dmel.pdf ':ignore') and [**S. pombe**](output/reports/spombe.pdf ':ignore').

## Comparing against a reference

To compare the predictions against a reference annotation (or another gene set), you can use the included  `compare_intervals_exact.pl` script. For example, to compare the exact CDS matches, use:

```bash
compare_intervals_exact.pl --f1 annotation.gtf --f2  genemark.gtf --cds --verbose
```

The output will look as follows:

```
# CDS in file annotation.gtf: 62887
# CDS in file genemark.gtf: 58085

#in     match   unique  %match  CDS
62887   45225   17662   71.91   annotation.gtf
58085   45225   12860   77.86   genemark.gtf
```

?> In this case with reference annotation used in comparison, the first `%match` (71.91) represents prediction **sensitivity** and the value in the second row (77.86) is equivalent to prediction **specificity**.

The compare script supports comparisons on many different levels, e.g. `--trans` to compare exact transcript matches, `--intron` to compare exact intron matches, etc. Run the script without any arguments to see the full list of available options.


## BUSCO evaluation

[BUSCO](https://busco.ezlab.org/) is a tool to assess annotation completeness by assessing the presence of universal single-copy genes. Please refer to [BUSCO download page](https://busco.ezlab.org/busco_userguide.html) for installation instructions.

To assess the completeness of a gene set with BUSCO, you first need to select a lineage dataset corresponding to your species. The list of available BUSCO lineages can be displayed with the following command:

```bash
python3 busco --list-datasets
```

BUSCO score is then computed for the [predicted protein set](output/description?id=predicted-protein-and-gene-sequences) as follows:

```bash
python3 busco -m protein -i prot_seq.faa -o busco_output -l LINEAGE --cpu 8
```

A plot which visualizes the BUSCO result can be generated with the following command:

```bash
python3 generate_plot.py -wd busco_output
```

See [one of the examples](examples/novel_genome?id=busco-evaluation) for such visualization.