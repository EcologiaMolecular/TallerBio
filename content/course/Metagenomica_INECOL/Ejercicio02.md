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
mkdir $HOME/04.Ensamble
```

## Creación de enlaces simbólicos

A continuación, crearemos enlaces simbólicos a los archivos FASTQ que utilizaremos para el ensamblaje.

```bash
ln -s $HOME/03.Interleave/SRR10997048_HQ.fastq $HOME/04.Ensamble
```

¡Listo para continuar!

## Configuración de Conda y Megahit

Verificaremos las configuraciones de Conda para asegurarnos de que todo esté en orden.

```bash
conda env list
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
megahit --12 $HOME/04.Ensamble/SRR10997048_HQ.fastq --k-list 21,33,55,77,99,121 --min-count 2 --verbose -t 4 -o $HOME/04.Ensamble/SRR10997048 --out-prefix SRR10997048_megahit
```

## Intentemos otro ensamblador: MetaSPAdes

Para utilizar [**MetaSPAdes**](https://cab.spbu.ru/software/meta-spades/), que vive en un entorno de Conda específico, primero lo activamos.

```bash
conda activate metagenomics
```

Luego, crearemos un directorio para los resultados de MetaSPAdes.

```bash
mkdir $HOME/05.Metaspades
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

