---
title: Ejercicio_01_Filtrado de lecturas
date: '2023-10-24'
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
ln -s /databases/material_curso_2023/00.RawReads/* $HOME/00.RawData
```
#Contexto de los datos con los que se trabajara
*Datos de la fermentación del pulque*
Sinopsis: En este trabajo se secuenciaron librerias de Illumina MiSeq (2 X 149 pb) de muestras a diferentes tiempos de fermentación: Aguamiel, 0, 3, 6 y 12 horas. En general observaron que la abundancia de los géneros cambió durante la fermentación y se asoció con una disminución de sacarosa, aumento de etanol y ácido láctico (Medidos por HPLC). Predijeron, entre otros, la biosíntesis de folato y vitamina B. Uno de los MAG obtenidos correspondió a Saccharomyces cereviseae relacionado filogenéticamente con S. cerevisiae aislado de sake y bioetanol y otro correspondió a Zymomonas mobilis que propusieron como un nuevo linaje.

##Trabajaremos con un subset de los datos originales.##

Podemos acceder a los datos crudos a traves de: PRJNA603591 https://www.ebi.ac.uk/ena/browser/view/PRJNA603591

El articulo original de los datos es: Genomic profiling of bacterial and fungal communities and their predictive functionality during pulque fermentation by whole-genome shotgun sequencing https://www.nature.com/articles/s41598-020-71864-4

#Ya con contexto continuemos con los ejercicios del curso
*¡Muy bien ya tenemos los datos con los que vamos a trabajar!*

Una primera cosa que nos gustaría hacer sería ver la calidad de las secuencias a utilizar, para eso vamos a analizar la calidad de las lecturas con  [**fastqc**](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/).

Entonces vamos a crear un nuevo directorio donde van a vivir los resultados de fastqc.

```
mkdir $HOME/01.Fastqc
```

Y ahora vamos a correr el programa.

```
fastqc -t 4 $HOME/00.RawData/*.fastq -o $HOME/01.Fastqc
```

Ahora tenemos que activar conda y usando el entorno Qiime podremos observar las calidades de nuestras secuencias 

```
conda activate /botete/TuUsuario/.conda/envs/qiime2-2022.11
multiqc 01.Fastqc/*.zip -o 01.Fastqc/multiqc
```
En resumen, esta parte del código ejecuta multiqc en los archivos ZIP en el directorio results/01.fastqc/ y genera un informe de calidad en el directorio 

#Conoczcamos a Trimgalore
La herramienta TrimGalore nos permite eliminar lecturas de baja calidad, adaptadores, etc. Y con MultiQC, podemos ver las calidades del conjunto de lecturas. Existen otros programas para limpiar las lecturas como Trimmomatic.

```
#Creamos el directorio para Trimgalore
mkdir 02.trimgalore
#Ejecutamos trim_galore para filtrar
trim_galore --fastqc -j 45 --paired pulquet0_1_10M.fastq pulquet0_2_10M.fastq -o 02.trimgalore/pulquet0_trimgalore
```

¡Muy bien!

Ahora vamos a filtrar las secuencias usando Trimmomatic

Vamos a filtrar las lecturas por calidad utilizando [**Trimmomatic**](http://www.usadellab.org/cms/?page=trimmomatic).

Vamos a crear un directorio de salida.

```
mkdir $Home/results/02.Trimmomatic/zips
```

Y ahora vamos a correr el programa.

```
java -jar /opt/Trimmomatic-0.39/trimmomatic-0.39.jar PE -threads 4 -phred33 -trimlog $HOME/02.Trimmomatic/triminfo.txt $HOME/00.RawData/SRR10997048_1.fastq $HOME/00.RawData/SRR10997048_2.fastq  $HOME/02.Trimmomatic/SRR10997048_R1_trimm.fastq  $HOME/02.Trimmomatic/SRR10997048_R1_unpair.fastq $HOME/02.Trimmomatic/SRR10997048_R2_trimm.fastq $HOME/02.Trimmomatic/SRR10997048_R2_unpair.fastq ILLUMINACLIP:/opt/Trimmomatic-0.39/adapters/TruSeq2-PE.fa:2:30:10:8:True LEADING:5 TRAILING:5 SLIDINGWINDOW:5:15 MINLEN:50
```

*Ahora evaluemos nuestros resultados, tanto de timgalore, como de trimomatic con multiqc*
Para evaluar los resultados de calidad tras el filtrado, podemos ejecutar *multiqc*.

Creamos el directorio donde alojaremos las entradas de multiqc y movemos estos archivos al nuevo directorio.

```
mkdir -p results/02.trimgalore/zips
mv 02.trimgalore/*/*.zip results/02.trimgalore/zips/
mv 02.trimomatic/*/*.zip results/02.trimomatic/zips/
```

Ahora si ejecutamos multiqc que esta en en el ambiente de qiime2, por eso lo vamos a activar. OJO, multiqc puede estar alojado en un ambiente independiente o instalarse fuera de ambientes, en el servidor esta dentro de qiime pero no es una regla.

```
#Activamos el ambiente
conda activate /botete/TuUsuario/.conda/envs/qiime2-2022.11
#Ejecutamos multiqc
multiqc 02.trimgalore/zips/*.zip -o 02.trimgalore/zips/multiqc
#Podmeos ejecutar multiqc para trimomatic
multiqc 02.trimomatic/zips/*.zip -o 02.trimomatic/zips/multiqc 
```

Movemos nuestros datos limpios al subdirectorio data/clean

```
mv 02.trimgalore/*/*.fq data/clean/
mv 02.trimoamtic/*/*.fq data/clean/
```
Y descativamos el ambiente

```
conda deactivate
```
#Yey terminamos la primera parte 
*¡Felicidades a todos los alumnos por completar estas actividades de análisis de metagenomas!*
 Han aprendido y aplicado varias herramientas y conceptos importantes en este proceso. 
 Aquí un resumen de lo que han logrado y cómo lo han hecho:

 1. Creación de Espacio de Trabajo: Comenzaron por organizar su espacio de trabajo, creando directorios clave para los diferentes pasos del análisis. Esta organización es esencial para mantener los datos y resultados organizados.

 2. Análisis de Calidad y Filtrado de Lecturas: Utilizaron la herramienta FastQC para evaluar la calidad de las lecturas y obtener informes que les ayudaron a identificar problemas potenciales en los datos. También, ejecutaron TrimGalore y trimomatic para eliminar lecturas de baja calidad y adaptadores, mejorando la calidad de los datos.

 3. Generación de Informes de Calidad: Aplicaron MultiQC para resumir y visualizar la calidad del conjunto de lecturas, lo que les permitió una visión más completa de la calidad de los datos después del filtrado.

 4. Organización de Datos Limpios: Organizaron sus datos limpios moviéndolos al directorio correspondiente en data/clean, lo que facilita su acceso y uso en pasos posteriores del análisis.

 5. Manejo de Ambientes Conda: Utilizaron ambientes Conda para gestionar las dependencias de las herramientas utilizadas en el análisis, lo que garantiza la reproducibilidad y compatibilidad de las herramientas.

En general, han adquirido habilidades fundamentales para la manipulación de datos de secuenciación de alto rendimiento y la evaluación de su calidad, este conocimiento y experiencia les servirán en su camino hacia un análisis de metagenomas exitoso (tanto ahora como en el futuro.

**¡Sigan adelante con su trabajo y exploración en el fascinante mundo de la microbiología y la bioinformática!**
