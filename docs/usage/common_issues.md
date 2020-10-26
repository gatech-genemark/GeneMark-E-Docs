# Common issues

This page describes problems commonly encountered by GeneMark-ES suite and BRAKER users.

When encountering an error with GeneMark, first [run the test located in the `GeneMark-E-tests` folder](installation/installation_verification). This helps to determine whether the problem is caused by a specific input or whether there is something wrong with the installation of the program. The same applies for BRAKER, its tests are described at [github.com/Gaius-Augustus/BRAKER#example-data](https://github.com/Gaius-Augustus/BRAKER#example-data).

## GeneMark-ES suite problems

### Expired or missing key

**Problem**: The program fails with a message similar to:

```
GeneMark.hmm eukaryotic 3
License key ".gm_key" not found.
This file is neccessary in order to use GeneMark.hmm eukaryotic 3.
```

**Solution:** Re-install `gm_key` according to instructions shown in the [download](installation/download) and [installation](installation/installation) pages.

### Short contigs in a fragmented assembly

**Problem**: The program fails with a message similar to:

```
error, not enough sequence to run training in file /data/training.fna
```

**Solution**: Adjust the `--min_contig` parameter. See details on the [general usage page](usage/general?id=fragmented-assemblies).

### Low amount of RNA-Seq or protein data in GeneMark-ET/EP+

**Problem**: The program fails with a message similar to:

```
error, no valid sequences were found
error on call: make_nt_freq_mat.pl --cfg run.cfg --section stop_TAA --format TERM_TAA
```

**Solution**

It is likely there are not enough relevant RNA-Seq or protein hints to guide model training. This can be caused by:

* Something went wrong during RNA-Seq or protein alignment. Please check the [run of STAR](usage/preparing_rna) or [ProtHint](usage/EP?id=using-prothint-output-in-genemark-ep) for errors.
* RNA-Seq library comes from another species.
* Reference protein sequences are too remote (for example from a different kingdom) or too sparse (not enough protein sequences). Try getting a [relevant protein set from OrthoDB](usage/preparing_proteins).

### Low RNA-Seq or protein depth in GeneMark-ET/EP+

**Problem**: The program fails with a message similar to:

```
build initial ET/EP model
error, no valid sequences were found
error on call: make_nt_freq_mat.pl --cfg run.cfg --section donor_GT --format DONOR
```

**Solution**

It is likely there are not enough confident RNA-Seq or protein hints to build the initial intron model. Likely causes are:

* In case of RNA-Seq, this can be caused by low read depth. Try adding more RNA-Seq data or adjust the [`et_score` threshold](usage/et?id=adjusting-the-intron-initialization-score-threshold).
* There are not enough protein homologs within protein families. For example, this can happen when you use proteins of just one single relative on input. Try [adding more proteins from OrthoDB](usage/preparing_proteins) or adjust the `ep_score` threshold.

### Errors in cleanup

**Problem**: The program prints error messages similar to:

```
(in cleanup)   (in cleanup) at /usr/local/share/perl/5.30.0/Object/IndiseOut.pm line 1953 during global destruction.
```

**Solution**: This problem can be safely ignored. The errors are caused by problems with memory cleanup in one of the Perl modules and do not affect the results.

### Too many false positives

**Problem**: The program predicts too many false positives, i.e. the number of predicted genes is much higher than expected.

**Solution**
* The genome can be undermasked -- the interspersed and simple repeats were not sufficiently masked. See the [repeat masking section](/usage/general?id=repeat-masking) for more details.
* You can [filter out predictions which are not supported by external evidence](output/reliable_subset.md) to obtain a more reliable gene set.

### Poor prediction on fungal genomes

Make sure to use the `--fungus` flag when running GeneMark on a fungal genome. See [one of the examples](examples/fungal_genome?id=annotation-of-an-intron-sparse-fungal-genome) for a discussion of issues associated with intron-sparse fungal genomes.

### Poor prediction result

In general, poor predictions can be caused by several factors:

* The genome is not sufficiently masked for repeats. See the [repeat masking section](/usage/general?id=repeat-masking) for more details.
* GeneMark-E* does not support multiple models needed for genomes with heterogeneous nucleotide composition, like genomes of mammals and some plants (grasses, e.g. rice). While the current version of GeneMark-ET and -EP+ outperforms GeneMark-ES when running on such genomes, the overall accuracy could be significantly improved with more accurate modeling of genome heterogeneity.
* GeneMark-E* does not support [non-standard genetic codes](https://www.ncbi.nlm.nih.gov/Taxonomy/Utils/wprintgc.cgi). Support for genomes with alternative genetic codes is coming soon.

## BRAKER problems

The list of common BRAKER problems is available in [its documentation on GitHub](https://github.com/Gaius-Augustus/BRAKER#common-problems).