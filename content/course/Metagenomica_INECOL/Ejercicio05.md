---
title: Ejercicio_05-Inferencia_Metabólica
date: '2023-10-26'
type: book
weight: 20
---

# Inferencia Metabólica

En esta sección, exploraremos la inferencia metabólica de los bins de nuestro análisis.

## Preparación de Carpetas

Comenzaremos creando dos carpetas llamadas "Genoma" y "Proteoma" para organizar nuestros archivos:

```bash
mkdir $HOME/10.Bins/Genoma
mkdir $HOME/10.Bins/Proteoma
```

Luego, moveremos todos los archivos con extensión ".fa" a la carpeta "Genoma":

```bash
mv $HOME/10.Bins/*.fa $HOME/10.Bins/Genoma
```

## Predicción de Proteínas

Utilizaremos [Prodigal](https://github.com/hyattpd/Prodigal) para predecir las proteínas a partir de los genomas:

```bash
module load prodigal/2.6.3/gcc/9.3.0-ztxs
cd $HOME/10.Bins/Genoma/
for i in $(ls *.fa); do prodigal -i $i -o $HOME/10.Bins/Proteoma/$i.txt -a $HOME/10.Bins/Proteoma/$i.faa ; done
cd
```

Podemos revisar las proteínas predichas con:

```bash
grep ">" $HOME/10.Bins/Proteoma/*.faa
```

## Anotación de Proteínas con KEGG

Utilizaremos KEGG, una base de datos que proporciona información sobre genes, proteínas, rutas metabólicas y más. 

Ahora, utilizaremos [kofam_scan](https://github.com/takaram/kofam_scan) para anotar las proteínas:

```bash
module load kofam-scan/1.3.0/gcc/8.3.1-n3v4
```

Dividiremos el trabajo en equipos de cuatro personas:

```bash
mkdir ../12.KOFAM
cd ../10.Bins/Proteoma/
for i in *.faa; do /opt/kofam_scan-1.3.0/exec_annotation -o $HOME/12.KOFAM/$i.txt $i --report-unannotated --cpu 4 -p /databases/kofamscan_dbs/profiles -k /databases/kofamscan_dbs/ko_list; done
cd
```

## Exploración Metabólica con Rbims

Vamos a explorar el metabolismo utilizando [Rbims](https://mirnavazquez.github.io/RbiMs/index.html). Primero, instalamos las bibliotecas necesarias en R:

```R
install.packages("devtools")
library(devtools)
install_github("mirnavazquez/RbiMs")
library(rbims)
library(tidyverse)
```

A continuación, leemos los resultados de KEGG y los mapeamos con la base de datos de KEGG:

```R
pulque_mapp <- read_ko("08.Kofamscan/02.KO_results/") %>%
    mapping_ko()
```

Nos centraremos en las vías metabólicas relacionadas con la obtención de energía:

```R
Overview <- c("Central Metabolism", "Carbon Fixation", "Nitrogen Metabolism", "Sulfur Metabolism", "Fermentation", "Methane Metabolism")
Energy_metabolisms_pulque <- pulque_mapp %>%
  drop_na(Cycle) %>%
  get_subset_pathway(rbims_pathway, Overview) 
```

Visualizamos los datos con un gráfico de burbujas:

```R
plot_bubble(tibble_ko = Energy_metabolisms_pulque,
            x_axis = Bin_name, 
            y_axis = Pathway_cycle,
            analysis = "KEGG",
            calc = "Percentage",
            range_size = c(1, 10),
            y_labs = FALSE,
            x_labs = FALSE)  
```

Añadiremos metadatos, como la taxonomía:

```R
Metadatos <- read_delim("11.GTDBTK/Metadatos.txt", delim = "\t")
```

Y generaremos un gráfico de burbujas con metadatos:

```R
plot_bubble(tibble_ko = Energy_metabolisms_pulque,
            x_axis = Bin_name, 
            y_axis = Pathway_cycle,
            analysis = "KEGG",
            data_experiment = Metadatos,
            calc = "Percentage",
            color_character = Class,
            range_size = c(1, 10),
            y_labs = FALSE,
            x_labs = FALSE) 
```

## Exploración de una Vía Específica

Podemos explorar una sola vía, como el "Secretion system," y crear un mapa de calor para visualizar los genes relacionados con esta vía:

```R
Secretion_system_pulque <- pulque_mapp %>%
  drop_na(Cycle) %>%
  get_subset_pathway(Cycle, "Secretion system")
```

Y, finalmente, generamos un mapa de calor:

```R
plot_heatmap(tibble_ko = Secretion_system_pulque, 
             y_axis = Genes,
             analysis = "KEGG",
             calc = "Binary")
```

También podemos agregar metadatos para obtener una visión más completa:

```R
plot_heatmap(tibble_ko = Secretion_system_pulque, 
             y_axis = Genes,
             data_experiment = Metadatos,
             order_x = Phylum,
             analysis = "KEGG",
             calc = "Binary")
```

```R
plot_heatmap(tibble_ko = Secretion_system_pulque, 
             y_axis = Genes,
             data_experiment = Metadatos,
             order_y = Pathway_cycle,
             order_x = Phylum,
             analysis = "KEGG",
             calc = "Binary")
```

Estas visualizaciones te ayudarán a explorar y comprender mejor los datos metabólicos de tus bins.
