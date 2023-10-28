# EP2-somatico-G
## Trabalho EP2 Somático
* Instalando sratoolskit e baixando arquivo WP312
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
``bash
time parallel-fastq-dump --sra-id SRR8856724 \
--threads 4\
--outdir./ \
--split-files \
--gzip
``
