# Installation Verification

Check if all the required models are installed and the validity of gm_key:

```bash
./check_install.bash
```

Run a test example:

```bash
cd GeneMark-E-tests/EP
mkdir test; cd test
../../../gmes_petap.pl --seq ../input/genome.fasta --EP --dbep ../input/proteins.fasta \
    --verbose --cores=8 --max_intergenic 10000
```

?> This example tests the execution of the algorithm in the **EP+** mode, with the integration of protein evidence. See the `GeneMark-E-tests` folder for other tests.

The expected runtime of this example is **~7 minutes**.

If everything is configured correctly, the resulting `genemark.gtf` file  in the `test` folder should match the contents of the `genemark.gtf` in the `output` folder. This can be checked by running the following command:

```bash
../../../compare_intervals_exact.pl  --f1 genemark.gtf  --f2 ../output/genemark.gtf  --v
```

The output should look as follows:

```
# CDS in file genemark.gtf: 1461
# CDS in file ../output/genemark.gtf: 1461

#in match   unique  %match  CDS

1461    1461    0   100.00  genemark.gtf
1461    1461    0   100.00  ../output/genemark.gtf
```