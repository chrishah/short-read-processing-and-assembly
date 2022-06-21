
***TASK 1***
```bash
(user@host)-$ zcat data/reads.1.fastq.gz | head -n 12
```

***TASK 2***
```bash
#reduce the file to only every fourth line and count
(user@host)-$ zcat data/reads.1.fastq.gz | sed -n '1~4p' | wc -l

#find a pattern that should only occur in one line for each read and count the number of lines
(user@host)-$ zcat data/reads.1.fastq.gz | grep "^+$" | wc -l

#count total number of lines and divide result by 4 (manual)
(user@host)-$ zcat data/reads.1.fastq.gz | wc -l

#count total number and divide by four on the fly
(user@host)-$ echo "$(zcat data/reads.1.fastq.gz | wc -l) / 4" | bc
```

***TASK 3***
```bash
(user@host)-$ zcat data/reads.1.fastq.gz | head -n 8
(user@host)-$ zcat data/reads.2.fastq.gz | head -n 8
```

***TASK 4***
```bash
(user@host)-$ docker run --rm -u $(id -u):$(id -g) -v $(pwd):/in -w /in chrishah/fastp:0.23.1 \
                fastp --in1 trimmed/reads.trimmed.pe.1.fastq.gz --in2 trimmed/reads.trimmed.pe.2.fastq.gz \
                --merge --merged_out trimmed/reads.fastp.merged.fastq.gz \
                --out1 trimmed/reads.fastp.notmerged.1.fastq.gz --out2 trimmed/reads.fastp.notmerged.2.fastq.gz \
                --thread 4 --html trimmed/merging.report.html --json trimmed/merging.report.json
```

***TASK 5***
```bash
(user@host)-$ docker run --rm -u $(id -u):$(id -g) -v $(pwd):/in -w /in chrishah/flash:1.2.11 \
                flash -z -t 4 -o trimmed/reads.flash trimmed/reads.trimmed.pe.1.fastq.gz trimmed/reads.trimmed.pe.2.fastq.gz
```

***TASK 6***
```bash
(user@host)-$ for k in {51,61,71,81,91}
do
        docker run --rm -u $(id -u):$(id -g) -v $(pwd):/in -w /in chrishah/minia:3.2.4 \
        minia -in trimmed/reads.trimmed.pe.1.fastq.gz -in trimmed/reads.trimmed.pe.2.fastq.gz \
        -kmer-size $k -abundance-min 2 -max-memory 8000 -out minia/minia.$k -nb-cores 1
done
```

***TASK 7***
```bash
(user@host)-$ mkdir abyss

(user@host)-$ mkdir abyss/abyss.51
(user@host)-$ docker run --rm -u $(id -u):$(id -g) -v $(pwd)/:/in -w /in reslp/abyss:2.2.5 \
               abyss-pe -C abyss/abyss.51 k=51 name=abyss np=4 \
               in="/in/trimmed/reads.trimmed.pe.1.fastq.gz /in/trimmed/reads.trimmed.pe.2.fastq.gz" default 2>&1 | tee abyss/abyss.51/abyss.log

(user@host)-$ mkdir abyss/abyss.81
(user@host)-$ docker run --rm -u $(id -u):$(id -g) -v $(pwd)/:/in -w /in reslp/abyss:2.2.5 \
               abyss-pe -C abyss/abyss.81 k=81 name=abyss np=4 \
               in="/in/trimmed/reads.trimmed.pe.1.fastq.gz /in/trimmed/reads.trimmed.pe.2.fastq.gz" default 2>&1 | tee abyss/abyss.81/abyss.log


(user@host)-$ mkdir abyss/abyss.merged.81
(user@host)-$ docker run --rm -u $(id -u):$(id -g) -v $(pwd)/:/in -w /in reslp/abyss:2.2.5 \
               abyss-pe -C abyss/abyss.merged.81 k=81 name=abyss np=4 \
               in="/in/trimmed/reads.fastp.notmerged.1.fastq.gz /in/trimmed/reads.fastp.notmerged.2.fastq.gz" \
               se="/in/trimmed/reads.fastp.merged.fastq.gz /in/trimmed/reads.trimmed.unpaired.1.fastq.gz /in/trimmed/reads.trimmed.unpaired.2.fastq.gz" default 2>&1 | tee abyss/abyss.merged.81/abyss.log

(user@host)-$ mkdir abyss/abyss.merged.51
(user@host)-$ docker run --rm -u $(id -u):$(id -g) -v $(pwd)/:/in -w /in reslp/abyss:2.2.5 \
               abyss-pe -C abyss/abyss.merged.51 k=51 name=abyss np=4 \
               in="/in/trimmed/reads.fastp.notmerged.1.fastq.gz /in/trimmed/reads.fastp.notmerged.2.fastq.gz" \
               se="/in/trimmed/reads.fastp.merged.fastq.gz /in/trimmed/reads.trimmed.unpaired.1.fastq.gz /in/trimmed/reads.trimmed.unpaired.2.fastq.gz" default 2>&1 | tee abyss/abyss.merged.51/abyss.log

#quast
(user@host)-$ docker run --rm -v $(pwd):/in -u $(id -u):$(id -g) -w /in reslp/quast:5.0.2 \
        quast -o quast_results -m 1000 --labels minia.k51,minia.k61,abyss.k51,abyss.k81 -t 4 \
        minia/minia.51.contigs.fa minia/minia.61.contigs.fa abyss/abyss.51/abyss-scaffolds.fa abyss/abyss.81/abyss-scaffolds.fa
```

