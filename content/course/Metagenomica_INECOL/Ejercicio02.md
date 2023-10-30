---
title: Ejercicio 02 - Ensamble
date: '2021-01-01'
type: book
weight: 20
---

# Ensamble

Ahora, nos sumergiremos en el proceso de ensamblaje.

## Configuración del directorio de ensamblaje

Primero, crearemos un directorio donde almacenaremos los resultados del ensamblaje.

```bash
mkdir $HOME/06.Ensamble
```

## Creación de enlaces simbólicos

A continuación, crearemos enlaces simbólicos a los archivos FASTQ que utilizaremos para el ensamblaje.

```bash
ln -s $HOME/03.Trimgalore/pulquet0_trimgalore/*fq $HOME/06.Ensamble
```

Luego, activaremos el entorno de [**Megahit**](https://github.com/voutcn/megahit).

```bash
conda activate megahit
```

Explore las opciones de Megahit y ajuste los parámetros según su experiencia.

```bash
megahit
```

Realicemos el ensamblaje con Megahit y especificaremos los parámetros necesarios.

```bash
srun --mem 16G -n 1 -p q1 megahit -1 $HOME/06.Ensamble/pulquet0_1_10M_val_1_val_1.fq -2 $HOME/06.Ensamble/pulquet0_2_10M_val_2_val_2.fq --k-list 21,33,55,77,99,121 --min-count 2 --verbose -t 4 -o $HOME/06.Ensamble/01.Megahit --out-prefix pulquet0_megahit
```

## Intentemos otro ensamblador: MetaSPAdes

Para utilizar [**MetaSPAdes**](https://cab.spbu.ru/software/meta-spades/), que vive en un entorno de Conda específico, primero lo activamos.

```bash
conda activate metagenomica
```

Luego, crearemos un directorio para los resultados de MetaSPAdes.

```bash
mkdir $HOME/06.Ensamble/02.Metaspades
```

Ejecutaremos MetaSPAdes con los parámetros adecuados.

```bash
spades.py --meta -k 21,33,55,77,99,127 -t 30 -1 data/clean/fermentation_1.fastq -2 data/clean/fermentation_2.fastq -o results/04.Ensamble/03.metaspades
```

Una vez completado, desactivamos el entorno de Conda.

```bash
conda deactivate
```

## Comparación de ensamblajes con QUAST

Ahora, compararemos los ensamblajes utilizando [**QUAST**](https://github.com/ablab/quast), una herramienta que evalúa la calidad de los ensamblajes genómicos.

Para utilizar QUAST, activaremos el entorno "metagenomica".

```bash
conda activate metagenomica
```

Una vez activado, ejecutaremos QUAST para comparar los resultados de MetaSPAdes y Megahit.

```
quast.py results/03.metaspades/contigsftr.fasta results/03.megahit/final.contigs.fa -o results/05.quast
```

Luego, desactivamos Conda.

```
conda deactivate
```

Finalmente, examine la salida de QUAST para determinar cuál opción es la más adecuada.

## Mapeo

**Profundidad**: La profundidad de cada contig se calcula mapeando las lecturas al ensamble. Este paso permite evaluar la calidad del ensamble y es necesario para hacer la reconstrucción de genomas ya que, como veremos más adelante, es uno de los parámetros que toman en cuenta los "bineadores". 

Vamos a mapear utilizando la herramienta BBMap del programa **[BBtools](https://jgi.doe.gov/data-and-tools/software-tools/bbtools/)**. Y [**samtools**](http://www.htslib.org/doc/samtools.html). 

**¡Manos a la obra!**

Crea tu carpeta y una liga simbólica a los datos:

```
mkdir  $HOME/07.Mapeo/
```

```
ln -s $HOME/06.Ensamble/01.Megahit/pulquet0_megahit.contigs.fa $HOME/07.Mapeo/
ln -s $HOME/03.Trimgalore/pulquet0_trimgalore/*fq .
```

Ahora ¡sí! explora las opciones de bbmap, y vamos a hacer nuestro primer mapeo.

```
bbmap.sh ref=$HOME/07.Mapeo/pulquet0_megahit.contigs.fa in=$HOME/07.Mapeo/pulquet0_1_10M_val_1_val_1.fq in2=$HOME/07.Mapeo/ pulquet0_2_10M_val_2_val_2.fq out=$HOME/07.Mapeo/pulquet0.sam kfilter=22 subfilter=15 maxindel=80 threads=4
```

```
samtools view -bShu $HOME/07.Mapeo/pulquet0.sam | samtools sort -@ 5 -o $HOME/07.Mapeo/pulquet0_sorted.bam
samtools index $HOME/07.Mapeo/pulquet0_sorted.bam
```

Como cualquier otro programa **jgi_summarize_bam_contig_depths** tiene opciones, podemos revisarlas. 

```
jgi_summarize_bam_contig_depths --outputDepth $HOME/07.Mapeo/pulquet0-depth.txt $HOME/07.Mapeo/pulquet0_sorted.bam
```

