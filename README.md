# short-read-processing-and-assembly
A short introduction to processing and assembly of short (Illumina) NGS sequencing data

## Introduction

The massively parallel generation of nucleotide sequence data has had an enormous effect on modern biology, and is commonly referred to as high-throughput sequencing or also Next generation sequencing (NGS). 
Different sequencing instruments have specific error profiles, which are important to understand before getting started with analyses. The Illumina sequencing platform is currently the most widely used and a great option for genomic sequencing (among many other applications) with large amounts of data produced at relatively low cost. In the following part of the course you will explore some Illumina data and familiarize yourself with basic characteristics. Furthermore, there will be a brief introduction into quality filtering of Illumina data.

This tutorial is basically self contained and ships with testdata. You'll need a few pieces of software though. How to get these setup? A few options:
 - use Docker (tested with version 20.10.7, build 20.10.7-0ubuntu5~18.04.3)
 - use Singularity (tested with version 3.6.3)
 - install things locally (see list below)

If you have either Docker or Singularity, there is no more installation needed. Running things through container engines as the above at first glance may appear a little bit more complicated, but it has the big advantage that you don't need to have any of the software that we'll be using actually installed locally on your computer/server and secondly, this ensures full reproducibility of this session. The below exercises assume you are using Docker, but if you have things installed locally you can always omit the lines calling Docker and call software directly instead. If you want to try things with Singularity you can switch to the branch 'singularity' on Github to see the corresponding hints (or go [here](https://github.com/chrishah/short-read-processing-and-assembly/tree/singularity), if you're unsure how to change the branch.

If you need a recap on Docker usage, we have a tutorial for this [here](https://github.com/chrishah/docker-intro/blob/master/README.md).

If you don't want to use these, you'll need to install the following things on your system:
***ATTENTION***
> To ensure for things to work as expected we **highly** recommend to set up the specific versions of the software we note below - what might happen if you use other versions is anyones guess.. - you've been warned ;-) 

Bioinformatics software:
 - fastqc
 - fastp (version 0.23.1)
 - flash (version 1.2.11 - optional)
 - kmc3 (version 3.0.0)
 - minia (version 3.2.4)
 - quast (version 5.0.2)
 - spades (version 3.15.3)
 - abyss (version 2.2.5 - optional)
 - platanus (version 1.2.4 - optional)

There's a fair chance you already have some experience at the command line and you've done at least some pre-processing of read data before so the first few examples should be fairly straightforward and should simply help to get you back into the swing of things. 

Now, let's get cracking!

Start by cloning this repository:
```bash
(user@host)-$ git clone https://github.com/chrishah/short-read-processing-and-assembly.git
```

Move into the directory you've just downloaded:
```bash
(user@host)-$ cd short-read-processing-and-assembly
```

There are a few tasks to solve throughout the tutorial. If you get stuck we have solutions prepared for you [here](https://github.com/chrishah/short-read-processing-and-assembly/tree/main/solutions/README.md) - use them wisely ;-)


***ATTENTION***
> If you are doing this exercise as part of a course you may be given separate test data. Please halt here and ask your instructors about this if you haven't received any information at this point. In the meantime maybe what you're looking for is explained [here](https://github.com/chrishah/short-read-processing-and-assembly/blob/main/data/download-and-subsample.md). Enjoy!

__Optional__

As backup, if you are doing this as part of a course, your instructors may have set up a conda environment with software installed for you. Please consult your instructors if you are interested. Assuming an environment with the name 'short_assembly' is present you would activate it like so:
```bash
(user@host)-$ conda activate short_assembly
```

For an example on how the necessary software for this exercise could be setup with conda please see [here](https://github.com/chrishah/short-read-processing-and-assembly/tree/main/conda_setup/README.md).


## Illumina data basics

Illumina sequence data usually comes in the so-called __fastq__ format, which can be seen as an extension of the __fasta__ format that you are already familiar with, I am sure. In addition to nucleotide sequences, fastq files also contain quality scores associated with each nucleotide. These describe the probability that a given nucleotide has been called incorrectly and are expressed as [phred scores](https://en.wikipedia.org/wiki/Phred_quality_score). A phred score of 30 (often used as ‘gold-standard’) indicates a probability of 1 in 1000 for a base to be miscalled (ie 99.9% accuracy). Phred scores are coded as single characters in ASCII format. 

Let’s now get familiar with the fastq format by exploring some read data. The typical length of Illumina sequences (‘reads’) is currently 150-300 bp (depending on technology) and data usually comes in paired-ends (pe), i.e. ‘the same’ DNA fragment is sequenced from both ends. The so-called forward and reverse reads are mostly provided in two separate files (as in our case) with the first forward read pairing with the first reverse read and so on. Pairing information will also be encoded in the read headers. Depending on the length of the actual fragment the read pairs might overlap. You can identify forward/reverse reads based on a *1* or *2*, respectively, in the filename, while the remainder of the filename should be identical. 
Fastq files usually come in compressed form (usually ‘gzipped’ indicated by a `.gz` suffix) to save space. Many programs are able to process gzipped files automatically but others are not, so make sure you know how to decompress such files on the command line (`gunzip`). Alternatively there are ways to display the content of gzipped files without actually writing a much larger decompressed version of your data onto your disk.

***TASK 1***
> Find a way to look at the first 12 lines of the compressed file (gzipped) that we provide in `data/reads.1.fastq.gz` without actually decompressing it.

Now that you had your first look at a fastq file, can you find at least two differences between fasta and fastq files in terms of formatting conventions? You remember that in fasta format sequence headers (the first line of every sequence record) were characterized by `>`. What about fastq? How many lines does a single sequence in a fastq file typically have?

***TASK 2***
> Given the general characteristics of fastq files can you come up with at least two ways to determine the number of sequences in the file `data/reads.1.fastq.gz` using your command line skills? 

***TASK 3***
> Display the first 8 lines of the files `data/reads.1.fastq.gz` and `data/reads.2.fastq.gz` and explore in order to understand how read pairing works in Illumina paired-end data. Question: What might an ‘interleaved’ fastq file be?

Now, let's have a quick look at the data quality in our files. You've probably seen [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) before so the most straightforward thing would be the following (if FastQC was installed on your machine):
```bash
(user@host)-$ fastqc data/reads.1.fastq.gz
```

With Docker it could be done, like so (in this case I am using an image I have made for [trim_galore](https://www.bioinformatics.babraham.ac.uk/projects/trim_galore/), which uses FastQC):
```bash
(user@host)-$ docker run --rm -u $(id -u):$(id -g) -v $(pwd)/:/in -w /in biocontainers/fastqc:v0.11.9_cv8 \
               fastqc data/reads.1.fastq.gz
(user@host)-$ docker run --rm -u $(id -u):$(id -g) -v $(pwd)/:/in -w /in biocontainers/fastqc:v0.11.9_cv8 \
               fastqc data/reads.2.fastq.gz
```

Do inspect the resulting `*.html` reports.

## Read trimming

Quick quality trimming with [fastp](https://github.com/OpenGene/fastp) may be done like this (to keep things clean, make a directory for the trimmed reads first): 
```bash
(user@host)-$ mkdir trimmed

(user@host)-$ docker run --rm -u $(id -u):$(id -g) -v $(pwd):/in -w /in chrishah/fastp:0.23.1 \
               fastp --in1 data/reads.1.fastq.gz --in2 data/reads.2.fastq.gz \
               --out1 trimmed/reads.trimmed.pe.1.fastq.gz --out2 trimmed/reads.trimmed.pe.2.fastq.gz
```

If you wanted to be a bit more explicit about how you trim and also keep reads that are missing it's mate after trimming (sometimes called 'orphaned reads'), you could e.g. do something like this:
```bash
(user@host)-$ docker run --rm -u $(id -u):$(id -g) -v $(pwd):/in -w /in chrishah/fastp:0.23.1 \
               fastp --in1 data/reads.1.fastq.gz --in2 data/reads.2.fastq.gz \
               --out1 trimmed/reads.trimmed.pe.1.fastq.gz --out2 trimmed/reads.trimmed.pe.2.fastq.gz \
               --unpaired1 trimmed/reads.trimmed.unpaired.1.fastq.gz --unpaired2 trimmed/reads.trimmed.unpaired.2.fastq.gz \
               --detect_adapter_for_pe --length_required 100 --qualified_quality_phred 30 --average_qual 20 --cut_right --cut_mean_quality 25 \
               --thread 2 --html trimmed/trimming.report.html --json trimmed/trimming.report.json 
```

## Read merging

We noted before that under certain circumstances read pairs might overlap. The extent will depend on the particular library prep, but it may be useful to merge overlapping reads. FastP can also do that.

***TASK 4***
> Use FastP to merge trimmed read pairs (where possible) and retain the unmerged reads.

There are many other tools that do read-merging. Another example is [FlasH](https://ccb.jhu.edu/software/FLASH/). 

***TASK 5***
> Try out FlasH - a docker image is prepared for you: `chrishah/flash:1.2.11`

## Kmer counting

We assume you've been introduced to the concept of __k-mers__, and how, before actual assembly, they can be used to get a feel for your data. Let's use a took called [kmc3](https://github.com/refresh-bio/KMC) to count the __21-mers__ in our data:
```bash
(user@host)-$ mkdir kmer.db.21
(user@host)-$ ls -1 data/reads.*.fastq.gz > fastq.files.txt
(user@host)-$ docker run --rm -u $(id -u):$(id -g) -v $(pwd)/:/in -w /in chrishah/kmc3-docker:v3.0 \
               kmc -k21 -m4 -v -sm -ci2 -cx1000000000 -cs255 -n64 -t2 @fastq.files.txt kmers.21 kmer.db.21

(user@host)-$ docker run --rm -u $(id -u):$(id -g) -v $(pwd)/:/in -w /in chrishah/kmc3-docker:v3.0 \
               kmc_tools histogram kmers.21 -ci2 kmers.21.hist.txt
```

The file we have produced `kmers.21.hist.txt` is a simple text file with two columns. You could plot out the k-mer counts with e.g. `R`.

A neat online tool to explore kmer frequencies is [GenomeScope](http://qb.cshl.edu/genomescope/). You can simply upload the file we've produced after a small change in formatting, specifically we want to modify the text file so that the two columns are separated by a single space only - some `awk` magic will do it:
```bash
(user@host)-$ cat kmers.21.hist.txt | awk '{print $1" "$2}' > kmers.21.hist.genomescope.txt
```

## De novo genome assembly

Once we are satisfied with the trimmed data quality, we can now start to create longer sequences out of the short FASTQ reads. The resulting longer sequences are called contigs and the process of creating these contigs is called assembly. Sequence assembly is a complex problem and short read assembly can be computationally intense, especially when it comes to memory usage. Because there are many different kinds of assemblers for this purpose, (almost) all claiming to be better then the others but in reality each assembler has different pros and cons. A list of some commonly used assemblers is provided at the end of this chapter. At a certain stage of the assembly process most contemporary short read assemblers use a concept called De Bruijn graphs. A de Bruijn graph is a method to represent a large number of short sequences in terms of their k-mer content in the form of graph structure. k-mers are all possible sequences (read sub-sequences) of length k. 

***QUESTION***
> What limits the maximum k-mer size one can use in an assembly?

As mentioned previously there exists a large number of genome assemblers. Our tasks will encourage you to try the following:
 - [Minia](http://minia.genouest.org)
 - [Spades](http://bioinf.spbau.ru/en/spades)
 - [Platanus](http://platanus.bio.titech.ac.jp)
 - [ABySS](https://github.com/bcgsc/abyss)

They are not necessarily the best possible assemblers. It's just a selection of popular ones that tend to be very fast (Minia) and/or usually return decent results. Of course you can try any assembler you want - in fact we dare you to try an additional one. Each assembler has different qualities and problems. You will now assemble a test dataset with different assemblers, it is up to you to choose which one you want to use, but please try at least three different parameter settings and/or assemblers to see how the choice of software and parameters can influence assembly.

***ATTENTION***
> If you do this exercise as part of a course you might be asked to enter your results in an online table. 

Please evaluate your assembly results with Quast - see below - and identify your ‘best’ assembly based on the Quast stats. Make sure that you remember/note the parameters used for different assemblies. In the end you’ll try to find the best assembly overall and it would be a shame if you couldn’t reproduce it.. 

It may also be useful to create seperate directories in your assembly folder for each assembler / parameter setting so you know which files belong to which assembly, but this is up to you.

Minia is one of the fastest assemblers around. It uses a filtering algorithm to make storing de Bruijn graphs more memory efficient. With Minia it is possible to assembly the human genome on a desktop computer within a day. 
A typical Minia command looks like this:
```bash
(host-$ minia -in reads.1.fq -in reads.2.fq -kmer-size 31 -abundance-min 3 -out minia_k31
```

The `–in` flag specifies the fastq read file from which an assembly should be made (if you have multiple files you'll need multiple `-in`. Minia accepts both plain FASTQ and gzipped FASTQ files. `–kmer-size` specifies the k-mer size. `–abundance-min` tells minia how often a k-mer must be seen in the reads to be considered correct. `–out` specifies the prefix for the output files, so if you run multiple assemblies you can still assign the output files.

The above will work if Minia is installed on your server. As usual, we've made Docker container. Running Minia via Docker for a k-mer size of 41 could look like this (remember to make a `minia` directory first to keep things tidy):
```bash
(host)-$ mkdir minia

(host)-$ docker run --rm -u $(id -u):$(id -g) -v $(pwd):/in -w /in chrishah/minia:3.2.4 \
          minia -in trimmed/reads.trimmed.pe.1.fastq.gz -in trimmed/reads.trimmed.pe.2.fastq.gz \
          -kmer-size 41 -abundance-min 2 -out minia/minia.41 -nb-cores 2
```

***TASK 6***
> Try to assemble your data a few times with Minia, with different kmer sizes; trimmed reads vs. merged reads, etc, be creative. Make sure you change the output prefix between assemblies. Try to pick informative names - this will make your life easier in the long run.

__Congratulations, you have just assembled your first genomes!__ .. took me about a month to get there..

Below you'll find tasks and hints for trying further assemblers, but first let's have a quick look on a neat tool for assessing contiguity of assemblies - Quast.

```bash
(host)-$ docker run --rm -u $(id -u):$(id -g) -v $(pwd):/in -w /in reslp/quast:5.0.2 \
          quast -o quast_results \
          minia/minia.41.contigs.fa
```

Assuming you've run Minia three times with a number of different k-mer sizes, but following the naming convention from above you could create a joint report for all (adjust if you did something else), for example if you have:
 - `minia/minia.41.contigs.fa`
 - `minia/minia.51.contigs.fa`
 - `minia/minia.61.contigs.fa`

```bash
(host)-$ docker run --rm -u $(id -u):$(id -g) -v $(pwd):/in -w /in reslp/quast:5.0.2 \
          quast -o quast_results \
          minia/minia.*.contigs.fa
```
Check out the file `quast_results/report.html` that has bee created (if you do this on a server you'll need to download the full directory `quast_results/` - double-clicking on quast_results/report.html will open the file in a webbrowser).

or, more explicitly:
```bash
(host)-$ docker run --rm -u $(id -u):$(id -g) -v $(pwd):/in -w /in reslp/quast:5.0.2 \
          quast -o quast_results \
          -m 1000 --labels minia.k41,minia.k51,minia.k61 \
          minia/minia.41.contigs.fa minia/minia.51.contigs.fa minia/minia.61.contigs.fa
```

***Note***
> The third line in the above command is entirely optional and specifies that we want to consider only contigs of minimal length 1000 (`-m 1000`) and specify short labels for the report that will be created. Just leave out this line if you want to see what different it makes.


__Well done for reaching this point!!!!__


Now to some other assemblers..

Let's try SPAdes:
```bash
(user@host)-$ docker run --rm -u $(id -u):$(id -g) -v $(pwd)/:/in -w /in reslp/spades:3.15.3 \
                spades.py -o spades-default \
                   -1 trimmed/reads.trimmed.pe.1.fastq.gz -2 trimmed/reads.trimmed.pe.2.fastq.gz \
                   -t 2 \
                   -m 8 --only-assembler
```

Compare the minia and spades results with quast.
```bash
(host)-$ docker run --rm -u $(id -u):$(id -g) -v $(pwd):/in -w /in reslp/quast:5.0.2 \
          quast -o quast_results \
          -m 1000 \
          minia/minia.*.contigs.fa spades-default/scaffolds.fasta
```

Have a look at the html file `quast_results/report.html`.

***ATTENTION***
> A quast report including some more assembly variations also ships with the repository in `solutions/results/quast_results/report.html`. 


***ATTENTION***
> Some assembly results and a quast report of toy bacterial dataset (2 Million reads from EBI accession: ERR022075) also ship with the repository in `solutions/ERR022075_2M/`. 

__Well Done!!!!__


And then some more optional tasks, if you have time..

***TASK 7***
> Try out ABySS - Docker image: `reslp/abyss:2.2.5`

***TASK 8***
> Try out SpAdes with error correction - Docker image: `reslp/spades:3.15.3`

***TASK 9***
> Try out Platanus - Docker image: `chrishah/platanus:v1.2.4`


Are you still feeling adventurous? Do you want to try another assembler? We can help you install or find a Docker container..

Don't forget to share you results with us!!!

__Thanks for joining us today!__

If you have any questions, comments, feedback (good OR bad), let me know!

__What do you think?__

# Contact
Christoph Hahn - <christoph.hahn@uni-graz.at>

