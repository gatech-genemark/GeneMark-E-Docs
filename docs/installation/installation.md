# Installation

!> GeneMark-ES Suite depends on `perl` (version 5.10 or higher) and `python3` (version 3.3 or higher). Both are usually pre-installed on modern Linux distributions.

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
cpan YAML Hash::Merge Logger::Simple Parallel::ForkManager MCE::Mutex Thread::Queue threads Math::Utils
```

!> You need to have `make` installed (for example with `sudo apt-get install build-essential` on Debian-based Linux) before installing the Perl modules . See the `INSTALL` file for alternative ways to install Perl modules.

Add GeneMark-ES Suite directory to your `PATH`, to make it available anywhere on the machine:

```bash
echo "export PATH=$(pwd)":'$PATH' >> ~/.bashrc
source ~/.bashrc 
```

> [!WARNING] Perl scripts are configured with default Perl location at `/usr/bin/perl`. Path to Perl can be changed by script `change_path_in_perl_scripts.pl`, e.g.:
> ```bash
> ./change_path_in_perl_scripts.pl "/usr/bin/env perl"
> ```
