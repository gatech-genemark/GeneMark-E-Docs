# Output Evaluation

## Prediction Report

To get summary statistics about the predicted gene set and trained model, run the following script:

```bash
./predictionReport.py genemark.gtf output/gmhmm.mod report.pdf
```

The report is saved in the specified `report.pdf` file. The report includes:

* Number of predicted genes
    * Number of single-exon and multi-exon genes
* Average number of exons per transcript
* Histograms of exon, gene, and intron lengths
* Fraction of exons and genes supported by external evidence (see the [output description](output/description.md))
* Estimated splice site motifs
* Estimated start and stop motifs
* If the `--fungus` option was used during GeneMark run, the report also includes
    * Estimated branch motif
    * Estimated branch point spacer duration

?> An example report can be [viewed here]() (**TODO**)

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