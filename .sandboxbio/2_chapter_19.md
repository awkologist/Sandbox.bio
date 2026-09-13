# In Search of Diﬀerences in Proteomes

Adapted from [Chapter 19](https://link.springer.com/chapter/10.1007/978-3-031-70314-0_19) of my book *Computational Biology*.

This project introduces two serotypes of *Escherichia coli*: one pathogenic and one non-pathogenic variety. The serotype O157:H7 emerges as a significant cause of foodborne illness, notably linked to undercooked meat since its detection in 1982. Phylogenetic analyses suggest that O157:H7 diverged from a common ancestor around 4.5 million years ago, acquiring its pathogenicity possibly through horizontal gene transfer. Can we identify proteins associated with pathogenicity among those acquired genes? To answer this question, we compare the translated, annotated genomes of one non-pathogenic and one pathogenic serotype. This project aims to uncover the presence of diﬀerent genes in different but related genomes. Central to this analysis is the Basic Local Alignment Search Tool (BLAST+) that we run locally and in the terminal. For sequence download, I introduce the rather new tool NCBI Databases.

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
mv ./GCF_000008865.2_ASM886v2_protein.faa ecoli_h7.fasta
```

Next, we do the same for the other proteome, K12:

```bash
curl -O  https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/005/845/GCF_000005845.2_ASM584v2/GCF_000005845.2_ASM584v2_protein.faa.gzgunzip ./GCF_000005845.2_ASM584v2_protein.faa.gz
mv ./GCF_000005845.2_ASM584v2_protein.faa ecoli_k12.fasta
```

How many proteins are there?

```bash
grep -c ">" ec*.fasta
```

## Creating BLAST DB
We use the NCBI BLAST+ command `makeblastdb` to create a local BLAST database:

```bash
makeblastdb -in ec-k12.fasta -dbtype prot -title "Escherichia coli K12" -out ecolik12 -parse_seqids
```

These are the database files:

```bash
ls -l ecolik12*
```

## BLASTing
Now, we perform the BLAST query:

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
awk '/Query=/ || /No hits/{print $0}' h7vsk12.txt | awk '{line[NR]=$0; if($0~/No hits/){print line[NR-1]}}' | egrep -v "([Uu]nknown| [Pp]utative|[Hh]ypothetical|[Uu]ncharacterized)" | head -20
```

```bash
awk '/Query=/ || /No hits/{print $0}' h7vsk12.txt | awk '{line[NR]=$0; if($0~/No hits/){print line[NR-1]}}' | egrep -v "([Uu]nknown| [Pp]utative|[Hh]ypothetical|[Uu]ncharacterized)" | wc -l
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
blastp -db ecolik12 -query ec-h7.faa -out h7vsk12-$i.txt -evalue $i
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
for i in h7vsk12-*; do echo -n $i" : "; awk '/Query=/ || /No hits/{print $0}' $i | awk '{line[NR]=$0; if($0~/No hits/){print line[NR-1]}}' | egrep -v "([Uu]nknown|[Pp]utative|[Hh]ypothetical)|[Uu]ncharacterized)" | wc -l; done
```
