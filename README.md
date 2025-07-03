# Methanogen-Comparison-Pangenome-w-Anvio

This pipeline allows you to (relatively) quickly conduct a pangenome analysis of up to 100 genomes. 

## Step 1: Download the genomes you want to include in the analysis
If you have genomes that aren't included in the "methanogen_MAGs.zip" file in this repo, download other genomes of interest from [NCBI](https://www.ncbi.nlm.nih.gov/home/genomes/)
The "methanogen_MAGs.zip" contains the following genomes: 
```
Mnat_SPAdes.Assembly_copy.fa
NSHQ14B.bins.3.fasta
NSHQ14C.bins.4.fasta
accession_taxonomy_modified.csv
bin.009_Methanobacterium.fasta_assembly.fa
bin.019_Methanobacterium.fasta_assembly.fa
Lost_city_Methanosarcaceae.fna
Lost_city_Methanocellales.fna
Methanobacteriaceae_archaeon_OMB002.fna
Methanobacteriaceae_archaeon_OMB007.fna
Methanobacterium_aggregans_5_bin_148.fna
Methanobacterium_articum_strain_M2.fna
Methanobacterium_bryantii strain_MoH..fna
Methanobacterium_formicicum_Mb9.fna
Methanobacterium_formicicum_strain_DSM_1535.fna
Methanobacterium_sp_ERen5.fna
Methanobacterium_sp_IBPM_KMB_004.fna
Methanobacterium_sp_OMB001.fna
Methanobacterium_sp_OMB003.fna
Methanobacterium_sp_OMB004.fna
Methanobacterium_sp_OMB005.fna
Methanobacterium_sp_OMB006.fna
Methanobacterium_sp_strain_34x.fna
Methanobacterium_spitsbergense_VT.fna
Methanobacterium_veterum_MK4.fna
```

## Step 2: Install Anvi'o 
Follow the directions here: https://anvio.org/install/linux/stable/

## Step 3: Build all of the contig_db's, run the hmms, and annotate the COGs and KEGGs
### [contig_db](https://anvio.org/help/main/artifacts/contigs-db/) - An anvi'o database that stores the sequence's analysis information in one place. 
### [hmmer](https://anvio.org/help/7/programs/anvi-run-hmms/) - a program to to identify homologous protein or nucleotide sequences and to perform sequence alignments
### annotate [COGs](https://anvio.org/help/7/programs/anvi-run-ncbi-cogs/)and [KEGGs](https://anvio.org/help/7/programs/anvi-run-kegg-kofams/)
