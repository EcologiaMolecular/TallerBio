---
title: pt4_Taxonomia
date: '2023-10-26'
author: Mirna Vazquez-Landa, Yaxche Lona-Tellez, Diego Montes-Gabriel
type: book
weight: 20
---

## Taxonomía

Vamos a explorar la taxonomía de estos bins con [GTDB-tk](https://github.com/Ecogenomics/GTDBTk).

```
{bash eval=F}
# Crear un directorio para GTDB-tk
mkdir ../11.GTDBTK
```
Activemos el ambiente

```{bash  eval=F}
conda activate gtdbtk-2.1.1
```

Indiquemos dónde está la base de datos:

```{bash  eval=F}
export GTDBTK_DATA_PATH=/botete/mvazquez/release207_v2/
```

Explora la ayuda de GTDB-tk

```
gtdbtk --help
```

Excelente, ahora en equipos vamos a correr GTDB-tk, como somos 16 alumnos, por favor dividanse en equipos de 4

```{bash, eval=FALSE}
gtdbtk classify_wf --genome_dir $HOME/10.Bins --out_dir $HOME/11.GTDBTK  --cpus 5 -x fa 
```

Vamos a visualizar los datos. *Todos a R!!*

# Leamos los datos

```{r, eval=FALSE}
library(tidyverse)
```
"tidyverse" es un conjunto de paquetes de R que incluye herramientas para manipulación y visualización de datos. Al cargar este paquete, se habilitan funciones y capacidades adicionales para trabajar con datos de manera más eficiente.
```{r, eval=FALSE}
GTDBK<-read.table("11.GTDBTK/gtdbtk.bac120.summary.tsv", 
  sep = "\t", header = T, 
  na.strings ="", stringsAsFactors= F)%>%
  as_tibble()
```
Leemos un archivo de datos tabulados con delimitador de pestaña ("\t") ubicado en "11.GTDBTK/gtdbtk.bac120.summary.tsv".
header = T indica que la primera fila del archivo contiene nombres de columnas. *ESTO TAL VEZ YA SE EXPLICO ANTES*
na.strings = "" especifica que los valores nulos en el archivo se representan como cadenas vacías.
stringsAsFactors = F evita que las cadenas se conviertan en factores en R, lo que es útil en el análisis de datos.
as_tibble() convierte los datos en un formato "tibble", que es una estructura de datos similar a un DataFrame en R. Tibble es parte del paquete "tidyverse" y facilita el análisis de datos. *Por hoy que viva Tydyverse*

#Continuemos

```{r, eval=FALSE}
pulque_gtdbtk<-GTDBK %>%
  select(user_genome, classification) %>%
  separate(classification, c("Domain", "Phylum", "Class", "Order",
                             "Family", "Genus", "Species"), sep= ";") %>%
  rename(Bin_name=user_genome)  %>%
  unite(Bin_name_2, c("Bin_name", "Phylum"), remove = FALSE) %>%
  select(Bin_name, Domain, Phylum, Class, Order, Family, Genus, 
         Species)
```
Utilizamos la función "separate()" para dividir la columna "classification" en varias columnas ("Domain", "Phylum", "Class", "Order", "Family", "Genus", "Species") utilizando el punto y coma como separador.
Renombramos la columna "user_genome" como "Bin_name".
Utilizamos la función "unite()" para combinar las columnas "Bin_name" y "Phylum" en una nueva columna llamada "Bin_name_2".
Finalmente, selecciona las columnas "Bin_name", "Domain", "Phylum", "Class", "Order", "Family", "Genus", y "Species".

*Paréntesis*, vamos a imprimir esta tabla para convertirla en metadatos.

```{r eval=FALSE}
write.table(pulque_gtdbtk, file = "11.GTDBTK/Metadatos.txt", sep="\t", quote = F,
            row.names = F, col.names = T)
```
Este código toma el tibble "pulque_gtdbtk" y lo guarda como un archivo de texto separado por tabulaciones (TSV), aclaracion extra, este archivo es un tanto similar a lo que podemos observar en los excel, por lo que nos podria parecer mas familiar. Esto puede ser útil para convertir los datos en un formato adecuado para su posterior análisis o visualización.
**Vamos a hacer un plot**

```{r, eval=FALSE}
GTDBtk<-pulque_gtdbtk %>%
  count(Domain, Phylum) %>%
  rename(Number_of_MAGs = n) %>%
  ggplot(aes(x = Domain, 
             y = Number_of_MAGs, fill = Phylum)) + 
  geom_bar(stat = "identity", position=position_dodge())+
  theme_minimal()
```
Este fragmento de código crea un gráfico de barras que muestra cuántos MAGs pertenecen a cada dominio y los colorea según el filo. Esto proporciona una visualización de la distribución taxonómica de los bins en tu análisis.

**Auqnue puede ser interactivo también**

```{r, eval=FALSE}
library(plotly)
GTDBtk_p_fig <- ggplotly(GTDBtk)
```
Agregamos interactividad a tu gráfico, lo que te permitirá, por ejemplo, hacer clic en las barras del gráfico para obtener más información o realizar zoom y desplazamiento en el gráfico si es necesario, esto es útil para explorar visualmente los datos de manera más dinámica.

**No es lindo?**