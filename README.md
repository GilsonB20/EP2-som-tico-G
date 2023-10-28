# EP2-somatico-G
## Trabalho EP2 Somático
Instalando sratoolskit e baixando arquivo WP312
  ```bash
  brew install sratoolkit
  ```
Instalando parallel-fastq-dump
```bash
pip install parallel-fastq-dump
```
```bash
echo "Aexyo" | vdb-config -i
```
```bash
Baixando os fastq

time parallel-fastq-dump --sra-id SRR8856724 \
--threads 10\
--outdir./ \
--split-files \
--gzip
```

Instalando o bwa para mapeamento dos arquivos
```bash
brew install bwa 
```
Realizar download do cromossomo 9 UCSC hg19:https://hgdownload.soe.ucsc.edu/goldenPath/hg19/chromosomes/ chr9.fa.gz: https://hgdownload.soe.ucsc.edu/goldenPath/hg19/chromosomes/chr9.fa.gz
```bash
wget -c https://hgdownload.soe.ucsc.edu/goldenPath/hg19/chromosomes/chr9.fa.gz
```

Descompactando
```bash
gunzip chr9.fa.gz
```
Criando index para o chr9
```bash
bwa index chr9.fa
```
intalando samtools
````bash
brew install samtools 
````
````bash
samtools faidx chr9.fa
````
combinando samtools com bwa (view e sort)
````bash
NOME=WP312; Biblioteca=Nextera; Plataforma=illumina;
bwa mem -t 10 -M -R "@RG\tID:$NOME\tSM:$NOME\tLB:$Biblioteca\tPL:$Plataforma" chr9.fa SRR8856724_1.fastq.gz SRR8856724_2.fastq.gz | samtools view -F4 -Sbu -@2 - | samtools sort -m4G -@2 -o WP312_sorted.bam
````

Removendo duplicatas de pcr com rmdup:
````bash
samtools rmdup WP312_sorted.bam WP312_sorted_rmdup.bam
````

Instalando bedtools
````bash
brew install bedtools
````

Gerando arquivo .bed apartir do .bam
````bash
bedtools bamtobed -i WP312_sorted_rmdup.bam > WP312_sorted_rmdup.bed
````
````bash
bedtools merge -i WP312_sorted_rmdup.bed > WP312_sorted_rmdup_merged.bed
````
````bash
bedtools sort -i WP312_sorted_rmdup_merged.bed > WP312_sorted_rmdup_merged_sorted.bed
````

Calculando a cobertuda
````bash
bedtools coverage -a WP312_sorted_rmdup_merged_sorted.bed \
-b WP312_sorted_rmdup_F4.bam -mean \
> WP312_coverageBed.bed
````

Filtro por total de reads >= 20
````bash
cat WP312_coverageBed.bed | \
awk -F "\t" '{if($4>=20){print}}' \
> WP312_coverageBed20x.bed
````


#trabalhando com fasta e VCFs
A amostras WP312 serão executadas com a versão do genoma humano HG19. 

baixando os arquivos de Referência: Panel of Normal (PoN)
````bash
wget -c https://storage.googleapis.com/gatk-best-practices/somatic-b37/Mutect2-WGS-panel-b37.vcf
````
````bash
wget -c https://storage.googleapis.com/gatk-best-practices/somatic-b37/Mutect2-WGS-panel-b37.vcf.idx
````

baixando os arquivos de Referência: Gnomad AF
````bash
wget -c  https://storage.googleapis.com/gatk-best-practices/somatic-b37/af-only-gnomad.raw.sites.vcf
````
````bash
wget -c  https://storage.googleapis.com/gatk-best-practices/somatic-b37/af-only-gnomad.raw.sites.vcf.idx
````

instalação do GATK 
````bash
wget -c https://github.com/broadinstitute/gatk/releases/download/4.2.2.0/gatk-4.2.2.0.zip
````
descompactação do GATK
````bash
unzip gatk-4.2.2.0.zip
````
Gerando dict
````bash
./gatk-4.2.2.0/gatk CreateSequenceDictionary -R chr9.fa -O chr9.dict
````
Gerando interval_list
````bash
./gatk-4.2.2.0/gatk ScatterIntervalsByNs -R chr9.fa -O chr9.interval_list -OT ACGT
````

Converter Bed para Interval_list
````bash
./gatk-4.2.2.0/gatk BedToIntervalList -I WP312_coverageBed20x.bed \
-O WP312_coverageBed20x.interval_list -SD chr9.dict
````

#Trabalhando os VCFs
````bash
grep "\#" af-only-gnomad.raw.sites.vcf > af-only-gnomad.raw.sites.chr.vcf
grep  "^9" af-only-gnomad.raw.sites.vcf |  awk '{print("chr"$0)}' >> af-only-gnomad.raw.sites.chr.vcf
````
indexing
````bash
bgzip af-only-gnomad.raw.sites.chr.vcf
tabix -p vcf af-only-gnomad.raw.sites.chr.vcf.gz
````

````bash
grep "\#" Mutect2-WGS-panel-b37.vcf > Mutect2-WGS-panel-b37.chr.vcf 
grep  "^9" Mutect2-WGS-panel-b37.vcf |  awk '{print("chr"$0)}' >> Mutect2-WGS-panel-b37.chr.vcf 
````

````bash
bgzip Mutect2-WGS-panel-b37.chr.vcf 
tabix -p vcf Mutect2-WGS-panel-b37.chr.vcf.gz
````

#GATK Mutect Call
````bash
./gatk-4.2.2.0/gatk GetPileupSummaries \
	-I WP312_sorted_rmdup.bam  \
	-V af-only-gnomad.raw.sites.chr.vcf.gz  \
	-L WP312_coverageBed20x.interval_list \
	-O WP312.table
````

````bash
./gatk-4.2.2.0/gatk CalculateContamination \
-I WP312.table \
-O WP312.contamination.table
````


#falta rodar
pegando arquivos no drive para comparação
````bash
git clone https://github.com/circulosmeos/gdown.pl.git
./gdown.pl/gdown.pl  https://drive.google.com/file/d/1pTMpZ2eIboPHpiLf22gFIQbXU2Ow26_E/view?usp=drive_link WP312_sorted_rmdup_F4.bam
./gdown.pl/gdown.pl  https://drive.google.com/file/d/10utrBVW-cyoFPt5g95z1gQYQYTfXM4S7/view?usp=drive_link WP312_sorted_rmdup_F4.bam.bai
````

#falta rodar
executando Mutect2
````bash
./gatk-4.2.2.0/gatk Mutect2 \
  -R chr9.fa \
  -I WP312_sorted_rmdup.bam \
  --germline-resource af-only-gnomad.raw.sites.chr.vcf.gz  \
  --panel-of-normals Mutect2-WGS-panel-b37.chr.vcf.gz \
  --disable-sequence-dictionary-validation \
  -L WP312_coverageBed20x.interval_list \
  -O WP312.somatic.pon.vcf.gz
````



* ./gatk-4.2.2.0/gatk FilterMutectCalls \
-R chr9.fa \
-V WP312.somatic.pon.vcf.gz \
--contamination-table WP312.contamination.table \
-O WP312.filtered.pon.vcf.gz

* vcf-compare WP312.filtered.pon.vcf.gz ../WP312.filtered.chr.vcf.gz 
