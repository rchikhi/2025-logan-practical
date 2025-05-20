# Part B — Using **Logan Search** to Locate Sequences Across the SRA

> “Query the planet in minutes.”  

Logan Search lets you search up to **1 kb** of DNA/RNA and instantly identifies every public *Sequence Read Archive* (SRA) dataset that is likely to contain it. It does so by finding shared k-mers between the query sequence and each SRA accession. It’s built on a petabyte-scale Bloom filter index covering >27 million runs assembled by **Logan**.

---

## 0 . Prerequisites

| Tool | Why you need it | Install |
|------|-----------------|---------|
| Web browser & email | Submit queries + receive result links | Any modern browser |
| *Optional* `aws` CLI | Fetch assemblies for the hits | See Part A or `aws help` |
| *Optional* `parallel`, `seqkit` | Batch downloads & sequence fiddling | `GitHub`, `brew`, `apt`, etc. |

---


## 1 . Prepare a Query Sequence

* **Limit:** ≤ 1 000 bp (service cap)  
* **Formats:** FASTA
* **Tip:** pick a region *specific* to your organism/gene—avoid conserved rRNA or adapters.

```bash
# Example: extract 800 bp from the E.coli assembly from PartA
seqkit subseq -r 1001:1800 SRR2584403.contigs.fa --quiet | seqkit seq -m 800 -w 0  | head -n 2 > query.fa
```

Trivia: what does the command above really do?

But actually, let us not query that particular sequence, because E. coli is everywhere, there will be too many hits and Logan Search takes more time.

Let us instead query this mysterious sequence:

```
TATGTCCTGTAACCATGAATTAAACTGCACCATATGATTGACTGATGCGGATGAACAGTATTGATCAAGTCCAGCTACAATACATTCTCTGCGACGTAGGCCTATTTACTAGGCCGTAGCTGAGTCACAGCAACTTCCCCCCCCAAAATTAAAAGATACCAAATGCATGACACCACAGTTATGTGTGATCATGGGCTGATTAGTCATTAGTCCATCGAGATGTCCCATTTGCAGGATCAGTAGGATTGAGTATGCGATGGCATGGCCCTGAAGTAAGAACCAGATGCCAGGTATAGTTCCAGTATAGAAACCCCCACATTTTATGGGCCCGGAGCGAGAAGAGGTACACTTCGCGCAAGGATTGATGGTTTCTCGAGGCTTGGTGATTAAGCTCGTGATCTAGGTGTGTTGCACCTATGTAAGATGAACCATAATATGTACATGCTTATATGCATGGGGCAAACCACCAATGCACGACGTACATAGGGTAGTTTAAAGGTTATATATGGGGTGGTGCAGTGGGTATTTGATATGTAGGTGTTGGGGTGGG
```


---

## 2 . Open the Interface

1. Navigate to **<https://logan-search.org>**.  
2. Click **“Query the planet”** (upper‑right).  
3. Under **Input**, choose **Text** (paste) *or* **File** (upload) and provide your sequence.  
4. *(Optional)* adjust the sidebar settings (see table below).  
5. Hit **Submit**. Wait about 5 minutes.

### Sidebar settings

| Setting | What it does |
|---------|--------------|
| **Groups** | Research search to a subset of SRA (*Genomic*, *Single‑cell*, etc). By default, use *all* |
| **Minimum coverage** | Fraction of 31‑mers that must match (default = 0.5). Raising it reduces false positives but increases false negatives. |
| **Email** | Necessary, and results links are sent there. |

> **Service limits**  
> • One sequence per submission  
> • Top 20 000 hits returned  
> • Results retained for **1 month**—download them!

---

## 3 . Look At The Results

Once Logan Search has finished searching (takes about 5 minutes), browse through the results webpage, and take a look at the map and plots!

* What do you think is the organism?
* Where is it mostly found?
* Is the sequence specific to that organism?

## 4 . Retrieve Your Results

The confirmation e‑mail contains **two links**:

| Link suffix | Purpose |
|-------------|---------|
| `/viz.html` | Interactive *kmviz* dashboard (taxonomy barplot, hit table, coverage slider). |
| `.zip` | Raw output: `hits.tsv`, `metadata.tsv`, static plots, your query. |

```bash
# Unzip & extract the accession column
unzip kmviz-<session>.zip
cut -f1 hits.tsv | tail -n +2 > hits.acc   # skip header
```

---

## 5 . Download Assemblies for the Hits

```bash
# Fetch the top 25 hits using 4 parallel threads
head -n 25 hits.acc | parallel -j4 '
  aws s3 cp s3://logan-pub/c/{}/{}.contigs.fa.zst . &&
  zstd -d {}.contigs.fa.zst
'
```

You now have contigs ready for alignment, variant calling, or pangenome analysis.

---

## 6 . Power‑User Tricks

| Goal | Trick |
|------|-------|
| Faster search | Trim query to 400–800 bp of the most specific region. |
| Skip isolates | In **Groups**, untick *Genomic* and *Single‑cell*. |
| Filter low‑coverage hits | `awk '$3 < 0.9' hits.tsv` (assuming coverage in column 3). |
| Random subset | `shuf hits.acc | head -n 50 > subset.acc` |

---

## 7 . Troubleshooting

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| “Too many concurrent queries” | High server load | Wait a few minutes, then resubmit. |
| Empty result set | Sequence too short or repetitive | Lengthen the query or choose a unique region. |
| No e‑mail received | Spam filter or typo | Check spam; resubmit with correct address. |
| “Top 20 k cutoff” notice | Query matched *too many* datasets | Raise **Minimum coverage** or pick a more specific sequence. |

---

## 8 . Next Steps

* **Variant scanning:** align all retrieved contigs to a reference.  
* **Phylogenetics:** build a tree from the matched runs.  
* **Epidemiology:** combine `metadata.tsv` with sampling dates/locations.  
* **Iterative discovery:** pull novel haplotypes → search again → expand the network.

---

### 🏁 Quick Recap

1. Craft ≤ 1 kb sequence.  
2. Submit on Logan Search, tweak filters, add your e‑mail.  
3. Explore *kmviz*, download the ZIP.  
4. Script‑fetch assemblies, and dive into analysis.

You’ve just queried **every** public sequencing run on Earth in minutes—welcome to planetary‑scale genomics!
