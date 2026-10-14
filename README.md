# Brca-pangenome
fastqc Raw/*.gz -o FastQC -t 8
multiqc FastQC -o MultiQC
./run_fastp.sh Raw Fastp
