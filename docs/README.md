# GeneMark-ES Suite

Novel genomes can be analyzed by [GeneMark-ES](usage/es.md), an algorithm utilizing models parameterized by unsupervised training. Notably, GeneMark-ES has a special option for fungal genomes to account for fungal-specific intron organization. To integrate into GeneMark-ES information on mapped RNA-Seq reads, we made semi-supervised [GeneMark-ET](usage/et.md). Recently, we have developed [GeneMark-EP+](usage/EP.md) that uses homologous protein sequences of any evolutionary distance in both training and predictions.

This documentation covers installation and usage instructions for algorithms from the GeneMark-E* family. Visit the example section for step-by-step instructions on how to annotate a novel genome.

#### Relevant Publications

* [**GeneMark-EP+**](https://academic.oup.com/nargab/article/2/2/lqaa026/5836691)<br>
Tomáš Brůna, Alexandre Lomsadze, Mark Borodovsky<br>
"GeneMark-EP+: eukaryotic gene prediction with self-training in the space of genes and proteins"<br>
NAR Genomics and Bioinformatics, Volume 2, Issue 2, 2020

* [**GeneMark-ET**](https://academic.oup.com/nar/article/42/15/e119/2434516)<br>
Lomsadze A., Burns P.D. and Borodovsky M.<br>
"Integration of mapped RNA-Seq reads into automatic training of eukaryotic gene finding algorithm."<br>
Nucleic Acids Research, 2014, July 2

* [**GeneMark-ES, version 2**](https://genome.cshlp.org/content/18/12/1979)<br>
Ter-Hovhannisyan V., Lomsadze A., Chernoff Y. and Borodovsky M.<br>
"Gene prediction in novel fungal genomes using an ab initio algorithm with unsupervised training."<br>
Genome Research, 2008, Dec 18(12):1979-90

* [**GeneMark-ES, version 1**](https://academic.oup.com/nar/article/33/20/6494/1082033)<br>
Lomsadze A., Ter-Hovhannisyan V., Chernoff Y. and Borodovsky M.<br>
"Gene identification in novel eukaryotic genomes by self-training algorithm."<br>
Nucleic Acids Research, 2005, Vol. 33, No. 20, 6494-6506
