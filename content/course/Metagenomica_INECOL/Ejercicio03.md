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

Primero me puedes decir ¿Cuántas lineas tiene tu archivo?

Okay... ahora vamos a remover esas lineas feas. 

```{bash, eval=FALSE}
sed -e '1,3d' $HOME/09.CheckM/CheckM-DAS_Tool_bins.txt | sed -e '6d' > $HOME/09.CheckM/CheckM-DAS_Tool_bins_mod.txt
```


```{r, eval=FALSE}
library(tidyverse)
# CheckM -------------------------------------------------------------------####
checkm<-read.table("CheckM-DAS_Tool_bins_mod.txt", sep = "", header = F, na.strings ="", stringsAsFactors= F)
# Extracting good quality bins Megahit ------------------------------------####
colnames(checkm)<-c("Bin_Id", "Marker", "lineage", "Number_of_genomes", 
                         "Number_of_markers", "Number_of_marker_sets", 
                         "0", "1", "2", "3", "4", "5", "Completeness", 
                         "Contamination", "Strain_heterogeneity")  

good_bins<-checkm %>%
  select(Bin_Id, Marker, Completeness, Contamination) %>%
  filter(Completeness >= 50.00 & Contamination <= 10.00) 
```

Muy bien, vamos a extraer esos bins.

```{r, eval=FALSE}
bins<-good_bins$Bin_Id

write.table(bins, "lista_medium_bins", quote = F, row.names = F, col.names = F)
```

### Mis Bins

```{bash, eval=FALSE}
 sed 's#bin#cp $HOME/08.DasTool/SRR10997048_bins_DASTool_bins/bin#g' lista_medium_bins | sed 's#$#.fa $HOME/10.Bins#g' > copy_bins.sh
```

```{bash, eval=F}
mkdir  $HOME/10.Bins
```

```{bash, eval=F}
cp  copy_bins.sh $HOME/10.Bins
cd
```

```{bash, eval=F}
bash $HOME/10.Bins/copy_bins.sh
```

Ahora un ejercicio.

```{bash, eval=FALSE}
grep ">" $HOME/10.Bins/*.fa
```

¿Cuál es el problema?

```{r, eval=FALSE}
change_bin_name<-function(ruta, ambiente){
ruta_original<-getwd()
setwd(ruta)
filez <- list.files()
newname<-paste0(ambiente, "_", filez)
file.rename(from=filez, to=newname)
filez <- list.files()
file.rename(from=filez, to=sub(pattern="\\.", replacement="_", filez))
setwd(ruta_original)
}
```

```{r, eval=FALSE}
change_bin_name("/botete/mvazquez/10.Bins", "pulque")
```


```{r, eval=FALSE}
library(phylotools)
library(tidyverse)

add_names_to_seqs <- function(nombre_del_archivo){
  filenames <- unlist(strsplit(nombre_del_archivo, "/"))
  filenames <- filenames[[grep("fa", filenames)]]
  divide <- unlist(strsplit(filenames, "\\."))
  bin_name <- divide[1]
  termination <- divide[2]
  old_name <- get.fasta.name(nombre_del_archivo)
  new_name <- paste0( bin_name, "-scaffold-", old_name) 
  ref2 <- data.frame(old_name, new_name)
  out_file <- paste0(bin_name, "_renamed", ".", termination)
  rename.fasta(infile = nombre_del_archivo, ref_table = ref2, outfile = out_file)
}

files <- list.files("10.Bins/")
files <- paste0("/botete/mvazquez/10.Bins/", files)

map(files, add_names_to_seqs)
```

Veamos si funcionó

```{bash, eval=FALSE}
grep ">" *.fa
```

Muy bien, pongamos eso en una nueva carpeta y esperemos lo mejor jaja. No es cierto, sí movámoslo a otra carpeta, pero quitemos el renamed.

```{r, eval=FALSE}
rename 's/_renamed//g' *
```

¿Creen que puedan optimizar esos scripts? ¡Discute en tu equipo si tienes una mejor idea!








