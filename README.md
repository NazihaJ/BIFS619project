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

sudo apt install fastp

mkdir -p ~/BIFS_619_Group_Project/cleaned_reads

#assuming you’re in /raw_data
#Clean DRR034568
fastp \
-i DRR034568_1.fastq \
-I DRR034568_2.fastq \
-o ~/BIFS_619_Group_Project/raw_data/DRR034568_1.clean.fastq \
-O ~/BIFS_619_Group_Project/raw_data/DRR034568_2.clean.fastq \
-h ~/BIFS_619_Group_Project/raw_data/DRR034568.html \
-j ~/BIFS_619_Group_Project/raw_data/DRR034568.json \
 --length_required 50

#DRR034570
fastp \
  -i DRR034570_1.fastq \
  -I DRR034570_2.fastq \
  -o ~/BIFS_619_Group_Project/raw_data/DRR034570_1.clean.fastq \
  -O ~/BIFS_619_Group_Project/raw_data/DRR034570_2.clean.fastq \
  -h ~/BIFS_619_Group_Project/raw_data/DRR034570.html \
  -j ~/BIFS_619_Group_Project/raw_data/DRR034570.json \
  --length_required 50

#DRR034563
fastp \
  -i DRR034563_1.fastq \
  -I DRR034563_2.fastq \
  -o ~/BIFS_619_Group_Project/raw_data/DRR034563_1.clean.fastq \
  -O ~/BIFS_619_Group_Project/raw_data/DRR034563_2.clean.fastq \
  -h ~/BIFS_619_Group_Project/raw_data/DRR034563.html \
  -j ~/BIFS_619_Group_Project/raw_data/DRR034563.json \
  --length_required 50

#move the clean files into cleaned_reads
mv ~/BIFS_619_Group_Project/raw_data/*.clean.fastq ~/BIFS_619_Group_Project/cleaned_reads

#this should run fastqc again on the cleaned reads and use multiqc to compile them into an html file that can be opened with firefox

cd ~/BIFS_619_Group_Project/cleaned_reads
fastqc *.fastq
multiqc . -f -n cleaned_multiqc_report.html
firefox cleaned_multiqc_report.html & 



