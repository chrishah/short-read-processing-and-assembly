# How some of the testdata was prepared

Download data from ENA (European Nucleotide Archive). 

Imagine you were browsing around ENA finding data from a paper you've read recently, e.g. [here](https://www.ebi.ac.uk/ena/browser/view/ERR022075).There's a bunch of ways how you could get the data but one example would be to right click on the \*fastq.gz file that you're interested in, 'select 'Copy link address' (or something like that - depends on the operating system you're working on, and then paste it to the command line after a `wget` command, like so:

```bash
(user@host)-$ wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR022/ERR022075/ERR022075_1.fastq.gz

(user@host)-$ wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR022/ERR022075/ERR022075_2.fastq.gz 
```

This would download the files the files to wherever you are on your system.

Now let's downsample the data and only take the first 2,000,000 (2M) reads from it. Given that you know how a fastq files is structured you can simply take the first 8M lines, right? There are a few other things thrown into the command below:
 - gunzip on the fly
 - using pipe `|` to stream data through multiple commands consecutively
 - compress data on the fly

Try it out:
```bash
(user@host)-$ zcat ERR022075_1.fastq.gz | head -n 8000000 | gzip > ERR022075_2M_1.fastq.gz 
(user@host)-$ zcat ERR022075_2.fastq.gz | head -n 8000000 | gzip > ERR022075_2M_2.fastq.gz 
```


