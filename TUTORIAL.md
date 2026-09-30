# SyntenyScan — Tutorial

A complete guide for someone who has never run a bioinformatics pipeline.

**Contents**

1. [What this tool is](#1-what-this-tool-is)
2. [The problem it solves](#2-the-problem-it-solves)
3. [What synteny and paralogy mean](#3-what-synteny-and-paralogy-mean)
4. [Opening the tool](#4-opening-the-tool)
5. [Running it — step by step](#5-running-it--step-by-step)
6. [A worked example](#6-a-worked-example-pigeonpea-and-soybean)
7. [Reading every output file](#7-reading-every-output-file)
8. [What researchers gain from it](#8-what-researchers-gain-from-it)
9. [Using it on your own crop](#9-using-it-on-your-own-crop)
10. [Limitations](#10-limitations-stated-plainly)
11. [When something goes wrong](#11-when-something-goes-wrong)
12. [Citing the tool](#12-citing-the-tool)

---

## 1. What this tool is

SyntenyScan compares two plant genomes and answers two questions:

- **Which stretches of gene order have survived since the two species last shared an ancestor?**
- **Which genes inside each genome are recent copies of one another?**

It is a notebook that runs in your web browser on Google's computers. You
install nothing. You do not use a command line. You press one button and, after
ten to thirty minutes, you download a set of spreadsheets and a figure.

The pipeline itself is not new — it chains two established programs, DIAMOND
and MCScanX. What SyntenyScan provides is that the chaining, the parameter
choices, the awkward installation steps and the output parsing are all done for
you, correctly and identically every time.

## 2. The problem it solves

Anyone who has tried to run this analysis by hand knows the obstacles:

| Obstacle | What normally happens | With SyntenyScan |
|---|---|---|
| Installing MCScanX | Fails to compile on any modern system without patching two header files | Patched automatically |
| Getting genome files | Navigating NCBI, choosing among several file types | Two accession numbers |
| One gene, many proteins | A gene with ten transcripts is counted ten times, inflating everything | Longest coding sequence per gene, automatically |
| Formatting for MCScanX | A position file in an undocumented four-column format | Written for you |
| Reading the output | A custom text format that is not a table | Parsed into spreadsheets |
| Hit filtering | A subtle trap that silently biases the result (see §7.3) | Handled correctly |

A researcher attempting this for the first time typically loses several days to
installation alone. SyntenyScan takes a morning, most of it waiting.

## 3. What synteny and paralogy mean

Skip this section if the terms are familiar.

### Synteny

Picture a chromosome as a street and the genes as houses standing along it in a
fixed order.

Two related species inherited the same street from their shared ancestor. Since
then the street has been cut up and rearranged in both, but **long stretches
still have the houses in the same order**. Finding those stretches is synteny
analysis.

Each preserved stretch is a **block**. Each matched pair of genes inside it is
an **anchor**.

The essential point, easy to miss: *similarity alone is not synteny.* Two genes
can resemble each other anywhere in a genome. Synteny requires **order** —
several similar genes appearing in the same sequence in both species.

### Paralogy

Two genes are **paralogs** when they are copies of each other **inside the same
genome**, produced by duplication.

The contrast is worth fixing clearly:

- **Ortholog** — a gene in species A and its counterpart in species B. Separated by *speciation*.
- **Paralog** — a gene in species A and another gene in species A. Separated by *duplication*.

So paralogy analysis means comparing a genome **with itself**. SyntenyScan does
this for both species automatically, in the same run, because both genomes go
into a single search.

### Why the two belong together

A gene that still sits in its ancestral position, in the same order as its
counterpart in the other species, has usually **not** been duplicated recently
— a duplication would have disturbed the arrangement.

That is the link that makes the tool useful for practical work: **synteny is an
easily measured stand-in for "this gene has no recent copies."**

## 4. Opening the tool

**From GitHub** — click the blue **Open in Colab** badge on the repository's
front page. The notebook opens ready to run.

**Without GitHub** — go to [colab.research.google.com](https://colab.research.google.com),
choose **File → Upload notebook**, and select `SyntenyScan.ipynb`.

Either way you need a free Google account and a browser. Nothing else.

## 5. Running it — step by step

Choose **Runtime → Run all** from the menu, then leave it alone.

| Step | What it does | Roughly |
|---|---|---|
| 1 | Reads your settings and checks them | instant |
| 2 | Installs DIAMOND, MCScanX and the NCBI download tool | 2–3 min |
| 3 | Downloads both genomes from NCBI | 2–5 min |
| 4 | Picks one representative protein per gene | 1–2 min |
| 5 | **Compares every protein with every other** | **10–30 min** |
| 6 | Keeps the best five hits per query per species | seconds |
| 7 | Finds the collinear blocks | 1–2 min |
| 8 | Writes the result tables | seconds |
| 9 | Draws the dot plot | seconds |
| 10 | Packs everything and downloads it | seconds |

A spinning circle beside a cell means it is working. A green tick means it
finished.

**Keep the browser tab open and visible.** Google disconnects notebooks left
idle. If it does disconnect, simply run it again from the top: every step
checks whether it has already done its work and skips it, so a second run is
much faster.

## 6. A worked example: pigeonpea and soybean

The notebook's default settings compare pigeonpea with soybean. Running it
unchanged reproduces a published analysis, which is a useful way to confirm
that everything is working before you try your own species.

**Settings (Step 1, unchanged):**

```python
SPECIES_A_ACCESSION = "GCF_000340665.2"   # pigeonpea
SPECIES_B_ACCESSION = "GCF_000004515.6"   # soybean
```

**What `summary.txt` should say:**

```
between the two species                   2,429 blocks    38,865 anchors
Cajanus cajan against itself                198 blocks     1,949 anchors
Glycine max against itself                  947 blocks    30,791 anchors

Cajanus cajan genes inside a shared block: 16,343 of 29,045 (56.3%)
```

**What those numbers mean.**

Pigeonpea and soybean still share 2,429 stretches of preserved gene order,
covering 56.3% of all pigeonpea genes. That is a strong signal and makes
soybean a good reference for pigeonpea.

The two self-comparison lines are the more interesting result. Soybean
compared with itself gives **947** duplicated blocks; pigeonpea gives only
**198**. Soybean's ancestor doubled its entire genome and the duplicated
regions are still visible. Pigeonpea's ancestor did not.

That single contrast — 947 against 198 — is a real biological finding produced
in one run, and it has a practical consequence: pigeonpea has fewer recent gene
copies to confuse a gene-based marker.

## 7. Reading every output file

### 7.1 `summary.txt` — read this first

Five lines. The first gives shared ancestry between your two species; the next
two give duplication history within each. Everything else is detail.

A rough guide to the first line:

| Genes in shared blocks | Interpretation |
|---|---|
| above 50% | a close, well-assembled relative — a good reference |
| 25–50% | usable, but blocks will be shorter and fewer |
| below 25% | too distant, or one assembly is too fragmented |

### 7.2 `blocks.tsv` — one row per collinear block

Opens directly in Excel.

| Column | Meaning |
|---|---|
| `block` | block number |
| `chr_a`, `chr_b` | the two chromosomes joined |
| `n_anchors` | how many gene pairs it contains — its size |
| `orient` | `plus` = same direction, `minus` = the block is inverted |
| `score`, `evalue` | MCScanX's own confidence measures |
| `kind` | **the most useful column** — see below |

`kind` tells you what sort of block it is. With codes `cc` and `gm`:

- `cc-gm` — joins the two species. **Shared ancestry.**
- `gm-gm` — soybean with itself. **Duplication within soybean.**
- `cc-cc` — pigeonpea with itself. **Duplication within pigeonpea.**

Sort or filter on this column first. Almost every question you have is
answered by looking at one `kind` at a time.

### 7.3 `anchors.tsv` — one row per matched gene pair

Every gene pair inside every block, with the gene identifiers. This is the file
you join to your own data.

If you have a list of candidate genes — markers, QTL genes, a gene family —
look them up in the `gene_a` and `gene_b` columns. A gene that appears is
inside a conserved block. A gene that does not, is not.

> **A design decision worth understanding.** MCScanX expects roughly the top
> five matches per gene. Taking the top five *overall* from a two-species
> database is a trap: a gene with several close copies of itself lets those
> copies fill all five slots, so its matches in the other species are discarded
> and it can never be placed in a shared block.
>
> Since those same copies are exactly what make a gene-based marker amplify
> more than one locus, that shortcut would **manufacture part of the very
> pattern such a study sets out to test**. SyntenyScan splits the quota per
> target species, which removes the problem. The setting is `HITS_PER_SPECIES`.

### 7.4 `tandem.tsv` — genes in tandem arrays

Genes sitting immediately beside their own near-identical copies. These are the
worst behaved genes in any marker set: a primer pair designed in one of them
will usually find its neighbours too.

Disease-resistance genes cluster here heavily. In the pigeonpea run, 35.7% of
disease-resistance markers were in tandem arrays against 11.8% of markers
overall.

### 7.5 `dotplot.png` and `.pdf`

Every point is one matched gene pair, positioned by where it sits on each
genome. Diagonal streaks are conserved blocks.

**How to read it.** If one genome has doubled since the two species split, each
region of the other lines up against **two** parallel streaks instead of one.
That doubling is visible at a glance, without any statistics. In the
pigeonpea–soybean plot the one-to-many pattern is unmistakable.

The PDF is vector, so it can be enlarged without blurring and is suitable for
publication.

## 8. What researchers gain from it

### Choosing the right reference species, with evidence

Before designing markers in any crop, most groups pick a reference relative by
reputation. SyntenyScan lets you pick it by measurement: run the candidate
against two or three relatives, compare the shared-block counts, and use the
one that actually shows the most conserved gene order. What used to be weeks of
setup is now three runs.

### Predicting which markers will misbehave, before spending money

This is the application the tool was built for. In the pigeonpea study,
markers whose genes lay inside a conserved block amplified more than one locus
**3.9%** of the time. Markers outside those blocks did so **25.4%** of the time
— a six-and-a-half-fold difference.

Filtering a marker panel on that basis raised the expected single-locus rate
from **87.8% to 96.1%**, at the cost of discarding 38.5% of candidates. The
screening costs computer time only: no reagents, no gels.

For a laboratory ordering hundreds of primer pairs, that is the difference
between a usable panel and a season of failed reactions.

### Reading duplication history directly

The two self-comparison lines tell you whether your crop carries a recent
whole-genome duplication. That determines how much trouble you should expect
from gene families in every downstream analysis — marker design, expression
work, candidate-gene studies.

### Making an analysis reproducible

Anyone — a reviewer, a collaborator, a student — can click the badge and repeat
your analysis exactly, with the same parameters, without installing anything.
That is increasingly what journals and funders expect.

### Teaching

The notebook is readable top to bottom. Each step says what it does and why.
It works as a practical class in comparative genomics for students who would
otherwise spend the session fighting a compiler.

## 9. Using it on your own crop

Change four lines in **Step 1**:

```python
SPECIES_A_ACCESSION = "GCF_xxxxxxxxx.x"   # your crop
SPECIES_A_CODE      = "ab"                # any two lowercase letters
SPECIES_A_NAME      = "Genus species"

SPECIES_B_ACCESSION = "GCF_yyyyyyyyy.y"   # the reference relative
SPECIES_B_CODE      = "cd"
SPECIES_B_NAME      = "Genus species"
```

**Finding an accession.** Search at
[NCBI Datasets](https://www.ncbi.nlm.nih.gov/datasets/genome/). Use the
**GCF_** number, not GCA_ — the pipeline reads RefSeq annotation, and only GCF_
records carry it. Check that the genome's page shows an **Annotation Release**;
not every assembly at NCBI is annotated.

**Choosing a reference relative.** In order of importance: same family or tribe;
a chromosome-level assembly; mature annotation. A distant relative with a superb
assembly usually beats a close relative with a fragmented one.

Some pairs to consider:

| Crop | Reasonable reference |
|---|---|
| Chickpea | *Medicago truncatula*, soybean |
| Mungbean, urdbean | Common bean, soybean |
| Lentil | *Medicago truncatula* |
| Pearl millet | Foxtail millet, sorghum |
| Finger millet | Foxtail millet, rice |

## 10. Limitations, stated plainly

**It measures gene order, not function.** A gene inside a conserved block is
not thereby important, or expressed, or useful. It is merely unlikely to have
recent copies.

**Assembly quality sets the ceiling.** A fragmented, scaffold-level assembly
cannot show long blocks, because the blocks are broken across scaffolds. Low
numbers may say more about the assembly than about the biology.

**It does not design markers or run e-PCR.** SyntenyScan produces the synteny
and duplication layer. To repeat a full marker study you still need a marker
set to join to it.

**The two programs are the authors' own.** DIAMOND and MCScanX do the
scientific work. SyntenyScan automates and connects them. Cite them.

**Annotation quality matters.** Genes missing from the annotation are missing
from the analysis. Two genomes annotated to different standards will give an
artificially low shared-block count.

## 11. When something goes wrong

| Message | Meaning | What to do |
|---|---|---|
| `protein.faa not found for GCF_...` | that genome has no RefSeq protein annotation | pick a genome whose NCBI page lists an Annotation Release |
| `MCScanX produced no output` | no detectable blocks | the two species are too distant, or an assembly is too fragmented |
| Session disconnected during Step 5 | Colab stops idle notebooks | keep the tab visible; re-run from the top, it resumes |
| Out of memory | two very large genomes | **Runtime → Change runtime type → High-RAM**, if your account offers it |
| `MCScanX BUILD FAILED` | the compiler patch did not apply | report it as an issue on the repository, with the printed log |

A dot plot is only drawn when **both** genomes have numbered chromosomes. For a
scaffold-level assembly the tables are still complete; only the figure is
omitted, and the notebook says so.

## 12. Citing the tool

SyntenyScan runs two programs, and they are what does the scientific work:

- Buchfink B, Xie C, Huson DH (2015) Fast and sensitive protein alignment using
  DIAMOND. *Nature Methods* 12:59–60. https://doi.org/10.1038/nmeth.3176
- Wang Y, Tang H, DeBarry JD, Tan X, Li J, Wang X, Lee TH, Jin H, Marler B,
  Guo H et al (2012) MCScanX: a toolkit for detection and evolutionary analysis
  of gene synteny and collinearity. *Nucleic Acids Research* 40:e49.
  https://doi.org/10.1093/nar/gkr1293

Also cite the genome papers for whichever species you use, and this repository.

---

*SyntenyScan was developed at ICAR–National Bureau of Plant Genetic Resources,
New Delhi, from the analysis pipeline of a study of intron-spanning marker
specificity in pigeonpea.*
