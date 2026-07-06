Pilot Study 1
Connecting GC-MS Metabolites to Candidate Terpene Synthase (TPS) Genes in Pinus taeda

Objective

Develop a reproducible bioinformatics workflow that links metabolites detected by GC-MS to candidate terpene synthase genes in the Pinus taeda genome.

#Phase I – Candidate Discovery

1. Navigate to the downloaded genome directory
cd "C:\Users\norba\Downloads\ncbi_dataset\ncbi_dataset\data\GCA_000404065.3"

2. Confirm the genome files exist
ls *.fna
ls *.gbff

Expected files

GCA_000404065.3_Ptaeda2.0_genomic.fna
genomic.gbff

3. Inspect the GenBank annotation file
Get-Content genomic.gbff -TotalCount 50

Purpose

Verify the genome downloaded correctly.
Examine metadata and annotation format.

4. Search for coding sequence (CDS) annotations
Select-String -Pattern "     CDS     " -Path genomic.gbff | Select-Object -First 10

Purpose

Determine whether coding sequences are annotated.

Result

No useful TPS annotations were identified from this search.

#Phase II – Reference Protein Collection

Search NCBI Protein using terms such as:

Pinus taeda terpene synthase

Pinus taeda alpha-pinene synthase

Pinus taeda alpha-farnesene synthase

Pinus taeda alpha-terpineol synthase

Pinus diterpene synthase

Pinus limonene synthase

Pinus myrcene synthase

Pinus longifolene synthase

Pinus ent-kaurene synthase

Download the protein FASTA sequences as

C:\Users\norba\Downloads\sequence.fasta

Verify the downloaded proteins

Select-String -Pattern ">" -Path "C:\Users\norba\Downloads\sequence.fasta"

#Phase III – Build the Local Genome Database

Create a searchable BLAST database

& "C:\Users\norba\Downloads\ncbi-blast-2.17.0+-x64-win64\ncbi-blast-2.17.0+\bin\makeblastdb.exe" `
-in "GCA_000404065.3_Ptaeda2.0_genomic.fna" `
-dbtype nucl `
-out PinusGenome

Later, rebuild the database with accession information so sequences can be extracted directly:

& "C:\Users\norba\Downloads\ncbi-blast-2.17.0+-x64-win64\ncbi-blast-2.17.0+\bin\makeblastdb.exe" `
-in "GCA_000404065.3_Ptaeda2.0_genomic.fna" `
-dbtype nucl `
-parse_seqids `
-out PinusGenome

Purpose

Convert the FASTA genome into an indexed BLAST database.

#Phase IV – Search the Genome with tblastn
& "C:\Users\norba\Downloads\ncbi-blast-2.17.0+-x64-win64\ncbi-blast-2.17.0+\bin\tblastn.exe" `
-query "C:\Users\norba\Downloads\sequence.fasta" `
-db PinusGenome `
-evalue 1e-20 `
-outfmt 6 `
-num_threads 8 `
-out TPS_hits.tsv

Inspect results

ls TPS_hits.tsv

Get-Content TPS_hits.tsv -TotalCount 10

#Phase V – Rank Candidate Hits

Generate the best hit for each protein

Import-Csv TPS_hits.tsv -Delimiter "`t" `
-Header query,subject,pident,length,mismatch,gapopen,qstart,qend,sstart,send,evalue,bitscore |
Sort-Object query,@{Expression={[double]$_.bitscore};Descending=$true} |
Group-Object query |
ForEach-Object {$_.Group | Select-Object -First 1} |
Export-Csv TPS_best_hits.csv -NoTypeInformation

Open

start TPS_best_hits.csv

Generate the top five hits

Import-Csv TPS_hits.tsv -Delimiter "`t" `
-Header query,subject,pident,length,mismatch,gapopen,qstart,qend,sstart,send,evalue,bitscore |
Sort-Object query,@{Expression={[double]$_.bitscore};Descending=$true} |
Group-Object query |
ForEach-Object {$_.Group | Select-Object -First 5} |
Export-Csv TPS_top5_hits.csv -NoTypeInformation

Open

start TPS_top5_hits.csv
Phase VI – Expand the Reference Protein Dataset

If multiple FASTA files were downloaded

Get-Content "C:\Users\norba\Downloads\expanded_TPS_refs.fasta\*.fasta" |
Out-File "C:\Users\norba\Downloads\expanded_TPS_refs_combined.fasta" -Encoding ascii

Verify the combined FASTA

Select-String -Pattern ">" `
-Path "C:\Users\norba\Downloads\expanded_TPS_refs_combined.fasta"

Run an expanded tblastn search

