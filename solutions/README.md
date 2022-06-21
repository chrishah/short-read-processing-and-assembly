
***TASK 1***
```bash
(user@host)-$ zcat data/reads.1.fastq.gz | head -n 12
```

***TASK 2***
```bash
#count total number of lines and divide result by 4 (manual)
(user@host)-$ zcat data/reads.1.fastq.gz | wc -l

#reduce the file to only every fourth line and count
(user@host)-$ zcat data/reads.1.fastq.gz | sed -n '1~4p' | wc -l

#find a pattern that should only occur in one line for each read and count the number of lines
(user@host)-$ zcat data/reads.1.fastq.gz | grep "^+$" | wc -l
```



