# In Search of Diﬀerences in Proteomes

Adapted from [Chapter 19](https://link.springer.com/chapter/10.1007/978-3-031-70314-0_19) of my book *Computational Biology*.

This project introduces two serotypes of *Escherichia coli*: one pathogenic and one non-pathogenic variety. The serotype O157:H7 emerges as a significant cause of foodborne illness, notably linked to undercooked meat since its detection in 1982. Phylogenetic analyses suggest that O157:H7 diverged from a common ancestor around 4.5 million years ago, acquiring its pathogenicity possibly through horizontal gene transfer. Can we identify proteins associated with pathogenicity among those acquired genes? To answer this question, we compare the translated, annotated genomes of one non-pathogenic and one pathogenic serotype. This project aims to uncover the presence of diﬀerent genes in different but related genomes. Central to this analysis is the Basic Local Alignment Search Tool (BLAST+) that we run locally and in the terminal. For sequence download, I introduce the rather new tool NCBI Databases.

## Installation of NCBI Datasets
First, we install the NCBI tool `datasets` to download data from NCBI. 

```bash
curl -O https://ftp.ncbi.nlm.nih.gov/pub/datasets/command-line/v2/linux-amd64/datasets"
```

Check the content of your working directory:

```bash
ls -l"
```

We must make the code executable:

```bash
chmod u+x datasets"
```

Check the content of your working directory, again:

```bash
ls -l"
```

> What difference do you observe?


## Downloading Proteoms
Let us use a `for` loop in the Bash shell:

```bash
for i in GCF_000005845.2 GCF_000008865.2; do ./datasets download genome accession $i --include protein --filename $i.zip; done"
```

Now, we extract the compress file archive:

```bash
unzip -jo GCF_000005845.2.zip"
```

An then we rename the file:

```bash
mv protein.faa ec-k12.fasta"
```

Next, we do the same for the other proteome:

```bash
unzip -jo GCF_000008865.2"
```

```bash
mv protein.faa ec-h7.fasta"
```

How many proteins are there?

```bash
grep -c ">" ec*.fasta
```