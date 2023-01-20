---
title: Ejercicio_05_Metabolismo
date: '2021-01-01'
type: book
weight: 20
---
# Inferencia metabolica

Primero vamos a crear una carpeta llamada Genoma y otra llamada Proteoma.

```
mkdir $HOME/10.Bins/Genoma
mkdir $HOME/10.Bins/Proteoma
```

Vamos a mover todos los *.fa a Genoma.

```
mv $HOME/10.Bins/*.fa  $HOME/10.Bins/Genoma
```

Vamos a utilizar [prodigal](https://github.com/hyattpd/Prodigal) para predecir las proteínas:

```
cd $HOME/10.Bins/Genoma/
for i in $(ls *.fa); do prodigal -i $i -o $HOME/10.Bins/Proteoma/$i.txt -a $HOME/10.Bins/Proteoma/$i.faa ; done
cd
```

Veamos un poco la salida

```
grep ">" $HOME/10.Bins/Proteoma/*.faa
```

# KEEG

Okay, ahora vamos a utilizar [kofam_scan](https://github.com/takaram/kofam_scan) para anotar las proteínas.

Vamos a dividirnos en equipos para hacer la anotación de los bins!


```
mkdir $HOME/12.KOFAM
```

```
cd $HOME/10.Bins/Proteoma/
for i in *.faa ; do  /opt/kofam_scan-1.3.0/exec_annotation -o  $HOME/12.KOFAM/$i.txt  $i  --report-unannotated  --cpu 4 -p /databases/kofamscan_dbs/profiles -k /databases/kofamscan_dbs/ko_list; done
cd 
```

## Explorando el metabolismo con rbims.

Vamos a hacer una exploración rápida del metabolismo con [rbims](https://mirnavazquez.github.io/RbiMs/index.html).

Okay iniciamos con la librería de Rbims

```
install.packages("devtools")
library(devtools)
install_github("mirnavazquez/RbiMs")
library(rbims)
library(tidyverse)
```

Ahora, vamos a leer los resultados de KEEG y mapearlos con el resto de la base de datos de KEEG

```
pulque_mapp<-read_ko("08.Kofamscan/02.KO_results/") %>%
    mapping_ko()
```

Okay, vamos a enfocarnos en los metabolismos encargados de la obtención de energía. 

```
Overview<-c("Central Metabolism", "Carbon Fixation", 
            "Nitrogen Metabolism", "Sulfur Metabolism", "Fermentation", 
            "Methane Metabolism")
Energy_metabolisms_pulque<-pulque_mapp %>%
  drop_na(Cycle) %>%
  get_subset_pathway(rbims_pathway, Overview) 
```

Vamos a visualizar los datos.

```
plot_bubble(tibble_ko = Energy_metabolisms_pulque,
            x_axis = Bin_name, 
            y_axis = Pathway_cycle,
            analysis="KEGG",
            calc="Percentage",
            range_size = c(1,10),
            y_labs=FALSE,
            x_labs=FALSE)  
```

Okay, incorporemos metadatos, por ejemplo la taxonomía. 

```
Metadatos<-read_delim("11.GTDBTK/Metadatos.txt", delim="\t")
```

Hagamos un plot

```
plot_bubble(tibble_ko = Energy_metabolisms_pulque,
            x_axis = Bin_name, 
            y_axis = Pathway_cycle,
            analysis="KEGG",
            data_experiment = Metadatos,
            calc="Percentage",
            color_character = Class,
            range_size = c(1,10),
            y_labs=FALSE,
            x_labs=FALSE) 
```

Ahora, vamos a explorar una sola vía

```
Secretion_system_pulque<-pulque_mapp %>%
  drop_na(Cycle) %>%
  get_subset_pathway(Cycle, "Secretion system")
```

Y hagamos un heatmap

```
plot_heatmap(tibble_ko=Secretion_system_pulque, 
             y_axis=Genes,
             analysis = "KEGG",
             calc="Binary")
```

Ahora agreguemos metadatos

```
plot_heatmap(tibble_ko=Secretion_system_pulque, 
             y_axis=Genes,
             data_experiment = Metadatos,
             order_x = Phylum,
             analysis = "KEGG",
             calc="Binary")
```

```
plot_heatmap(tibble_ko=Secretion_system_pulque, 
             y_axis=Genes,
             data_experiment = Metadatos,
             order_y = Pathway_cycle,
             order_x = Phylum,
             analysis = "KEGG",
             calc="Binary")
```


