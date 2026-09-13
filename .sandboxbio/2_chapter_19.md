<script>
import Execute from "$components/Execute.svelte";
import Quiz from "$components/Quiz.svelte";
import Link from "$components/Link.svelte";

import Exercise from "$components/Exercise.svelte";

const criteria = [{
	name: "File <code>exons.fixed.bed</code> no longer causes a <code>bedtools merge</code> error",
	checks: [{
		type: "file",
		path: "exons.fixed.bed",
		action: "contents",
		commandExpected: `sed 's/ /\\t/g' exons.bed`
	}]
}];

const hints = [
	`As suggested by the error message, run <code>cat -t exons.bed</code>. Also try <code>head exons.bed</code>. Do any lines stand out from the others?`,
	`In the output of <code>cat -t exons.bed</code>, the first line uses spaces as the column delimiter instead of tabs.`,
	`You can use a <code>sed</code> command to replace spaces with tabs (<code>\\t</code>). You can also use <code>vim</code> to modify the file manually.`,
	`With <code>sed</code>, don't forget to specify that you want the replacement logic to be global. With <code>vim</code>, make sure you convert each space in the first row to a tab.`
];
</script>

# In Search of Diﬀerences in Proteomes

Adapted from [Chapter 19](https://link.springer.com/chapter/10.1007/978-3-031-70314-0_19) of my book *Computational Biology*.

This project introduces two serotypes of *Escherichia coli*: one pathogenic and one non-pathogenic variety. The serotype O157:H7 emerges as a significant cause of foodborne illness, notably linked to undercooked meat since its detection in 1982. Phylogenetic analyses suggest that O157:H7 diverged from a common ancestor around 4.5 million years ago, acquiring its pathogenicity possibly through horizontal gene transfer. Can we identify proteins associated with pathogenicity among those acquired genes? To answer this question, we compare the translated, annotated genomes of one non-pathogenic and one pathogenic serotype. This project aims to uncover the presence of diﬀerent genes in different but related genomes. Central to this analysis is the Basic Local Alignment Search Tool (BLAST+) that we run locally and in the terminal. For sequence download, I introduce the rather new tool NCBI Databases.

## Installation of NCBI Datasets
First, we install the NCBI tool `datasets` to download data from NCBI. 
<Execute command={"curl -O https://ftp.ncbi.nlm.nih.gov/pub/datasets/command-line/v2/linux-amd64/datasets"} />

Check the content of your working directory:
<Execute command={"ls -l"} />

We must make the code executable:

<Execute command={"chmod u+x datasets"} />

Check the content of your working directory, again:
<Execute command={"ls -l"} />

What difference do you observe?
<Quiz
	id="quiz1"
	choices={[
		{ valid: false, value: `file size changed` },
		{ valid: false, value: `ownership changed` },
		{ valid: true, value: `permissions changed` },
		{ valid: false, value: `file date changed` },
    ]}>
	<span slot="prompt"></span>
</Quiz>

## Downloading Proteoms
let us use a `for` loop in the Bash shell:
<Execute command={"for i in GCF_000005845.2 GCF_000008865.2; do ./datasets download genome accession $i --include protein --filename $i.zip; done"} />

Now, we extract the compress file archive:
<Execute command={"unzip -jo GCF_000005845.2.zip"} />

An then we rename the file:
<Execute command={"mv protein.faa ec-k12.fasta"} />

Next, we do the same for the other proteome:
<Execute command={"unzip -jo GCF_000008865.2"} />

<Execute command={"mv protein.faa ec-h7.fasta"} />

How many proteins are there?

```bash
grep -c ">" ec*.fasta
```


As of version `2.21.0`, bedtools is able to intersect an "A" file against one or more "B" files. This greatly simplifies analyses involving multiple datasets relevant to a given experiment. For example, let's intersect exons with CpG islands, GWAS SNPs, an the ChromHMM annotations:

<Execute command={"bedtools intersect -a exons.bed -b cpg.bed gwas.bed hesc.chromHmm.bed -sorted | head"} />

Now by default, this isn't incredibly informative as we can't tell which of the three "B" files yielded the intersection with each exon. However, if we use the `-wa` and `wb` options, we can see from which file number (following the order of the files given on the command line) the intersection came. In this case, the 7th column reflects this file number:

<Execute command={"bedtools intersect -a exons.bed -b cpg.bed gwas.bed hesc.chromHmm.bed -sorted -wa -wb \\ | head -n 10000 \\ | tail -n 10"} />

Additionally, one can use file "labels" instead of file numbers to facilitate interpretation, especially when there are _many_ files involved:

<Execute command={"bedtools intersect -a exons.bed -b cpg.bed gwas.bed hesc.chromHmm.bed -sorted -wa -wb -names cpg gwas chromhmm \\ | head -n 10000 \\ | tail -n 10"} />

> You should by now have learned how to analyse data

## Exercises

<Exercise {criteria} {hints} />

## Quiz

Take a look at the `lsd2_out` log file. When did the MRCA exist?

<Quiz
	id="step3-quiz1"
	choices={[
		{ valid: false, value: `2019-08-12` },
		{ valid: false, value: `2019-03-11` },
		{ valid: true, value: `2020-03-02` },
		{ valid: false, value: `2020-01-28` },
    ]}>
	<span slot="prompt"></span>
</Quiz>