***TASK 8***
```bash
(user@host)-$ docker run --rm -u $(id -u):$(id -g) -v $(pwd)/:/in -w /in reslp/spades:3.15.3 \
               spades.py -o spades-default \
                -1 trimmed/reads.trimmed.pe.1.fastq.gz -2 trimmed/reads.trimmed.pe.2.fastq.gz \
                --checkpoints last \
                -t 4 \
                -m 8 --only-assembler

#with ec
(user@host)-$ docker run --rm -u $(id -u):$(id -g) -v $(pwd)/:/in -w /in reslp/spades:3.15.3 \
               spades.py -o spades-ec-default \
                -1 trimmed/reads.trimmed.pe.1.fastq.gz -2 trimmed/reads.trimmed.pe.2.fastq.gz \
                --checkpoints last \
                -t 4 \
                -m 8
```

***TASK 9***
```bash
(user@host)-$ mkdir platanus 

#platanus does not accept gzipped input, so we'll first need to decompress
(user@host)-$ gunzip trimmed/reads.trimmed.pe.1.fastq.gz  
(user@host)-$ gunzip trimmed/reads.trimmed.pe.2.fastq.gz  

#run in three stages
(user@host)-$ docker run --rm -u $(id -u):$(id -g) -v $(pwd)/:/in -w /in chrishah/platanus:v1.2.4 \
               platanus assemble -o platanus/platanus \
               -f /in/trimmed/reads.trimmed.pe.1.fastq /in/trimmed/reads.trimmed.pe.2.fastq -t 4 -m 8 2>&1 | tee platanus/platanus.assemble.log 
(user@host)-$ docker run --rm -u $(id -u):$(id -g) -v $(pwd)/:/in -w /in chrishah/platanus:v1.2.4 \
               platanus scaffold -o platanus/platanus -c platanus/platanus_contig.fa -b platanus/platanus_contigBubble.fa \
               -IP1 /in/trimmed/reads.trimmed.pe.1.fastq /in/trimmed/reads.trimmed.pe.2.fastq -t 4 2>&1 | tee platanus/platanus.scaffold.log 
(user@host)-$ docker run --rm -u $(id -u):$(id -g) -v $(pwd)/:/in -w /in chrishah/platanus:v1.2.4 \
               platanus gap_close -o platanus/platanus -c platanus/platanus_scaffold.fa \
               -IP1 /in/trimmed/reads.trimmed.pe.1.fastq /in/trimmed/reads.trimmed.pe.2.fastq -t 4 2>&1 | tee platanus/platanus.gapclose.log 
 
#quast
(user@host)-$ docker run --rm -v $(pwd):/in -w /in reslp/quast:5.0.2 \
               quast -o quast_results -m 1000 -t 4\
               --labels minia.k51,minia.k61,abyss.k51,abyss.k81,abyss.m.k51,abyss.m.k81,spades-default,spades-ec-default,platanus \
               minia/minia.51.contigs.fa minia/minia.61.contigs.fa \
               abyss/abyss.51/abyss-scaffolds.fa abyss/abyss.81/abyss-scaffolds.fa \
               abyss/abyss.merged.51/abyss-scaffolds.fa abyss/abyss.merged.81/abyss-scaffolds.fa \
               spades-default/scaffolds.fasta spades-ec-default/scaffolds.fasta \
               platanus/platanus_gapClosed.fa 
```

