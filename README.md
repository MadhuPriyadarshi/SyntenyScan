# SyntenyScan

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/YOUR-USERNAME/SyntenyScan/blob/main/SyntenyScan.ipynb)

Find conserved gene order (**synteny**) between any two annotated genomes at
NCBI, and the recent duplications (**paralogy**) inside each one — without
installing anything.

Click the badge above. The notebook opens in Google Colab, installs its own
software on Google's machine, downloads the genomes, runs the analysis and
hands back a ZIP of results. Nothing is installed on your computer, and it is
free.

---

**New to this? Read the [Tutorial](TUTORIAL.md)** — what the tool is,
what synteny and paralogy mean, a worked example, how to read every output
file, and what to use it for.

---

## What it does

| Step | What happens |
|---|---|
| 1 | You give two NCBI RefSeq accessions |
| 2 | DIAMOND and MCScanX are installed automatically |
| 3 | Both genomes are downloaded from NCBI |
| 4 | One representative protein is chosen per gene (longest coding sequence) |
| 5 | Every protein is compared with every other (DIAMOND) |
| 6 | Hits are cut to the best five per query **per target species** |
| 7 | Collinear blocks are detected (MCScanX) |
| 8 | Results are written as tables |
| 9 | A whole-genome dot plot is drawn |
| 10 | Everything downloads as a ZIP |

Because both genomes go into one search, the run finds blocks **between** the
two species and **within** each of them at the same time. The within-species
blocks are the signature of past genome duplication.

## What you get

| File | Contents |
|---|---|
| `blocks.tsv` | every collinear block, with chromosomes, size and orientation |
| `anchors.tsv` | every matched gene pair inside those blocks |
| `tandem.tsv` | genes lying in tandem arrays |
| `summary.txt` | the headline counts |
| `dotplot.png` / `.pdf` | the whole-genome comparison figure |

## Changing the species

Edit four lines in **Step 1** of the notebook:

```python
SPECIES_A_ACCESSION = "GCF_000340665.2"   # pigeonpea, C. cajan V1.1
SPECIES_A_CODE      = "cc"
SPECIES_A_NAME      = "Cajanus cajan"

SPECIES_B_ACCESSION = "GCF_000004515.6"   # soybean, Glycine max v4.0
SPECIES_B_CODE      = "gm"
SPECIES_B_NAME      = "Glycine max"
```

Find accessions at [NCBI Datasets](https://www.ncbi.nlm.nih.gov/datasets/genome/).
Use the **GCF_** (RefSeq) number, not GCA_ — the pipeline reads RefSeq
annotation. Not every assembly at NCBI is annotated; check that the genome's
page lists an Annotation Release.

The two-letter codes are short labels that appear in the output (`cc01`,
`gm14`). They must be two lowercase letters and must differ from each other.

## One design decision worth knowing about

MCScanX expects roughly the top five matches per gene. Taking the top five
*overall* from a two-species database is a trap: a gene with several close
copies of itself lets those copies fill all five slots, so its matches in the
other species are discarded and it can never be placed in a shared block.

Since those same copies are what make a gene-based marker amplify more than one
locus, that shortcut would manufacture part of the very pattern such a study
sets out to test. SyntenyScan splits the quota **per target species**, which
removes the problem. This is `HITS_PER_SPECIES` in Step 1.

## Validation

The analysis code is taken from the scripts behind the pigeonpea–soybean study
this notebook was built from, and is checked against that published run:

| Check | Expected | Notebook |
|---|---|---|
| Pigeonpea representative proteins | 29,045 | 29,045 |
| Soybean representative proteins | 47,068 | 47,068 |
| Pigeonpea × soybean blocks | 2,429 | 2,429 |
| Pigeonpea × soybean anchors | 38,865 | 38,865 |
| Soybean self-comparison blocks | 947 | 947 |
| Pigeonpea self-comparison blocks | 198 | 198 |
| Pigeonpea genes in a shared block | 16,343 (56.3%) | 16,343 (56.3%) |

## How long it takes

Ten to thirty minutes for two plant genomes on Colab's free tier, almost all of
it in the DIAMOND step. Keep the browser tab open — Colab disconnects notebooks
left idle. Re-running from the top is safe: every step skips work it has already
done.

## Requirements

None on your computer. A free Google account, and a browser.

## Citing

SyntenyScan runs two programs. If you publish results from it, cite them:

- Buchfink B, Xie C, Huson DH (2015) Fast and sensitive protein alignment using
  DIAMOND. *Nat Methods* 12:59–60. https://doi.org/10.1038/nmeth.3176
- Wang Y, Tang H, DeBarry JD, Tan X, Li J, Wang X, Lee TH, Jin H, Marler B,
  Guo H et al (2012) MCScanX: a toolkit for detection and evolutionary analysis
  of gene synteny and collinearity. *Nucleic Acids Res* 40:e49.
  https://doi.org/10.1093/nar/gkr1293

and the genome papers for whichever species you use.

## Licence

MIT — see [LICENSE](LICENSE).

## Authors

Madhu Bala Priyadarshi, Rakesh Singh and Jeshima Khan Yasin
ICAR–National Bureau of Plant Genetic Resources, Pusa Campus, New Delhi 110012, India
