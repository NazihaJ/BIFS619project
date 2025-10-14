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
fasterq-dump DRR034568 --split-files --threads 6 && gzip DRR034568_*.fastq
fasterq-dump DRR034570 --split-files --threads 6 && gzip DRR034570_*.fastq
fasterq-dump DRR034563 --split-files --threads 6 && gzip DRR034563_*.fastq
fastqc *.fastq
```

The output of this should be the HTML to the FastQC 
```bash
#in ~/raw_data

multiqc . -f 
firefox multiqc_report.html
```
#this is not required, but it generates a nice comparison of all the fastqc data into one report

```bash
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
```
```bash
#move the clean files into cleaned_reads
mv ~/BIFS_619_Group_Project/raw_data/*.clean.fastq ~/BIFS_619_Group_Project/cleaned_reads

#this should run fastqc again on the cleaned reads and use multiqc to compile them into an html file that can be opened with firefox

cd ~/BIFS_619_Group_Project/cleaned_reads
fastqc *.fastq
multiqc . -f -n cleaned_multiqc_report.html
firefox cleaned_multiqc_report.html & 
```
```bash
#Try this for compressing into .gz, it shouldn't take very long because it uses multiple CPUs
sudo apt install pigz
pigz DRR034563_1.clean.fastq
```

##Assembly using SPades

Go to the project folder

```bash
~/BIFS_619_Group_Project
```

Then command editing usign nano

```bash
nano run_all_spades.sh

#copy this to the edit page

#!/bin/bash
#The command to execute is: run_all_spades.sh
#This will sequentially builds genome assemblies for all cleaned read pairs using SPAdes

#directories given that the clean reads are in the following folder. If they arent create a folder and call it cleaned_reads.
CLEAN_DIR="/home/StudentFirst/BIFS_619_Group_Project/cleaned_reads"
OUT_DIR="/home/StudentFirst/BIFS_619_Group_Project/02_spades_assembly"

#this is the output directory
mkdir -p $OUT_DIR

#samples
SAMPLES=("DRR034563" "DRR034568" "DRR034570")

#loop
for sample in "${SAMPLES[@]}"; do
    spades.py \
        -1 ${CLEAN_DIR}/${sample}_1.clean.fastq.gz \
        -2 ${CLEAN_DIR}/${sample}_2.clean.fastq.gz \
        -o ${OUT_DIR}/${sample} \
        -t 4 \
        -m 12
done
```
To exit do Crtl O, Enter, Crtl X

Execute command in terminal 
*nohup is to make sure it keeps running even if you lose internet connection like what happened to me*

```bash
chmod +x run_all_spades.sh
nohup ./run_all_spades.sh > batch_spades.log 2>&1 &
```
 Now you wait around 8 hours.. or less if you have more CPU than I do. 



