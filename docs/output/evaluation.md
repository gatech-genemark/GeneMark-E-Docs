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

The compare script supports comparisons on many different levels, e.g. `--trans` to compare exact transcript matches, `--intron` to compare exact intron matches, etc. Run the script without any arguments to see the full list of avaliable options.


## BUSCO evaluation

?> TODO