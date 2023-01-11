---
title: Ejercicio día 03
date: '2021-01-01'
type: book
weight: 20
---

## Ejercicio de filtrado de lecturas


- Vamos a filtrar las lecturas por calidad utilizando [**Trimmomatic**](http://www.usadellab.org/cms/?page=trimmomatic).


```
java -jar /opt/Trimmomatic-0.39/trimmomatic-0.39.jar PE -threads 10 -phred33 -trimlog triminfo.txt PB1_S43_L001_R1_001.fastq PB1_S43_L001_R2_001.fastq  PB1_R1_trimm.fastq  PB1_R1_unpair.fastq PB1_R2_trimm.fastq PB1_R2_unpair.fastq ILLUMINACLIP:/opt/Trimmomatic-0.39/adapters/TruSeq2-PE.fa:2:30:10:8:True LEADING:5 TRAILING:5 SLIDINGWINDOW:5:15 MINLEN:50
```


- Repite el ejercicio con los resultados de [**PEAR**](https://cme.h-its.org/exelixis/web/software/pear/)


En equipos corran fastqc y definan que cambios vieron en la corrida.


## Ejemplo de la vida real


Amplicones de columna de agua en distintos puntos del Golfo e México, las muestras se llaman: 


  - WC1.fasta  
  - WC2.fasta  
  - WC3.fasta  
  - WC4.fasta  
  - WC5.fasta
  - WC6.fasta  
  - WC7.fasta  
  - WC8.fasta  
  - WC9.fasta


- Copia los archivos que estan en el directorio el material del curso.


- ¡Haz un **head** de los archivos! 


```
head *.fasta
```


-¿Qué hace la opcion -n?


```
head --h
```


¡Muy bien!


- Ahora con el script **cambiar_nombre.pl** vamos a agregar algunos identificadores. 


```
perl cambiar_nombre.pl sample1 WC1.fasta
```


- ¿Qué hizo el script?


## Ejercicio a casa 1


- ¿Como utilizarias este script de forma automatizada?


## Explorando las expresiones regulares


- Seguimos, ahora vamos a concatenar todo.


```
cat *.numbered.fas > ColumnaDeAgua.fas
```


¡Perfecto! 


- Ahora vamos a remover del nombre la parte fea.


```
perl -i.bak -pe "s/\ .*//g" ColumnaDeAgua.fas
```


- Discute con tu equipo que esta haciendo ese comando


  - hint: Haz un **head** ¿Qué paso? 


## Agrupamiento


- Vamos a agrupar las secuencias acorde a un porcentaje de identida. En este caso es del 97%.


```
cd-hit-est -i ColumnaDeAgua.fas -c 0.97 -aL 0.97 -o ColumnaDeAgua.fasout -T 2
```


- Ahora hagamos un grep del archivo **fasout** y **fas**, ¿Qué observas?


```
grep -c ">" *fasout
```


- ¿Qué hay en el archivo **clstr**?


- Ahora vamos a modificar el archivo para utilizarlo dentro de [**QIIME**](https://qiime2.org/) como mapa e OTUs.


```
perl -pne 's/\t//g;s/^.*,//g;s/\.\.\..*$//g;s/\n/\t/g;s/\>Cluster\ /\n/g;s/\>//g; eof && do{chomp; print "$_ \n"; exit}' ColumnaDeAgua.fasout.clstr > ColumnaDeAgua.otu
```


- ¿Qué puedes decir de este archivo?


```
sed -i '1d' ColumnaDeAgua.otu
```


- ¿Cuantos OTUs tienes?
