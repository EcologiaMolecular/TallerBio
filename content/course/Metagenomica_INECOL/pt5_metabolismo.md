---
title: Ejercicio_05_Metabolismo
date: '2023-10-26'
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

# KEGG
*Pero que es KEGG?*
KEGG es una base de datos y un recurso en línea que proporciona información sobre la función y la asociación de genes y proteínas en una variedad de organismos.

KEGG se utiliza ampliamente en la investigación en biología molecular, bioquímica, genómica y otros campos relacionados para entender la función de genes y proteínas, así como su papel en vías metabólicas y enfermedades. La base de datos contiene información sobre rutas metabólicas, genes, proteínas, compuestos químicos y enfermedades, y ofrece herramientas y recursos para el análisis de datos genómicos y la interpretación de resultados experimentales.

Okay... mucho texto, ahora vamos a utilizar [kofam_scan](https://github.com/takaram/kofam_scan) para anotar las proteínas.

**Vamos a dividirnos en equipos para hacer la anotación de los bins! ya saben de 4 en 4 **
Ya saben lo primero, organizar su espacio de trabajo creando directorios
```
mkdir ../12.KOFAM
```

```
cd ../10.Bins/Proteoma/
for i in *.faa ; do  /opt/kofam_scan-1.3.0/exec_annotation -o  $HOME/12.KOFAM/$i.txt  $i  --report-unannotated  --cpu 4 -p /databases/kofamscan_dbs/profiles -k /databases/kofamscan_dbs/ko_list; done
cd 
```
Se automatiza la anotación funcional de proteinas en archivos ".faa" utilizando KOFAM-Scan para cada archivo de proteinas en el directorio "Proteoma", los resultados de la anotación se almacenan en archivos de texto dentro de un nuevo directorio

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
Esto es importantisimo ya que sin estas librerias no funcionaria absolutamente nada

Ahora, vamos a leer los resultados de KEGG y mapearlos con el resto de la base de datos de KEGG

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
crea un gráfico de burbujas que representa datos relacionados con la energía y el metabolismo utilizando KEGG como fuente de datos, las burbujas en el gráfico se escalan en tamaño según los valores de porcentaje y se les asignan colores según la clase de los datos. Los ejes X e Y muestran los nombres de los bins y los ciclos de vías, respectivamente.

**Ahora, vamos a explorar una sola vía**

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
esta llamada a la función "plot_heatmap" crea un mapa de calor que representa datos relacionados con genes y su participación en rutas metabólicas utilizando KEGG como fuente de datos, el eje Y muestra los nombres de los genes, y se utiliza una representación binaria para resaltar la presencia o ausencia de genes en las rutas metabólicas. El mapa de calor proporcionará una visualización de la relación entre genes y rutas metabólicas en el contexto del ciclo de "Secretion system"

**Ahora agreguemos metadatos**

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
Esto ordena las filas de acuerdo con el ciclo de vías al que pertenecen.

#Conclusion y aprendizajes 
*Uso de Prodigal para predecir proteínas:*
   Se utilizo la herramienta "Prodigal" para predecir proteínas a partir de secuencias genómicas en formato FASTA, Se ejecuto un ciclo que procesa multiples archivos FASTA y genera archivos de proteínas predichas.
Anotacion de proteínas con Kofam Scan:

*Se utilizó la herramienta "Kofam Scan" para anotar proteínas* Se realizaron anotaciones en paralelo para varios archivos de proteínas utilizando múltiples núcleos de CPU.

*Exploración del metabolismo con RbiMs:*
Se utilizó la librería "RbiMs" en R para explorar los datos relacionados con el metabolismo.
Se leyeron los resultados de KEGG y se mapearon con la base de datos de KEGG.
Se enfocó en metabolismos relacionados con la obtencion de energía, como "Central Metabolism," "Carbon Fixation," "Nitrogen Metabolism," "Sulfur Metabolism," "Fermentation," y "Methane Metabolism."

*Visualizacion de datos con plots:*
Se crearon graficos de burbujas para visualizar la relacion entre los metabolismos y las secuencias genómicas. Estos gráficos incluyeron información sobre el analisis de KEGG y el calculo de porcentaje.
*Se incorporaron metadatos, como la taxonomaa, para enriquecer la visualizacion de datos*

*Creacion de mapas de calor:*
Se generaron mapas de calor para visualizar la relacion entre genes y rutas metabolicas. Se utilizo una representacion binaria para resaltar la presencia o ausencia de genes en las rutas metabolicas.

**En resumen** esta actividad permitio aprender sobre la manipulacion de datos genomicos y proteomicos, la anotación de proteinas, la exploración del metabolismo y la visualizacion de datos utilizando herramientas y librerias como Prodigal, Kofam Scan, RbiMs y R,  se exploraron diferentes enfoques para representar datos, como graficos de burbujas y mapas de calor, lo que proporciona una comprension mas profunda de la relación entre genes, rutas metabolicas y filogenia en el contexto del metabolismo microbiano.