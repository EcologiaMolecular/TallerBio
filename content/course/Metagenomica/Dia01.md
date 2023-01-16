---
title: Filtrado de lecturas
date: '2021-01-01'
type: book
weight: 20
---

## Filtrado de lecturas

La primer cosa que tenemos que hacer es crear una carpeta donde van a vivir los datos que vamos a trabajar durante el curso.

```
mkdir $HOME/00.RawData
```

Ahora creemos una liga simbólica al lugar a donde viven los datos.

```
ln -s /botete/databases/material_curso_2023/00.RawReads/* $HOME/00.RawData
```

¡Muy bien!

Una primera cosa que nos gustaría hacer sería ver la calidad de las secuencias a utilizar, para eso vamos a analizar la calidad de las lecturas con  [**fastqc**](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/).

Entonces vamos a crear un nuevo directorio donde van a vivir los resultados de fastqc.


```
mkdir $HOME/01.Fastqc
```

Y ahora vamos a correr el programa.

```
fastqc -t 4 $HOME/00.RawData/*.fastq -o $HOME/01.Fastqc
```

Okay ahora vamos a pasar los resultados a nuestras computadoras para ver los reportes de calidad. 

Abramos una terminal nueva y vamos a nuestro directorio de escritorio dentro de nuestras computadoras:

```
cd $HOME/Desktop/
```

Y ahora copiemos los archivos

```
scp -P Numero_de_puerto usuario@IP:/ruta/a/mi/archivo .
```

¡Muy bien!

Ahora vamos a filtrar las secuencias usando Trimmomatic

Vamos a filtrar las lecturas por calidad utilizando [**Trimmomatic**](http://www.usadellab.org/cms/?page=trimmomatic).

Vamos a crear un directorio de salida.

```
mkdir $HOME/02.Trimmomatic
```

Y ahora vamos a correr el programa.

```
java -jar /opt/Trimmomatic-0.39/trimmomatic-0.39.jar PE -threads 4 -phred33 -trimlog $HOME/02.Trimmomatic/triminfo.txt $HOME/00.RawData/SRR10997048_1.fastq $HOME/00.RawData/SRR10997048_2.fastq  $HOME/02.Trimmomatic/SRR10997048_R1_trimm.fastq  $HOME/02.Trimmomatic/SRR10997048_R1_unpair.fastq $HOME/02.Trimmomatic/SRR10997048_R2_trimm.fastq $HOME/02.Trimmomatic/SRR10997048_R2_unpair.fastq ILLUMINACLIP:/opt/Trimmomatic-0.39/adapters/TruSeq2-PE.fa:2:30:10:8:True LEADING:5 TRAILING:5 SLIDINGWINDOW:5:15 MINLEN:50
```

Ahora en equipos definan los parámetros para el resto de las secuencias hagan el filtrado y veamos finalmente los archivos de salida.

Ahora vamos a crear un archivo intercalado con un script de [**bbtools**](https://jgi.doe.gov/data-and-tools/software-tools/bbtools/).

```
mkdir $HOME/03.Interleave
```

```
/opt/anaconda3/bin/reformat.sh threads=10 in1=02.Trimmomatic/SRR10997048_R1_trimm.fastq in2=02.Trimmomatic/SRR10997048_R2_trimm.fastq out=03.Interleave/SRR10997048_HQ.fastq
```
