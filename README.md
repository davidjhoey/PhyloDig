# PhyloDig
`PhyloDig` is a command-line pipeline for identifying and extracting homologous sequences from collections of protein or coding sequence (CDS) databases.

It was developed to streamline a common comparative genomics workflow: searching databases for homologs, filtering candidate hits, extracting matching sequences from the original databases, and preparing datasets for downstream phylogenetic analysis. `PhyloDig` integrates established homology search and domain-filtering approaches into a single reproducible workflow, reducing the need for manual sequence processing during large-scale gene family analyses across diverse genomic datasets.

With `PhyloDig`, you can curate a set of locally stored databases which can be easily queried with your sequence of interest. It allows species set or genome version to be adjusted with ease, important considerations when building alignments and phylogenetic trees. It is capable of multifasta queries, and will produce a summary table of all hits at the end of a multifasta run, allowing for quick and easy presence-absence assessment. The output files are organised to be easily integrated into phylogenetic pipelines.

## Overview
`PhyloDig` automates the following steps:

- Automatically detects whether the query and database are protein or nucleotide input (multifasta supported for queries).
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
`PhyloDig` uses the following tools:

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
You will also need to prepare the HMM profiles from Pfam-A.hmm using `hmmpress`. You will only need to do this once.
```
hmmpress ./Pfam-A.hmm
```
This produces four index files which will allow `hmmsearch` to work.
### Usage
`PhyloDig` is currently distributed as a shell script. Make the script executable:
```
chmod +x phylodig.sh
```
You can then run it directly:
```
./phylodig.sh [options] query.fasta /path/to/databases
```
Or, without changing permissions:
```
bash phylodig.sh [options] query.fasta /path/to/databases
```
For Pfam filtering, provide the location of Pfam-A.hmm using the `--pfam-db` option.

## Quickstart
```
./phylodig.sh [options] query.fasta /path/to/databases
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

### Typical directory structure
```
database_directory/
│
├── Input databases (*.fa)
│
├── Query_results/
│   ├── Summary statistics (.csv)
│   ├── Run log (phylodig.txt)
│   ├── Homologous protein sequences/
│   ├── Corresponding CDS sequences/
│   ├── Search log files (optional, --keep-temp)
│   └── Temporary working files (deleted by default)
│
└── .phylodig_cache/
    └── translated_databases/
        └── Cached protein translations of CDS databases
```
In multifasta mode, `PhyloDig` also produces a master `phylodig.txt` log and a summary table of all queries.

## Below-threshold searches

- By default, `PhyloDig` follows the standard HMMER inclusion thresholds and reports only statistically significant homologs. However, `PhyloDig` can optionally retain hits below the default HMMER inclusion threshold (`--include-below-threshold`).
- This is a practical way to search for highly divergent homologs, and is particularly useful when used in combination with the optional Pfam domain filtering (`--motif-hmm HMM_ID`) to recover true family members that might otherwise be missed.
- Including below-threshold hits may also be desirable when searching distantly related species, poorly annotated genomes, or attempting to trace the broader evolutionary origins of a particular gene family.

## Multifasta mode
- Input queries can be in multifasta format, as well as single-sequence input.
- Each query will get its own folder with a folder name derived from the sequence header name.
- Please note: applying PFAM filters in multifasta mode will result in the same PFAM filter being applied to all queries which may not be desirable.
- In multifasta mode, PhyloDig will produce a final summary table with all final protein hits for each query.

## Other notes
- If a database is nucleotide-based, both protein and CDS outputs are written.
- Input databases should be in `.fa` FASTA format, which are protected from deletion by the script.
- `PhyloDig` could be combined with _ab initio_ annotation software, such as `Helixer`, to reduce biases introduced by different annotation softwares for lineage-specific gene discovery.

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

## Statement on the use of AI in development
I have used the assistance of ChatGPT for refining the code and some of the documentation of this pipeline. I have not used it to decide on the steps of the pipeline, nor for the conceptualisation of it. Each stage of the pipeline has been tested thoroughly and is expected to be robust for that reason - the pipeline's outputs are not expected to be affected by the use of AI. However, if you encounter any unusual errors or find that the pipeline is not behaving as expected for a particular dataset, feel free to get in contact and I am happy to look into any issues.

## Citation
If you use `PhyloDig` in published research, please consider citing this repository or the associated publication, when available:
- Hoey DJ. *PhyloDig: automated database mining for comparative phylogenomics*. GitHub repository: https://github.com/davidjhoey/phylodig

## Licence
PhyloDig is released under the GNU General Public License v3.0 (GPLv3). This means that the software and derivative versions will remain freely available, modifiable, and open source.
