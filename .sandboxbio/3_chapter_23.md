# Exploring Early SARS-CoV2 Mutations

Adapted from [Chapter 23](https://link.springer.com/chapter/10.1007/978-3-031-70314-0_23) of my book *Computational Biology*.

We have tackled the SARS-CoV-2 pandemic, but the lives lost were tragic. Our project delves into using bioinformatics tools to analyze mutations in the SARS-CoV-2 genome. Employed tools include IGV, SAMtools, BCFtools, Minimap, NCBI EDirect, AWK, and Jmol. IGV aids in visually inspecting genomic data for mutation identification. SAMtools and BCFtools process sequencing data to identify mutations from alignment (SAM/BAM) and variant call format (VCF) files. Minimap aligns SARS-CoV-2 sequences to a reference genome for mutation detection. NCBI EDirect retrieves SARS-CoV-2 sequences for mutation analysis. AWK filters and manipulates mutation data. Jmol visualizes the three- dimensional structure of SARS-CoV-2 spike proteins, aiding in understanding mutation implications. Integrating these tools enables comprehensive mutation
analyses, offering insights into viral evolution and impacts on disease strategies.

## Working Directory
We are going to work in the following directory:

```bash
cd /home/sandbox/github/awkologist/sandbox.bio/SARS
```

## Provided Programs 
The AWK-Scripts *fasta2tbl* and *compare-cov2.awk* are available in the working directory.

Make *fasta2tbl* executable:

```bash
chmod u+x ./fasta2tbl
```

## Download Virus Reference Sequences
Download reference genome and save in file *wuhan-1.fasta*:

```bash
efetch -db nuccore -id NC_045512 -format fasta > wuhan-1.fasta
```

Now, create a copy in tab-delimited format:

```bash
./fasta2tbl wuhan-1.fasta > wuhan-1.tab
```

## Download other Virus Sequences from NCBI
Download from [NCBI](https://www.ncbi.nlm.nih.gov/sars-cov-2/) viruses from Europe, 
- from human hosts, 
- without ambigious characters, 
- complete nucleotide sequences, 
- and a sequence length of exactly 29,903 nt. 
Build a **custom** FASTA annotation line with
- 
- 
- 
in step 3 in the download window They are downloaded via the web browser as *sequences_timestamp.fasta*. Move them into your current working directory.

Detailed instruction are in chapter "**23.2.3 Data and GitHub Repository**"

Move the file (ca 100 MB) from your local computer to JuypterHub. It takes a while – check in the GitHub file browser if the complete file has been uploaded.

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
