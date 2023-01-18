---
title: Ejercicio_03_Binning
date: '2021-01-01'
type: book
weight: 20
---

## Binning

Utilizaremos varios programas para hacer la reconstrucción de los genomas y haremos una comparación de estos.

**NOTA**: Cada programa tiene una ayuda y un manual de usuario, es **importante** revisarlo y conocer cada parámetro que se ejecute. En terminal se puede consultar el manual con el comando `man` y también se puede consultar la ayuda con `-h` o `--help`, por ejemplo `fastqc -h`.

La presente práctica sólo es una representación del flujo de trabajo, sin embargo, no sustituye los manuales de cada programa y el flujo puede variar dependiendo del tipo de datos y pregunta de investigación.

# [MetaBat](https://bitbucket.org/berkeleylab/metabat/src/master/)

Crea una carpeta para metabat. 

```
mkdir $HOME/06.Metabat
```

Para metabat sólo necesitamos dos archivos principales:

- El ensamble
- El archivo de profundidad
 
```
metabat -i $HOME/04.Ensamble/SRR10997048/SRR10997048_megahit.contigs.fa -a $HOME/05.Mapeo/SRR10997048-depth.txt -o $HOME/06.Metabat/bins -t 5 --minCVSum 0 --saveCls -d -v --minCV 0.1 -m 1500
```

# [MaxBin](https://sourceforge.net/p/maxbin/code/ci/master/tree/)

Crea tu espacio de trabajo.

```
mkdir $HOME/07.MaxBin
```

Okay, ahora activa tu ambiente.

```
conda activate maxbin
```

Explora las opciones y ahora sí, a calcular bins. 

```
run_MaxBin.pl -contig $HOME/04.Ensamble/SRR10997048/SRR10997048_megahit.contigs.fa -out  $HOME/07.MaxBin/bin -abund $HOME/05.Mapeo/SRR10997048-depth.txt -max_iteration 2
```
