---
title: Ejercicio_03_Binning
date: '2023-10-25'
type: book
weight: 20
---

## Binning 
Una vez que hemos escogido que ensamble usar, podemos proceder con el Binning 
Utilizaremos varios programas para hacer la reconstrucción de los genomas y haremos una comparación de estos.

**NOTA**: Cada programa tiene una ayuda y un manual de usuario, es **importante** revisarlo y conocer cada parámetro que se ejecute. En terminal se puede consultar el manual con el comando `man` y también se puede consultar la ayuda con `-h` o `--help`, por ejemplo `fastqc -h`.

La presente práctica sólo es una representación del flujo de trabajo, sin embargo, no sustituye los manuales de cada programa y el flujo puede variar dependiendo del tipo de datos y pregunta de investigación.
# Preparación del Entorno

Antes de comenzar, asegúrate de que tienes todas las herramientas y ambientes instalados adecuadamente. Si necesitas ayuda con la instalación de programas, consulta la documentación respectiva.
```
# Eliminación de archivos innecesarios
rm results/04.depth/fermentation.megahit.sam
```
 *Conversión y ordenamiento del archivo de mapeo*
 Ahora si, para poder agrupar en bins el coensamble, los binneadores requieren de un archivo con las coverturas de cada contig. Afortunadamente ya habíamos mapeado las lecturas, asi que ya tenemos un archivo de mapeo, nos falta convertirlo a formato bam y ordenarlo. Para ello ejecutamos **samtools** (https://www.htslib.org/doc/samtools.html)
 (module load samtools/1.11/gcc/9.3.0-3fme)

 ```
samtools view -bShu results/04.depth/fermentation.metaspades.sam | samtools sort -@ 80 -o results/04.depth/fermentation_metaspades_sorted.bam
samtools index results/04.depth/fermentation_metaspades_sorted.bam
rm results/04.depth/fermentation.metaspades.sam
```
Y borramos el .samn para poder ahorrar espacio
```
rm results/04.depth/fermentation.metaspades.sam
```
Ahora vamos a obtener la información en el formato que necesitan los programas que harán el binning. Para ello usamos jgi_summarize_bam_contig_depths .

```
# Obtención de información de cobertura
conda activate metabat
jgi_summarize_bam_contig_depths --outputDepth results/04.depth/fermentation_metaspades_depth.txt results/04.depth/fermentation_metaspades_sorted.bam 
```
# BINNING con METABAT
# [MetaBat](https://bitbucket.org/berkeleylab/metabat/src/master/)
(conda activate megabat2)
Crea una carpeta para metabat. 

```
mkdir ../06.Metabat
```

Para metabat sólo necesitamos dos archivos principales:

- El ensamble
- El archivo de profundidad
 
```
metabat2 -i $HOME/04.Ensamble/SRR10997048/SRR10997048_megahit.contigs.fa -a ../05.Mapeo/SRR10997048-depth.txt -o ../06.Metabat/bins -t 5 --minCVSum 0 --saveCls -d -v --minCV 0.1 -m 1500
#Y desactivamos CONDA 
conda deactivate
```
# Maxbin
Ahora ejecutemos *Maxbin*(https://sourceforge.net/p/maxbin/code/ci/master/tree/)
*Activamos el ambiente*
```
conda activate maxbin
#Creamos el directorio de estos resultados
mkdir -p results/07.maxbin/
```
Ahora si, corremos Maxbin
```
run_MaxBin.pl -thread 48 -min_contig_length 1500 -contig results/03.metaspades/contigsftr.fasta -out results/07.maxbin/maxbin  -abund results/04.depth/fermentation_metaspades_depth.txt
#Y desactivamos el ambiente
conda deactivate
```
Ya casi....

# Vamb 
(https://github.com/RasmussenLab/vamb)
Corramos Vamb
```
/botete/../.local/bin/vamb
vamb --fasta results/03.metaspades/contigsftr.fasta --jgi results/04.depth/fermentation_metaspades_depth.txt --minfasta 500000 --outdir results/08.vamb
```
## Refinamiento
# DAS_Tool
(https://github.com/cmks/DAS_Tool)
*Tenemos los resultados de tres binneadores*, muchos de estos bins estarán repetidos, ejecutaremos DAS_Tool para desreplicar estos bins, el flujo para DAS_Tool es el siguiente:

Primero activamos el ambiente.
```
conda activate das_tool
```
Creamos un directorio de trabajo para los resultados
```
mkdir -p results/09.dastool
```
Y creamos archivos tabulares que son legibles para DAS_Tool
```
#Para metabat
Fasta_to_Contig2Bin.sh -i results/06.metabat/ -e fa > results/09.dastool/fermentation_metabat.dastool.tsv
#Para maxbin
Fasta_to_Contig2Bin.sh -i results/07.maxbin/ -e fasta > results/09.dastool/fermentation_maxbin.dastool.tsv
#Para vamb
Fasta_to_Contig2Bin.sh -i results/08.vamb/bins/ -e fna > results/09.dastool/fermentation_vamb.dastool.tsv
```
Ahora si, ejecutemos DAS_Tool:
```
DAS_Tool -i results/09.dastool/fermentation_maxbin.dastool.tsv,results/09.dastool/fermentation_metabat.dastool.tsv,results/09.dastool/fermentation_vamb.dastool.tsv -l maxbin,metabat,vamb -c results/03.metaspades/contigsftr.fasta -o results/09.dastool/fermentation_bins -t 12 --write_bins
```
# CheckM
Ya desreplicamos los bins que obtuvimos, pero nos falta evaluar la calidad de estos, para ello ejecutaremos **CheckM**(https://ecogenomics.github.io/CheckM/)
(conda activate checkm)

```
checkm lineage_wf results/09.dastool/fermentation_bins_DASTool_bins/ results/10.checkm/ -x fa -t 40  -f results/10.checkm/checkm_dastool_bins.txt
```

Recordemos que hay más herramientas para desreplicar como *dRep*(https://github.com/MrOlm/drep) y también para refinar los bins que obtengamos, como *RefineM*(https://github.com/donovan-h-parks/RefineM). Si te es posible, detente a observar tus datos, analiza los resultados, prueba lo que puedas y toma las mejores decisiones.

# Conclusión
**¡Felicidades!** 
Has aprendido a realizar binning utilizando diversas herramientas en un flujo de trabajo coherente. Asegúrate de explorar las salidas de los binners y las evaluaciones para obtener resultados precisos.

*En este curso, hemos cubierto los siguientes puntos clave:*

 1. Eliminación de archivos innecesarios.
 2. Conversión y ordenamiento de archivos de mapeo.
 3. Obtención de información de cobertura.
 4. Binning con Metabat2.
 5. Binning con MaxBin.
 6. Refinamiento con DASTool.
 7. Evaluación con CheckM.
 
 
###Parte complementaria y en duda 
 YAXCHE esta parte esta en el curso de la DRA. pero tendriamos que ver como encaja con lo anterior hecho, simplemente voy a copiar y pegar el flujo de trabajo que tuvo, aunque aclaro que aparentemente cometio errores a proposito, para despues corregirlos que DIANA NO COMETIO, pero por fa checalo, igual puede ser importante ya que esta ultima parte, se conecta con la parte 4 de taxonomia y 5 de funcionalidad 
 
**AQUI EMPIEZA EL DESMADRE**

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
