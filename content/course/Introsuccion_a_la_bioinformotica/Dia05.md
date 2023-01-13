---
title: Ejercicio día 05
date: '2021-01-01'
type: book
weight: 20
---

## Predicción de genes en un genoma bacteriano

Okay, primero creamos una carpeta que se llame **Ejercicio_05**.

```
mkdir ~/Ejercicio_05
```

Y copia el genoma que vamos a anotar


```
cp /databases/material_curso_2023/final.assembly.fasta Ejercicio_05/
``` 

Muy bien 


Ahora vamos a correr [**Prodigal**](https://bmcbioinformatics.biomedcentral.com/articles/10.1186/1471-2105-11-119).

Veamos el programa

```
prodigal
``` 

Corramos un ejemplo

```
prodigal -i final.assembly.fasta.fasta -o test.txt -a test.faa
``` 

Ahora vamos a anotarlo con [**KEGG**](https://www.genome.jp/kegg/)

```
/opt/kofam_scan-1.3.0/exec_annotation Ejercicio_05/test.faa -p /databases/kofamscan_dbs/profiles -k /databases/kofamscan_dbs/ko_list --cpu 4 -f mapper > Ejercicio_05/salida
``` 

Descarga el archivo **salida**.


Y existen cosas super "pro" que hacen todo por ti como [**PROKKA**](https://github.com/tseemann/prokka), chequemos la ayuda.

```
/opt/prokka-1.14.5/bin/prokka
``` 

Vamos a correrlo :) 

¿Qué parámetros le ponemos?
