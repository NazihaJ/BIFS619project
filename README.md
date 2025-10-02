# BIFS619project

Group 4 microbiome project -- e.coli


## Setting up

This is to setup the folders we need

```bash
mkdir -p ~/BIFS_619_Group_Project
cd ~/BIFS_619_Group_Project
mkdir raw_data
```

now run the following to download the Fasta file

```bash
cd ~/raw_data
fasterq-dump DRR034568 --split-files --threads 6
fasterq-dump DRR034570 --split-files --threads 6
fasterq-dump DRR034563 --split-files --threads 6
fastqc *.fastq
```

The output of this should be the HTML to the FastQC 

