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
Methanobacterium_bryantii strain_MoH.fna
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
#### [contig_db](https://anvio.org/help/main/artifacts/contigs-db/) - An anvi'o database that stores the sequence's analysis information in one place. 
#### [hmmer](https://anvio.org/help/7/programs/anvi-run-hmms/) - a program to to identify homologous protein or nucleotide sequences and to perform sequence alignments
#### annotate [COGs](https://anvio.org/help/7/programs/anvi-run-ncbi-cogs/) and [KEGGs](https://anvio.org/help/7/programs/anvi-run-kegg-kofams/)
I have set up Anvi'o to run on the high-performance computing (HPC) cluster, since it runs a lot faster. I'll include two versions of the code - one for running on your local computer, and a second for running on an HPC. 
### Local computer
create a shell script file in nano or textedit
save the file as **Methanogen_comparison_pangenome.sh**
```
#!/bin/bash

#Go into the directory containing your genomes
cd /vortexfs1/omics/huber/selkassas/Methanogen_Comparison_Alta/genomes/

#Optional command - this may be needed for some of the genomes: Reformat assemblies so they can run thru Anvi'o
#for file in *.f
#do
#SAMPLE=$(basename ${file} | cut -d '.' -f 1)
#anvi-script-reformat-fasta ${file} -o ${SAMPLE}.fa -l 0 --simplify-names
#done

#Load COG and KEGG/KOFAM db's into Anvi'o (you only need to do this once)
anvi-setup-ncbi-cogs
anvi-setup-kegg-kofams

#for loop to build all of the contig_db, run the hmms, and annotate the COGs and KEGGs at once for all of your genomes: 
for file in *.f
do
SAMPLE=$(basename ${file} | cut -d '.' -f 1)
#generate a contigs database for each genome
anvi-gen-contigs-database -f "${file}" --project-name ${SAMPLE} -o ${SAMPLE}_contigs-db.db 
#sequence homology and alignment
anvi-run-hmms -c "${SAMPLE}_contigs-db.db"
#annotate gene clusters using COG db
anvi-run-ncbi-cogs -c "${SAMPLE}_contigs-db.db" --num-threads 4
#annotate using KEGG db
anvi-run-kegg-kofams -c "${SAMPLE}_contigs-db.db" --num-threads 4
done 

#generate genomes file that puts your sample name in one column and the path to your sample in a second column
anvi-script-gen-genomes-file --input-dir /vortexfs1/omics/huber/selkassas/Methanogen_Comparison_Alta/genomes/ \
                             --output-file Methanogen_genomes.txt

#stores all the info about your genomes
anvi-gen-genomes-storage -e Methanogen_genomes.txt \
                         -o METHANOGEN_GENOMES.db

#generates your pangenome
anvi-pan-genome -g METHANOGEN_GENOMES.db \
                --project-name "METHANOGEN_PANGENOME" \
                --output-dir METHANOGEN_PANGENOME \
                --num-threads 4 \
                --minbit 0.5
```
Make the script executable: `chmod +x Methanogen_comparison_pangenome.sh`\
Execute the script: `./Methanogen_comparison_pangenome.sh`

### HPC: 
create in nano
save the file as **Methanogen_comparison_pangenome.sh**
```
#!/bin/bash
#SBATCH --partition=compute                         # Queue selection
#SBATCH --job-name=generate_pangenome               # Job name
#SBATCH --mail-type=ALL                             # Mail events (BEGIN, END, FAIL, ALL)
#SBATCH --mail-user=selkassas@whoi.edu              # Where to send mail
#SBATCH --ntasks=1                                  # Run a single task
#SBATCH --cpus-per-task=4                           # Number of CPU cores per task
#SBATCH --mem=80gb                                  # Job memory request
#SBATCH --time=24:00:00								              # Time limit hrs:min:sec
#SBATCH --output=generate_pangenome.log     		    # Job log name
export OMP_NUM_THREADS=4

eval "$(/vortexfs1/home/selkassas/miniforge3/bin/conda shell.bash hook)" 
conda activate /vortexfs1/home/selkassas/miniforge3/envs/anvio-8

#Go into the directory containing your genomes
cd /vortexfs1/omics/huber/selkassas/Methanogen_Comparison_Alta/genomes/

#Optional command - this may be needed for some of the genomes: Reformat assemblies so they can run thru Anvi'o
#for file in *.f
#do
#SAMPLE=$(basename ${file} | cut -d '.' -f 1)
#anvi-script-reformat-fasta ${file} -o ${SAMPLE}.fa -l 0 --simplify-names
#done

#Load COG and KEGG/KOFAM db's into Anvi'o (you only need to do this once)
anvi-setup-ncbi-cogs
anvi-setup-kegg-kofams

#for loop to build all of the contig_db, run the hmms, and annotate the COGs and KEGGs at once for all of your genomes: 
for file in *.f
do
SAMPLE=$(basename ${file} | cut -d '.' -f 1)
#generate a contigs database for each genome
anvi-gen-contigs-database -f "${file}" --project-name ${SAMPLE} -o ${SAMPLE}_contigs-db.db 
#sequence homology and alignment
anvi-run-hmms -c "${SAMPLE}_contigs-db.db"
#annotate gene clusters using COG db
anvi-run-ncbi-cogs -c "${SAMPLE}_contigs-db.db" --num-threads 4
#annotate using KEGG db
anvi-run-kegg-kofams -c "${SAMPLE}_contigs-db.db" --num-threads 4
done 

#generate genomes file that puts your sample name in one column and the path to your sample in a second column
anvi-script-gen-genomes-file --input-dir /vortexfs1/omics/huber/selkassas/Methanogen_Comparison_Alta/genomes/ \
                             --output-file Methanogen_genomes.txt

#stores all the info about your genomes
anvi-gen-genomes-storage -e Methanogen_genomes.txt \
                         -o METHANOGEN_GENOMES.db

#generates your pangenome
anvi-pan-genome -g METHANOGEN_GENOMES.db \
                --project-name "METHANOGEN_PANGENOME" \
                --output-dir METHANOGEN_PANGENOME \
                --num-threads 4 \
                --minbit 0.5
```
Submit the job to slurm `sbatch Methanogen_comparison_pangenome.sh`

## Step 4: Display pangenome 
### Local computer: Anvi'o interactive interface 
This will launch Anvi'o's interactive interface on your default browser
```
anvi-display-pan -p /vortexfs1/omics/huber/selkassas/Methanogen_Comparison_Alta/genomes//METHANOGEN_PANGENOME/METHANOGEN_PANGENOME-PAN.db \
-g METHANOGEN_GENOMES.db
```
### HPC: SSH tunneling
#### Check out [this](https://github.com/emilieskoog/SSH-tunneling/blob/main/SSH%20tunneling%20(specific%20example%20for%20anvi%E2%80%99o).md) awesome guide from Dr. Emily Skoog
```
#Enter a Local SSH tunnel
local ~ $ ssh -L 8090:localhost:8090 selkassas@poseidon-l2.whoi.edu
#enter your password

#activate your conda environment
conda activate anvio-8

anvi-display-pan -p /vortexfs1/omics/huber/selkassas/Methanogen_Comparison_Alta/genomes/METHANOGEN_PANGENOME/METHANOGEN_PANGENOME-PAN.db \
-g METHANOGEN_GENOMES.db \
--server-only -P 8090

Go to a webbrowser and type http://localhost:8090 and an interactive window will pop up
```


