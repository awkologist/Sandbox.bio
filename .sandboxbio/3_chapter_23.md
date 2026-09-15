# Exploring Early SARS-CoV2 Mutations

Adapted from [Chapter 23](https://link.springer.com/chapter/10.1007/978-3-031-70314-0_23) of my book *Computational Biology*.

We have tackled the SARS-CoV-2 pandemic, but the lives lost were tragic. Our project delves into using bioinformatics tools to analyze mutations in the SARS-CoV-2 genome. Employed tools include IGV, SAMtools, BCFtools, Minimap, NCBI EDirect, AWK, and Jmol. IGV aids in visually inspecting genomic data for mutation identification. SAMtools and BCFtools process sequencing data to identify mutations from alignment (SAM/BAM) and variant call format (VCF) files. Minimap aligns SARS-CoV-2 sequences to a reference genome for mutation detection. NCBI EDirect retrieves SARS-CoV-2 sequences for mutation analysis. AWK filters and manipulates mutation data. Jmol visualizes the three- dimensional structure of SARS-CoV-2 spike proteins, aiding in understanding mutation implications. Integrating these tools enables comprehensive mutation
analyses, offering insights into viral evolution and impacts on disease strategies.

## Working Directory
We are going to work in the following directory:

```bash
cd /home/sandbox/github/awkologist/SARS
```

## Provided Programs 
The AWK-Scripts *fasta2tbl* and *compare-cov2.awk* are available in the working directory.


## Downloading Proteoms
One after the other:

```bash
curl -O https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/008/865/GCF_000008865.2_ASM886v2/GCF_000008865.2_ASM886v2_protein.faa.gz
```

Now, we extract the compress file archive:

```bash
gunzip ./GCF_000008865.2_ASM886v2_protein.faa.gz
```

An then we rename the file:

```bash
mv ./GCF_000008865.2_ASM886v2_protein.faa ec-h7.fasta
```

Next, we do the same for the other proteome, K12:

```bash
curl -O  https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/005/845/GCF_000005845.2_ASM584v2/GCF_000005845.2_ASM584v2_protein.faa.gz
gunzip ./GCF_000005845.2_ASM584v2_protein.faa.gz
mv ./GCF_000005845.2_ASM584v2_protein.faa ec-k12.fasta
```

How many proteins are there?

```bash
grep -c ">" ec*.fasta
```

## Creating BLAST DB
We use the NCBI BLAST+ command `makeblastdb` to create a local BLAST database:

```bash
makeblastdb -in ec-k12.fasta -dbtype prot -title "Escherichia coli K12" -out ecolik12 -parse_seqids -blastdb_version 4 
```

The option `-blastdb_version 4` dictates using the older formatting structure for BLAST databases. The newer version 5 is not compatible with this Linux system.

These are the database files:

```bash
ls -l ecolik12*
```

## BLASTing
Now, we perform the BLAST query. **Attention**: The query runs appr. 25 minutes.

```bash
time blastp -db ecolik12 -query ec-h7.fasta -out h7vsk12.txt -evalue .00001 
```

The result is in file *h7vsk12.txt*:
```bash
ls -lh ec-* h7*
```

Let us count the number of lines:

```bash
wc -l h7vsk12.txt
```

## Processing the BLAST Result File
Finally, we analyse the result file step-by-step:

```bash
awk '/Query=/ || /No hits/{print}' h7vsk12.txt | head -20
```

```bash
awk '/Query=/ || /No hits/{print $0}' h7vsk12.txt | awk '{line[NR]=$0; if($0~/No hits/){print line[NR-1]}}' | head
```

```bash
awk '/Query=/ || /No hits/{print $0}' h7vsk12.txt | awk '{line[NR]=$0; if($0~/No hits/){print line[NR-1]}}' | wc -l
```

```bash
awk '/Query=/ || /No hits/{print $0}' h7vsk12.txt | awk '{line[NR]=$0; if($0~/No hits/){print line[NR-1]}}' | grep -Ev "([Uu]nknown| [Pp]utative|[Hh]ypothetical|[Uu]ncharacterized)" | head -20
```

```bash
awk '/Query=/ || /No hits/{print $0}' h7vsk12.txt | awk '{line[NR]=$0; if($0~/No hits/){print line[NR-1]}}' | grep -Ev "([Uu]nknown| [Pp]utative|[Hh]ypothetical|[Uu]ncharacterized)" | wc -l
```

## Playing with the E-Value
Let us analyse the effect of the e-value setting on the result. Therefore, we need the Bash/AWK script in *autoblast.sh*:

```
#!/bin/bash
# save as autoblast.sh
# loops through E-value
for i in 1 0.001 0.00001
do
echo "Working on h7vsk12-$i.txt"
blastp -db ecolik12 -query ec-h7.fasta -out h7vsk12-$i.txt -evalue $i
done
```

```bash
time ./autoblast.sh
```

And analyse the result:

```bash
for i in h7vsk12-*; do echo -n $i" : "; awk '/Query=/ || /No hits/{print $0}' $i | awk '{line[NR]=$0; if($0~/No hits/){print line[NR-1]}}' | wc -l; done
```

```bash
for i in h7vsk12-*; do echo -n $i" : "; awk '/Query=/ || /No hits/{print $0}' $i | awk '{line[NR]=$0; if($0~/No hits/){print line[NR-1]}}' | grep -Ev "([Uu]nknown|[Pp]utative|[Hh]ypothetical)|[Uu]ncharacterized)" | wc -l; done
```
