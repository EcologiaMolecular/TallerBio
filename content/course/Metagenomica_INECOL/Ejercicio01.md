---
title: Ejercicio_01_Filtrado de lecturas
date: '2023-10-25'
type: book
weight: 20
---

## Filtrado de lecturas

Para comenzar, debemos crear una carpeta donde almacenaremos los datos que utilizaremos en el curso.

```
mkdir $HOME/00.RawData
```

Ahora, creemos un enlace simbólico hacia la ubicación de los datos.

```
ln -s /lustre/dhoaxaca/Datos_taller_27_10_23/* $HOME/00.RawData
```

Asegurémonos de que los datos se encuentren en la carpeta.

```
ls $HOME/00.RawData
```

¡Excelente!

Uno de nuestros primeros pasos será evaluar la calidad de las secuencias que vamos a utilizar. Para esto, utilizaremos [**fastqc**](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/).

Primero, creemos un directorio para almacenar los resultados de FastQC.

```
mkdir $HOME/01.Fastqc
```

Luego, carguemos el módulo necesario.

```
module load fastqc/0.11.7/gcc
```

Ahora, ejecutemos el programa.

```
srun --mem 16G -n 1 -p q2 fastqc -t 4 $HOME/00.RawData/*.fq -o $HOME/01.Fastqc
```

Luego, copiemos los resultados a nuestras computadoras para examinar los informes de calidad.

Abre una nueva terminal y navega hasta tu directorio de escritorio en tu computadora local.

```
cd $HOME/Desktop/
```

Ahora, copiemos los archivos desde el servidor.

```
scp -P Numero_de_puerto usuario@IP:/ruta/a/mi/archivo .
```

¡Perfecto!

A continuación, procederemos a filtrar las secuencias utilizando [**Trimmomatic**](http://www.usadellab.org/cms/?page=trimmomatic).

Utilizaremos Trimmomatic para filtrar las lecturas basándonos en la calidad.

Primero, creemos un directorio de salida.

```
mkdir $HOME/02.Trimmomatic
```

Luego, carguemos el módulo correspondiente.

```
module load trimmomatic/0.38/gcc/8.3.1-jwk4
```

Finalmente, ejecutemos el programa.

```
srun --mem 16G -n 1 -p q2 trimmomatic PE -threads 1 -phred33 -trimlog $HOME/02.Trimmomatic/triminfo.txt $HOME/00.RawData/pulquet0_1_10M_val_1.fq $HOME/00.RawData/pulquet0_2_10M_val_2.fq $HOME/02.Trimmomatic/pulquet0_2_10M_val_R1_trimm.fastq $HOME/02.Trimmomatic/pulquet0_2_10M_val_R1_unpair.fastq $HOME/02.Trimmomatic/pulquet0_2_10M_val_R2_trimm.fastq $HOME/02.Trimmomatic/pulquet0_2_10M_val_R2_unpair.fastq LEADING:5 TRAILING:5 SLIDINGWINDOW:5:15 MINLEN:50
```

Si deseas, también podemos probar  [TrimGalore](https://github.com/FelixKrueger/TrimGalore/blob/master/Docs/Trim_Galore_User_Guide.md), que permite eliminar lecturas de baja calidad y adaptadores.

Primero, creemos un directorio para los resultados de TrimGalore.

```
mkdir $HOME/03.Trimgalore
```

A continuación, carguemos el módulo necesario.

```
module load trimgalore/0.6.4/gcc/9.3.0-4hih
```

Finalmente, ejecutemos TrimGalore.

```
srun --mem 16G -n 1 -p q1 trim_galore --fastqc -j 1 --paired 00.RawData/pulquet0_1_10M_val_1.fq 00.RawData/pulquet0_2_10M_val_2.fq -o 03.Trimgalore/pulquet0_trimgalore
```

Ahora con [**MultiQC**] podemos ver las calidades del conjunto de lecturas.

```
module load py-multiqc/1.7/gcc/9.3.0-yslf
```

Ahora corramos el programa.

```
srun --mem 16G -n 1 -p q1 multiqc 03.Trimgalore/*.zip -o 04.Multiqc
```

Recuerda adaptar los nombres de archivo y directorios según tu estructura de carpetas y archivos.

