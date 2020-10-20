## Installation


> :warning: GeneMark-ES Suite depends on `perl` (version 5.10 or higher) and `python3` (version 3.3 or higher). Both are pre-installed in modern Linux distributions.

Extract the downloaded archive:

```bash
tar xvf gmes_linux_64.tar.gz
```

Install the key:

```bash
cd gmes_linux_64
cp gm_key ~/.gm_key
```

Install required Perl modules, for example with `cpan`

```bash
cpan YAML Hash::Merge Logger::Simple Parallel::ForkManager MCE::Mutex Thread::Queue threads
```
> :information_source: See the `INSTALL` file for alternative ways to install Perl modules.

Verify the installation:

```bash
./check_install.bash
```

> :warning: Perl scripts are configured with default Perl location at `/usr/bin/perl`. Path to Perl can be changed by script `change_path_in_perl_scripts.pl`, e.g.:
> ```bash
> ./change_path_in_perl_scripts.pl "/usr/bin/env perl"
> ```

