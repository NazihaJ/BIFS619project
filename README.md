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
#Clean DRR034568, if you're cleaning .gz files - just add .gz to the filenames below except for html and json
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

## Assembly using SPades

Go to the project folder

```bash
~/BIFS_619_Group_Project
```

Then command editing usign nano

```bash
# execute this command in terminal
nano run_all_spades.sh
```
This will pull up a text editor, you can copy the following directly into the edit page

```bash
#!/bin/bash
#The command to executes run_all_spades.sh
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
#run the following two commands, you can remove the notes if you feel so inclined
# this gives permission to execute command
chmod +x run_all_spades.sh

#this is will allow the script to run in the background even if things get disconnected
nohup ./run_all_spades.sh > batch_spades.log 2>&1 &
```
 Now you wait around 8 hours.. or less if you have more CPU than I do. 

QUAST
```bash
#evaluating assembly quality
#in the quast directory I entered the following
./quast.py ~/BIFS_619_Group_Project/02_spades_assembly/DRR034563/scaffolds.fasta -o ~/BIFS_619_Group_Project/02_spades_assembly/DRR034563/quast_result

./quast.py ~/BIFS_619_Group_Project/02_spades_assembly/DRR034568/scaffolds.fasta -o ~/BIFS_619_Group_Project/02_spades_assembly/DRR034568/quast_result

./quast.py ~/BIFS_619_Group_Project/02_spades_assembly/DRR034570/scaffolds.fasta -o ~/BIFS_619_Group_Project/02_spades_assembly/DRR034570/quast_result

```

Tried to install Prokka directly in the VM environment using Conda and Mamba for dependency resolution. It didnt work for me but this is how I would have approached it.  

```bash
conda install -n base -c conda-forge mamba
mamba create -n prokka_env -c bioconda -c conda-forge prokka

# Activate environment and attempt database setup
conda activate prokka_env
prokka --setupdb
prokka --listdb
```

HISAT2 Alignment _shannons version
This creates the conda environment using alignment tools. This uses the contig.fasta file from spades to create genome index for alignment

```bash
conda create -n rnaseq_env -c bioconda -c conda-forge hisat2 bowtie2 samtools subread -y
conda activate rnaseq_env
cd ~/BIFS_619_Group_Project/02_spades_assembly/DRR034563
hisat2-build contigs.fasta DRR034563_index
 
```

# HISAT2 Alignment 
This step aligns the cleaned RNA-seq reads against the assembled
E. coli contigs (from SPAdes) using the HISAT2 indices created earlier. Repeat the following for each sample in their respective directories.
```bash 
#Example running for sample DRR034568
cd ~/BIFS_619_Group_Project/02_spades_assembly/DRR034568 

#Run HISAT2 alignment
hisat2 -x DRR034568_index \
  -1 ~/BIFS_619_Group_Project/cleaned_reads/DRR034568_1.clean.fastq.gz \
  -2 ~/BIFS_619_Group_Project/cleaned_reads/DRR034568_2.clean.fastq.gz \
  -S DRR034568.sam \
  -p 6 
```
hisat2_${ID}.log → run summary with % alignment, read counts, etc.

