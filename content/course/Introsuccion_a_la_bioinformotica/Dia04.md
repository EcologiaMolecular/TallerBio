---
title: Ejercicio día 04
date: '2021-01-01'
type: book
weight: 20
---

## Búsqueda de secuencias en genomas anotados

Tenemos una colección de genomas de bacterias que no conservan una relación filogenética, pero que son endosimbiontes obligadas (Wolbachia, Holsopora obtusa, Helicobacter pylori, y Rickettsia rickettsia). Además, tenemos como referencia un genoma de una bacteria de vida libre (Salmonella enterica). Todo esto se encuentra en el directorio:

```
/databases/material_curso_2023/genomas_microbianos.tar.gz
```

Ahora vamos a traerlos a nuestro directorio y descomprimirlos con tar (recuerda las opciones -z (compresión zip), -x (extraer) -v (verbose) y -f (preservar archivo original, si gustas)

Qué tenemos? un directorio con tres tipos de archivos:
*.ptt
*.fna
*.faa

¿Qué continen estos archivos?

¿Cuál es el genoma más grande? ¿y el más pequeño?

¿Cuáles tienen el genoma más compacto en términos de número de proteínas codificadas?

¿Qué especies tienen (posiblemente) flagelo?

Supongamos que queremos buscar remanescencias de proteínas flagelares en el genoma de intracelulares, utilizando como modelo las de Salmonella. 

Cómo podríamos rescatar secuencias proteicas de Salmonella que puedan ser modelos de búsqueda?

Una vez que tengamos esos modelos, podríamos "blastearlas" contra el proteoma de las intracelulares obligadas, para identificar posibles secuencias que

Para emplear Blast, primero necesitamos identificar una (o varias) secuencia(s) modelo de búsqueda, así como una base de datos (generalmente una colección grande de secuencias.

Podemos generar una base de datos con el siguiente comando:

```
makeblastdb -dbtype [prot=proteinas; nucl=nucleótidos] -in [archivo FASTA]
```

Y la sintaxis para buscar mediante alineamientos locales con Blast
blast arguments (blastn, blastp, blastx, tblastx...)

```
  -query 	<archivo>
  -db		<archivo>
  -evalue	1e-#
  -outfmt	<String>
   		alignment view options:
   			  0 = Pairwise,
   			  1 = Query-anchored showing identities,
  			  2 = Query-anchored no identities,
  			  6 = Tabular,
     		  7 = Tabular with comment lines,
   			  11 = BLAST archive (ASN.1),
   		      18 = Organism Report
```  		      
   		      
Ejercicio libre: piensa en la proteína bacteriana favorita y búscala en Salmonella. La puedes encontrar en las intracelulares obligadas?

## ENSAMBLE DE UN GENOMA

Tenemos unas lecturas en /databases/material_curso_2023/lecturas_ensamble_genoma
Vamos a ver las instrucciones de Spades para hacer un ensamble sencillo. Para ejecutar este programa, primero tenemos que exportar la ruta de su ubicación a PATH:

```
export PATH=/usr/local/bin/racon/build/bin:/usr/local/bin/SPAdes-3.15.5/bin:$PATH
```

# aquí es importante de anteceder la ruta a $PATH, ta que $PATH tiene una versión de un programa (spades) más viejita a la que requiere usar nuestro programa.

Primero vamos a ensamblas las lecturas cortas únicamente con Spades, definiendo un directorio de salida. ¿Qué te parece el resultado?


Ahora incorporemos las lecturas largas, pero en baja cobertura. ¿Mejora?

¿ Y si mejor agregamos lecturas largas en alta cobertura?

¿puedes ensamblar sólamente con lecturas largas? 
