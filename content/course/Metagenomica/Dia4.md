---
title: Ejercicio_03_MAGs
date: '2021-01-01'
type: book
weight: 20
---

## Binning

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



