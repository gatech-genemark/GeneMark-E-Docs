# Output Description

All output files are located in the folder from which GeneMark-E* was executed.

## Gene coordinates

Predicted gene coordinates are outputted to `genemark.gtf` which looks as follows:

```
Chr1    GeneMark.hmm    exon          14412408    14412758   0   +   .   gene_id "1_g"; transcript_id "1_t";
Chr1    GeneMark.hmm    start_codon   14412408    14412410   .   +   0   gene_id "1_g"; transcript_id "1_t"; count "1_1";
Chr1    GeneMark.hmm    CDS           14412408    14412758   .   +   0   gene_id "1_g"; transcript_id "1_t"; cds_type "Single"; count "1_1";
Chr1    GeneMark.hmm    stop_codon    14412756    14412758   .   +   0   gene_id "1_g"; transcript_id "1_t"; count "1_1";
Chr1    GeneMark.hmm    exon          14412872    14413141   0   +   .   gene_id "2_g"; transcript_id "2_t";
Chr1    GeneMark.hmm    start_codon   14412872    14412874   .   +   0   gene_id "2_g"; transcript_id "2_t"; count "1_1";
Chr1    GeneMark.hmm    CDS           14412872    14413141   .   +   0   gene_id "2_g"; transcript_id "2_t"; cds_type "Initial"; count "1_2";
Chr1    GeneMark.hmm    intron        14413142    14413482   0   +   0   gene_id "2_g"; transcript_id "2_t"; count "1_1";
Chr1    GeneMark.hmm    exon          14413483    14413536   0   +   .   gene_id "2_g"; transcript_id "2_t";
Chr1    GeneMark.hmm    CDS           14413483    14413536   .   +   0   gene_id "2_g"; transcript_id "2_t"; cds_type "Terminal"; count "2_2";
Chr1    GeneMark.hmm    stop_codon    14413534    14413536   .   +   0   gene_id "2_g"; transcript_id "2_t"; count "1_1";
```

!> Stop codons are included in the `CDS` for terminal and single exons. 

If you ran GeneMark-ET or -EP+, each CDS includes an **anchored** status in the 9th column:

* `anchored "1_1"`: Both exon boundaries are supported by external evidence (intron, start or stop codon).
* `anchored "1_0"` or `anchored "0_1"`: One of the exon boundaries is supported by external evidence.
* If the feature is missing, none of the boundaries is supported by external evidence, the prediction is purely computational.

This information can be used to [select a reliable subset of predicted genes](reliable_subset.md).

## Predicted protein and gene sequences

The following script (inlucded in the GeneMark-ES suite) can be used to get protein and nucleotide sequences of predicted genes:

```bash
get_sequence_from_GTF.pl genemark.gtf genome.fasta
```

The outputs are located in `prot_seq.faa` and `nuc_seq.fna` files.

## Model file

The final model file is saved to `output/gmhmm.mod`.

## ProtHint output

If you ran GeneMark-EP+, the ProtHint output can be found in the `prothint` subfolder.

ProtHint generates two main outputs:

* `prothint.gff` contains a set of all reported hints
* `evidence.gff` contains a high-confidence subset of `prothint.gff`. Hints in this output pass stringent filtering criteria and generally have > 95% specificity.

?> ProtHint also creates `prothint_augustus.gff` output which can be directly used by [BRAKER](https://github.com/Gaius-Augustus/BRAKER) and [AUGUSTUS](https://github.com/Gaius-Augustus/Augustus).
