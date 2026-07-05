# Pilot-Study_1
Pilot Study with metabolomic root/tissue data from Pinus taeda

# 1. Go to your downloaded Pinus taeda genome folder
cd "C:\Users\norba\Downloads\ncbi_dataset\ncbi_dataset\data\GCA_000404065.3"

# 2. Confirm the genome FASTA exists
ls *.fna
ls *.gbff

# 3. Optional: inspect the GenBank file
Get-Content genomic.gbff -TotalCount 50

# 4. Optional: check whether CDS annotations exist
Select-String -Pattern "     CDS     " -Path genomic.gbff | Select-Object -First 10



#NCBI searches to do first

Search NCBI Protein for terpene synthase reference proteins:

Pinus taeda terpene synthase
Pinus taeda alpha-farnesene synthase
Pinus taeda alpha-pinene synthase
Pinus taeda alpha-terpineol synthase
Pinus taeda diterpene synthase
Pinus terpene synthase limonene
Pinus terpene synthase myrcene
Pinus terpene synthase longifolene
Pinus ent-kaurene synthase



#Download selected protein sequences as FASTA and save them as:

C:\Users\norba\Downloads\sequence.fasta

Check your downloaded protein references:

Select-String -Pattern ">" -Path "C:\Users\norba\Downloads\sequence.fasta"
Build the BLAST database
& "C:\Users\norba\Downloads\ncbi-blast-2.17.0+-x64-win64\ncbi-blast-2.17.0+\bin\makeblastdb.exe" `
-in "GCA_000404065.3_Ptaeda2.0_genomic.fna" `
-dbtype nucl `
-out PinusGenome
Run tblastn
& "C:\Users\norba\Downloads\ncbi-blast-2.17.0+-x64-win64\ncbi-blast-2.17.0+\bin\tblastn.exe" `
-query "C:\Users\norba\Downloads\sequence.fasta" `
-db PinusGenome `
-evalue 1e-20 `
-outfmt 6 `
-num_threads 8 `
-out TPS_hits.tsv
Check the BLAST results
ls TPS_hits.tsv
Get-Content TPS_hits.tsv -TotalCount 10
Convert BLAST output to best-hit CSV
Import-Csv TPS_hits.tsv -Delimiter "`t" -Header query,subject,pident,length,mismatch,gapopen,qstart,qend,sstart,send,evalue,bitscore |
Sort-Object query, @{Expression={[double]$_.bitscore}; Descending=$true} |
Group-Object query |
ForEach-Object { $_.Group | Select-Object -First 1 } |
Export-Csv TPS_best_hits.csv -NoTypeInformation

#Open it:

start TPS_best_hits.csv

#Optional: top 5 hits per protein
Import-Csv TPS_hits.tsv -Delimiter "`t" -Header query,subject,pident,length,mismatch,gapopen,qstart,qend,sstart,send,evalue,bitscore |
Sort-Object query, @{Expression={[double]$_.bitscore}; Descending=$true} |
Group-Object query |
ForEach-Object { $_.Group | Select-Object -First 5 } |
Export-Csv TPS_top5_hits.csv -NoTypeInformation

start TPS_top5_hits.csv

#If you downloaded many FASTA files into a folder

Your folder was named like a file:

C:\Users\norba\Downloads\expanded_TPS_refs.fasta

but it was actually a directory. Combine all FASTA files inside it:

Get-Content "C:\Users\norba\Downloads\expanded_TPS_refs.fasta\*.fasta" |
Out-File "C:\Users\norba\Downloads\expanded_TPS_refs_combined.fasta" -Encoding ascii

#Check headers:

Select-String -Pattern ">" -Path "C:\Users\norba\Downloads\expanded_TPS_refs_combined.fasta"

#Run expanded BLAST:

& "C:\Users\norba\Downloads\ncbi-blast-2.17.0+-x64-win64\ncbi-blast-2.17.0+\bin\tblastn.exe" `
-query "C:\Users\norba\Downloads\expanded_TPS_refs_combined.fasta" `
-db PinusGenome `
-evalue 1e-20 `
-outfmt 6 `
-num_threads 8 `
-out TPS_expanded_hits.tsv

#Best expanded hits:

Import-Csv TPS_expanded_hits.tsv -Delimiter "`t" -Header query,subject,pident,length,mismatch,gapopen,qstart,qend,sstart,send,evalue,bitscore |
Sort-Object query, @{Expression={[double]$_.bitscore}; Descending=$true} |
Group-Object query |
ForEach-Object { $_.Group | Select-Object -First 1 } |
Export-Csv TPS_expanded_best_hits.csv -NoTypeInformation

start TPS_expanded_best_hits.csv

#Simple interpretation

Use TPS_best_hits.csv for your first-pass candidate loci. Use TPS_top5_hits.csv to see possible gene family duplicates, paralogs, fragmented hits, or multiple scaffold matches. Use TPS_expanded_best_hits.csv when you want broader terpene synthase candidates beyond the first Pinus taeda references.