This step was repeated for DRR034568 and DRR034570 to generate the .sam files.
The disk space is very limited so the .sam file is converted to .bam and removed for space. Then the .bam can be sorted. 
```bash
samtools view -bS -o DRR034568.bam DRR034568.sam
samtools sort -o DRR034568_sorted.bam DRR034568.bam

#move the sorted bams into a sorted_bam directory to generate a table/plot with total reads and mapping percentage.
mkdir ~/BIFS_619_Group_Project/sorted_bam
mv DRR034568_sorted.bam ~/BIFS_619_Group_Project/sorted_bam

#repeated for DRR034563 and DRR034570
```
### Alignment rate per sample
```bash
cd ~/BIFS_619_Group_Project/sorted_bam
#in the sorted_bam directory, copy and past the following to generate each sample_flagstat.txt

for bam in *.bam; do
    sample=$(basename "$bam" .bam)
    samtools flagstat "$bam" > "${sample}_flagstat.txt"
done

#copy and paste the following to generate a CSV of the sample, total_reads, mapped_reads, and mapping_percent.

echo "Sample,Total_Reads,Mapped_Reads,Mapping_Percent" > alignment_summary.csv
for file in *_flagstat.txt; do
    sample=${file%_flagstat.txt}
    total=$(grep "in total" $file | awk '{print $1}')
    mapped=$(grep "mapped (" $file | head -n1 | awk '{print $1}')
    percent=$(grep "mapped (" $file | head -n1 | sed 's/.*(\(.*%\).*/\1/')
    echo "$sample,$total,$mapped,$percent" >> alignment_summary.csv
done

#check the output
cat alignment_summary.csv 
```
### Total read count and mapping percentage
```bash
#If you don’t have gnuplot
sudo apt install gnuplot-nox # version 5.4.2+dfsg2-2

#copy and paste the following code block to generate a .png of the mapping percentage

gnuplot -e "
set terminal png size 800,400;
set output 'mapping_percentage.png';
set title 'HISAT2 Alignment Rate per Sample';
set boxwidth 0.5;
set style fill solid;
set ylabel 'Mapping %';
set yrange [0:100];
set datafile separator ',';
plot 'alignment_summary.csv' using 0:4:xtic(1) with boxes lc rgb 'skyblue' title 'Mapping %';
"

#copy and paste the following code block to generate a .png plot for the total reads 
gnuplot -e "
set terminal pngcairo size 900,500 enhanced font 'Arial,12' background rgb 'white';
set output 'total_reads.png';
set title 'HISAT2 Total Reads per Sample';
set ylabel 'Total Reads';
set boxwidth 0.5;
set style fill solid;
set datafile separator ',';
set xtics rotate by -45;
plot 'alignment_summary.csv' using 2:xtic(1) with boxes lc rgb '#1f77b4' title 'Total Reads';
"

# to view these .pngs you can use the following commands
display total_reads.png
display mapping_percentage.png
```

# Heat Map
Install R and then install the following packages to R to make the heat map from the featured gene counts.
```bash 
install.packages(c("dplyr", "tidyr", "ggplot2", "pheatmap"))
 
setwd("~/BIFS_619_Group_Project")
library(dplyr)
library(tidyr)
library(ggplot2)
library(pheatmap)
 

f1$Geneid <- sub("^.*_", "", f1$Geneid)
f2$Geneid <- sub("^.*_", "", f2$Geneid)
f3$Geneid <- sub("^.*_", "", f3$Geneid)
 
library(dplyr)
library(tidyr)
library(ggplot2)
library(pheatmap)

```
```bash
# merge
counts <- f1 %>%
  select(Geneid, DRR034563 = ncol(f1)) %>%
  left_join(f2 %>% select(Geneid, DRR034568 = ncol(f2)), by = "Geneid") %>%
  left_join(f3 %>% select(Geneid, DRR034570 = ncol(f3)), by = "Geneid")
 
#ensure numeric
counts_num <- counts
counts_num[, -1] <- lapply(counts_num[, -1], function(x) as.numeric(as.character(x)))
 
#convert to matrix
counts_mat <- as.matrix(counts_num[,-1])
rownames(counts_mat) <- counts_num$Geneid
 
# check numeric values now
summary(counts_mat)
 
#select top 10 genes
top10 <- counts_mat[order(rowSums(counts_mat, na.rm = TRUE), decreasing = TRUE)[1:10], ]
```
The following code is for obtaining the heat map with pink–blue color scale, axis labels, and title.
```bash
# define pink-blue palette
pink_blue <- colorRampPalette(c("deepskyblue3", "white", "deeppink3"))(50)
 
#generate heatmap
pheatmap(log2(top10 + 1),
         color = pink_blue,
         cluster_rows = TRUE,
         cluster_cols = TRUE,
         scale = "row",
         main = "Top 10 Expressed Genes Across E. coli Samples",
         labels_row = rownames(top10),
         labels_col = colnames(top10),
         angle_col = 45,
         fontsize = 10,
         fontsize_row = 9,
         fontsize_col = 10,
         legend = TRUE,
         legend_labels = "Expression Level (log2 scaled)")
```












