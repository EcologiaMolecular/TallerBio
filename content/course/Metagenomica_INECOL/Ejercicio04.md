---
title: Ejercicio_04_Taxonomia
date: '2023-10-26'
type: book
weight: 20
---

# Taxonomía

En esta sección, exploraremos la taxonomía de los bins utilizando [GTDB-tk](https://github.com/Ecogenomics/GTDBTk).

## Configuración de GTDB-tk

Empezaremos por crear un directorio para GTDB-tk:

```bash
mkdir ../11.GTDBTK
```

Luego, activaremos el entorno de Conda requerido:

```bash
conda activate gtdbtk-2.1.1
```

Asegurémonos de que GTDB-tk pueda encontrar la base de datos:

```bash
export GTDBTK_DATA_PATH=/botete/mvazquez/release207_v2/
```

Para explorar las opciones disponibles en GTDB-tk, ejecutamos:

```bash
gtdbtk --help
```

## Clasificación de los Bins

Ahora, en grupos de cuatro estudiantes, ejecutaremos GTDB-tk para clasificar los bins. La siguiente línea de código es un ejemplo de cómo ejecutarlo:

```bash
gtdbtk classify_wf --genome_dir $HOME/10.Bins --out_dir $HOME/11.GTDBTK --cpus 5 -x fa
```

Después de ejecutar GTDB-tk, continuaremos en R para visualizar los datos.

## Análisis en R

Para el análisis en R, cargaremos la biblioteca "tidyverse", que incluye herramientas para la manipulación y visualización de datos:

```R
library(tidyverse)
```

Luego, importamos los datos desde el archivo "gtdbtk.bac120.summary.tsv" y los almacenamos en un tibble:

```R
GTDBK <- read.table("11.GTDBTK/gtdbtk.bac120.summary.tsv", 
  sep = "\t", header = TRUE, na.strings = "", stringsAsFactors = FALSE) %>%
  as_tibble()
```

El archivo contiene información sobre la clasificación taxonómica de los bins.

Continuamos limpiando y transformando los datos:

```R
pulque_gtdbtk <- GTDBK %>%
  select(user_genome, classification) %>%
  separate(classification, c("Domain", "Phylum", "Class", "Order", "Family", "Genus", "Species"), sep = ";") %>%
  rename(Bin_name = user_genome) %>%
  unite(Bin_name_2, c("Bin_name", "Phylum"), remove = FALSE) %>%
  select(Bin_name, Domain, Phylum, Class, Order, Family, Genus, Species)
```

Finalmente, guardamos los datos en un archivo de metadatos:

```R
write.table(pulque_gtdbtk, file = "11.GTDBTK/Metadatos.txt", sep = "\t", quote = FALSE, row.names = FALSE, col.names = TRUE)
```

## Visualización de Datos

Creamos un gráfico de barras interactivo que muestra la distribución taxonómica de los bins:

```R
GTDBtk <- pulque_gtdbtk %>%
  count(Domain, Phylum) %>%
  rename(Number_of_MAGs = n) %>%
  ggplot(aes(x = Domain, y = Number_of_MAGs, fill = Phylum)) + 
  geom_bar(stat = "identity", position = position_dodge()) +
  theme_minimal()
```

Si prefieres una visualización interactiva:

```R
library(plotly)
GTDBtk_p_fig <- ggplotly(GTDBtk)
```

Esta adición proporciona interactividad al gráfico, permitiéndote explorar visualmente los datos de manera dinámica.

