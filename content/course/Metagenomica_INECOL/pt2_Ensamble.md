---
title: Ejercicio_02_Ensamble
date: '2023-10-25'
type: book
weight: 20
---

## Ensamble Metagenomico

*Bien, ahora si a ensamblar!*
En este ejemplo ensamblaremos todas las muestras que dan lugar a la fermentación del pulque, es decir, haremos un coensamble. Sin embargo, también podríamos correr un esamble por muestra de manera independiente.

Ensamblaremos con *MetaSPAdes* y *Megahit*, después compararemos ambos coensambles basándonos en longitudes de los contigs con QUAST y el porcentaje de lecturas que dan lugar al coensamble, esto con bbmap de bbtools.

Para acceder a metaspades y megahit activamos el ambiente metagenomics

```
conda activate metagenomics
```

Crea ligas simbólicas a los archivos fastq que vamos a utilizar para ensamblar y creamos el directorio de los ensambles. 

```
ln -s /../03.Analisis_de_metagenomas/data/clean/SRR10997048_HQ.fastq  /results/04.Ensamble
```

¡Ahora si!

Ahora vamos a utilizar conda.
Explora la ayuda y según tu experiencia qué parámetros se deberían utilizar del ensamblador [**megahit**](https://github.com/voutcn/megahit).

```
megahit
```
Una vez que conozcamos los parametros *Hay que ejecutarlo*

```
megahit -t 34 --k-list 21,33,55,77,99,127 --min-contig-len 1000 -1 data/clean/fermentation_1.fastq -2 data/clean/fermentation_2.fastq -o results/04.Ensamble/03.megahit 
```
Ahora podriamos hacer el ensamble pero con [**MetaSpades**](https://github.com/ablab/spades)
Para *SPAdes* necesitamos crear el directorio en el que se alojarán los resultados

```
mkdir -p results/03.metaspades
```
Y lo ejecutamos....
```
spades.py --meta -k 21,33,55,77,99,127 -t 30 -1 data/clean/fermentation_1.fastq -2 data/clean/fermentation_2.fastq -o results/04.Ensamble/03.metaspades 
```
*Desactivamos el ambiente*
```
conda deactivate
```
En spades no pudimos delimitar el tamaño mínimo de contig, por lo tanto tenemos que eliminar los de menor tamaño.

**Veamos cuanto miden los contigs de nuestro ensamble.**
```
infoseq -auto -nocolumns -delimiter '\t' -only -noheading -name -length results/03.metaspades/contigs.fasta | sort -k2n | cut -f2 | sort | uniq -c | sort -k2n | less
```
Vamos a filtrarlo ... Para el filtrado necesitamos entrar al directorio de resultados de spades y ejecutar el script desde ahí.
```
python filter_contig_length_sp.py 1000 1 contigs.fasta
```
**Comparemos el número de contigs que había antes y después de filtrar.**
```
grep -c '>' contigs.fasta 
grep -c '>' contigs.fltr.fasta 
```
##Comparacion de ensambles a traves del mapeo
**Profundidad**:
La profundidad de cada contig se calcula mapeando las lecturas al ensamble. Este paso permite evaluar la calidad del ensamble y es necesario para hacer la reconstrucción de genomas ya que, como veremos más adelante, es uno de los parámetros que toman en cuenta los "bineadores". 

Vamos a mapear utilizando la herramienta BBMap del programa **[BBtools](https://jgi.doe.gov/data-and-tools/software-tools/bbtools/)**. Y [**samtools**](http://www.htslib.org/doc/samtools.html). 

**¡Manos a la obra!**
Primero creamos el directorio para los mapeos
```
mkdir  ../05.Mapeo/04.Depth
```
Mapeamos las lecturas al ensamble de metaspades con bbmap
```
#Hacer ligas simbolicas y correr dentro del directorio 04.depth
nohup bbmap.sh ref=results/03.metaspades/contigsftr.fasta in=data/clean/fermentation_1.fastq in2=data/clean/fermentation_2.fastq out=results/05.Mapeo/04.depth/fermentation.metaspades.sam   scafstats=fermentation_metaspades.scafstats &
```
Bien, veamos cuantas lecturas mapearon al ensamble
```
#Número de lecturas iniciales
95588026
#Obtenemos las que mapearon
cut -f8 results/04.depth/fermentation_metaspades.scafstats | grep -v assignedReads | awk '{sum += $1;}END{print sum;}'
91148282
#Sacamos el porcentaje
95.35 %
```
*Hagamos lo mismo para el de megahit*
```
nohup bbmap.sh ref=results/03.megahit/final.contigs.fa in=data/clean/fermentation_1.fastq in2=data/clean/fermentation_2.fastq out=results/05.Mapeo/04.depth/fermentation.megahit.sam maxindel=80 scafstats=results/05.Mapeo/04.depth/fermentation_megahit.scafstats &
```
*Cuantas lecturas se mapearon?*
```
95588026
cut -f8 results/04.depth/fermentation_megahit.scafstats | grep -v assignedReads | awk '{sum += $1;}END{print sum;}'
91218466
95.428
```
**Conozcamos a QUAST** (https://github.com/ablab/quast)
"Quality Assessment Tool for Genome Assemblies" (Herramienta de Evaluación de Calidad para Ensamblajes de Genomas) es una herramienta de software diseñada para evaluar y analizar la calidad de los ensamblajes de genomas. QUAST es una herramienta versatil que proporciona informacion valiosa sobre la calidad de los ensamblajes genómicos, lo que es fundamental en la investigacion genomica y la bioinformatica para determinar la confiabilidad de los resultados y para orientar mejoras en los ensamblajes si es necesario.
*Para usar QUAST, debemos activar a un viejo conocido, o sea metagenomics*
```
conda activate metagenomics
```
Una vez activado metagenomics, ahora si podemos ejecutar QUAST 
```
quast.py results/03.metaspades/contigsftr.fasta results/03.megahit/final.contigs.fa -o results/05.quast
```
Desactivemos conda
```
conda deactivate
```
Y veamos la salida de QUAST para determinar la mejor opción.
(Para revisar la salida de QUAST, puedes abrir los archivos de texto generados por la herramienta en un editor de texto o visor de documentos).

#Que aprendimos esta vez? 
En esta lección de ensamblaje metagenómico, se llevaron a cabo una serie de pasos para ensamblar y evaluar la calidad de los ensambles generados a partir de datos metagenomics, veamos un pequeno resumen las herramientas utilizadas y la utilidad de cada paso:

 1. MetaSPAdes y Megahit: Se utilizaron dos ensambladores de genomas metagenómicos, MetaSPAdes y Megahit. Estos ensambladores permiten ensamblar secuencias genómicas a partir de datos metagenómicos, donde múltiples genomas pueden estar presentes en la muestra.

 2. Parámetros de ensamblaje: Se exploraron los parámetros de ensamblaje para Megahit a través del comando megahit.
 3. Filtrado de contigs: Despues de ensamblar con MetaSPAdes, se examinaron las longitudes de los contigs y se realizó un filtrado para eliminar los contigs con una longitud menor a 1000 bases, esto se hizo para eliminar contigs de baja calidad o ruido del ensamblaje.

 4. Mapeo de lecturas al ensamble: Se utiliz0 la herramienta BBMap de BBtools para mapear las lecturas metagenómicas originales al ensamblaje generado por MetaSPAdes y Megahit, lo hicimos para calcular la profundidad de cobertura de cada contig y evaluar la calidad del ensamblaje.

 5. Uso de QUAST: se utilizo para comparar los ensambles generados por MetaSPAdes y Megahit. QUAST proporciona estadísticas detalladas sobre la calidad del ensamblaje, incluyendo medidas como el tamaño de N50, la cantidad de contigs, y mas, la salida de QUAST se utiliza para determinar la calidad del ensamblaje y orientar mejoras si es necesario.

*En resumen* 
Esta lección se centro en el ensamblaje y la evaluación de calidad de datos metagenómicos utilizando MetaSPAdes y Megahit, una vez que ensamblamos, se compararon los ensambles utilizando QUAST, lo que proporciona información crítica para evaluar la calidad de los ensambles y tomar decisiones informadas en la investigación genómica y la bioinformática.