# BRAKER

BRAKER is a pipeline for fully automated prediction of protein coding gene structures with GeneMark-ES/ET/EP+ and AUGUSTUS in novel eukaryotic genomes.

Initially developed **BRAKER1** [[1](https://academic.oup.com/bioinformatics/article/32/5/767/1744611)] combines strengths of GeneMark-ET and AUGUSTUS to automatically generate full gene structure annotations in novel eukaryotic genomes.

**GeneMark-ET**

* Automatic semi-supervised training
* Does not use RNA-Seq information in prediction steps
* Does not predict alternative isoforms

**AUGUSTUS**

* Supervised training (solved by using GeneMark-ET result for training)
* Uses RNA-Seq data in prediction steps
* Predicts alternative isoforms


BRAKER2 [[2](https://www.biorxiv.org/content/10.1101/2020.08.10.245134v1)] is an extension of BRAKER1 that combines protein-supported GeneMark-EP+ and AUGUSTUS.

?> In contrast to other available methods that rely on protein homology information, BRAKER2 reaches high gene prediction accuracy even in the absence of the annotation of very closely related species and in the absence of RNA-Seq data.

Full documentation of BRAKER2 is available on its GitHub page at https://github.com/Gaius-Augustus/BRAKER. Please refer to the documentation for installation instructions.

## Overview of BRAKER usage

The most common use cases of BRAKER are:

**BRAKER with RNA-Seq data**

```bash
braker.pl --genome genome.fa --bam RNAseq.bam --softmasking --cores N
```

**BRAKER with proteins of any evolutionary distance**

```bash
braker.pl --genome genome.fa --prot_seq proteins.fa --softmasking --cores N
```

**BRAKER with RNA-Seq and protein data**

```bash
braker.pl --genome genome.fa --prot_seq proteins.fa --bam ../RNAseq.bam --etpmode --softmasking --cores N
```

* If you wish to run GeneMark separately (or re-use an existing GeneMark result), you can supply GeneMark result to BRAKER with `--geneMarkGtf genemark.gtf` option.
* Similarly, ProtHint protein hints can be used directly with `--hints prothint_augustus.gff` option.

Please refer to the [BRAKER manual](https://github.com/Gaius-Augustus/BRAKER#running-braker) for description of more options.

