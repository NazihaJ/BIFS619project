# BIFS619project

Group 4 microbiome project -- e.coli


## Setting up

This is to setup the folders we need

```bash
mkdir -p ~/BIFS_619_Group_Project
cd ~/BIFS_619_Group_Project
mkdir raw_data
```

Now run the following to download the Fasta file
the SSR IDs we will be focusing on are:

DRR34568, DRR034570, and DRR034563

```bash
cd ~/raw_data
fasterq-dump DRR034568 --split-files --threads 6
fasterq-dump DRR034570 --split-files --threads 6
fasterq-dump DRR034563 --split-files --threads 6
fastqc *.fastq
```

The output of this should be the HTML to the FastQC 

#in ~/raw_data

multiqc . -f 
firefox multiqc_report.html

#this is not required, but it generates a nice comparison of all the fastqc data into one report




