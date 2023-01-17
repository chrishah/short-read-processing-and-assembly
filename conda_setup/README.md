
One of the easiest ways to install bioinformatics software is through the package management system [conda](https://docs.conda.io/en/latest/).

As an exercise we will setup a reduced version of conda, called [miniconda](https://docs.conda.io/en/latest/miniconda.html), which contains many relevant packages.

Set it up like so:
First, find an appropriate version of miniconda on the the miniconda [webpage](https://docs.conda.io/en/latest/miniconda.html).

To ensure that we'll all get the same version we'll download a particular installation file. Let's also make sure that we're all in our home directory first.
```bash
(user@host)-$ cd ~
(user@host)-$ wget https://repo.anaconda.com/miniconda/Miniconda3-py38_4.12.0-Linux-x86_64.sh
```

Now, install miniconda, by executing the installation script that you've just downloaded.
```bash
(user@host)-$ bash ./Miniconda3-py38_4.12.0-Linux-x86_64.sh
```

You will have to confirm with ENTER and 'yes' a few times. Let's keep if simple for now.

***Attention***
> To make the installation active you'll have to exit and reconnect to the server.

```bash
(user@host)-$ exit
```

Then you can start installing tools, e.g.:
```bash
(user@host)-$ conda install -c bioconda fastqc
```

Note, that conda allows you to explicitly install particular versions of software. Let's try for the genome assembler SPAdes - please only do one of the below.

```bash
(user@host)-$ conda install -c bioconda spades

(user@host)-$ conda install -c bioconda spades=3.11.1

(user@host)-$ conda install -c bioconda spades=3.15.3
```

```bash
(user@host)-$ conda install -c bioconda quast

(user@host)-$ conda install -c bioconda minia

```

Software is per default installed in the so-called 'base' environment, which you have to activate as follows:
```bash
(user@host)-$ conda activate
```

If you want to deactivate the currently active environment, you can do this like so:
```bash
(user@host)-$ conda deactivate
```


__Congratulations, you have just installed software using conda!! __


