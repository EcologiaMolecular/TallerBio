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
mkdir $HOME/08.Metabat
```

Para metabat sólo necesitamos dos archivos principales:

- El ensamble
- El archivo de profundidad
 
```
metabat -i $HOME/06.Ensamble/01.Megahit/pulquet0_megahit.contigs.fa -a $HOME/07.Mapeo/pulquet0-depth.txt -o $HOME/08.Metabat/bins -t 5 --minCVSum 0 --saveCls -d -v --minCV 0.1 -m 1500
```

# [MaxBin](https://sourceforge.net/p/maxbin/code/ci/master/tree/)

Crea tu espacio de trabajo.

```
mkdir $HOME/09.MaxBin
```

Okay, ahora activa tu ambiente.

```
conda activate maxbin
```

Explora las opciones y ahora sí, a calcular bins. 

```
run_MaxBin.pl -contig $HOME/06.Ensamble/01.Megahit/pulquet0_megahit.contigs.fa -out  $HOME/09.MaxBin/bin -abund $HOME/07.Mapeo/pulquet0-depth.txt -max_iteration 2
```

# Refinamiento

# [DASTool](https://github.com/cmks/DAS_Tool)

```
mkdir $HOME/10.DasTool
```

Preparing input files.

```
Fasta_to_Contig2Bin.sh -i $HOME/09.MaxBin -e fasta > $HOME/10.DasTool/pulquet0_maxbin.scaffolds2bin.tsv
```

```
Fasta_to_Contig2Bin.sh -i $HOME/08.Metabat/ -e fa > $HOME/10.DasTool/pulquet0_metabat.scaffolds2bin.tsv
```

Ahora si :)

```
DAS_Tool -i $HOME/10.DasTool/pulquet0_maxbin.scaffolds2bin.tsv,$HOME/10.DasTool/pulquet0_metabat.scaffolds2bin.tsv -l metabat,maxbin -c $HOME/04.Ensamble/pulquet0/pulquet0_megahit.contigs.fa -o $HOME/10.DasTool/pulquet0_bins --debug -t 4  --search_engine diamond --write_bins
```

# [CheckM](https://github.com/Ecogenomics/CheckM/wiki)

Muy bien, crea un nuevo directorio y entra en él.

```
mkdir $HOME/11.CheckM
```

Exportemos el path a donde viven las bases de datos.

```
export CHECKM_DATA_PATH=/databases
```

```
checkm  lineage_wf -t 4 -x fa 10.DasTool/pulquet0_bins_DASTool_bins/ pulquet0-log  -f 11.CheckM/CheckM-DAS_Tool_bins.txt
```

Vamos a explorar la salida de checkM