& "C:\Users\norba\Downloads\ncbi-blast-2.17.0+-x64-win64\ncbi-blast-2.17.0+\bin\tblastn.exe" `
-query "C:\Users\norba\Downloads\expanded_TPS_refs_combined.fasta" `
-db PinusGenome `
-evalue 1e-20 `
-outfmt 6 `
-num_threads 8 `
-out TPS_expanded_hits.tsv

Create the best-hit table

Import-Csv TPS_expanded_hits.tsv -Delimiter "`t" `
-Header query,subject,pident,length,mismatch,gapopen,qstart,qend,sstart,send,evalue,bitscore |
Sort-Object query,@{Expression={[double]$_.bitscore};Descending=$true} |
Group-Object query |
ForEach-Object {$_.Group | Select-Object -First 1} |
Export-Csv TPS_expanded_best_hits.csv -NoTypeInformation

Open

start TPS_expanded_best_hits.csv

Interpretation

TPS_best_hits.csv → strongest candidate loci.
TPS_top5_hits.csv → possible paralogs, duplicated genes, or fragmented alignments.
TPS_expanded_best_hits.csv → broader TPS candidates from multiple Pinaceae reference proteins.
Phase VII – Candidate Locus Extraction

The strongest candidate identified in this pilot study was:

APFE031366321.1

Extract the candidate region

& "C:\Users\norba\Downloads\ncbi-blast-2.17.0+-x64-win64\ncbi-blast-2.17.0+\bin\blastdbcmd.exe" `
-db PinusGenome `
-entry APFE031366321.1 `
-range 1-20000 `
-out APFE031366321_TPS_20kb.fasta

Purpose

Extract a 20 kb genomic region surrounding the candidate TPS locus for detailed analysis.

#Phase VIII – Build a Candidate-Specific Database

& "C:\Users\norba\Downloads\ncbi-blast-2.17.0+-x64-win64\ncbi-blast-2.17.0+\bin\makeblastdb.exe" `
-in APFE031366321_TPS_20kb.fasta `
-dbtype nucl `
-parse_seqids `
-out APFE031366321_TPS_20kb_DB

Purpose

Create a BLAST database containing only the candidate region, enabling higher-resolution local analyses.

#Phase IX – High-Resolution Local tblastn

& "C:\Users\norba\Downloads\ncbi-blast-2.17.0+-x64-win64\ncbi-blast-2.17.0+\bin\tblastn.exe" `
-query "C:\Users\norba\Downloads\expanded_TPS_refs_combined.fasta" `
-db APFE031366321_TPS_20kb_DB `
-evalue 1e-5 `
-outfmt "6 qseqid sseqid pident length qstart qend sstart send evalue bitscore qseq sseq" `
-num_threads 8 `
-out APFE031366321_TPS_20kb_tblastn.tsv

Purpose

Map protein alignments at high resolution within the candidate genomic region.

#Phase X – Candidate Validation (CURRENT STAGE)
Step 15. Create a Protein Evidence Map

Question: Where do all TPS proteins align on the 20 kb candidate region?

Output

A graphical map showing every protein alignment along the genomic DNA.

Step 16. Infer Candidate Exon-Like Alignment Blocks

Use the clustered protein alignments to identify candidate coding regions.

Important: These are evidence-supported alignment blocks and are not yet confirmed exons.

Step 17. Reconstruct the Candidate Gene

Combine the alignment evidence to infer:

exon order
intron positions
coding sequence (CDS)

Step 18. Translate the Predicted Coding Sequence

Generate the predicted amino acid sequence encoded by the reconstructed gene.

Step 19. Identify Conserved TPS Motifs

Search the predicted protein for conserved terpene synthase motifs, including:

RR(X)₈W
DDXXD
NSE/DTE

These motifs are characteristic of functional TPS enzymes.

Step 20. Validate with BLASTP

Compare the reconstructed protein against the NCBI Protein database to identify the closest characterized homologs.

Step 21. Perform Phylogenetic Analysis

Construct a phylogenetic tree to determine whether the candidate protein clusters with:

α-pinene synthases
α-farnesene synthases
humulene synthases
longifolene synthases
diterpene synthases
other TPS families
Step 22. Integrate Genomic and Metabolomic Evidence

Combine:

GC-MS metabolite profiles
protein homology
gene structure
conserved motifs
phylogenetic placement

to infer the most likely function of the candidate TPS gene.

Phase XI – Scale the Workflow

After validating the workflow on Candidate 1, repeat Phases II–X for each remaining metabolite identified in the GC-MS dataset.

This pilot study establishes a reproducible framework for linking metabolomic observations to candidate biosynthetic genes in Pinus taeda. It is intended to serve as a reusable pipeline for future metabolite-to-gene discovery analyses.