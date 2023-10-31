---
title: Ejercicio_02_Ensamble
date: '2021-01-01'
type: book
weight: 20
---

## Ensamble

Bien, ahora si a ensamblar.

```
mkdir  $HOME/04.Ensamble
```

Crea ligas simbólicas a los archivos fastq que vamos a utilizar para ensamblar. 

```
ln -s $HOME/03.Interleave/SRR10997048_HQ.fastq  $HOME/04.Ensamble
```

¡Ahora si!

Ahora vamos a utilizar conda.

```
conda env list
```

Y el ensamblador [**megahit**](https://github.com/voutcn/megahit).

```
conda activate megahit
```

Explora la ayuda y según tu experiencia qué parámetros se deberían utilizar.

```
megahit
```

```
megahit --12 $HOME/04.Ensamble/SRR10997048_HQ.fastq  --k-list 21,33,55,77,99,121 --min-count 2 --verbose -t 4 -o $HOME/04.Ensamble/SRR10997048 --out-prefix SRR10997048_megahit

## Ensamblamos una muestra, pero podríamos hacer un ensamble que incluya todas las muestras. Ejemplo:
#/opt/anaconda3/pkgs/megahit-1.2.9-h2e03b76_1/bin/megahit --12 SRR10997046_HQ.fastq,SRR10997047_HQ.fastq,SRR10997048_HQ.fastq,SRR10997049_HQ.fastq,SRR10997050_HQ.fastq  --k-list 21,33,55,77,99,121 --min-count 2 --verbose -t 10 -o pulque --out-prefix megahit
```


## Mapeo

**Profundidad**: La profundidad de cada contig se calcula mapeando las lecturas al ensamble. Este paso permite evaluar la calidad del ensamble y es necesario para hacer la reconstrucción de genomas ya que, como veremos más adelante, es uno de los parámetros que toman en cuenta los "bineadores". 

Vamos a mapear utilizando la herramienta BBMap del programa **[BBtools](https://jgi.doe.gov/data-and-tools/software-tools/bbtools/)**. Y [**samtools**](http://www.htslib.org/doc/samtools.html). 

**¡Manos a la obra!**

Crea tu carpeta y una liga simbólica a los datos:

```
mkdir  $HOME/05.Mapeo/
```

```
ln -s $HOME/04.Ensamble/SRR10997048/SRR10997048_megahit.contigs.fa $HOME/05.Mapeo/
ln -s $HOME/02.Trimmomatic/SRR10997048_*_trimm.fastq .
```

Ahora ¡sí! explora las opciones de bbmap, y vamos a hacer nuestro primer mapeo.

```
/opt/anaconda3/bin/bbmap.sh ref=$HOME/05.Mapeo/SRR10997048_megahit.fasta in=$HOME/05.Mapeo/SRR10997048_R1_trimm.fastq in2=$HOME/05.Mapeo/SRR10997048_R2_trimm.fastq out=$HOME/05.Mapeo/SRR10997048.sam kfilter=22 subfilter=15 maxindel=80 threads=4
```

```
samtools view -bShu $HOME/05.Mapeo/SRR10997048.sam | samtools sort -@ 5 -o $HOME/05.Mapeo/SRR10997048_sorted.bam
samtools index $HOME/05.Mapeo/SRR10997048_sorted.bam
```

Como cualquier otro programa **jgi_summarize_bam_contig_depths** tiene opciones, podemos revisarlas. 

```
/opt/anaconda3/envs/metabat/bin/jgi_summarize_bam_contig_depths --outputDepth $HOME/05.Mapeo/SRR10997048-depth.txt $HOME/05.Mapeo/SRR10997048_sorted.bam 
```

