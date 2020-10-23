# Selecting a reliable subset of predictions

Set of genes predicted by GeneMark-ET/EP+ can contain many predictions which are unsupported or only partially supported by external evidence (protein or RNA-Seq). While these computational predictions are often correct, they are generally less reliable than predictions with some or full external support.

?> The number of false positive predictions is generally increasing with genome size, due to longer intergenic regions.

To obtain a more conservative prediction set, you can run the following script (located in the GeneMark-ES suite):

```bash
selectSupportedSubsets.py genemark.gtf --fullSupport fullSupport.gtf --anySupport anySupport.gtf --noSupport noSupport.gtf
```

?> The selection procedure only works for GeneMark-ET/EP+ since all predictions in GeneMark-ES are purely computational

The script splits GeneMark prediction into three categories:
* **Fully supported genes**: All introns in a gene are supported by external evidence. In case of single-exon genes, both start and stop codon are supported by external evidence (works with protein evidence only). This is the most reliable set.
* **Genes with any support**: At least one intron, start or stop codon of a predicted gene is found.
* **Genes no support**: None of the gene features is supported by external evidence. This is the least reliable set.

See [one of the examples](examples/novel_genome?id=selection-of-a-reliable-gene-set) for prediction accuracy differences between these sets.

!> Genes with no external support are not necessarily false positives. In GeneMark-ET, they can be lowly expressed or single-exon genes. In GeneMark-EP+, many correct but unsupported genes can be present if the set of reference proteins is taxonomically distant.

