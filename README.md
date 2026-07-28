# PhyloMiner
`PhyloMiner` is a command-line pipeline for identifying and extracting homologous sequences from collections of protein or coding sequence (CDS) databases.

It was developed to streamline a common comparative genomics workflow: searching databases for homologs, filtering candidate hits, extracting matching sequences from the original databases, and preparing datasets for downstream phylogenetic analysis. `PhyloMiner` integrates established homology search and domain-filtering approaches into a single reproducible workflow, reducing the need for manual sequence processing during large-scale gene family analyses across diverse genomic datasets.

## Overview
`PhyloMiner` automates the following steps:

- Automatically detects whether the query and database are protein or nucleotide input.
- Searches protein databases directly with `phmmer` and retains homologs above the inclusion threshold.
- There is an optional permissive mode using `--keep-below-threshold` which retains HMMER hits which did not meet the inclusion threshold. 
- Translates CDS databases to protein before searching using `transeq`.
- Optionally filters candidate hits with one or more Pfam profile HMMs using `hmmsearch`.
- Extracts matching protein sequences.
- Extracts corresponding CDS sequences when a nucleotide database is used.
- Writes summary tables and run information for downstream analysis and record keeping.
- Optionally retains intermediate files for inspection and troubleshooting.
- The pipeline is designed to be robust in handling inconsistent gene identifiers and difficult datasets.

## Installation 
### Dependencies
`PhyloMiner` uses the following tools:

- HMMER (`phmmer`, `hmmsearch`, `hmmfetch`)
- EMBOSS (`transeq`)
- SeqKit (`seqkit`)

### Download
Download dependencies using anaconda:
```
conda install -c bioconda hmmer emboss seqkit
```
or homebrew:
```
brew install hmmer emboss seqkit
```
If you plan to use Pfam-based filtering, you will also need a Pfam HMM database such as `Pfam-A.hmm`.
Download Pfam-A.hmm from https://www.ebi.ac.uk/interpro/download/Pfam/ (last accessed 02-07-2026).

### Usage
`PhyloMiner` is currently distributed as a shell script. Make the script executable:
```
chmod +x phylominer.sh
```
You can then run it directly:
```
./phylominer.sh [options] query.fasta /path/to/databases
```
Or, without changing permissions:
```
bash phylominer.sh [options] query.fasta /path/to/databases
```
For Pfam filtering, provide the location of Pfam-A.hmm using the `--pfam-db` option.

## Quickstart
```
./phylominer.sh [options] query.fasta /path/to/databases
```
### Required arguments
- `query.fasta`
Protein or CDS FASTA query
- `/path/to/databases`
Directory containing FASTA databases.
Note: these should be in `.fa` format as this file format is protected from accidental deletion.

### Options
- `-f`
Overwrite existing outputs
- `--threads N`
Number of threads for phmmer and hmmsearch (default: 1)
- `--keep-temp`
Retain translated files, hit lists, and search tables
- `--include-below-threshold`
Retain phmmer hits below the default HMMER inclusion threshold
- `--motif-hmm HMM_ID`
Filter extracted proteins using one or more HMM profiles
- `--pfam-db PATH`
HMM database used for `--motif-hmm`
- `-h`, `--help`
Show help and exit.

## Workflow
```
                Query FASTA
                     │
                     ▼
        Detect protein or CDS database
                     │
         ┌───────────┴───────────┐
         ▼                       ▼
 Protein database         Coding sequence database
         │                       │
  (use directly)          (translate to protein)
         └───────────┬───────────┘
                     ▼
               phmmer search
                     ▼
             Candidate homologs
                     ▼
      (Optional) Pfam HMM filtering
                     ▼
Extract protein homologs from original database
                     ▼
If nucleotide database: Extract CDS sequences too
                     ▼
 Protein FASTAs + CDS FASTAs + summary table
```
## Output

A typical run produces:

- extracted protein FASTA files
- extracted CDS FASTA files, where applicable
- a summary table of hits across databases
- a run log containing the command used and version information
- optional intermediate files if `--keep-temp` is used

These outputs are useful for:

- presence/absence screening
- gene family surveys across many genomes
- downstream multiple sequence alignment
- phylogenetic inference and gene family evolution analyses

## Below-threshold searches

- By default, `PhyloMiner` follows the standard HMMER inclusion thresholds and reports only statistically significant homologs. However, `PhyloMiner` can optionally retain hits below the default HMMER inclusion threshold (`--include-below-threshold`).
- This is a practical way to search for highly divergent homologs, and is particularly useful when used in combination with the optional Pfam domain filtering (`--motif-hmm HMM_ID`) to recover true family members that might otherwise be missed.
- Including below-threshold hits may also be desirable when searching distantly related species, poorly annotated genomes, or attempting to trace the broader evolutionary origins of a particular gene family.

## Other notes
- If a database is nucleotide-based, both protein and CDS outputs are written.
- Input databases should be in `.fa` FASTA format, which are protected from deletion by the script.
- `PhyloMiner` could be combined with _ab initio_ annotation software, such as `Helixer`, to reduce biases introduced by different annotation softwares for lineage-specific gene discovery. 

## Downstream analysis
After extracting homologs, standard phylogenetic processing can be carried out with the following recommended tools:
- `mafft` for alignment
- `trimal` or `phyx` for automatic trimming
- `iqtree` or `iqtree2` for tree inference
- `iTOL` or `FigTree` for tree visualisation

A typical workflow is:
```
mafft --localpair --maxiterate 1000 input.fasta > output_align.fasta
trimal -in output_align.fasta -out output_trim.fasta -fasta -gappyout
iqtree2 -s output_trim.fasta -m MFP -bb 10000 -ninit 10000 -nm 10000 -T AUTO
```
## Citation
If you use `PhyloMiner` in published research, please consider citing this repository or the associated publication, when available:
- Hoey DJ. *PhyloMiner: automated homology mining for comparative phylogenomics*. GitHub repository: https://github.com/davidjhoey/phylominer

## Licence
PhyloMiner is released under the GNU General Public License v3.0 (GPLv3). This means that the software and derivative versions will remain freely available, modifiable, and open source.
