# Preparation of Protein Evidence

We recommend to use a relevant section of [OrthodDB](https://www.orthodb.org/?) as a general source of protein evidence for GeneMark-EP+.

!> **Different protein sources:** GeneMark-EP+ works best with a database of protein families, i.e. many representatives of each protein family must be present in the database. Therefore if you want to add additional proteins of closely related species, we recommend to add them to the OrthoDB fasta file.

For example, if your genome of interest is an insect, download arthropoda proteins:

```bash
wget https://v100.orthodb.org/download/odb10_arthropoda_fasta.tar.gz
tar xvf odb10_arthropoda_fasta.tar.gz
```

and concatenate proteins from all species into a single file:

```bash
cat arthropoda/Rawdata/* > proteins.fasta
```

For other genomes of interest, you can select the most specific OrthoDB
section from the list below and repeat the procedure described above.

* **Fungi**: https://v100.orthodb.org/download/odb10_fungi_fasta.tar.gz
* **Metazoa**: https://v100.orthodb.org/download/odb10_metazoa_fasta.tar.gz
    * **Arthropoda**: https://v100.orthodb.org/download/odb10_arthropoda_fasta.tar.gz
    * **Vertebrata**: https://v100.orthodb.org/download/odb10_vertebrata_fasta.tar.gz
* **Protozoa**: https://v100.orthodb.org/download/odb10_protozoa_fasta.tar.gz
* **Viridiplantae**: https://v100.orthodb.org/download/odb10_plants_fasta.tar.gz
