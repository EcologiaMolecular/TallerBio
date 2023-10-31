---
title: Ejercicio_04_Taxonomia
date: '2021-01-01'
type: book
weight: 20
---

## Taxonomia

Vamos a explorar la taxonomía de estos bins con [GTDB-tk](https://github.com/Ecogenomics/GTDBTk).

```{bash eval=F}
mkdir $HOME/11.GTDBTK
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


Excelente, ahora en equipos vamos a correr GTDB-tk.

```{bash, eval=FALSE}
gtdbtk classify_wf --genome_dir $HOME/10.Bins --out_dir $HOME/11.GTDBTK  --cpus 5 -x fa 
```

Vamos a visualizar los datos. Todos a R!!

# Leamos los datos

```{r, eval=FALSE}
library(tidyverse)
```

```{r, eval=FALSE}
GTDBK<-read.table("11.GTDBTK/gtdbtk.bac120.summary.tsv", 
  sep = "\t", header = T, 
  na.strings ="", stringsAsFactors= F)%>%
  as_tibble()
```

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

Paréntesis, vamos a imprimir esta tabla para convertirla en metadatos.

```{r eval=FALSE}
write.table(pulque_gtdbtk, file = "11.GTDBTK/Metadatos.txt", sep="\t", quote = F,
            row.names = F, col.names = T)
```

Vamos a hacer un plot

```{r, eval=FALSE}
GTDBtk<-pulque_gtdbtk %>%
  count(Domain, Phylum) %>%
  rename(Number_of_MAGs = n) %>%
  ggplot(aes(x = Domain, 
             y = Number_of_MAGs, fill = Phylum)) + 
  geom_bar(stat = "identity", position=position_dodge())+
  theme_minimal()
```

Puede ser interactivo también.

```{r, eval=FALSE}
library(plotly)
GTDBtk_p_fig <- ggplotly(GTDBtk)
```
