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

BWA Index para o cromossomo 9
```bash
gunzip chr9.fa.gz
```
```bash
