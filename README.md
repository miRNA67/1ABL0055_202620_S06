# Semana 06: Anotación de genomas

## Logro de la sesión:

Al finalizar la sesión, el estudiante utiliza herramientas bioinformáticas para anotar estructural y funcionalmente un genoma bacteriano e identificar en él genes asociados a la promoción del crecimiento vegetal.

## Estructura de la práctica:

1. Acceso al servidor de cómputo
2. Preparación del genoma y anotación con Bakta
3. Envío de los análisis en línea (eggNOG-mapper y antiSMASH)
4. Evaluación del proteoma con BUSCO
5. Visualización del genoma anotado
6. Anotación funcional (COG y KEGG)
7. Identificación de genes promotores del crecimiento vegetal (PGP)
8. Identificación de enzimas activas sobre carbohidratos (CAZymes)
9. Identificación de clústeres de metabolitos secundarios (antiSMASH)
10. Evaluación de bioseguridad: genes de resistencia y de virulencia (ResFinder y ABRicate)
11. Identificación de plásmidos y elementos genéticos móviles (MOB-suite y MobileElementFinder)
12. Genómica comparativa con OrthoVenn3
13. Anotación del genoma ensamblado y validado en la Semana 05

> **Cómo está organizada esta práctica:** las secciones 2 a 12 son **demostrativas** y se realizan con un genoma de ejemplo, `m01` (`/data/2025_1/database/m01_flye.racon.fasta`), una cepa del género *Vibrio* cuyo genoma tiene dos cromosomas y plásmidos. Al final (sección 13), cada grupo repite todo el proceso con **el genoma que ensambló y validó en la Semana 05**, que corresponde a una bacteria promotora del crecimiento vegetal, y con esos resultados elabora la bitácora.
>
> El genoma de demostración **no** es una bacteria promotora del crecimiento vegetal: sirve para aprender a ejecutar cada programa y a leer sus salidas. Los resultados de las secciones 7 a 11 serán distintos con su cepa, y ese contraste es parte de la discusión.

## Flujo de trabajo:

### Preparación y anotación (secciones 2 a 6):

```mermaid
flowchart LR
    subgraph ANN["Anotación"]
        direction LR
        A["Ensamblaje (m01_flye.racon.fasta)"] --> B["PRINSEQ (filtrado y renombrado de contigs)"]
        B --> C["Bakta (CDS, rRNA, tRNA, función)"]
        C --> D["m01.faa / m01.gff3 / m01.gbff"]
        D --> E["BUSCO (modo proteins)"]
        D --> F["Proksee (mapa circular)"]
        D --> G["eggNOG-mapper (KEGG) + COG"]
    end
```

### Análisis dirigidos y comparativos (secciones 7 a 12):

```mermaid
flowchart LR
    subgraph PGP["Potencial PGP y comparación"]
        direction LR
        A["Genoma + proteoma anotado"] --> B["Búsqueda dirigida + PGPg_finder (rasgos PGP)"]
        A --> C["run_dbcan (CAZymes, CGC y sustratos)"]
        A --> D["antiSMASH (metabolitos secundarios)"]
        A --> S["ResFinder + ABRicate (resistencia y virulencia)"]
        A --> M["MOB-suite + MobileElementFinder (plásmidos y elementos móviles)"]
        A --> E["OrthoVenn3 (núcleo, accesorio, únicos)"]
        R["Genomas de referencia del género"] --> B
        R --> E
    end
```

## Programas requeridos:

### Programas de acceso al servidor:

PuTTY v0.79 https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
   - **Descripción:** cliente SSH para establecer conexiones de línea de comandos con el servidor.

WinSCP v6.1 https://winscp.net/eng/download.php
   - **Descripción:** cliente SFTP con interfaz gráfica para transferir archivos entre su computadora y el servidor. En esta práctica se usa con frecuencia: los análisis en línea requieren descargar archivos del servidor y subir los resultados de vuelta.

### Programas bioinformáticos:

PRINSEQ-lite v0.20.4 https://prinseq.sourceforge.net/
   - **Descripción:** PRINSEQ (PReprocessing and INformation of SEQuence data) es un programa para filtrar, formatear y recortar secuencias. En esta práctica se usa para eliminar los contigs muy cortos del ensamblaje y dar a los contigs nombres cortos y uniformes antes de la anotación.

Bakta v1.12.1 https://github.com/oschwengers/bakta
   - **Descripción:** Bakta es un anotador de genomas bacterianos y plásmidos. Realiza la anotación estructural (¿dónde están los genes?) y funcional (¿qué hacen?) en un solo paso: predice CDS (con Pyrodigal, una implementación de Prodigal), rRNA, tRNA, tmRNA, ncRNA, CRISPR y orígenes de replicación, y asigna función a las proteínas mediante identificadores estables (UniRef, RefSeq), lo que hace la anotación reproducible y lista para depositar en bases de datos públicas. Es el sucesor recomendado de Prokka. En el servidor está instalado con su base de datos completa (esquema v6.0), que incluye la de AMRFinderPlus.

BUSCO v6.1.0 https://busco.ezlab.org/
   - **Descripción:** ya usado en la Semana 05 en modo `genome`. Aquí se usa en modo `proteins`, sobre el proteoma anotado, para comprobar que la anotación recuperó los genes que el ensamblaje contenía.

PGPg_finder v1.1.0 https://github.com/tpellegrinetti/PGPg_finder
   - **Descripción:** PGPg_finder identifica genes asociados a rasgos promotores del crecimiento vegetal (PGPT, *Plant Growth-Promoting Traits*). Predice las proteínas con Prodigal (un predictor de genes procariotas) y las compara con DIAMOND contra la base de datos PLaBAse, que organiza los rasgos en una ontología de cinco niveles (por ejemplo, efectos directos → biofertilización → adquisición de nitrógeno → fijación de nitrógeno). Genera tablas de conteo y mapas de calor por nivel.

run_dbcan v5 https://github.com/bcb-unl/run_dbcan
   - **Descripción:** versión local de dbCAN3 para anotar enzimas activas sobre carbohidratos (CAZymes). Combina tres métodos (HMM de familias, HMM de subfamilias y DIAMOND contra CAZy), identifica clústeres de genes de CAZymes (CGC) y predice su sustrato.

ResFinder v4 https://github.com/genomicepidemiology/resfinder
   - **Descripción:** ResFinder identifica genes **adquiridos** de resistencia a antimicrobianos en genomas bacterianos y predice el fenotipo de resistencia asociado. Incluye PointFinder (mutaciones cromosómicas de resistencia, solo para algunas especies de importancia clínica) y DisinFinder (resistencia a desinfectantes y biocidas).

ABRicate v1.0.1 https://github.com/tseemann/abricate
   - **Descripción:** ABRicate busca, por comparación de secuencias (BLAST), genes de resistencia y de virulencia en genomas ensamblados, usando bases de datos como VFDB (factores de virulencia), CARD, ResFinder y NCBI (resistencia) o PlasmidFinder (replicones de plásmidos). En esta práctica se usa con VFDB.

MOB-suite v3.1.9 https://github.com/phac-nml/mob-suite
   - **Descripción:** MOB-suite identifica y caracteriza plásmidos en ensamblajes bacterianos. `mob_recon` separa los contigs en cromosoma y plásmidos, y `mob_typer` clasifica cada plásmido según su replicón, su relaxasa y su capacidad de transferencia (conjugativo, movilizable o no movilizable).

MobileElementFinder v1.1.2 https://pypi.org/project/MobileElementFinder/
   - **Descripción:** MobileElementFinder (`mefinder`) detecta elementos genéticos móviles en genomas ensamblados: secuencias de inserción (IS), transposones (Tn), elementos integrativos y conjugativos (ICE), entre otros.

SeqKit v2 https://bioinf.shenwei.me/seqkit/
   - **Descripción:** herramienta para manipular archivos FASTA/FASTQ; aquí se usa para obtener estadísticas y extraer secuencias de proteínas por su identificador.

### Herramientas bioinformáticas en línea:

eggNOG-mapper https://eggnog-mapper.cgmlab.org/
   - **Descripción:** asigna función a las proteínas por ortología (no por el mejor hit de similitud): ubica cada proteína en un grupo de ortólogos de la base de datos eggNOG y le transfiere el grupo COG, el nombre del gen, los ortólogos y rutas de KEGG, los términos GO, el número EC y los dominios PFAM.

KEGG Mapper Reconstruct https://www.genome.jp/kegg/mapper/reconstruct.html
   - **Descripción:** a partir de una lista de genes con su KO (KEGG Orthology), reconstruye las rutas metabólicas y módulos presentes en el genoma.

Proksee https://proksee.ca/
   - **Descripción:** genera mapas circulares de genomas bacterianos a partir de un archivo GenBank o FASTA, con pistas de genes, contenido GC y sesgo GC.

antiSMASH (versión bacteriana) https://antismash.secondarymetabolites.org/
   - **Descripción:** identifica clústeres de genes biosintéticos (BGC) de metabolitos secundarios: sideróforos, péptidos no ribosomales (NRPS), policétidos (PKS), bacteriocinas, terpenos, entre otros.

OrthoVenn3 https://orthovenn3.bioinfotoolkits.net/
   - **Descripción:** compara los proteomas de varios genomas, agrupa las proteínas en clústeres de ortólogos y muestra cuántos clústeres son compartidos por todos (núcleo), por algunos (accesorio) o exclusivos de un genoma, con diagramas de Venn/UpSet y enriquecimiento funcional.

PLaBAse (PGPT-Pred) https://plabase.cs.uni-tuebingen.de/
   - **Descripción:** recurso web de bacterias asociadas a plantas del que proviene la ontología PGPT. Su herramienta PGPT-Pred es la alternativa en línea a PGPg_finder.

dbCAN3 https://pro.unl.edu/dbCAN2/
   - **Descripción:** servidor web de dbCAN, alternativa en línea a run_dbcan.

NCBI Datasets Genome https://www.ncbi.nlm.nih.gov/datasets/genome/
   - **Descripción:** ya usado en la Semana 05; aquí se usa para descargar los genomas y proteomas de referencia del género.

## Metodología:

## 1. Acceso al servidor de cómputo:

### Abrir el programa PuTTY, colocar el hostname ( 10.142.250.66 ) y port ( 22 ), y dar clic en Open:

<img width="500" alt="image" src="https://github.com/user-attachments/assets/92f89dbb-1a21-411d-adb5-38fe486a5567" />

**Figura 1.** Ventana de configuración de PuTTY. En *Host Name* se coloca la dirección IP del servidor y en *Port* el puerto 22 (SSH); el tipo de conexión debe ser SSH.

### En la terminal abierta, escribir su usuario y contraseña correspondiente para tener acceso al servidor de cómputo Tensor:

<img width="700" alt="image" src="https://github.com/user-attachments/assets/4d246e93-c59c-4749-a2dd-03db25c53654" />

**Figura 2.** Terminal de PuTTY al iniciar la sesión. Al escribir la contraseña no se muestran caracteres en pantalla; es el comportamiento normal. Una vez dentro, la línea de comandos indica su usuario y el nombre del servidor.

### Abrir el programa WinSCP, colocar el hostname ( 10.142.250.66 ) y port ( 22 ), escribir su usuario y contraseña correspondiente para tener acceso al servidor de cómputo Tensor, y hacer clic en Login:

<img width="500" alt="image" src="https://github.com/user-attachments/assets/ef4dc253-ce4a-417d-b761-39692d2a011a" />

**Figura 3.** Ventana de inicio de sesión de WinSCP, con el protocolo SFTP, la dirección del servidor, el puerto 22, el usuario y la contraseña.

<img width="500" alt="image" src="https://github.com/user-attachments/assets/577debca-6085-47c5-9bbd-73688bfa8bb0" />

**Figura 4.** Sesión de WinSCP abierta. El panel izquierdo muestra las carpetas de su computadora y el derecho, las del servidor; los archivos se transfieren arrastrándolos de un panel al otro.

### Crear la estructura de carpetas de trabajo

```bash
cd ~/genomics

mkdir -p annotation/{bakta,busco,eggnog,pgp,cazymes,antismash,resfinder,virulence,plasmid,mobile,orthovenn}

tree -L 2 ~/genomics/annotation
```

> **Comentario:**
> - `cd ~/genomics`: entra a la carpeta de trabajo del curso (`~` es su carpeta personal).
> - `mkdir -p annotation/{...}`: crea la carpeta `annotation` y, dentro de ella, una subcarpeta por cada análisis de la práctica, para que los resultados no se mezclen. La opción `-p` crea las carpetas intermedias y no da error si ya existen.
> - Las carpetas se crean **dentro de `~/genomics`**, junto a `assembly/`, `validation/` y `taxonomy/` de la Semana 05.
> - `tree -L 2`: muestra la estructura creada, hasta dos niveles de profundidad. Debe ver las 11 subcarpetas dentro de `annotation`.

## 2. Preparación del genoma y anotación con Bakta

### Filtrar y renombrar los contigs del ensamblaje

Antes de anotar conviene dejar el ensamblaje "listo para publicar": sin contigs muy cortos y con nombres de contig cortos, uniformes, **únicos** y que identifiquen a la cepa. Esos nombres aparecerán en todas las tablas de la práctica y en los archivos que se depositan en las bases de datos públicas.

```bash
mkdir -p ~/genomics/assembly/nanopore

cd ~/genomics/assembly/nanopore

conda activate genome

grep ">" /data/2025_1/database/m01_flye.racon.fasta

>contig_bts_jin_01
>contig_bts_suga_02
>contig_bts_j-hope_03
>contig_bts_rm_04
>contig_bts_jimin_05
>contig_bts_v_06

prinseq-lite.pl -fasta /data/2025_1/database/m01_flye.racon.fasta -min_len 500 -seq_id "m01_00" -out_good m01_genome_final -out_bad null

Input and filter stats:
        Input sequences: 6
        Input bases: 6,068,252
        Input mean length: 1011375.33
        Good sequences: 6 (100.00%)
        Good bases: 6,068,252
        Good mean length: 1011375.33
        Bad sequences: 0 (0.00%)
        Sequences filtered by specified parameters:
        none

grep ">" m01_genome_final.fasta

>m01_001
>m01_002
>m01_003
>m01_004
>m01_005
>m01_006

conda deactivate
```

> **Comentario (comandos):**
> - `mkdir -p ~/genomics/assembly/nanopore`: crea la carpeta de ensamblajes si no existe (ya la tiene si realizó la Semana 05).
> - `conda activate genome`: activa el entorno donde está instalado PRINSEQ.
> - `grep ">" archivo.fasta`: en un archivo FASTA, cada secuencia empieza con una línea que comienza con `>`; este comando muestra solo esas líneas, es decir, los nombres de los contigs.
> - `prinseq-lite.pl`: programa PRINSEQ en su versión de línea de comandos.
> - `-fasta`: archivo de entrada, el ensamblaje final (ensamblado con Flye y pulido con Racon).
> - `-min_len 500`: descarta los contigs de menos de 500 pb, que suelen ser artefactos del ensamblaje y no se aceptan en las bases de datos públicas.
> - `-seq_id "m01_00"`: **renombra los contigs**. PRINSEQ reemplaza el nombre de cada contig por el prefijo indicado seguido de un número correlativo: `m01_001`, `m01_002`, `m01_003`, etc., en el mismo orden del archivo original.
> - `-out_good m01_genome_final`: prefijo del archivo de salida con los contigs que pasan el filtro; se crea `m01_genome_final.fasta`.
> - `-out_bad null`: no guarda en un archivo los contigs descartados.
> - `conda deactivate`: cierra el entorno antes de activar otro.

> **Comentario (resultados):**
> - **Primer `grep`:** el ensamblaje original tiene 6 contigs, pero **dos de ellos tienen el mismo nombre** (`contig_bts_03` aparece dos veces). Los nombres repetidos hacen fallar a muchos programas (o, peor, mezclan los resultados de dos contigs distintos sin avisar); es un motivo más para renombrar los contigs antes de anotar.
> - **Resumen de PRINSEQ:** `Input sequences: 6` e `Input bases: 6,068,252` son los contigs y las bases del archivo de entrada (un genoma de 6,07 Mb); `Input mean length` es la longitud promedio de los contigs. `Good sequences: 6 (100.00%)` indica que los 6 contigs pasaron el filtro, y `Bad sequences: 0`, que ninguno fue descartado: ningún contig medía menos de 500 pb. Por eso `Sequences filtered by specified parameters` indica `none`.
> - **Segundo `grep`:** los 6 contigs tienen ahora nombres únicos y correlativos (`m01_001` a `m01_006`). Como el orden se conserva, `m01_001` es el primer contig del archivo original, `m01_002` el segundo, y así sucesivamente; los dos contigs que se llamaban `contig_bts_03` son ahora `m01_003` y `m01_004`.
> - **`m01_genome_final.fasta` es el genoma que se usa en el resto de la práctica.**

### Anotación del genoma con Bakta

La anotación **estructural** responde a la pregunta *¿dónde están los genes?* y la **funcional**, a *¿qué hacen?* Bakta realiza las dos en una sola ejecución.

```bash
cd ~/genomics/annotation/bakta

conda activate bakta

echo $BAKTA_DB

bakta --output m01_bakta --prefix m01 --locus-tag M01 --genus Vibrio --keep-contig-headers --threads 10 ~/genomics/assembly/nanopore/m01_genome_final.fasta

Parse genome sequences...
        imported: 6
        filtered & revised: 6
        contigs: 6

Start annotation...
predict tRNAs...
        found: 132
predict tmRNAs...
        found: 1
predict rRNAs...
        found: 37
predict ncRNAs...
        found: 29
predict ncRNA regions...
        found: 27
predict CRISPR arrays...
        found: 0
predict & annotate CDSs...
        predicted: 5521 
        discarded length: 0
        discarded spurious: 3
        revised translational exceptions: 0
        detected IPSs: 4410
        found PSCs: 786
        found PSCCs: 167
        lookup annotations...
        conduct expert systems...
                amrfinder: 3
                protein sequences: 40
        combine annotations and mark hypotheticals...
        detect pseudogenes...
                candidates: 84
                verified: 70
        analyze hypothetical proteins: 383
                detected Pfam hits: 10 
                calculated proteins statistics
        revise special cases...
detect & annotate sORF...
        detected: 84051
        discarded due to overlaps: 69602
        discarded spurious: 0
        detected IPSs: 1
        found PSCs: 3
        lookup annotations...
        filter and combine annotations...
        filtered sORFs: 0
detect gaps...
        found: 0
detect oriCs/oriVs...
        found: 3
detect oriTs...
        found: 0
apply feature overlap filters...
select features and create locus tags...
        selected: 5740
improve annotations...
        revised gene symbols: 19

Genome statistics:
        Genome size: 6,068,252 bp
        Contigs/replicons: 6
        GC: 45.5 %
        N50: 3,620,046
        N90: 2,216,041
        N ratio: 0.0 %
        coding density: 87.2 %

annotation summary:
        tRNAs: 132
        tmRNAs: 1
        rRNAs: 37
        ncRNAs: 29
        ncRNA regions: 26
        CRISPR arrays: 0
        CDSs: 5512
                hypotheticals: 378
                pseudogenes: 70
        sORFs: 0
        gaps: 0
        oriCs/oriVs: 3
        oriTs: 0

Export annotation results to: /home/alumno01/genomics/annotation/bakta/m01_bakta
        human readable TSV...
        GFF3...
        INSDC GenBank & EMBL...
        genome sequences...
        feature nucleotide sequences...
        translated CDS sequences...
        feature inferences...
        circular genome plot...
        hypothetical TSV...
        translated hypothetical CDS sequences...
        machine readable JSON...
        Genome and annotation summary...
```

> **Comentario (comandos):**
> - `conda activate bakta`: al activar el entorno se define automáticamente la variable `$BAKTA_DB`, que apunta a la base de datos completa de Bakta instalada en el servidor; por eso no hace falta indicar `--db`.
> - `echo $BAKTA_DB`: comprueba la ruta de la base de datos; debe mostrar `/data/db/bakta/db`.
> - `--output m01_bakta`: carpeta de salida.
> - `--prefix m01`: prefijo de todos los archivos de salida.
> - `--locus-tag M01`: prefijo de los identificadores de cada gen (`M01_00005`, `M01_00010`, ...). Debe tener entre 3 y 12 caracteres alfanuméricos en mayúscula y empezar con letra. **Con sus datos, use el código de su barcode** (por ejemplo, `B01`).
> - `--genus Vibrio`: género de la cepa, para los metadatos de la anotación. **Con sus datos, use el género que identificó en la Semana 05**; como ya conoce la especie por ANI, añada también `--species` (por ejemplo, `--genus Enterobacter --species cloacae`).
> - `--keep-contig-headers`: conserva los nombres de los contigs que asignó PRINSEQ (`m01_001`, `m01_002`, ...); sin esta opción Bakta los renombraría como `contig_1`, `contig_2`, ...
> - `--threads 10`: número de hilos.
> - `~/genomics/assembly/nanopore/m01_genome_final.fasta`: genoma de entrada.
> - Si todos los contigs de su ensamblaje son replicones circulares cerrados (columna `circ.` = `Y` en `assembly_info.txt` de Flye), puede añadir `--complete`.

> **Comentario (resultados):** Bakta informa en pantalla cada etapa a medida que avanza.
> - **`Parse genome sequences`:** leyó los 6 contigs (`imported`) y ninguno fue eliminado (`filtered & revised: 6`).
> - **ARN:** encontró 132 tRNA, 1 tmRNA (ARN que rescata ribosomas detenidos; las bacterias tienen uno), 37 rRNA y 29 ncRNA (ARN no codificantes reguladores), además de 27 regiones reguladoras (`ncRNA regions`, como riboswitches) y ningún sistema CRISPR. Los 37 rRNA y 132 tRNA son muchos para una bacteria, pero es lo característico del género *Vibrio*, que tiene numerosos operones ribosomales (lo que le permite crecer muy rápido).
> - **`predict & annotate CDSs`:** se predijeron 5 521 genes codificantes; 3 se descartaron por espurios. La función se busca en tres pasos, de mayor a menor confianza: `detected IPSs: 4410` son proteínas **idénticas** a una proteína ya conocida de la base de datos (*identical protein sequences*); `found PSCs: 786` son proteínas asignadas por similitud a un grupo de proteínas conocidas (*protein sequence clusters*, ≥ 90 % de identidad); y `found PSCCs: 167`, a un grupo más amplio (≥ 50 % de identidad). Es decir, el 80 % de las proteínas de esta cepa ya existía, idéntica, en las bases de datos.
> - **`conduct expert systems`:** anotaciones especializadas: `amrfinder: 3` indica que AMRFinderPlus reconoció 3 genes de resistencia a antimicrobianos (se retoman en la sección 10).
> - **`detect pseudogenes`:** de 84 candidatos, 70 se confirmaron como pseudogenes: genes interrumpidos por un codón de parada o un cambio del marco de lectura. Pueden ser reales o deberse a errores de secuenciación (inserciones/deleciones en homopolímeros, típicas de Nanopore).
> - **`analyze hypothetical proteins: 383`:** proteínas sin función asignada; en 10 de ellas se detectó al menos un dominio Pfam.
> - **`detect & annotate sORF`:** se evaluaron 84 051 marcos de lectura cortos; casi todos se descartan por solaparse con otros genes y ninguno quedó en la anotación final (`filtered sORFs: 0`).
> - **`detect oriCs/oriVs: 3`:** se detectaron 3 orígenes de replicación; **`detect oriTs: 0`:** ningún origen de transferencia.
> - **`select features`:** la anotación final tiene 5 740 elementos, cada uno con su locus tag.
> - **`Genome statistics`:** tamaño de 6 068 252 pb en 6 contigs, 45,5 % de GC, N50 de 3 620 046 pb (el contig mayor, el cromosoma 1) y N90 de 2 216 041 pb (el cromosoma 2). La densidad codificante es 87,2 %: ese porcentaje del genoma corresponde a genes (en bacterias suele estar entre 85 y 90 %).
> - **`annotation summary`:** resumen final: 5 512 CDS, de los cuales 378 son proteínas hipotéticas y 70 pseudogenes.
> - **`Export annotation results`:** lista de los archivos generados en la carpeta de salida.

```bash
ls -lh m01_bakta

total 83M
-rw-rw-r-- 1 alumno01 alumno01  16M oct  3 08:50 m01.embl
-rw-rw-r-- 1 alumno01 alumno01 1,9M oct  3 08:50 m01.faa
-rw-rw-r-- 1 alumno01 alumno01 5,3M oct  3 08:50 m01.ffn
-rw-rw-r-- 1 alumno01 alumno01 5,9M oct  3 08:50 m01.fna
-rw-rw-r-- 1 alumno01 alumno01  15M oct  3 08:50 m01.gbff
-rw-rw-r-- 1 alumno01 alumno01 7,6M oct  3 08:50 m01.gff3
-rw-rw-r-- 1 alumno01 alumno01  63K oct  3 08:50 m01.hypotheticals.faa
-rw-rw-r-- 1 alumno01 alumno01  49K oct  3 08:50 m01.hypotheticals.tsv
-rw-rw-r-- 1 alumno01 alumno01 482K oct  3 08:50 m01.inference.tsv
-rw-rw-r-- 1 alumno01 alumno01  24M oct  3 08:50 m01.json
-rw-rw-r-- 1 alumno01 alumno01 1,7M oct  3 08:50 m01.log
-rw-rw-r-- 1 alumno01 alumno01 1,6M oct  3 08:50 m01.png
-rw-rw-r-- 1 alumno01 alumno01 4,3M oct  3 08:50 m01.svg
-rw-rw-r-- 1 alumno01 alumno01 1,2M oct  3 08:50 m01.tsv
-rw-rw-r-- 1 alumno01 alumno01  395 oct  3 08:50 m01.txt

cat m01_bakta/m01.txt

Sequence(s):
Length: 6068252
Count: 6
GC: 45.5
N50: 3620046
N90: 2216041
N ratio: 0.0
coding density: 87.2

Annotation:
tRNAs: 132
tmRNAs: 1
rRNAs: 37
ncRNAs: 29
ncRNA regions: 26
CRISPR arrays: 0
CDSs: 5512
pseudogenes: 70
hypotheticals: 378
sORFs: 0
gaps: 0
oriCs: 3
oriVs: 0
oriTs: 0

Bakta:
Software: v1.12.1
Database: v6.0, full
DOI: 10.1099/mgen.0.000685
URL: github.com/oschwengers/bakta
```

> **Comentario (comandos):**
> - `ls -lh m01_bakta`: lista los archivos de la carpeta de salida, con sus permisos, propietario, tamaño en unidades legibles (`-h`: K, M) y fecha.
> - `cat m01_bakta/m01.txt`: muestra en pantalla el contenido del resumen de la anotación.

> **Comentario (resultados):** Bakta generó 15 archivos (83 MB en total):
> - `m01.txt`: resumen de la anotación.
> - `m01.tsv`: tabla con una fila por elemento anotado.
> - `m01.gff3`: anotación en formato GFF3.
> - `m01.gbff`: anotación en formato GenBank (secuencia + anotación); es la entrada de Proksee y antiSMASH.
> - `m01.embl`: anotación en formato EMBL.
> - `m01.fna`: secuencia del genoma (los contigs), en nucleótidos.
> - `m01.ffn`: secuencias nucleotídicas de cada gen.
> - `m01.faa`: secuencias de las proteínas (**el proteoma**); es la entrada de BUSCO, eggNOG-mapper, run_dbcan y OrthoVenn3.
> - `m01.hypotheticals.tsv` y `m01.hypotheticals.faa`: proteínas sin función asignada.
> - `m01.inference.tsv`: evidencia usada para asignar cada función.
> - `m01.json`: toda la información en formato legible por programas.
> - `m01.png` / `m01.svg`: mapa circular del genoma.
> - `m01.log`: registro de la ejecución.
>
> El resumen `m01.txt` tiene tres bloques:
> - **Sequence(s):** los mismos valores de `Genome statistics` (longitud, número de contigs, GC, N50, N90, proporción de bases indeterminadas y densidad codificante).
> - **Annotation:** el número de elementos de cada tipo. Los 3 orígenes de replicación son de tipo cromosómico (`oriCs: 3`, `oriVs: 0`).
> - **Bakta:** versión del programa (v1.12.1) y de la base de datos (v6.0, completa). Anótelas para la bitácora.

### Explorar la tabla de anotación

```bash
head -n 20 m01_bakta/m01.tsv

# Annotated with Bakta
# Software: v1.12.1
# Database: v6.0, full
# DOI: 10.1099/mgen.0.000685
# URL: github.com/oschwengers/bakta
#Sequence Id    Type    Start   Stop    Strand  Locus Tag       Gene    Product DbXrefs
m01_001 rRNA    1       110     -       M01_00001       rrf     (3' truncated) 5S ribosomal RNA GO:0003735, GO:0005840, KEGG:K01985, RFAM:RF00001, SO:0000652
m01_001 rRNA    199     3087    -       M01_00002       rrl     23S ribosomal RNA       GO:0003735, GO:0005840, KEGG:K01980, RFAM:RF02541, SO:0001001
m01_001 cds     3181    3336    -       M01_00003               (pseudo) hypothetical protein   SO:0001217, UniRef:UniRef50_A7MTN2, UniRef:UniRef90_A7MTN2
m01_001 tRNA    3369    3444    -       M01_00004       trnV    tRNA-Val(tac)   SO:0000273
m01_001 tRNA    3482    3557    -       M01_00005       trnA    tRNA-Ala(tgc)   SO:0000254
m01_001 tRNA    3567    3642    -       M01_00006       trnK    tRNA-Lys(ttt)   SO:0000265
m01_001 tRNA    3645    3720    -       M01_00007       trnE    tRNA-Glu(ttc)   SO:0000259
m01_001 rRNA    3812    5365    -       M01_00008       rrs     16S ribosomal RNA       GO:0003735, GO:0005840, KEGG:K01977, RFAM:RF00177, SO:0001000
m01_001 cds     5816    6343    -       M01_00009       hemG    menaquinone-dependent protoporphyrinogen IX dehydrogenase       EC:1.3.5.3, KEGG:K00230, RefSeq:WP_086368835.1, SO:0001217, UniParc:UPI000A2FBE66, UniRef:UniRef100_UPI000A2FBE66, UniRef:UniRef50_A0A377IXM2, UniRef:UniRef90_A0A0H0YCE0
m01_001 cds     6353    7705    -       M01_00010       trkH    Trk system potassium uptake protein TrkH        COG:COG0168, COG:P, GO:0005267, GO:0005886, GO:0015379, GO:0030955, GO:0042802, GO:0042803, GO:0071805, SO:0001217, UniRef:UniRef50_Q87TN7, UniRef:UniRef90_Q87TN7
m01_001 cds     7858    8481    -       M01_00011       yigZ    YigZ family protein     RefSeq:WP_050909549.1, SO:0001217, UniParc:UPI0006836678, UniRef:UniRef100_UPI0006836678, UniRef:UniRef50_U3B6M3, UniRef:UniRef90_A7N1D3
m01_001 cds     8750    10921   +       M01_00012       fadB    fatty acid oxidation complex subunit alpha FadB COG:COG1250, COG:I, GO:0003857, GO:0004165, GO:0004300, GO:0006635, GO:0008692, GO:0009062, GO:0016509, GO:0036125, GO:0070403, RefSeq:WP_050909548.1, SO:0001217, UniParc:UPI0006817788, UniRef:UniRef100_UPI0006817788, UniRef:UniRef50_Q87TN9, UniRef:UniRef90_Q87TN9
m01_001 cds     10936   12111   +       M01_00013       fadA    acetyl-CoA C-acyltransferase FadA       EC:2.3.1.16, GO:0003988, GO:0005737, GO:0006631, GO:0016042, RefSeq:WP_005426042.1, SO:0001217, UniParc:UPI0001544D72, UniRef:UniRef100_A0A1Q1PE66, UniRef:UniRef50_P28790, UniRef:UniRef90_F9T299
m01_001 cds     12348   13328   +       M01_00014       qor     Acryloyl-CoA reductase  COG:COG0604, COG:CR, RefSeq:WP_009708064.1, SO:0001217, UniParc:UPI00028C1383, UniRef:UniRef100_A0AAP9K8I8, UniRef:UniRef50_A0A6P1I601, UniRef:UniRef90_A0AAP9K8I8

grep -v "^#" m01_bakta/m01.tsv | cut -f 2 | sort | uniq -c

   5512 cds
     29 ncRNA
     26 ncRNA-region
      3 oriC
     37 rRNA
      1 tmRNA
    132 tRNA

grep -v "^#" m01_bakta/m01.tsv | awk -F'\t' '$2=="cds"' | cut -f 1 | sort | uniq -c

   3228 m01_001
   2023 m01_002
     87 m01_003
     82 m01_004
     64 m01_005
     28 m01_006

grep -c "hypothetical protein" m01_bakta/m01.tsv

378

grep -w "16S ribosomal RNA" m01_bakta/m01.tsv

m01_001 rRNA    3812    5365    -       M01_00008       rrs     16S ribosomal RNA       GO:0003735, GO:0005840, KEGG:K01977, RFAM:RF00177, SO:0001000
m01_001 rRNA    121855  123407  +       M01_00123       rrs     16S ribosomal RNA       GO:0003735, GO:0005840, KEGG:K01977, RFAM:RF00177, SO:0001000
m01_001 rRNA    167290  168842  +       M01_00166       rrs     16S ribosomal RNA       GO:0003735, GO:0005840, KEGG:K01977, RFAM:RF00177, SO:0001000
m01_001 rRNA    227237  228789  +       M01_00227       rrs     16S ribosomal RNA       GO:0003735, GO:0005840, KEGG:K01977, RFAM:RF00177, SO:0001000
m01_001 rRNA    232667  234219  +       M01_00231       rrs     16S ribosomal RNA       GO:0003735, GO:0005840, KEGG:K01977, RFAM:RF00177, SO:0001000
m01_001 rRNA    292090  293642  +       M01_00287       rrs     16S ribosomal RNA       GO:0003735, GO:0005840, KEGG:K01977, RFAM:RF00177, SO:0001000
m01_001 rRNA    297413  298965  +       M01_00290       rrs     16S ribosomal RNA       GO:0003735, GO:0005840, KEGG:K01977, RFAM:RF00177, SO:0001000
m01_001 rRNA    478717  480269  +       M01_00475       rrs     16S ribosomal RNA       GO:0003735, GO:0005840, KEGG:K01977, RFAM:RF00177, SO:0001000
m01_001 rRNA    483972  485524  +       M01_00478       rrs     16S ribosomal RNA       GO:0003735, GO:0005840, KEGG:K01977, RFAM:RF00177, SO:0001000
m01_001 rRNA    606805  608357  +       M01_00590       rrs     16S ribosomal RNA       GO:0003735, GO:0005840, KEGG:K01977, RFAM:RF00177, SO:0001000
m01_001 rRNA    3059895 3061447 -       M01_02878       rrs     16S ribosomal RNA       GO:0003735, GO:0005840, KEGG:K01977, RFAM:RF00177, SO:0001000
m01_002 rRNA    1299836 1301388 -       M01_04603       rrs     16S ribosomal RNA       GO:0003735, GO:0005840, KEGG:K01977, RFAM:RF00177, SO:0001000

grep -c ">" m01_bakta/m01.faa

5512
```

> **Comentario (comandos de procesamiento de la tabla):**
> - `head -n 20 archivo`: muestra las primeras 20 líneas del archivo.
> - `grep -v "^#"`: `grep` busca líneas que contengan un patrón; `-v` invierte la búsqueda (muestra las que **no** lo contienen) y `^#` significa "línea que empieza con `#`". El resultado es la tabla sin las líneas de comentario.
> - `|` (tubería): entrega la salida de un comando como entrada del siguiente.
> - `cut -f 2`: extrae la columna 2 de una tabla separada por tabulaciones (aquí, el tipo de elemento); `cut -f 1` extrae la columna 1 (el contig).
> - `sort | uniq -c`: `sort` ordena las líneas y `uniq -c` cuenta cuántas veces se repite cada una. Juntos producen una tabla de frecuencias.
> - `awk -F'\t' '$2=="cds"'`: `awk` procesa la tabla fila por fila; `-F'\t'` indica que las columnas están separadas por tabulaciones y `$2=="cds"` conserva solo las filas cuya columna 2 es exactamente `cds`.
> - `grep -c "patrón" archivo`: en lugar de mostrar las líneas, las **cuenta** (`-c`).
> - `grep -w "16S ribosomal RNA"`: `-w` exige que el patrón coincida como palabras completas.
> - `grep -c ">" m01_bakta/m01.faa`: cuenta las líneas con `>`, es decir, el número de proteínas del archivo FASTA.

> **Comentario (resultados):**
> - **`head`:** las primeras 5 líneas (con `# `) son comentarios con la versión de Bakta; la sexta es el encabezado. Columnas:
>   - `Sequence Id`: contig donde está el elemento (`m01_001`, ...).
>   - `Type`: tipo de elemento (`cds`, `tRNA`, `rRNA`, `tmRNA`, `ncRNA`, `ncRNA-region`, `crispr`, `sorf`, `oriC`, `oriV`, `oriT`, `gap`).
>   - `Start` y `Stop`: coordenadas de inicio y fin en el contig.
>   - `Strand`: hebra (`+` directa, `-` reversa).
>   - `Locus Tag`: identificador único del elemento (`M01_00001`, ...), asignado en orden a lo largo del genoma.
>   - `Gene`: nombre del gen, si se conoce (`hemG`, `fadB`); queda vacío si no tiene nombre.
>   - `Product`: producto o función.
>   - `DbXrefs`: referencias cruzadas a otras bases de datos: UniRef/UniParc/RefSeq (proteína de referencia), `COG` (grupo de ortólogos y, con una letra, su categoría funcional), `GO` (ontología génica), `EC` (número de enzima), `KEGG` (ortólogo KO), `RFAM` (familia de ARN) y `SO` (tipo de secuencia).
> - **Lectura de las primeras filas:** el contig `m01_001` empieza con un operón ribosomal en la hebra reversa: 5S (`rrf`), 23S (`rrl`), cuatro tRNA y 16S (`rrs`). El 5S aparece como `(3' truncated)` porque el contig comienza en medio del gen: en un cromosoma circular, el punto donde "se corta" la secuencia es arbitrario. `M01_00003` es un pseudogén hipotético (`(pseudo)`). `M01_00010` (`trkH`, transportador de potasio) tiene asignado el grupo `COG0168` y la categoría `P` (transporte de iones inorgánicos); `M01_00014` tiene dos categorías (`CR`).
> - **Elementos por tipo:** 5 512 CDS, 132 tRNA, 37 rRNA, 29 ncRNA, 26 regiones reguladoras, 3 orígenes de replicación y 1 tmRNA; coinciden con el resumen `m01.txt`.
> - **CDS por contig:** `m01_001` (3 228 genes) y `m01_002` (2 023) son los dos cromosomas; juntos contienen el 95 % de los genes. Los contigs `m01_003` a `m01_006` tienen entre 28 y 87 genes cada uno: son replicones pequeños, candidatos a plásmidos (se confirmará en la sección 11).
> - **Proteínas hipotéticas:** 378 de 5 512 CDS, es decir, 6,9 %. Es un valor bajo, propio de un género muy estudiado; en bacterias ambientales poco estudiadas puede superar el 20 o 30 %.
> - **Genes 16S:** hay **12 copias**: 11 en el cromosoma 1 (`m01_001`) y 1 en el cromosoma 2 (`m01_002`). Todas miden lo mismo (1 553 o 1 554 pb), lo esperado para un 16S completo.
> - **Proteoma:** 5 512 proteínas, el mismo número que los CDS.

> **Punto de control:** El genoma mide 6,07 Mb y tiene 5 512 CDS: aproximadamente un gen por cada 1 100 pb, coherente con la regla general de un gen por cada 1 000 pb en bacterias. Con su propio genoma, calcule el porcentaje de proteínas hipotéticas (hipotéticas / CDS × 100) y verifique que el número de copias del gen 16S coincida con el que encontró Barrnap en la Semana 05.

## 3. Envío de los análisis en línea

Los servidores web trabajan con colas de espera, por lo que conviene **enviar los trabajos apenas tenga la anotación de Bakta** y continuar con las secciones 4 y 5 mientras se procesan.

### Descargar con WinSCP los archivos `m01.faa` y `m01.gbff` de `~/genomics/annotation/bakta/m01_bakta/`

### eggNOG-mapper: ir a https://eggnog-mapper.cgmlab.org/, entrar a *Annotate*, cargar el archivo `m01.faa` como proteínas, colocar su correo y enviar el trabajo

<img width="1692" height="1488" alt="image" src="https://github.com/user-attachments/assets/b3133113-da4f-4714-8636-f315a2a63a88" />

**Figura 5.** Formulario de envío de eggNOG-mapper (*Annotate*). Se carga el proteoma `m01.faa`, se indica que las secuencias son proteínas y se coloca el correo al que llegará el enlace de resultados.

<img width="1620" height="1290" alt="image" src="https://github.com/user-attachments/assets/86bd4208-e4ff-4a1e-acd5-e8ba2aeaba42" />

**Figura 6.** Página de seguimiento del trabajo enviado a eggNOG-mapper. Muestra el identificador del trabajo y su estado (en cola, en ejecución o terminado); guarde el enlace para volver a consultarlo.

> **Comentario:**
> - El servidor permite **un solo trabajo en ejecución por correo electrónico**: envíe un único trabajo por grupo y no lo reenvíe si demora.
> - El tiempo depende de la cola (puede ir de varios minutos a más de una hora). La página principal muestra en vivo los trabajos en ejecución y en espera.
> - Los resultados se conservan 3 meses en el servidor.
> - Deje los parámetros por defecto: el ámbito taxonómico se ajusta automáticamente para cada proteína.

### antiSMASH: ir a https://antismash.secondarymetabolites.org/, cargar el archivo `m01.gbff`, colocar su correo y enviar el trabajo

<img width="3024" height="1437" alt="image" src="https://github.com/user-attachments/assets/682dc1a5-ac17-4e1a-8191-358f13b4a89c" />

**Figura 7.** Formulario de envío de antiSMASH (versión bacteriana). Se carga el archivo GenBank `m01.gbff`, se coloca el correo, y se dejan el nivel de detección (*relaxed*) y las opciones adicionales por defecto.

> **Comentario:** Al cargar el GenBank de Bakta, antiSMASH usa los genes ya anotados y los identificadores (`M01_...`) coinciden con los del resto de la práctica. Si se cargara solo el FASTA, antiSMASH tendría que predecir los genes por su cuenta y los identificadores serían distintos.

## 4. Evaluación del proteoma con BUSCO

En la Semana 05 se usó BUSCO en modo `genome` para evaluar el ensamblaje (BUSCO predice los genes por su cuenta). Ahora se evalúa además el **proteoma anotado por Bakta** (modo `proteins`): si la anotación es buena, ambos resultados deben ser muy similares.

```bash
cd ~/genomics/annotation/busco

conda activate busco

busco --list-datasets | grep -i -B 3 vibrio

    - desulfuromonadales_odb12.2 [626]
    - geobacteraceae_odb12.2 [783]
        - geobacter_odb12.2 [1043]
    - desulfovibrionales_odb12.2 [729]
--
                - eubacteriaceae_odb12.2 [377]
                - peptostreptococcaceae_odb12.2 [416]
                - lachnospiraceae_odb12.2 [444]
                    - butyrivibrio_odb12.2 [1031]
--
                - psychrobacter_odb12.2 [1408]
                - moraxella_odb12.2 [1053]
                - acinetobacter_odb12.2 [1486]
            - cellvibrionales_odb12.2 [782]
                - cellvibrionaceae_odb12.2 [1038]
            - pasteurellales_odb12.2 [1063]
            - aeromonadaceae_odb12.2 [1232]
                - aeromonas_odb12.2 [2492]
            - vibrionales_odb12.2 [1095]
                - vibrio_odb12.2 [1570]
--
                - rhodanobacteraceae_odb12.2 [1041]
            - chromatiales_odb12.2 [516]
                - ectothiorhodospiraceae_odb12.2 [636]
                    - thioalkalivibrio_odb12.2 [1198]
```

> **Comentario (comandos):**
> - `busco --list-datasets`: muestra el árbol de linajes disponibles (visto en la Semana 05).
> - `grep -i -B 3 vibrio`: busca la palabra en esa lista, sin distinguir mayúsculas de minúsculas (`-i`), y muestra también las 3 líneas anteriores a cada coincidencia (`-B 3`, de *before*), para ver el orden y la familia a los que pertenece. Los bloques de resultados se separan con `--`.

> **Comentario (resultados):**
> - `grep` encuentra todas las líneas que **contienen** el texto "vibrio", no solo el género *Vibrio*: por eso aparecen también `desulfovibrionales`, `butyrivibrio`, `cellvibrionales` y `thioalkalivibrio`, que son grupos bacterianos distintos.
> - El linaje que interesa está en el tercer bloque: `vibrionales_odb12.2 [1095]` (el orden) y, más sangrado, **`vibrio_odb12.2 [1570]`** (el género). La sangría indica la jerarquía y el número entre corchetes, la cantidad de genes marcadores: el linaje de género tiene más marcadores (1 570) que el de orden (1 095), por lo que la evaluación es más exigente y precisa.
> - Use siempre el linaje más específico disponible para su cepa; si no existe un linaje de género, use el de familia u orden.

```bash
busco -i ~/genomics/assembly/nanopore/m01_genome_final.fasta -l vibrio_odb12.2 -o m01_genome -m genome -c 10

busco -i ~/genomics/annotation/bakta/m01_bakta/m01.faa -l vibrio_odb12.2 -o m01_proteins -m proteins -c 10
```

> **Comentario (comandos):**
> - `-i`: archivo de entrada: en el primer comando el genoma (`.fasta`) y en el segundo el proteoma (`.faa`).
> - `-l vibrio_odb12.2`: linaje de referencia, el mismo en los dos comandos.
> - `-o`: nombre de la carpeta de salida (`m01_genome` y `m01_proteins`).
> - `-m genome` / `-m proteins`: modo de análisis. En modo `genome`, BUSCO predice primero los genes; en modo `proteins` evalúa directamente las proteínas que se le entregan.
> - `-c 10`: número de hilos.
> - Si necesita repetir un comando, añada `-f` para sobrescribir la carpeta de salida.
> - Con su propio genoma no necesita repetir el modo `genome`: ya lo ejecutó en la Semana 05 (carpeta `~/genomics/validation/busco/`).

```bash
cat m01_genome/short_summary.specific.vibrio_odb12.2.m01_genome.txt

# BUSCO version is: 6.1.0 
# The lineage dataset is: vibrio_odb12.2 (Creation date: 2026-05-22, number of genomes: 122, number of BUSCOs: 1570)
# Summarized benchmarking in BUSCO notation for file /home/alumno01/genomics/assembly/nanopore/m01_genome_final.fasta
# BUSCO was run in mode: prok_genome_prod
# Gene predictor used: prodigal

        ***** Results: *****

        C:97.6%[S:97.5%,D:0.2%],F:1.7%,M:0.6%,n:1570       
        1533    Complete BUSCOs (C)                        
        1530    Complete and single-copy BUSCOs (S)        
        3       Complete and duplicated BUSCOs (D)         
        27      Fragmented BUSCOs (F)                      
        10      Missing BUSCOs (M)                         
        1570    Total BUSCO groups searched                

Assembly Statistics:
        6       Number of scaffolds
        6       Number of contigs
        6068252 Total length
        0.000%  Percent gaps
        3 Mbp   Scaffold N50
        3 Mbp   Contigs N50


cat m01_proteins/short_summary.specific.vibrio_odb12.2.m01_proteins.txt

# BUSCO version is: 6.1.0 
# The lineage dataset is: vibrio_odb12.2 (Creation date: 2026-05-22, number of genomes: 122, number of BUSCOs: 1570)
# Summarized benchmarking in BUSCO notation for file /home/alumno01/genomics/annotation/bakta/m01_bakta/m01.faa
# BUSCO was run in mode: proteins

        ***** Results: *****

        C:97.6%[S:97.5%,D:0.2%],F:1.7%,M:0.6%,n:1570       
        1533    Complete BUSCOs (C)                        
        1530    Complete and single-copy BUSCOs (S)        
        3       Complete and duplicated BUSCOs (D)         
        27      Fragmented BUSCOs (F)                      
        10      Missing BUSCOs (M)                         
        1570    Total BUSCO groups searched  
```

> **Comentario (resultados):**
> - **Encabezado:** versión de BUSCO (6.1.0), linaje usado (`vibrio_odb12.2`, construido con 122 genomas del género y 1 570 genes marcadores), archivo evaluado y modo. En el modo `genome` se indica además que los genes se predijeron con Prodigal (`prok_genome_prod`).
> - **Línea resumen** `C:97.6%[S:97.5%,D:0.2%],F:1.7%,M:0.6%,n:1570`:
>   - `C` (Complete): 97,6 % de los marcadores se encontró completo (1 533 de 1 570).
>   - `S` (Single-copy): 1 530 están en una sola copia.
>   - `D` (Duplicated): 3 están duplicados (0,2 %). Un valor tan bajo indica que no hay contaminación ni regiones ensambladas dos veces.
>   - `F` (Fragmented): 27 se encontraron solo parcialmente (1,7 %).
>   - `M` (Missing): 10 no se encontraron (0,6 %).
>   - `n`: número total de marcadores del linaje.
> - **Assembly Statistics** (solo en modo `genome`): 6 contigs, 6 068 252 pb, sin huecos (`Percent gaps: 0.000%`) y N50 de 3 Mb. En el modo `proteins` este bloque no aparece, porque la entrada no es un genoma.
> - **Comparación:** los dos modos dan **exactamente el mismo resultado** (97,6 % completo). Es lo esperado: la anotación de Bakta recuperó todos los genes marcadores que el ensamblaje contiene, es decir, no se perdió ningún gen en la anotación.
> - Con 97,6 % de completitud y 0,2 % de duplicación, el genoma se considera de alta calidad. Los 27 marcadores fragmentados probablemente corresponden a genes interrumpidos (recuerde los 70 pseudogenes detectados por Bakta), que pueden deberse a errores de inserción/deleción en homopolímeros, típicos de las lecturas Nanopore.

### Comparar el modo genome con el modo proteins

```bash
mkdir -p m01_busco_summaries

cp m01_genome/short_summary.*.json m01_proteins/short_summary.*.json m01_busco_summaries/

busco --plot m01_busco_summaries

conda deactivate
```

> **Comentario (comandos):**
> - `mkdir -p m01_busco_summaries`: crea una carpeta para reunir los resúmenes.
> - `cp ... m01_busco_summaries/`: copia a esa carpeta el resumen en formato JSON de cada análisis; el asterisco (`*`) reemplaza cualquier texto, de modo que `short_summary.*.json` coincide con el archivo de resumen sin tener que escribir su nombre completo. Con su propio genoma, el resumen del modo `genome` está en `~/genomics/validation/busco/<ensamblaje>/`.
> - `busco --plot m01_busco_summaries`: genera `m01_busco_summaries/busco_figure.png` (y el script de R con el que se creó).

<img width="3000" height="1800" alt="image" src="https://github.com/user-attachments/assets/8d524257-24de-4269-b074-3d50a5d506f9" />

**Figura 8.** Gráfico comparativo de BUSCO para el genoma `m01` en modo `genome` (`m01_genome`) y en modo `proteins` (`m01_proteins`), con el linaje `vibrio_odb12.2`.

> **Comentario (figura):** Cada barra horizontal representa un análisis y está dividida según el porcentaje de marcadores en cada categoría: completos y de copia única (S, celeste), completos y duplicados (D, azul oscuro), fragmentados (F, amarillo) y ausentes (M, rojo). Sobre cada barra se repite la línea resumen. Las dos barras son idénticas, casi completamente celestes, con una franja mínima amarilla y roja al final: la anotación conserva toda la información del ensamblaje.

> **Punto de control:** El % `C` del modo `proteins` debería ser muy similar al del modo `genome`. Si fuera claramente menor, la anotación estaría perdiendo genes que sí están en el ensamblaje.

## 5. Visualización del genoma anotado

### Descargar con WinSCP el mapa circular generado por Bakta (`m01_bakta/m01.png`)

<img width="2603" height="2597" alt="image" src="https://github.com/user-attachments/assets/eb06eee6-7272-457f-8eb0-5db7641c9e7b" />

**Figura 9.** Mapa circular del genoma `m01` generado por Bakta (`m01.png`).

> **Comentario (figura):**
> - Los 6 contigs se dibujan uno a continuación del otro, como segmentos de un mismo círculo: los dos segmentos grandes son los cromosomas (`m01_001`, de 3,6 Mb, y `m01_002`, de 2,2 Mb) y los cuatro segmentos pequeños del final, los contigs `m01_003` a `m01_006`.
> - De afuera hacia adentro: genes de la hebra directa, genes de la hebra reversa (los CDS se colorean según su categoría funcional COG; los ARN tienen un color propio), contenido GC (desviación respecto al promedio del genoma) y sesgo GC.
> - Los genes cubren casi todo el círculo, sin grandes espacios vacíos: es la densidad codificante de 87,2 %.

### Exportar el archivo `m01.gbff` generado por Bakta e ir al programa Proksee (https://proksee.ca/)

### Hacer clic en Browse, seleccionar el archivo `m01.gbff`, esperar que el archivo se cargue, y hacer clic en Create Map

<img width="566" alt="image" src="https://github.com/user-attachments/assets/8bd5970a-5b02-442b-b058-f488bee5d354" />

**Figura 10.** Página de inicio de Proksee. Con *Browse* se selecciona el archivo GenBank y con *Create Map* se genera el mapa.

### Realizar zoom en el mapa utilizando la rueda del ratón y localizar los tRNA/rRNA

<img width="698" alt="image" src="https://github.com/user-attachments/assets/64f4d3aa-2636-4c72-917f-f85ead755c40" />

**Figura 11.** Mapa circular del genoma en Proksee, con los genes de la hebra directa (anillo externo) y de la hebra reversa (anillo interno). La leyenda indica el color de cada tipo de elemento (CDS, tRNA, rRNA).

### Hacer clic en la herramienta GC Content

<img width="198" alt="image" src="https://github.com/user-attachments/assets/4610c60c-8ac8-49b7-bf18-6c91766efa73" />

**Figura 12.** Herramienta *GC Content* en el panel *Tools* de Proksee.

### Hacer clic en OK

<img width="698" alt="image" src="https://github.com/user-attachments/assets/89839d38-5cf3-493c-a690-6448bbb6d628" />

**Figura 13.** Ventana de opciones de *GC Content* (tamaño de la ventana de cálculo y paso); se dejan los valores por defecto.

### Visualizar el contenido GC en el mapa del genoma

<img width="698" alt="image" src="https://github.com/user-attachments/assets/8e603f47-ff06-46df-9fbc-73be63f7f158" />

**Figura 14.** Mapa del genoma con la pista de contenido GC añadida como un anillo interno.

### Hacer clic en la herramienta GC Skew y visualizar el sesgo de GC en el mapa del genoma

<img width="698" alt="image" src="https://github.com/user-attachments/assets/48a7edcb-f22a-4303-b77d-df8eed927af7" />

**Figura 15.** Mapa del genoma con las pistas de contenido GC y de sesgo GC (*GC skew*); el sesgo positivo y el negativo se dibujan con colores distintos.

> **Comentario (figuras 10 a 15):**
> - Cada contig (cromosomas y plásmidos) se muestra como un segmento del círculo, separado de los demás por una marca. Con la rueda del ratón puede acercarse hasta ver cada gen con su nombre.
> - Los rRNA y tRNA se muestran con un color distinto al de los CDS. Búsquelos: están agrupados en operones (5S-23S-tRNA-16S, como el que vio al inicio de `m01.tsv`) y la mayoría se concentra en una región del cromosoma 1 (recuerde que 11 de las 12 copias del 16S están en `m01_001`).
> - **Contenido GC:** desviación del porcentaje de GC respecto al promedio del genoma (45,5 %). Los picos hacia afuera son regiones más ricas en GC que el promedio y los picos hacia adentro, más pobres. Las regiones con un GC muy distinto al promedio suelen ser ADN adquirido por transferencia horizontal (islas genómicas, profagos, plásmidos integrados).
> - **Sesgo GC (*GC skew*):** (G − C)/(G + C) calculado en ventanas. Cambia de signo en el origen y en el término de replicación, por lo que divide cada cromosoma circular en dos mitades de distinto color.
> - En el menú *Tools* puede añadir más pistas (por ejemplo, mobileOG-db para elementos móviles o CARD para genes de resistencia) y, en *Download*, exportar la figura para la bitácora.

## 6. Anotación funcional (COG y KEGG)

### Descargar el archivo de anotaciones del resultado de eggNOG-mapper, subirlo con WinSCP a `~/genomics/annotation/eggnog/` y renombrarlo como `m01.emapper.annotations`

<img width="1423" height="1010" alt="image" src="https://github.com/user-attachments/assets/d7ebcd80-35c6-4f71-ab11-48173fd74c7b" />

**Figura 16.** Página de resultados de eggNOG-mapper, con el resumen del trabajo y los enlaces de descarga. El archivo que se necesita es el de anotaciones (`annotations`).

### Preparar la tabla

```bash
cd ~/genomics/annotation/eggnog

grep -v "^##" m01.emapper.annotations > m01_eggnog.tsv

head -n 1 m01_eggnog.tsv | tr '\t' '\n' | cat -n

     1  #query
     2  seed_ortholog
     3  evalue
     4  score
     5  eggNOG_OGs
     6  tax_ceiling
     7  farthest_donor_lineage
     8  COG_category
     9  Preferred_name
    10  GOs
    11  EC
    12  KEGG_ko
    13  KEGG_Pathway
    14  KEGG_Module
    15  KEGG_Reaction
    16  KEGG_rclass
    17  BRITE
    18  KEGG_TC
    19  CAZy
    20  BiGG_Reaction
    21  PFAMs
    22  annotation_confidence
```

> **Comentario (comandos):**
> - `grep -v "^##" archivo > m01_eggnog.tsv`: elimina las líneas que empiezan con `##` (comentarios al inicio y al final del archivo) y guarda el resultado (`>`) en un archivo nuevo: una tabla con una línea de encabezado y una fila por proteína anotada.
> - `head -n 1`: toma solo la primera línea (el encabezado).
> - `tr '\t' '\n'`: reemplaza cada tabulación por un salto de línea, de modo que cada nombre de columna queda en una línea.
> - `cat -n`: numera las líneas. El resultado es la lista de columnas con su número.

> **Comentario (resultados):** La tabla tiene 22 columnas. Las más importantes:
> - `#query` (1): identificador de la proteína (el locus tag de Bakta, `M01_...`).
> - `seed_ortholog`, `evalue`, `score` (2 a 4): el ortólogo de referencia más parecido y la significancia del alineamiento.
> - `eggNOG_OGs` (5): grupos de ortólogos a los que pertenece la proteína, en cada nivel taxonómico.
> - `tax_ceiling` y `farthest_donor_lineage` (6 y 7): alcance taxonómico de los ortólogos desde los que se transfirió la anotación.
> - `COG_category` (8): grupo COG asignado a la proteína.
> - `Preferred_name` (9): nombre del gen (por ejemplo, `nifH`).
> - `GOs` (10), `EC` (11): términos de ontología génica y número de enzima.
> - `KEGG_ko` (12): ortólogo de KEGG (KO), por ejemplo `ko:K02588`; las columnas 13 a 18 dan las rutas, módulos, reacciones y transportadores de KEGG.
> - `CAZy` (19), `PFAMs` (21): familia de CAZymes y dominios proteicos.
> - `annotation_confidence` (22): nivel de confianza de la anotación.
> - Los comandos siguientes buscan las columnas **por su nombre**, no por su posición. Si en su archivo algún nombre fuera distinto, cámbielo dentro del comando.

### Porcentaje de proteínas anotadas

```bash
grep -c ">" ~/genomics/annotation/bakta/m01_bakta/m01.faa

5512

tail -n +2 m01_eggnog.tsv | wc -l

5278
```

> **Comentario (comandos):**
> - `grep -c ">"`: cuenta las proteínas del proteoma.
> - `tail -n +2`: muestra el archivo desde la línea 2 en adelante, es decir, sin el encabezado.
> - `wc -l`: cuenta líneas. El resultado es el número de proteínas que recibieron alguna anotación en eggNOG-mapper.

> **Comentario (resultados):** De las 5 512 proteínas, 5 278 recibieron anotación: 5 278 / 5 512 × 100 = **95,8 %**. Las 234 restantes no tienen ortólogos reconocibles en eggNOG; suelen ser proteínas muy cortas o exclusivas de la cepa o de la especie.

### Distribución de categorías COG

Las categorías COG clasifican cada proteína en una de unas 25 grandes funciones celulares, identificadas con una letra. En la versión actual del servidor de eggNOG-mapper, la columna `COG_category` contiene en la mayoría de las proteínas el identificador del grupo (por ejemplo, `COG0168`) y no la letra de la categoría. La letra está en la anotación de Bakta, en la columna `DbXrefs` de `m01.tsv` (por ejemplo, `COG:COG0168, COG:P`), y de ahí se obtiene la distribución:

```bash
grep -v "^#" ~/genomics/annotation/bakta/m01_bakta/m01.tsv | awk -F'\t' '$2=="cds" && $9 ~ /COG:COG[0-9]+/' | wc -l

1522

grep -v "^#" ~/genomics/annotation/bakta/m01_bakta/m01.tsv | awk -F'\t' '$2=="cds"{n=split($9,a,", "); for(j=1;j<=n;j++) if(a[j] ~ /^COG:[A-Z]+$/){sub("COG:","",a[j]); m=split(a[j],b,""); for(k=1;k<=m;k++) print b[k]}}' | sort | uniq -c | sort -k1,1nr > m01_cog_counts.txt

cat m01_cog_counts.txt

    186 E
    184 J
    130 K
    114 C
    108 G
    104 T
    101 H
     96 P
     82 R
     81 M
     79 O
     72 L
     58 U
     57 F
     50 I
     49 N
     43 V
     41 S
     28 D
     22 Q
      4 B
      4 W
      4 X
      2 Z
```

<!-- Pegar aquí la salida real de los comandos con el genoma m01 -->

> **Comentario (comandos de procesamiento de la tabla):**
> - **Primer comando:** cuenta cuántos CDS tienen un grupo COG asignado. `grep -v "^#"` quita los comentarios; `awk` conserva las filas de tipo `cds` (`$2=="cds"`) cuya columna 9 (`DbXrefs`) contiene el texto `COG:COG` seguido de números (`$9 ~ /COG:COG[0-9]+/`; el operador `~` significa "contiene el patrón"); `wc -l` cuenta las filas.
> - **Segundo comando**, paso a paso:
>   - `$2=="cds"{...}`: para cada CDS, ejecuta lo que está entre llaves.
>   - `n=split($9,a,", ")`: divide la columna `DbXrefs` en sus referencias (separadas por coma y espacio) y las guarda en la lista `a`.
>   - `for(j=1;j<=n;j++)`: recorre las referencias una por una.
>   - `if(a[j] ~ /^COG:[A-Z]+$/)`: conserva solo las referencias formadas por `COG:` seguido únicamente de letras mayúsculas (`COG:P`, `COG:CR`); así se excluyen los identificadores (`COG:COG0168`), que tienen números.
>   - `sub("COG:","",a[j])`: borra el prefijo `COG:` y deja solo las letras.
>   - `m=split(a[j],b,"")` y el segundo `for`: una proteína puede tener más de una categoría (por ejemplo, `CR`); se separa en letras y se imprime cada una en una línea, de modo que la proteína se cuenta en cada categoría.
>   - `sort | uniq -c`: cuenta cuántas veces aparece cada letra.
>   - `sort -k1,1nr`: ordena por la primera columna (`-k1,1`), numéricamente (`n`) y de mayor a menor (`r`).
>   - `> m01_cog_counts.txt`: guarda el resultado en un archivo.
> - `cat m01_cog_counts.txt`: muestra la tabla: número de proteínas y letra de la categoría.

> **Comentario (resultados):** Para leer la tabla use el significado de cada letra:
>
> | Grupo | Letra | Función |
> |---|---|---|
> | Almacenamiento y procesamiento de la información | J | Traducción, estructura y biogénesis del ribosoma |
> | | K | Transcripción |
> | | L | Replicación, recombinación y reparación |
> | Procesos celulares y señalización | D | Control del ciclo celular, división celular |
> | | V | Mecanismos de defensa |
> | | T | Transducción de señales |
> | | M | Biogénesis de pared, membrana y envoltura celular |
> | | N | Motilidad celular |
> | | U | Tráfico intracelular y secreción |
> | | O | Modificación postraduccional, chaperonas |
> | Metabolismo | C | Producción y conversión de energía |
> | | G | Transporte y metabolismo de carbohidratos |
> | | E | Transporte y metabolismo de aminoácidos |
> | | F | Transporte y metabolismo de nucleótidos |
> | | H | Transporte y metabolismo de coenzimas |
> | | I | Transporte y metabolismo de lípidos |
> | | P | Transporte y metabolismo de iones inorgánicos |
> | | Q | Biosíntesis, transporte y catabolismo de metabolitos secundarios |
> | Poco caracterizadas | R | Predicción de función general |
> | | S | Función desconocida |
>
> - En un genoma bacteriano típico, las categorías más abundantes son las de metabolismo (E, G, C, P), transcripción (K), envoltura celular (M) y transducción de señales (T), además de las poco caracterizadas (R, S).
> - No todos los CDS tienen categoría COG: compare el primer número con el total de CDS (5 512) para saber qué proporción del proteoma está clasificada.
> - Con `m01_cog_counts.txt` puede elaborar en Excel o R el gráfico de barras de categorías COG para la bitácora.

### Reconstrucción de rutas metabólicas con KEGG

```bash
awk -F'\t' 'NR==1{for(i=1;i<=NF;i++) if($i=="KEGG_ko") c=i; next} $c!="-" && $c!="" {n=split($c,a,","); for(j=1;j<=n;j++){sub("ko:","",a[j]); print $1"\t"a[j]}}' m01_eggnog.tsv > m01_ko.txt

head -n 5 m01_ko.txt

M01_05647       K07172
M01_03211       K00640
M01_03211       K00661
M01_03211       K03818
M01_03211       K13018

cut -f 2 m01_ko.txt | sort -u | wc -l

2852
```

> **Comentario (comandos de procesamiento de la tabla):**
> - `NR==1{...; next}`: en la primera línea (`NR` es el número de línea), recorre todas las columnas (`NF` es el número de columnas) y guarda en la variable `c` la posición de la columna llamada `KEGG_ko`; `next` pasa a la siguiente línea sin hacer nada más.
> - `$c!="-" && $c!=""`: en las demás líneas, continúa solo si la columna `KEGG_ko` no está vacía ni es un guion (proteínas sin KO).
> - `n=split($c,a,",")`: una proteína puede tener varios KO separados por comas; se dividen y se guardan en la lista `a`.
> - `sub("ko:","",a[j])`: quita el prefijo `ko:` de cada KO.
> - `print $1"\t"a[j]`: imprime el identificador de la proteína, una tabulación y el KO.
> - `> m01_ko.txt`: guarda la tabla en un archivo.
> - `cut -f 2 m01_ko.txt | sort -u | wc -l`: extrae la columna de los KO, elimina los repetidos (`sort -u`) y cuenta cuántos KO **distintos** hay.

> **Comentario (resultados):**
> - `m01_ko.txt` tiene dos columnas: locus tag y KO. Es el formato que acepta KEGG Mapper.
> - Una proteína puede aparecer en varias líneas: `M01_03211` tiene cuatro KO (`K00640`, `K00661`, `K03818`, `K13018`). Ocurre cuando la proteína pertenece a una familia cuyos miembros se han clasificado en varios KO parecidos (aquí, acetiltransferasas), y eggNOG-mapper los reporta todos.
> - El genoma tiene **2 852 KO distintos**: ese es el número de funciones diferentes que KEGG reconoce en esta cepa.

### Alternativa directa: obtener los KO de la anotación de Bakta (sin servidor web)

Bakta ya incluye, en la columna de referencias cruzadas (`DbXrefs`) de su tabla, el KO de KEGG de muchas proteínas. Si el trabajo de eggNOG-mapper aún no termina, puede avanzar con esta tabla:

```bash
grep -c "KEGG:K" ~/genomics/annotation/bakta/m01_bakta/m01.tsv

1431

grep -v "^#" ~/genomics/annotation/bakta/m01_bakta/m01.tsv | awk -F'\t' '{n=split($9,a,", "); for(j=1;j<=n;j++) if(a[j] ~ /^KEGG:K[0-9]+$/){sub("KEGG:","",a[j]); print $6"\t"a[j]}}' > m01_ko_bakta.txt

head -n 5 m01_ko_bakta.txt

M01_00001       K01985
M01_00002       K01980
M01_00008       K01977
M01_00009       K00230
M01_00020       K01878

cut -f 2 m01_ko_bakta.txt | sort -u | wc -l

1128
```

> **Comentario (comandos de procesamiento de la tabla):**
> - `grep -c "KEGG:K"`: cuenta cuántas filas de la tabla de Bakta tienen un KO asignado.
> - En el `awk`: `split($9,a,", ")` divide la columna 9 (`DbXrefs`) en sus referencias; `a[j] ~ /^KEGG:K[0-9]+$/` conserva las que tienen la forma `KEGG:K` seguido de números; `sub("KEGG:","",a[j])` quita el prefijo; y `print $6"\t"a[j]` imprime el locus tag (columna 6) y el KO. Es el mismo formato de dos columnas que `m01_ko.txt`.

> **Comentario (resultados):**
> - Bakta asignó un KO a 1 431 elementos del genoma, que corresponden a **1 128 KO distintos**; eggNOG-mapper asignó **2 852**. La diferencia se debe al método: Bakta solo transfiere el KO cuando la proteína coincide con una proteína de referencia que ya lo tiene anotado, mientras que eggNOG-mapper lo transfiere por ortología, lo que alcanza a muchas más proteínas.
> - Las tres primeras filas (`M01_00001`, `M01_00002` y `M01_00008`) no son proteínas, sino los genes de rRNA 5S, 23S y 16S, que también tienen KO en KEGG (`K01985`, `K01980` y `K01977`). eggNOG-mapper no los incluye porque solo analiza proteínas.
> - Para la bitácora use la tabla de eggNOG-mapper (`m01_ko.txt`), que es más completa, e indique cuántos KO aporta cada método.
> - El servidor KAAS (usado en años anteriores) hace esta misma asignación de KO a partir del archivo `.faa`; ya no es necesario, porque los KO se obtienen de eggNOG-mapper o de Bakta.

### Descargar `m01_ko.txt` con WinSCP, ir a KEGG Mapper Reconstruct (https://www.genome.jp/kegg/mapper/reconstruct.html), cargar el archivo y ejecutar

<img width="1587" height="1042" alt="image" src="https://github.com/user-attachments/assets/80578aaa-1571-4c82-8625-fa067d3da797" />

**Figura 17.** Formulario de KEGG Mapper Reconstruct. Se carga el archivo `m01_ko.txt` (o se pega su contenido) y se ejecuta con *Exec*.

<img width="1063" height="1438" alt="image" src="https://github.com/user-attachments/assets/4dcb0358-cb11-4c7c-8581-a1dbe9bd159e" />

**Figura 18.** Resultado de la reconstrucción, pestaña *Pathway*: rutas de KEGG agrupadas por categoría (metabolismo, procesamiento de la información genética, procesos celulares, etc.); el número entre paréntesis indica cuántos genes del genoma participan en cada ruta. <!-- VERIFICAR: leyenda escrita sin ver la captura -->

<img width="1028" height="1138" alt="image" src="https://github.com/user-attachments/assets/23043af3-7e61-4209-97d7-ccb8e7aee660" />

**Figura 19.** Detalle de la lista de rutas del metabolismo, con el número de genes asignados a cada una. <!-- VERIFICAR: leyenda escrita sin ver la captura -->

<img width="1781" height="1189" alt="image" src="https://github.com/user-attachments/assets/f74d8d5f-5614-4306-aca0-8e18ed22a7fe" />

**Figura 20.** Mapa de una ruta de KEGG con los genes del genoma `m01` resaltados: las cajas coloreadas son enzimas presentes en el genoma y las cajas en blanco, enzimas ausentes. <!-- VERIFICAR: leyenda escrita sin ver la captura -->

<img width="1319" height="713" alt="image" src="https://github.com/user-attachments/assets/1d3a5620-4c48-48c1-b5ce-1d943b938dd6" />

**Figura 21.** Pestaña *Module*: módulos funcionales de KEGG y su estado en el genoma (completo o con uno o dos bloques faltantes). <!-- VERIFICAR: leyenda escrita sin ver la captura -->

<img width="1174" height="1245" alt="image" src="https://github.com/user-attachments/assets/98cfd51e-490f-4717-9f27-aa467f2f784d" />

**Figura 22.** Detalle de un módulo de KEGG, con los pasos de la ruta y los KO del genoma que cubren cada uno. <!-- VERIFICAR: leyenda escrita sin ver la captura -->

> **Comentario (figuras 17 a 22):**
> - **Pestaña *Pathway*:** lista las rutas con genes presentes. Que una ruta aparezca en la lista **no significa que esté completa**: basta un solo gen para que aparezca. Al abrir un mapa, las enzimas del genoma aparecen resaltadas y puede verse si la ruta tiene todos sus pasos.
> - **Pestaña *Module*:** un módulo es una unidad funcional mínima (por ejemplo, "fijación de nitrógeno" o "biosíntesis de triptófano"). KEGG indica si está **completo** o cuántos bloques le faltan: es la forma más directa de saber si una ruta realmente puede funcionar.
> - **Pestaña *Brite*:** clasifica los genes en jerarquías funcionales (transportadores, sistemas de secreción, sistemas de dos componentes, etc.).
> - Rutas de interés para una bacteria promotora del crecimiento vegetal: metabolismo del nitrógeno (map00910), metabolismo del triptófano (map00380, biosíntesis de ácido indolacético), metabolismo de fosfonatos y fosfinatos (map00440), biosíntesis de sideróforos (map01053), metabolismo del butanoato (map00650, acetoína y 2,3-butanodiol), sistema de dos componentes (map02020) y quimiotaxis (map02030).

## 7. Identificación de genes promotores del crecimiento vegetal (PGP)

Se usan dos estrategias complementarias: una **búsqueda dirigida** de genes clave bien caracterizados y un **barrido global** con PGPg_finder.

### Búsqueda dirigida de genes PGP en la anotación de eggNOG-mapper

```bash
cd ~/genomics/annotation/pgp

for g in nifH nifD nifK gcd pqqB pqqC pqqD pqqE phoA appA phnC acdS ipdC entA entB entC entE entF fepA budA budB budC otsA otsB; do
  awk -F'\t' -v g="$g" 'NR==1{for(i=1;i<=NF;i++) if($i=="Preferred_name") c=i; next} tolower($c)==tolower(g){n++; ids=ids" "$1} END{print g"\t"n+0"\t"ids}' ~/genomics/annotation/eggnog/m01_eggnog.tsv
done > m01_pgp_genes.txt

cat m01_pgp_genes.txt
```

<!-- Pegar aquí la salida real con el genoma m01 -->

> **Comentario (comandos de procesamiento de la tabla):**
> - `for g in nifH nifD ...; do ... done`: bucle que repite el comando `awk` una vez por cada gen de la lista; en cada vuelta, la variable `g` contiene el nombre de un gen.
> - `-v g="$g"`: pasa a `awk` el nombre del gen que se está buscando.
> - `NR==1{...}`: en el encabezado, ubica la columna `Preferred_name`.
> - `tolower($c)==tolower(g)`: compara el nombre del gen de cada proteína con el gen buscado, sin distinguir mayúsculas de minúsculas (`tolower` convierte a minúsculas). La comparación es exacta: `entA` no coincide con `entA_2` ni con `mentA`.
> - `{n++; ids=ids" "$1}`: si coincide, suma 1 al contador `n` y agrega el locus tag a la lista `ids`.
> - `END{print g"\t"n+0"\t"ids}`: al terminar de leer la tabla, imprime el gen, el número de copias (`n+0` hace que se imprima `0` si no se encontró) y los locus tags.
> - `done > m01_pgp_genes.txt`: guarda en un archivo el resultado de todas las vueltas del bucle.

> **Comentario (resultados):**
> - La tabla tiene tres columnas: gen, número de copias y locus tags de las proteínas. Un `0` indica que eggNOG-mapper no asignó ese nombre a ninguna proteína.
> - Genes buscados y rasgo asociado:
>
> | Rasgo PGP | Genes | Función |
> |---|---|---|
> | Fijación de nitrógeno | `nifH`, `nifD`, `nifK` | Subunidades de la nitrogenasa |
> | Solubilización de fosfato inorgánico | `gcd`, `pqqBCDE` | Glucosa deshidrogenasa (produce ácido glucónico) y biosíntesis de su cofactor PQQ |
> | Mineralización de fosfato orgánico | `phoA`, `appA`, `phnC` | Fosfatasa alcalina, fitasa y transporte de fosfonatos |
> | Producción de ácido indolacético (AIA) | `ipdC` | Indol-3-piruvato descarboxilasa |
> | Reducción del etileno de la planta | `acdS` | ACC desaminasa |
> | Sideróforos | `entABCEF`, `fepA` | Biosíntesis de enterobactina y su receptor |
> | Compuestos volátiles | `budA`, `budB`, `budC` | Síntesis de acetoína y 2,3-butanodiol |
> | Tolerancia a estrés osmótico | `otsA`, `otsB` | Síntesis de trehalosa |
>
> - La lista es general e incluye los sideróforos de las enterobacterias; **adáptela al género de su cepa** (por ejemplo, en *Bacillus* los sideróforos son de tipo bacilibactina, genes `dhbABCEF`; en *Pseudomonas*, pioverdina, genes `pvd`; en *Vibrio*, vibriobactina, genes `vibABCDEF`).
> - En el genoma de demostración (*Vibrio*) la mayoría de estos genes dará 0 copias: es el resultado esperado para una bacteria que no es promotora del crecimiento vegetal, y sirve de contraste con su cepa.
> - **Presencia de un gen no es lo mismo que función.** El genoma indica potencial; la actividad debe confirmarse con ensayos (por ejemplo, medio NBRIP para solubilización de fosfato, reactivo de Salkowski para AIA, medio CAS para sideróforos).
> - Tenga cuidado con los falsos positivos por homología: `acdS` (ACC desaminasa) es homólogo de `dcyD` (D-cisteína desulfhidrasa) y con frecuencia se anotan uno por el otro. Para revisar un gen, extraiga su proteína y compárela con blastp del NCBI:

```bash
conda activate quality

seqkit grep -p M01_00005 ~/genomics/annotation/bakta/m01_bakta/m01.faa

conda deactivate
```

> **Comentario:**
> - `seqkit grep -p M01_00005`: extrae del proteoma la secuencia cuyo identificador coincide con el patrón indicado (`-p`), y la muestra en formato FASTA para copiarla en blastp.
> - Reemplace `M01_00005` por el locus tag del gen que quiera revisar (tercera columna de `m01_pgp_genes.txt`).

### Descargar genomas de referencia del género

### Ir a NCBI Datasets Genome (https://www.ncbi.nlm.nih.gov/datasets/genome/), buscar el género de su cepa y descargar de 3 a 4 genomas de referencia: marque *Genome sequences (FASTA)* y *Protein (FASTA)*

<img width="1953" height="1421" alt="image" src="https://github.com/user-attachments/assets/26a43b40-faf1-457e-8409-0065a14c5c21" />

**Figura 23.** Búsqueda de genomas en NCBI Datasets Genome y tabla de resultados, con los filtros de nivel de ensamblaje y de genomas de referencia. <!-- VERIFICAR: leyenda escrita sin ver la captura -->

<img width="2116" height="953" alt="image" src="https://github.com/user-attachments/assets/71d849ae-f72e-406c-8646-bf5aab253e45" />

**Figura 24.** Selección de los genomas que se van a descargar (casillas de la izquierda) y botón *Download*. <!-- VERIFICAR: leyenda escrita sin ver la captura -->

<img width="2138" height="1392" alt="image" src="https://github.com/user-attachments/assets/489782e0-87bd-4808-b9a9-034065b82a15" />

**Figura 25.** Ventana de descarga: se marcan *Genome sequences (FASTA)* y *Protein (FASTA)*. <!-- VERIFICAR: leyenda escrita sin ver la captura -->

> **Comentario (figuras 23 a 25):**
> - Incluya la **cepa tipo de la especie** que identificó por ANI en la Semana 05 y otras cepas o especies del mismo género; prefiera genomas con nivel de ensamblaje *Complete*.
> - La descarga es un archivo `.zip`: dentro, en la carpeta `ncbi_dataset/data/`, hay una carpeta por genoma (con su código de acceso `GCF_...` o `GCA_...`) que contiene el genoma (`.fna`) y el proteoma (`protein.faa`). El genoma se usa en PGPg_finder y el proteoma, en OrthoVenn3 (sección 12).
> - Renombre los archivos con un nombre corto y sin espacios que identifique a la cepa. En esta demostración se usaron dos genomas de *Vibrio campbellii*: `Vcampbellii_BoB-53.fna` / `Vcampbellii_BoB-53.faa` y `Vcampbellii_HJ-2023.fna` / `Vcampbellii_HJ-2023.faa`.
> - Anote el código de acceso de cada genoma: debe reportarlo en la bitácora.

### Subir con WinSCP los archivos `.fna` a `~/genomics/annotation/pgp/m01_genomes/` y los `.faa` a `~/genomics/annotation/orthovenn/`

```bash
cd ~/genomics/annotation/pgp

mkdir -p m01_genomes

cp ~/genomics/assembly/nanopore/m01_genome_final.fasta m01_genomes/m01.fasta

ls m01_genomes

m01.fasta  Vcampbellii_BoB-53.fna  Vcampbellii_HJ-2023.fna
```

> **Comentario:**
> - `mkdir -p m01_genomes`: crea la carpeta que recibirá los genomas (créela antes de subir los archivos con WinSCP).
> - `cp ... m01_genomes/m01.fasta`: copia el genoma de demostración con el nombre `m01.fasta`. PGPg_finder usa el nombre de cada archivo como nombre de la muestra en las tablas y figuras, por eso conviene un nombre corto.
> - `ls m01_genomes`: la carpeta contiene tres genomas: el de la cepa (`m01.fasta`) y los dos de referencia. PGPg_finder reconoce archivos con extensión `.fasta`, `.fna` o `.fa`.

### Barrido global de rasgos PGP con PGPg_finder

```bash
PGPg_finder -w genome_wf -i m01_genomes -o m01_pgpg -t 10 --piden 40 --qcov 70
```

> **Comentario (comandos):**
> - `PGPg_finder`: en el servidor está instalado como comando global, **no hace falta activar ningún entorno de conda**. Su base de datos está en `/data/db/PGPg_finder/` y el comando la ubica automáticamente.
> - `-w genome_wf`: flujo de trabajo para genomas. Para cada genoma predice las proteínas con Prodigal y las compara con DIAMOND (blastp) contra la base de datos PLaBAse, conservando el mejor hit de cada proteína.
> - `-i m01_genomes`: carpeta con los genomas.
> - `-o m01_pgpg`: carpeta de salida.
> - `-t 10`: número de hilos.
> - `--piden 40 --qcov 70`: identidad mínima de 40 % y cobertura mínima de la proteína de 70 %. Los valores por defecto del programa (30 % y 30 %) son muy permisivos: con una cobertura de 30 % basta que coincida un dominio para asignar el rasgo. El e-value máximo se deja en su valor por defecto (1e-5, `--evalue`).

```bash
ls -lh m01_pgpg

total 7,6M
drwxrwxr-x 5 alumno01 alumno01 4,0K oct  3 09:54 figures
-rw-rw-r-- 1 alumno01 alumno01 216K oct  3 09:54 gene_counts.txt
-rw-rw-r-- 1 alumno01 alumno01  955 oct  3 09:54 log.txt
-rw-rw-r-- 1 alumno01 alumno01 156K oct  3 09:53 m01_diamond.txt
-rw-rw-r-- 1 alumno01 alumno01 2,4M oct  3 09:52 m01_proteins.fa
drwxrwxr-x 5 alumno01 alumno01 4,0K oct  3 09:54 tables
-rw-rw-r-- 1 alumno01 alumno01 149K oct  3 09:54 Vcampbellii_BoB-53_diamond.txt
-rw-rw-r-- 1 alumno01 alumno01 2,1M oct  3 09:53 Vcampbellii_BoB-53_proteins.fa
-rw-rw-r-- 1 alumno01 alumno01 160K oct  3 09:52 Vcampbellii_HJ-2023_diamond.txt
-rw-rw-r-- 1 alumno01 alumno01 2,5M oct  3 09:51 Vcampbellii_HJ-2023_proteins.fa

head -n 3 m01_pgpg/m01_diamond.txt

m01_001_2       PGPT0008470_581 96.0    175     7       0       1       175     1       175     9.14e-123       348
m01_001_3       PGPT0002735_1517        99.5    435     2       0       16      450     51      485     5.33e-317       864
m01_001_5       PGPT0001865_345 100     723     0       0       1       723     1       723     0.0     1407

head -n 5 m01_pgpg/gene_counts.txt

Sample  ID      Count
Vcampbellii_HJ-2023     PGPT0000020_175 1
Vcampbellii_HJ-2023     PGPT0000020_73  1
Vcampbellii_HJ-2023     PGPT0000050_2668        1
Vcampbellii_HJ-2023     PGPT0000065_2111        1

cat m01_pgpg/tables/non-normalized/gene_counts_Lv3.txt

Lv3     Vcampbellii_BoB-53      Vcampbellii_HJ-2023     m01
NITROGEN_ACQUISITION    80      90      97
COLONIZATION-PLANT_DERIVED_SUBSTRATE_USAGE      383     405     432
CARBON_DIOXID_FIXATION  4       6       5
PHOSPHATE_SOLUBILIZATION        137     144     143
POTASSIUM_SOLUBILIZATION        12      12      13
SULFUR_ASSIMILATION|MINERALIZATION      23      24      26
IRON_ACQUISITION        115     109     113
HEAVY_METAL_DETOXIFICATION      82      81      86
CE-BACTERIAL_FITNESS    89      128     115
FLUORIDE_DETOXIFICATION 2       2       2
XENOBIOTICS_BIODEGRADATION      33      38      39
PHYTOHORMONE-ABSCISIC_ACID_DEGRADATION  18      18      18
PHYTOHORMONE-CYTOKININS|DERIVATE_PRODUCTION     11      11      11
PLANT_SIGNAL-OTHER_TERPENOID|DERIVATE_PRODUCTION        11      11      11
PHYTOHORMONE-GAMMA-AMINOBUTYRIC_ACID|GABA_PRODUCTION    3       3       3
PLANT_SIGNAL-PHOSPHOLIPID_PRODUCTION    14      14      14
PLANT_SIGNAL-BRANCHING_INHIBITION       14      16      13
PLANT_SIGNAL-GERMINATION_STIMULATION    30      30      31
PLANT_SIGNALLING_VOLATILES      24      24      25
PLANT_VITAMIN_PRODUCTION        92      92      95
PLANT_SIGNAL-UBIQUINONE|COENZYME_Q_PRODUCTION   10      10      10
PLANT_SIGNAL-LINOLENIC_ACID_PRODUCTION  2       2       2
NEUTRALIZING_BIOTIC_STRESS      42      46      51
NEUTRALIZING_ABIOTIC_STRESS     291     298     302
UNIVERSAL_STRESS_RESPONSE       59      63      64
INDUCTION_OF_SYSTEMIC_RESISTANCE|ISR    1       1       1
TRIGGERED_IMMUNITY      19      20      20
ROOT_COLONIZATION       12      13      13
COLONIZATION-MOTILITY|CHEMOTAXIS        167     179     179
CE-QUORUM_SENSING_RESPONSE|BIOFILM_FORMATION    138     172     168
COLONIZATION-SURFACE_ATTACHMENT 73      75      80
COLONIZATION-PLANT_CELL_WALL|MEMBRANE_DEGRADATION       3       3       4
COLONIZATION-ADAPTION_TO_PLANT_IMMUNE_SYSTEM    2       2       2
OTHER_COLONIZATION_RELATED_PROTEINS     15      15      17
CE-CELL_ENVELOPE_REMODELLING    78      79      77
CE-SPORE_PRODUCTION     6       5       6
CE-EXOPOLYSACCHARIDE_PRODUCTION|EPS     2       1       1
CE-BACTERIAL_SECRETION  70      76      78
PUTATIVE_FUNCTIONS-1    13      15      15
```

> **Comentario (comandos):**
> - `ls -lh m01_pgpg`: lista los archivos de salida.
> - `head -n 3` y `head -n 5`: muestran las primeras líneas de la tabla de DIAMOND y de la tabla de conteos.
> - `cat .../gene_counts_Lv3.txt`: muestra la tabla de conteos agrupada en el nivel 3 de la ontología.

> **Comentario (resultados):**
> - **Archivos:** para cada genoma hay un `<muestra>_proteins.fa` (proteínas predichas por Prodigal) y un `<muestra>_diamond.txt` (resultado de la comparación). Además: `gene_counts.txt` (conteos), `log.txt` (registro), `tables/` (tablas por nivel de la ontología, `Lv1` a `Lv5`, normalizadas y sin normalizar, y un resumen) y `figures/` (mapas de calor en formato SVG).
> - **`m01_diamond.txt`:** una fila por proteína con hit, en formato tabular de 12 columnas: (1) proteína del genoma, (2) referencia de PLaBAse, (3) % de identidad, (4) longitud del alineamiento, (5) diferencias, (6) aperturas de gap, (7 y 8) inicio y fin en la proteína, (9 y 10) inicio y fin en la referencia, (11) e-value y (12) puntaje (*bit score*). Por ejemplo, la proteína `m01_001_2` (el segundo gen predicho en el contig `m01_001`) coincide con `PGPT0008470_581` con 96,0 % de identidad a lo largo de sus 175 aminoácidos, con un e-value de 9,14e-123. En el identificador de la referencia, `PGPT0008470` es el rasgo de la ontología y el número final, la secuencia concreta dentro de ese rasgo.
> - Los identificadores de las proteínas son `m01_001_2`, `m01_001_3`, ... y no los locus tags de Bakta, porque PGPg_finder predice los genes por su cuenta. Note que `m01_001_1` y `m01_001_4` no aparecen: no tuvieron ningún hit que superara los umbrales.
> - **`gene_counts.txt`:** tres columnas: muestra, identificador del PGPT y número de genes.
> - **Tabla `Lv3`:** una fila por categoría de nivel 3 y una columna por genoma. Niveles de la ontología: `Lv1` separa efectos directos e indirectos; `Lv2` incluye, entre otros, biofertilización, fitohormonas, biorremediación, colonización, exclusión competitiva (`CE-`) y control del estrés; los niveles `Lv3` a `Lv5` son cada vez más específicos.
> - **Lectura de la tabla:** el genoma `m01` tiene 97 genes en `NITROGEN_ACQUISITION`, 143 en `PHOSPHATE_SOLUBILIZATION` y 113 en `IRON_ACQUISITION`, a pesar de que **no es una bacteria promotora del crecimiento vegetal**. Esto ocurre porque esas categorías incluyen funciones generales que casi cualquier bacteria tiene: transportadores de amonio y nitrato, asimilación de nitrógeno, transporte de fosfato, captación de hierro. Lo mismo pasa con las categorías más numerosas: uso de sustratos (`COLONIZATION-PLANT_DERIVED_SUBSTRATE_USAGE`, 432), respuesta a estrés abiótico (302), motilidad y quimiotaxis (179) y *quorum sensing* y biopelículas (168).
> - Los tres genomas tienen conteos muy parecidos en todas las categorías, lo esperado entre cepas del mismo género. Las diferencias mayores están en `CE-BACTERIAL_FITNESS` (89, 128 y 115) y `CE-QUORUM_SENSING_RESPONSE|BIOFILM_FORMATION` (138, 172 y 168), donde `Vcampbellii_BoB-53` tiene menos genes.

<img width="2668" height="1160" alt="image" src="https://github.com/user-attachments/assets/b7053eb2-7f40-44b7-b27b-409de9a59b3b" />

**Figura 26.** Mapa de calor de PGPg_finder con el número de genes por categoría de la ontología PGPT en los tres genomas (`m01` y las dos referencias de *Vibrio campbellii*). <!-- VERIFICAR: indicar el nivel (Lv2, Lv3...) y si es la tabla normalizada -->

<img width="1728" height="1042" alt="image" src="https://github.com/user-attachments/assets/09cde020-9e53-4767-af98-310fd7f29e12" />

**Figura 27.** Mapa de calor resumen de PGPg_finder (`figures/summary/`), con las categorías principales de la ontología en los tres genomas. <!-- VERIFICAR: leyenda escrita sin ver la captura -->

> **Comentario (figuras 26 y 27):**
> - En un mapa de calor, cada fila es una categoría de rasgos PGP, cada columna un genoma y el color de cada celda indica la abundancia de genes según la escala lateral: cuanto más intenso el color, más genes.
> - En las figuras **normalizadas**, el conteo se divide entre el total de genes del genoma, para que genomas de distinto tamaño sean comparables.
> - Lo informativo no es el color de una celda aislada, sino las **diferencias entre columnas en una misma fila**: una categoría con color más intenso en su cepa que en las referencias señala un rasgo que la distingue. Aquí las tres columnas son casi iguales, lo que indica que `m01` no tiene un perfil distinto al de su género.

> **Punto de control:** PGPg_finder asigna un PGPT a una gran parte de las proteínas del genoma, porque la ontología incluye funciones generales. Por eso el número total de hits **no** mide qué tan buena promotora es una cepa: un *Vibrio* marino obtiene casi 100 genes de "adquisición de nitrógeno" sin fijar nitrógeno. Interprete los resultados comparando su cepa con los genomas de referencia y contrástelos con la búsqueda dirigida: ¿los genes clave que encontró en `m01_pgp_genes.txt` aparecen en las categorías correspondientes de PGPg_finder?

> **Alternativa en línea:** también puede cargar el proteoma (`m01.faa`) en la herramienta PGPT-Pred de PLaBAse (https://plabase.cs.uni-tuebingen.de/), que usa la misma ontología y muestra los resultados en un gráfico jerárquico interactivo.

## 8. Identificación de enzimas activas sobre carbohidratos (CAZymes)

Las CAZymes participan en la degradación y síntesis de polisacáridos. En bacterias asociadas a plantas se relacionan con el aprovechamiento de exudados y restos vegetales, la colonización de la raíz, la formación de biopelículas (exopolisacáridos) y la degradación de la pared de hongos fitopatógenos (quitinasas, glucanasas).

```bash
cd ~/genomics/annotation/cazymes

conda activate run_dbcan

run_dbcan CAZyme_annotation --input_raw_data ~/genomics/annotation/bakta/m01_bakta/m01.faa --mode protein --output_dir m01_dbcan --threads 10

head -n 5 m01_dbcan/overview.tsv

Gene ID EC#     dbCAN_hmm       dbCAN_sub       DIAMOND #ofTools        Recommend Results       Substrate
M01_00066       -       -       -       GT58    1       -       -
M01_00112       -       -       CBM50_e2200(40-86)      CBM50   2       CBM50_e2200     chitin
M01_00130       -       AA1(57-457)     AA1_e63(42-457) -       2       AA1_e63 lignin
M01_00151       -       -       CBM32_e340(20-124)      -       1       -       host glycan
```

> **Comentario (comandos):**
> - `CAZyme_annotation`: anota CAZymes con tres métodos: HMM de familias (dbCAN_hmm), HMM de subfamilias (dbCAN_sub) y DIAMOND contra las proteínas de CAZy.
> - `--input_raw_data`: el proteoma anotado por Bakta.
> - `--mode protein`: indica que la entrada son proteínas (para un genoma en nucleótidos sería `--mode prok`).
> - `--output_dir m01_dbcan`: carpeta de salida.
> - `--threads 10`: número de hilos.
> - No se indica `--db_dir`: al activar el entorno `run_dbcan`, el servidor pasa automáticamente la ruta de las bases de datos (`/data/db/dbcan`) a cada subcomando.
> - `head -n 5 m01_dbcan/overview.tsv`: muestra las primeras líneas de la tabla que reúne los resultados de los tres métodos.

> **Comentario (resultados):** `overview.tsv` tiene una fila por proteína candidata y 8 columnas:
> - `Gene ID`: identificador de la proteína (locus tag de Bakta).
> - `EC#`: número EC de la actividad enzimática predicha.
> - `dbCAN_hmm`: familia asignada por los HMM de familias, con las posiciones del dominio en la proteína entre paréntesis.
> - `dbCAN_sub`: subfamilia asignada por los HMM de subfamilias (`_e` seguido de un número identifica la subfamilia).
> - `DIAMOND`: familia de la proteína más parecida de la base de datos CAZy.
> - `#ofTools`: número de métodos que detectaron la proteína como CAZyme (1 a 3).
> - `Recommend Results`: asignación final recomendada; si la proteína tiene varios dominios, se separan con `|`.
> - `Substrate`: sustrato probable de la subfamilia.
> - Un guion (`-`) indica que ese método no dio resultado.
>
> Lectura de las cuatro filas: `M01_00066` solo fue detectada por DIAMOND (familia GT58, `#ofTools` = 1), por lo que no tiene asignación recomendada y se descartará en el filtro. `M01_00112` fue detectada por dos métodos como CBM50 (dominio LysM, de unión a quitina y peptidoglicano, entre los aminoácidos 40 y 86). `M01_00130` es una AA1 (oxidasa multicobre, tipo lacasa) detectada por los dos HMM. `M01_00151` solo fue detectada por un método (CBM32).

### Filtrar las CAZymes detectadas por al menos dos métodos y contarlas por clase y por familia

```bash
awk -F'\t' 'NR==1{for(i=1;i<=NF;i++) if($i=="#ofTools") t=i; print; next} $t>=2' m01_dbcan/overview.tsv > m01_cazymes_filtered.tsv

tail -n +2 m01_cazymes_filtered.tsv | wc -l

143

awk -F'\t' 'NR==1{for(i=1;i<=NF;i++) if($i=="Recommend Results") r=i; next} {n=split($r,a,"|"); for(j=1;j<=n;j++){cl=a[j]; sub(/[0-9_].*/,"",cl); print cl}}' m01_cazymes_filtered.tsv | sort | uniq -c

     10 AA
     37 CBM
      5 CE
     76 GH
     39 GT

awk -F'\t' 'NR==1{for(i=1;i<=NF;i++) if($i=="Recommend Results") r=i; next} {n=split($r,a,"|"); for(j=1;j<=n;j++){fam=a[j]; sub(/_.*/,"",fam); print fam}}' m01_cazymes_filtered.tsv | sort | uniq -c | sort -k1,1nr | head -n 15

     16 GT4
     13 GH13
     11 CBM5
     11 GH23
     10 CBM50
      6 GH18
      6 GT2
      5 GH3
      5 GH92
      4 CBM73
      4 GH20
      3 CBM48
      3 CBM69
      3 GH188
      3 GH2
```

> **Comentario (comandos de procesamiento de la tabla):**
> - **Primer comando (filtro):** en el encabezado (`NR==1`) ubica la columna `#ofTools` y guarda su posición en `t`; `print` imprime el encabezado y `next` pasa a la línea siguiente. En las demás líneas, `$t>=2` conserva solo las proteínas detectadas por 2 o 3 métodos (criterio recomendado por los autores de dbCAN para reducir falsos positivos). El resultado se guarda en `m01_cazymes_filtered.tsv`.
> - **Segundo comando:** cuenta las filas sin el encabezado, es decir, el número de CAZymes.
> - **Tercer comando (clases):** ubica la columna `Recommend Results`; `split($r,a,"|")` separa los dominios de una proteína; `sub(/[0-9_].*/,"",cl)` borra desde el primer número o guion bajo en adelante, de modo que `GH13_e1` queda como `GH` y `CBM50_e2200` como `CBM`; `sort | uniq -c` cuenta cada clase.
> - **Cuarto comando (familias):** igual, pero `sub(/_.*/,"",fam)` borra solo desde el guion bajo, de modo que `GH13_e1` queda como `GH13` (se agrupan las subfamilias); `sort -k1,1nr | head -n 15` ordena de mayor a menor y muestra las 15 primeras.

> **Comentario (resultados):**
> - El genoma tiene **143 CAZymes** confiables. La suma por clase (10 + 37 + 5 + 76 + 39 = 167) es mayor que 143 porque una proteína con varios dominios se cuenta en cada uno (por ejemplo, una quitinasa GH18 con un módulo de unión CBM5).
> - Clases de CAZymes:
>
> | Clase | Nombre | Función |
> |---|---|---|
> | GH | Glicósido hidrolasas | Hidrolizan enlaces glicosídicos (celulasas, quitinasas, amilasas) |
> | GT | Glicosiltransferasas | Forman enlaces glicosídicos (síntesis de pared y exopolisacáridos) |
> | PL | Polisacárido liasas | Rompen polisacáridos ácidos, como la pectina |
> | CE | Carbohidrato esterasas | Eliminan grupos éster de los polisacáridos |
> | AA | Actividades auxiliares | Enzimas redox que actúan junto con las CAZymes |
> | CBM | Módulos de unión a carbohidratos | Dominios no catalíticos que unen la enzima a su sustrato |
>
> - Predominan las glicósido hidrolasas (76) y las glicosiltransferasas (39). **No se detectó ninguna polisacárido liasa (PL)**: esta cepa no tiene las enzimas típicas de degradación de pectina, un componente de la pared vegetal.
> - Las familias más abundantes reflejan el modo de vida de un *Vibrio* marino: GH18 (quitinasas), GH20 (β-N-acetilhexosaminidasas) y los módulos CBM5 y CBM73 (unión a quitina) forman la maquinaria de degradación de **quitina**, el polisacárido más abundante en el mar (exoesqueleto de crustáceos). GH13 corresponde a enzimas del metabolismo de almidón y glucógeno; GH23 y CBM50, al recambio de la propia pared bacteriana (peptidoglicano); y GT4 y GT2, a la síntesis de polisacáridos de la envoltura celular.
> - Familias de interés en bacterias promotoras del crecimiento vegetal: GH18 y GH19 (quitinasas, antagonismo de hongos), GH5 y GH9 (celulasas), GH16 (glucanasas), GH28 y PL1 (degradación de pectina), GT2 y GT4 (síntesis de exopolisacáridos).

### Identificación de clústeres de genes de CAZymes (CGC)

```bash
run_dbcan easy_CGC --input_raw_data ~/genomics/assembly/nanopore/m01_genome_final.fasta --mode prok --output_dir m01_dbcan_cgc --threads 10

step 1/3  CAZyme annotation...
step 2/3  GFF processing...
Generating Prodigal GFF: 6it [00:00, 37.02it/s]
step 3/3  CGC identification...
CGC analysis completed.

grep -v "^#" m01_dbcan_cgc/cgc_standard_out.tsv | cut -f 1 | sort -u | wc -l

72

head -n 15 m01_dbcan_cgc/cgc_standard_out.tsv

CGC#    Gene Type       Contig ID       Protein ID      Gene Start      Gene Stop       Gene Strand     Gene Annotation
CGC1    CAZyme  m01_001 m01_001_104     112940  114034  +       CAZyme|CBM50_e2200
CGC1    TC      m01_001 m01_001_105     114018  115127  +       TC|3.A.11.1.3
CGC2    CAZyme  m01_001 m01_001_115     127555  128949  +       CAZyme|AA1_e63+TC|1.B.76.1.4
CGC2    TC      m01_001 m01_001_116     129073  129456  +       TC|1.A.43.1.11
CGC2    null    m01_001 m01_001_117     129782  131722  +       null
CGC2    null    m01_001 m01_001_118     131722  133137  +       null
CGC2    TC      m01_001 m01_001_119     133124  133897  +       TC|3.A.25.2.1
CGC3    CAZyme  m01_001 m01_001_321     377865  379592  +       CAZyme|CBM50_e1982|CBM50_e1982|CBM50_e1982
CGC3    STP     m01_001 m01_001_322     379611  381518  +       STP|HATPase_c
CGC3    null    m01_001 m01_001_323     381654  382586  +       null
CGC3    TC      m01_001 m01_001_324     382781  383044  +       TC|9.B.468.1.1
CGC4    CAZyme  m01_001 m01_001_514     598096  599379  -       CAZyme|CE4_e337|CBM12_e13
CGC4    TC      m01_001 m01_001_515     599628  599933  +       TC|4.A.3.2.6
CGC4    TC      m01_001 m01_001_516     600022  601356  +       TC|4.A.3.2.6
```

> **Comentario (comandos):**
> - `easy_CGC`: además de anotar las CAZymes, busca transportadores (TC), factores de transcripción (TF) y proteínas de transducción de señales (STP), e identifica los **clústeres de genes de CAZymes (CGC)**: regiones del genoma donde una CAZyme está junto a transportadores o reguladores, lo que sugiere un sistema completo de utilización de un polisacárido.
> - `--input_raw_data`: aquí la entrada es el **genoma ensamblado** (nucleótidos), no el proteoma.
> - `--mode prok`: run_dbcan predice los genes por su cuenta con Prodigal; por eso los identificadores son `m01_001_104`, `m01_001_105`, ... (contig y número correlativo) y no los locus tags de Bakta. Para saber a qué gen de Bakta corresponde uno de ellos, compare sus coordenadas con las de `m01.tsv`.
> - `grep -v "^#" ... | cut -f 1 | sort -u | wc -l`: quita los comentarios, extrae la columna 1 (identificador del clúster), elimina los repetidos y cuenta: es el número de CGC. (La línea de encabezado no empieza con `#`, por lo que también se cuenta: reste 1 al resultado.)
> - `head -n 15`: muestra las primeras líneas de la tabla de clústeres.

> **Comentario (resultados):**
> - **Salida en pantalla:** el análisis tiene 3 pasos: anotación de CAZymes, predicción de genes con Prodigal en los 6 contigs (`6it`) e identificación de los clústeres.
> - El genoma tiene **71 clústeres de genes de CAZymes** (72 líneas distintas menos el encabezado).
> - `cgc_standard_out.tsv` tiene una fila por gen de cada clúster: `CGC#` (clúster), `Gene Type` (tipo de gen: `CAZyme`, `TC`, `TF`, `STP` o `null` si no es un gen firma), `Contig ID`, `Protein ID`, `Gene Start`, `Gene Stop`, `Gene Strand` y `Gene Annotation` (familia de CAZyme, clasificación del transportador en TCDB, o dominio del regulador).
> - Lectura de los primeros clústeres: **CGC1** tiene dos genes contiguos, una proteína con dominio CBM50 y un transportador (`TC|3.A.11.1.3`). **CGC2** tiene cinco genes: una AA1, dos transportadores y dos genes sin anotación (`null`) entre ellos. **CGC3** combina una proteína con tres dominios CBM50 con una proteína de transducción de señales (`STP|HATPase_c`, una histidina quinasa) y un transportador. **CGC4** combina una carbohidrato esterasa CE4 con módulo CBM12 (desacetilasa de quitina) y dos transportadores del sistema de fosfotransferasa (`4.A.3.2.6`): un clúster candidato para la utilización de quitina.
> - `total_cgc_info.tsv` (en la misma carpeta) contiene la anotación de todos los genes firma (CAZymes, TC, TF y STP) del genoma.

### Predicción del sustrato de los clústeres

```bash
run_dbcan easy_substrate --input_raw_data ~/genomics/assembly/nanopore/m01_genome_final.fasta --mode prok --output_dir m01_dbcan_substrate --threads 10

step 1/4  CAZyme annotation...
step 2/4  GFF processing...
Generating Prodigal GFF: 6it [00:00, 36.48it/s]
step 3/4  CGC identification...
step 4/4  Substrate prediction...
CGC substrate analysis completed

head -n 20 m01_dbcan_substrate/substrate_prediction.tsv

#cgcid  PULID   dbCAN-PUL substrate     bitscore        signature pairs dbCAN-sub substrate     dbCAN-sub substrate score
m01_001|CGC4    PUL0381 chitin  2530.0  CAZyme-CAZyme;TC-TC;TC-TC;TC-TC;CAZyme-CAZyme
m01_001|CGC6    PUL0168 galactose       743.0   CAZyme-CAZyme;TC-null
m01_001|CGC28   PUL0311 cellulose       2083.0  TC-TC;CAZyme-CAZyme;TC-TC;CAZyme-CAZyme
m01_001|CGC30   PUL0048 trehalose       1001.0  CAZyme-CAZyme;TC-TC
m01_002|CGC62   PUL0605 glycogen        1380.0  CAZyme-CAZyme;CAZyme-CAZyme     alpha-glucan    3.0
m01_001|CGC5    PUL0012 chitin  6830.0  null-null;CAZyme-CAZyme;CAZyme-CAZyme;null-null;CAZyme-CAZyme;TC-TC;TC-TC;TC-TC;TC-TC
m01_001|CGC12   PUL0230 starch  623.0   null-null;CAZyme-CAZyme;CAZyme-CAZyme
m01_002|CGC64   PUL0160 alpha-mannan    1229.0  CAZyme-CAZyme;CAZyme-CAZyme;CAZyme-CAZyme;CAZyme-CAZyme;CAZyme-CAZyme;CAZyme-CAZyme;CAZyme-CAZyme;CAZyme-CAZyme;CAZyme-CAZyme;CAZyme-CAZyme;CAZyme-CAZyme;CAZyme-CAZyme   hostglycan      5.0
m01_002|CGC51   PUL0573 beta-glucan     563.0   CAZyme-CAZyme;TC-TC
m01_001|CGC29                                   hostglycan      2.0
m01_002|CGC56   PUL0361 starch  509.8   TC-TC;CAZyme-CAZyme;CAZyme-CAZyme       alpha-glucan    2.0
m01_001|CGC21   PUL0208 chitin  157.9   CAZyme-CAZyme;CAZyme-CAZyme;CAZyme-CAZyme       chitin  2.0
m01_001|CGC13   PUL0213 galactomannan   593.0   CAZyme-CAZyme;TC-TC
m01_001|CGC16   PUL0227 xylan   540.0   TC-TC;CAZyme-CAZyme
m01_002|CGC42   PUL0722 xylan   188.3   CAZyme-CAZyme;STP-TC
m01_002|CGC59                                   alpha-glucan    6.0
m01_001|CGC11   PUL0111 melibiose       1744.0  TF-TF;CAZyme-CAZyme;TC-TC
m01_001|CGC25   PUL0267 glycogen        540.0   CAZyme-CAZyme;null-null
m01_001|CGC27   PUL0579 glycosaminoglycan       2212.0  TC-TC;CAZyme-CAZyme;TF-TF

awk -F'\t' '$2 != "" && $2 != "null" {print $1, $2}' m01_dbcan_substrate/substrate_prediction.tsv

#cgcid PULID
m01_001|CGC4 PUL0381
m01_001|CGC6 PUL0168
m01_001|CGC28 PUL0311
m01_001|CGC30 PUL0048
m01_002|CGC62 PUL0605
m01_001|CGC5 PUL0012
m01_001|CGC12 PUL0230
m01_002|CGC64 PUL0160
m01_002|CGC51 PUL0573
m01_002|CGC56 PUL0361
m01_001|CGC21 PUL0208
m01_001|CGC13 PUL0213
m01_001|CGC16 PUL0227
m01_002|CGC42 PUL0722
m01_001|CGC11 PUL0111
m01_001|CGC25 PUL0267
m01_001|CGC27 PUL0579

conda deactivate
```

> **Comentario (comandos):**
> - `easy_substrate`: repite los tres pasos de `easy_CGC` y añade un cuarto, la **predicción del sustrato** de cada clúster. (Por eso, si va a predecir sustratos, puede ejecutar directamente `easy_substrate`: su carpeta de salida incluye también `cgc_standard_out.tsv`.)
> - `head -n 20 .../substrate_prediction.tsv`: muestra las primeras líneas de la tabla de sustratos.
> - `awk -F'\t' '$2 != "" && $2 != "null" {print $1, $2}'`: conserva las filas cuya columna 2 (`PULID`) no está vacía ni es `null`, e imprime las columnas 1 y 2 separadas por un espacio. Es la lista de clústeres que se parecen a un PUL conocido (la primera línea es el encabezado).

> **Comentario (resultados):** run_dbcan predice el sustrato con dos métodos independientes, y la tabla tiene 7 columnas:
> - `#cgcid`: contig y clúster (`m01_001|CGC4`).
> - `PULID`, `dbCAN-PUL substrate` y `bitscore`: **primer método**. El clúster se compara con la base de datos dbCAN-PUL, que reúne loci de utilización de polisacáridos (PUL) cuyo sustrato se demostró experimentalmente. Se indica el PUL más parecido, su sustrato y el puntaje de la comparación (cuanto más alto, mayor parecido).
> - `signature pairs`: los pares de genes que coinciden entre el clúster y el PUL (`CAZyme-CAZyme`, `TC-TC`, `TF-TF`...). Cuantos más pares, y sobre todo más pares de CAZymes, más sólida es la predicción.
> - `dbCAN-sub substrate` y `dbCAN-sub substrate score`: **segundo método**. El sustrato se deduce de las subfamilias de las CAZymes del clúster; el puntaje es el número de CAZymes que lo apoyan.
>
> Lectura de la tabla:
> - **17 de los 71 clústeres** se parecen a un PUL conocido (lista del último comando). Otros, como `CGC29` y `CGC59`, no se parecen a ningún PUL, pero tienen sustrato predicho por las subfamilias (*host glycan* y α-glucano).
> - Las predicciones más sólidas son las de **quitina**: `CGC5` (PUL0012, puntaje 6 830, con 9 pares de genes coincidentes) y `CGC4` (PUL0381, puntaje 2 530); en `CGC21` los dos métodos coinciden en quitina. Concuerda con las familias GH18, GH20 y CBM5 encontradas antes.
> - También hay clústeres para carbohidratos de reserva y azúcares simples: glucógeno y almidón (`CGC62`, `CGC25`, `CGC12`, `CGC56`), trehalosa (`CGC30`), galactosa (`CGC6`) y melibiosa (`CGC11`).
> - **Interprete con cautela las predicciones débiles.** `CGC42` se asigna a xilano con un puntaje bajo (188,3) y un solo par de CAZymes; por sí sola, esa evidencia no permite afirmar que la cepa degrada xilano. Una predicción es más confiable cuando el puntaje es alto, hay varios pares de CAZymes coincidentes y los dos métodos concuerdan.

> **Alternativa en línea:** el servidor dbCAN3 (https://pro.unl.edu/dbCAN2/) acepta el mismo archivo `m01.faa` y devuelve la misma tabla *overview*.

## 9. Identificación de clústeres de metabolitos secundarios (antiSMASH)

### Abrir el enlace de resultados que antiSMASH envió a su correo

<img width="3024" height="534" alt="image" src="https://github.com/user-attachments/assets/819d4d1a-3fa3-4a05-abc8-c86252223d2b" />

**Figura 28.** Página de resultados de antiSMASH: barra superior con las regiones identificadas, numeradas por contig y coloreadas según el tipo de metabolito. <!-- VERIFICAR: leyenda escrita sin ver la captura -->

<img width="3024" height="1081" alt="image" src="https://github.com/user-attachments/assets/60df35d2-9f4b-4e64-abe4-22b21790f988" />

**Figura 29.** Tabla resumen de las regiones (*Overview*): contig, número de región, tipo de metabolito, coordenadas de inicio y fin, clúster conocido más parecido de la base de datos MIBiG y porcentaje de similitud. <!-- VERIFICAR: leyenda escrita sin ver la captura -->

<img width="3024" height="1253" alt="image" src="https://github.com/user-attachments/assets/8c371536-b5c3-4d72-808a-14b3d01f6e76" />

**Figura 30.** Vista de una región: genes del clúster dibujados como flechas y coloreados según su función (biosintéticos centrales, biosintéticos adicionales, transporte, regulación y otros). <!-- VERIFICAR: leyenda escrita sin ver la captura -->

<img width="3024" height="1297" alt="image" src="https://github.com/user-attachments/assets/9966c88d-5a15-41b1-97b6-49b4ca06b9f8" />

**Figura 31.** Detalle de una región, con la información del gen seleccionado y los dominios de las enzimas biosintéticas. <!-- VERIFICAR: leyenda escrita sin ver la captura -->

<img width="3024" height="1488" alt="image" src="https://github.com/user-attachments/assets/17f93604-17a5-4a89-bf80-ec5aeac25e54" />

**Figura 32.** Comparación de una región con los clústeres conocidos más parecidos (*KnownClusterBlast*): los genes homólogos se muestran con el mismo color. <!-- VERIFICAR: leyenda escrita sin ver la captura -->

> **Comentario (figuras 28 a 32):**
> - Cada **región** es un clúster de genes biosintéticos (BGC) candidato. Su nombre indica el contig y el número de región (por ejemplo, región 1.2 = segunda región del primer contig).
> - La tabla resumen indica, para cada región, el tipo de metabolito, las coordenadas, el clúster conocido más parecido de la base de datos MIBiG y el porcentaje de similitud (proporción de genes del clúster conocido que tienen un homólogo en la región).
> - Al hacer clic en una región se ven los genes del clúster. Los **genes biosintéticos centrales** son los que definen el tipo de metabolito; al seleccionar un gen se muestran su locus tag (el de Bakta), su función y sus dominios.
> - Tipos de interés en bacterias promotoras del crecimiento vegetal: sideróforos (`NI-siderophore`, o `NRPS` y `NRP-metallophore` en sideróforos peptídicos como la enterobactina), péptidos no ribosomales y policétidos con actividad antimicrobiana, bacteriocinas (`RiPP-like`), betalactonas, terpenos y aril-polienos.
> - Un porcentaje de similitud bajo con MIBiG no significa que el clúster sea falso: puede tratarse de un metabolito aún no descrito.

> **Punto de control:** ¿Qué tipos de metabolitos puede producir la cepa? ¿El clúster de sideróforos que detectó antiSMASH contiene los genes de sideróforos que encontró en la búsqueda dirigida (sección 7)? Compare los locus tags.

## 10. Evaluación de bioseguridad: genes de resistencia y de virulencia (ResFinder y ABRicate)

Una cepa que se propone como bioinoculante se libera al ambiente en grandes cantidades, por lo que no debería portar genes **adquiridos** de resistencia a antimicrobianos que pueda transferir a otras bacterias, ni factores de virulencia que la hagan un riesgo para personas, animales o plantas. Esta evaluación es especialmente importante en géneros que incluyen patógenos oportunistas (*Enterobacter*, *Klebsiella*, *Pseudomonas*, *Burkholderia*).

### Genes adquiridos de resistencia con ResFinder

```bash
cd ~/genomics/annotation/resfinder

conda activate resfinder

run_resfinder.py -ifa ~/genomics/assembly/nanopore/m01_genome_final.fasta -o m01_resfinder -s "Other" --acquired -l 0.6 -t 0.8 -db_res "$CGE_RESFINDER_RES_PATH" --disinfectant -db_disinf "$CGE_DISINFINDER_PATH"
```

> **Comentario (comandos):**
> - `-ifa`: genoma ensamblado en formato FASTA.
> - `-o m01_resfinder`: carpeta de salida.
> - `-s "Other"`: especie. Se usa `"Other"` cuando la especie no tiene un panel propio en ResFinder, que es lo habitual en bacterias ambientales.
> - `--acquired`: busca genes adquiridos de resistencia.
> - `-db_res "$CGE_RESFINDER_RES_PATH"`: ruta a la base de datos de ResFinder; la variable se define automáticamente al activar el entorno (`/data/db/resfinder/resfinder_db`).
> - `-l 0.6 -t 0.8`: cobertura mínima de 60 % e identidad mínima de 80 % respecto al gen de referencia. Son los valores por defecto; se indican explícitamente para dejar constancia de los parámetros usados.
> - `--disinfectant -db_disinf "$CGE_DISINFINDER_PATH"`: busca además genes de resistencia a desinfectantes y biocidas (DisinFinder).
> - **Mutaciones cromosómicas (`--point`):** PointFinder solo está disponible para algunas especies de importancia clínica (por ejemplo, *Escherichia coli*, *Klebsiella*, *Salmonella*, *Staphylococcus aureus*, *Enterococcus faecalis*, *Enterococcus faecium*). Si su cepa pertenece a una de ellas, indique la especie y añada `--point -db_point "$CGE_RESFINDER_POINT_PATH"`, por ejemplo: `-s "Escherichia coli" --acquired --point`.

```bash
ls -lh m01_resfinder

drwxrwxr-x 3 alumno01 alumno01 4,0K oct  3 10:14 disinfinder_blast
-rw-rw-r-- 1 alumno01 alumno01    0 oct  3 10:14 DisinFinder_Hit_in_genome_seq.fsa
-rw-rw-r-- 1 alumno01 alumno01    0 oct  3 10:14 DisinFinder_Resistance_gene_seq.fsa
-rw-rw-r-- 1 alumno01 alumno01   28 oct  3 10:14 DisinFinder_results_table.txt
-rw-rw-r-- 1 alumno01 alumno01  135 oct  3 10:14 DisinFinder_results_tab.txt
-rw-rw-r-- 1 alumno01 alumno01    0 oct  3 10:14 DisinFinder_results.txt
-rw-rw-r-- 1 alumno01 alumno01  31K oct  3 10:14 m01_genome_final.json
-rw-rw-r-- 1 alumno01 alumno01 5,1K oct  3 10:14 pheno_table.txt
drwxrwxr-x 2 alumno01 alumno01 4,0K oct  3 10:14 pointfinder_blast
drwxrwxr-x 3 alumno01 alumno01 4,0K oct  3 10:14 resfinder_blast
-rw-rw-r-- 1 alumno01 alumno01 1,3K oct  3 10:14 ResFinder_Hit_in_genome_seq.fsa
-rw-rw-r-- 1 alumno01 alumno01 1,2K oct  3 10:14 ResFinder_Resistance_gene_seq.fsa
-rw-rw-r-- 1 alumno01 alumno01  706 oct  3 10:14 ResFinder_results_table.txt
-rw-rw-r-- 1 alumno01 alumno01  233 oct  3 10:14 ResFinder_results_tab.txt
-rw-rw-r-- 1 alumno01 alumno01 4,9K oct  3 10:14 ResFinder_results.txt

cat m01_resfinder/ResFinder_results_tab.txt

Resistance gene Identity        Alignment Length/Gene Length    Coverage        Position in reference   Contig  Position in contig      Phenotype       Accession no.
tet(35) 99.01   1110/1110       100.0   1..1110 m01_001 2348230..2349339        Doxycycline, Tetracycline       AF353562

head -n 30 m01_resfinder/pheno_table.txt

# ResFinder phenotype results.
# Sample: m01_genome_final.fasta
# 
# The phenotype 'No resistance' should be interpreted with
# caution, as it only means that nothing in the used
# database indicate resistance, but resistance could exist
# from 'unknown' or not yet implemented sources.
# 
# The 'Match' column stores one of the integers 0, 1, 2, 3.
#      0: No match found
#      1: Match < 100% ID AND match length < ref length
#      2: Match = 100% ID AND match length < ref length
#      3: Match = 100% ID AND match length = ref length
# If several hits causing the same resistance are found,
# the highest number will be stored in the 'Match' column.

# Antimicrobial Class   WGS-predicted phenotype Match   Genetic background
gentamicin      aminoglycoside  No resistance   0
tobramycin      aminoglycoside  No resistance   0
streptomycin    aminoglycoside  No resistance   0
amikacin        aminoglycoside  No resistance   0
isepamicin      aminoglycoside  No resistance   0
dibekacin       aminoglycoside  No resistance   0
kanamycin       aminoglycoside  No resistance   0
neomycin        aminoglycoside  No resistance   0
lividomycin     aminoglycoside  No resistance   0
paromomycin     aminoglycoside  No resistance   0
ribostamycin    aminoglycoside  No resistance   0
unknown aminoglycoside  aminoglycoside  No resistance   0
butiromycin     aminoglycoside  No resistance   0
```

> **Comentario (comandos):**
> - `ls -lh m01_resfinder`: lista los archivos de salida.
> - `cat .../ResFinder_results_tab.txt`: muestra la tabla de genes de resistencia encontrados.
> - `head -n 30 .../pheno_table.txt`: muestra las primeras 30 líneas de la tabla de fenotipos. Para ver solo los antimicrobianos con resistencia predicha, puede usar `grep -w "Resistant" m01_resfinder/pheno_table.txt`.

> **Comentario (resultados):**
> - **Archivos:** los que empiezan con `ResFinder_` corresponden a los genes de resistencia a antimicrobianos y los que empiezan con `DisinFinder_`, a desinfectantes. En cada grupo, `_results_tab.txt` es la tabla de resultados, `_Hit_in_genome_seq.fsa` contiene la secuencia de cada gen encontrado en el genoma y `_Resistance_gene_seq.fsa`, la del gen de referencia. `pheno_table.txt` es la tabla de fenotipos y el archivo `.json` reúne todos los resultados. Las carpetas `*_blast` guardan los alineamientos.
> - **DisinFinder:** los archivos de secuencias pesan 0 bytes: no se encontró ningún gen de resistencia a desinfectantes.
> - **`ResFinder_results_tab.txt`:** se encontró **un gen adquirido de resistencia**, `tet(35)`:
>   - `Identity` 99,01 %: casi idéntico al gen de referencia.
>   - `Alignment Length/Gene Length` 1110/1110 y `Coverage` 100 %: el gen está completo.
>   - `Position in reference` 1..1110: el alineamiento cubre la referencia de principio a fin.
>   - `Contig` y `Position in contig`: está en `m01_001`, entre las posiciones 2 348 230 y 2 349 339, es decir, **en el cromosoma 1**, no en un plásmido.
>   - `Phenotype`: confiere resistencia a doxiciclina y tetraciclina.
>   - `Accession no.` AF353562: código de acceso de la secuencia de referencia en GenBank.
> - `tet(35)` codifica una bomba de eflujo de tetraciclinas descrita originalmente en una especie de *Vibrio*; su ubicación cromosómica en un genoma de este género es coherente con ese origen.
> - **`pheno_table.txt`:** las líneas con `#` explican cómo leer la tabla. Luego hay una fila por antimicrobiano, con su clase, el fenotipo predicho (`Resistant` o `No resistance`), el nivel de coincidencia (`Match`, de 0 a 3) y el gen que lo explica (`Genetic background`). Las 30 primeras líneas solo alcanzan a mostrar los aminoglucósidos, todos sin resistencia; las filas de tetraciclina y doxiciclina, más abajo, indican `Resistant` con `tet(35)`.
> - Como advierte el propio encabezado, `No resistance` solo significa que no hay nada en la base de datos que indique resistencia: no garantiza que la bacteria sea sensible.
> - **Fíjese en qué contig está cada gen.** Un gen de resistencia ubicado en un plásmido tiene mayor riesgo de transferencia horizontal que uno ubicado en el cromosoma; en la sección 11 se determina qué contigs son plásmidos.

### Comparar con los genes de resistencia anotados por Bakta

```bash
grep -i -E "lactamase|resistance|efflux" ~/genomics/annotation/bakta/m01_bakta/m01.tsv | cut -f 1,6,7,8 | head -n 30

m01_001 M01_00131       crcB    fluoride efflux transporter CrcB
m01_001 M01_00140       araJ    Bcr/CflA family efflux transporter
m01_001 M01_00149               Threonine efflux protein
m01_001 M01_00189       rhtB    homoserine/homoserine lactone efflux protein
m01_001 M01_00217       dinF    MATE family efflux transporter DinF
m01_001 M01_00324       fieF    CDF family cation-efflux transporter FieF
m01_001 M01_00326       norM    Multidrug resistance protein NorM
m01_001 M01_00408       kefG    glutathione-regulated potassium-efflux system ancillary protein KefG
m01_001 M01_00409       kefB    glutathione-regulated potassium-efflux system protein KefB
m01_001 M01_00710       corB    Magnesium/cobalt efflux protein
m01_001 M01_00771               RND efflux pump membrane fusion protein barrel-sandwich domain-containing protein
m01_001 M01_00772               Putative multidrug resistance protein
m01_001 M01_00797       vmrA    sodium-coupled multidrug efflux MATE transporter VmrA
m01_001 M01_00877       tehA    Tellurite resistance protein
m01_001 M01_00920       terB    Tellurite resistance TerB family protein
m01_001 M01_01278               RND efflux pump membrane fusion protein barrel-sandwich domain-containing protein
m01_001 M01_01384       gloB    Metallo-beta-lactamase domain-containing protein
m01_001 M01_01522       mdlB    Multidrug resistance-like ATP-binding protein MdlB
m01_001 M01_01569               Multidrug efflux SMR transporter
m01_001 M01_01573       fos     fosfomycin resistance glutathione transferase
m01_001 M01_01591       norM    Multidrug resistance protein NorM
m01_001 M01_01859       norM    Multidrug resistance protein NorM
m01_001 M01_01911       hdeD    HdeD family acid-resistance protein
m01_001 M01_01925               Efflux RND transporter periplasmic adaptor subunit
m01_001 M01_02108       norM    Multidrug resistance protein NorM
m01_001 M01_02114       araJ    Bcr/CflA family multidrug efflux MFS transporter
m01_001 M01_02145       mdtL    Multidrug resistance protein MdtL
m01_001 M01_02162               efflux RND transporter permease subunit
m01_001 M01_02165               RND efflux pump membrane fusion protein barrel-sandwich domain-containing protein
m01_001 M01_02187               Chlorhexidine efflux transporter domain-containing protein
```

> **Comentario (comandos de procesamiento de la tabla):**
> - `grep -i -E "lactamase|resistance|efflux"`: busca las filas que contengan cualquiera de las tres palabras. `-i` ignora mayúsculas y minúsculas; `-E` permite usar la barra vertical (`|`) como "o".
> - `cut -f 1,6,7,8`: conserva las columnas del contig, el locus tag, el gen y el producto.
> - `head -n 30`: muestra solo las primeras 30 filas (hay más).

> **Comentario (resultados):**
> - La búsqueda por palabras devuelve muchos más resultados que ResFinder, pero **la mayoría no son genes de resistencia a antibióticos**: transportadores de eflujo de fluoruro (`crcB`), de treonina y homoserina (`rhtB`), de potasio (`kefB`, `kefG`) o de magnesio y cobalto (`corB`), resistencia a telurito (`tehA`, `terB`) o a la acidez (`hdeD`), y una proteína con un dominio metalo-betalactamasa (`gloB`), que en realidad es una glioxalasa. Una búsqueda por palabras clave siempre debe revisarse fila por fila.
> - Sí son relevantes las bombas de eflujo de múltiples fármacos (`norM`, `vmrA`, `mdtL`, sistemas RND) y `fos` (resistencia a fosfomicina). Son genes **intrínsecos**: forman parte del cromosoma de la especie, no se adquirieron recientemente y por eso ResFinder no los reporta.
> - Recuerde que, durante la anotación, Bakta indicó `amrfinder: 3`: la base de datos especializada AMRFinderPlus reconoció solo 3 genes de resistencia en todo el genoma.
> - Conclusión: ResFinder (genes adquiridos) y la anotación general (genes intrínsecos y bombas de eflujo) responden preguntas distintas y se complementan.

### Genes de virulencia con ABRicate (VFDB)

```bash
cd ~/genomics/annotation/virulence

conda activate abricate

abricate --list

DATABASE        SEQUENCES       DBTYPE  DATE
vfdb    4392    nucl    2026-Oct-3
plasmidfinder   488     nucl    2026-Oct-3
ecoli_vf        2701    nucl    2026-Oct-3
resfinder       3206    nucl    2026-Oct-3
ncbi    5386    nucl    2024-Dec-15
ecoh    597     nucl    2026-Oct-3
argannot        2223    nucl    2024-Dec-15
megares 6635    nucl    2024-Dec-15
card    6059    nucl    2026-Oct-3

abricate --db vfdb --minid 80 --mincov 80 --threads 10 ~/genomics/assembly/nanopore/m01_genome_final.fasta > m01_vfdb.tab

cut -f 2,3,4,6,10,11,14 m01_vfdb.tab | column -t -s $'\t' | head -n 30

SEQUENCE  START    END      GENE  %COVERAGE  %IDENTITY  PRODUCT
m01_001   1066783  1067151  cheY  100.00     86.18      (cheY) chemotaxis protein CheY [Flagella (VF0519) - Motility (VFC0204)] [Vibrio cholerae O1 biovar El Tor str. N16961]
m01_001   1073402  1073896  cheW  100.00     80.00      (cheW) purine-binding chemotaxis protein CheW [Flagella (VF0519) - Motility (VFC0204)] [Vibrio cholerae O1 biovar El Tor str. N16961]
m01_001   2439976  2440467  vcrH  100.00     86.79      (vcrH) type III secretion system chaperone VcrH [T3SS1 (VF0408) - Effector delivery system (VFC0086)] [Vibrio parahaemolyticus RIMD 2210633]
m01_001   2447606  2448890  vscN  97.13      81.71      (vscN) type III secretion system ATPase VscN [T3SS1 (VF0408) - Effector delivery system (VFC0086)] [Vibrio parahaemolyticus RIMD 2210633]
m01_001   2465844  2466092  vscF  100.00     85.94      (vscF) type III secretion system needle protein VscF [T3SS1 (VF0408) - Effector delivery system (VFC0086)] [Vibrio parahaemolyticus RIMD 2210633]
m01_003   13218    14534    pirB  100.00     100.00     (pirB) Photorhabdus insect-related toxin subunit PirB [PirAB (VF1362) - Exotoxin (VFC0235)] [Vibrio parahaemolyticus str. 3HP]
m01_003   14547    14882    pirA  100.00     100.00     (pirA) Photorhabdus insect-related toxin subunit PirA [PirAB (VF1362) - Exotoxin (VFC0235)] [Vibrio parahaemolyticus str. 3HP]

abricate --summary m01_vfdb.tab

#FILE   NUM_FOUND       cheW    cheY    pirA    pirB    vcrH    vscF    vscN
/home/alumno01/genomics/assembly/nanopore/m01_genome_final.fasta        7       100.00  100.00  100.00  100.00  100.00  100.00  97.13

conda deactivate
```

> **Comentario (comandos):**
> - `abricate --list`: muestra las bases de datos instaladas.
> - `--db vfdb`: usa VFDB (*Virulence Factors Database*), una base de datos de factores de virulencia de bacterias patógenas.
> - `--minid 80 --mincov 80`: identidad y cobertura mínimas (son los valores por defecto; se indican explícitamente).
> - `--threads 10`: número de hilos.
> - `> m01_vfdb.tab`: guarda el resultado en una tabla separada por tabulaciones, con una fila por gen encontrado y 15 columnas: `#FILE` (archivo analizado), `SEQUENCE` (contig), `START` y `END` (coordenadas), `STRAND` (hebra), `GENE` (gen de la base de datos), `COVERAGE` (posiciones del gen cubiertas), `COVERAGE_MAP` (representación visual del alineamiento), `GAPS`, `%COVERAGE`, `%IDENTITY`, `DATABASE`, `ACCESSION`, `PRODUCT` y `RESISTANCE` (vacía en VFDB).
> - `cut -f 2,3,4,6,10,11,14`: extrae las columnas más informativas (contig, inicio, fin, gen, % de cobertura, % de identidad y producto).
> - `column -t -s $'\t'`: alinea las columnas para que la tabla se lea bien en pantalla; `-t` crea la tabla y `-s $'\t'` indica que el separador es la tabulación.
> - `abricate --summary`: resume en una fila por genoma el número de genes encontrados (`NUM_FOUND`) y el % de cobertura de cada gen; es útil para comparar varios genomas.

> **Comentario (resultados):**
> - **`abricate --list`:** hay 9 bases de datos; para cada una se indica el número de secuencias, el tipo (`nucl`: nucleótidos) y la fecha de actualización. VFDB contiene 4 392 genes de virulencia.
> - **Se encontraron 7 genes de virulencia**, de tres tipos. En la columna `PRODUCT`, el primer corchete indica el factor de virulencia y su categoría, y el segundo, el organismo del que proviene la secuencia de referencia:
>   - **`cheY` y `cheW`** (cromosoma 1; 86 % y 80 % de identidad con *Vibrio cholerae*): proteínas de quimiotaxis, clasificadas como "motilidad". Las tiene casi cualquier bacteria móvil, sea patógena o no: son funciones de colonización, no toxinas.
>   - **`vcrH`, `vscN` y `vscF`** (cromosoma 1; 82 a 87 % de identidad con *Vibrio parahaemolyticus*): componentes del sistema de secreción tipo III (T3SS1), una "jeringa molecular" que inyecta proteínas efectoras en células eucariotas. Este sí es un sistema característico de bacterias patógenas.
>   - **`pirA` y `pirB`** (contig `m01_003`; **100 % de identidad y de cobertura**): las dos subunidades de la toxina binaria PirAB, clasificada como exotoxina. Es la toxina que causa la enfermedad de la necrosis hepatopancreática aguda (AHPND) en langostinos.
> - **Fíjese en el contig:** los cinco primeros genes están en el cromosoma 1 (`m01_001`), mientras que la toxina está en `m01_003`, uno de los contigs pequeños. La identidad de 100 % con la referencia, frente al 80 a 87 % de los genes cromosómicos, sugiere que la toxina no evolucionó con el resto del genoma, sino que fue **adquirida** recientemente. Se retoma en la sección 11.
> - **`abricate --summary`:** `NUM_FOUND` = 7; todos los genes están cubiertos al 100 %, salvo `vscN` (97,13 %).
> - Este genoma de demostración corresponde, entonces, a una cepa patógena. Es el resultado opuesto al que se espera de un bioinoculante, y por eso sirve de ejemplo de lo que este análisis debe descartar.
> - **Interprete con cuidado en bacterias asociadas a plantas.** VFDB se construyó a partir de patógenos, y muchos de sus "factores de virulencia" son funciones de colonización que también usan las bacterias benéficas: flagelo y quimiotaxis, fimbrias y adhesinas, sideróforos, sistemas de secreción. Lo que debe preocupar es la presencia de **toxinas** y de sistemas de secreción con efectores característicos de patógenos.
> - ABRicate compara nucleótidos: si su cepa es lejana a los patógenos de VFDB puede no detectar homólogos divergentes. Una tabla vacía no demuestra ausencia de virulencia.

> **Punto de control:** ¿Su cepa porta genes adquiridos de resistencia o factores de virulencia? ¿Cuáles de los genes de VFDB corresponden realmente a toxinas y cuáles a funciones de colonización? Con esta evidencia, ¿consideraría segura la cepa para su uso como bioinoculante, o qué análisis adicionales pediría?

## 11. Identificación de plásmidos y elementos genéticos móviles (MOB-suite y MobileElementFinder)

Los plásmidos y los elementos móviles son los vehículos de la transferencia horizontal de genes. Interesan por dos razones: pueden portar los genes de resistencia o virulencia de la sección 10 y también genes PGP (por ejemplo, los genes de fijación de nitrógeno y nodulación de los rizobios suelen estar en plásmidos o islas simbióticas).

### Identificación y tipificación de plásmidos con MOB-suite

```bash
cd ~/genomics/annotation/plasmid

conda activate mob

mob_recon --infile ~/genomics/assembly/nanopore/m01_genome_final.fasta --outdir m01_plasmid --num_threads 10 --force

ls -lh m01_plasmid

-rw-rw-r-- 1 alumno01 alumno01  843 oct  3 10:24 biomarkers.blast.txt
-rw-rw-r-- 1 alumno01 alumno01 5,6M oct  3 10:24 chromosome.fasta
-rw-rw-r-- 1 alumno01 alumno01 1,5K oct  3 10:24 contig_report.txt
-rw-rw-r-- 1 alumno01 alumno01 9,5K oct  3 10:24 mge.report.txt
-rw-rw-r-- 1 alumno01 alumno01 1,1K oct  3 10:24 mobtyper_results.txt
-rw-rw-r-- 1 alumno01 alumno01  68K oct  3 10:24 plasmid_AD413.fasta
-rw-rw-r-- 1 alumno01 alumno01 138K oct  3 10:24 plasmid_AE795.fasta

cut -f 2,3,5,6,7,8 m01_plasmid/contig_report.txt | column -t -s $'\t'

molecule_type  primary_cluster_id  contig_id  size     gc                   md5
chromosome     -                   m01_001    3620046  0.45565498338971383  ab3305ab19034af2b5bb4378f5ddd0e6
chromosome     -                   m01_002    2216041  0.4542068490610056   683fad9c4d08882995585b0d374d10b3
plasmid        AE795               m01_003    73423    0.45751331326695993  7002bae1c33d13b9a12c844b53b14934
plasmid        AD413               m01_004    69340    0.44936544563022784  352832786a442f1f1dc995e791e5f382
plasmid        AE795               m01_005    66980    0.42441027172290235  5b7feb13b6c297143c554f216f4ab9bc
chromosome     -                   m01_006    22422    0.45053964855945056  7576057183e73eb2466466eb48dad006

cut -f 1,2,3,6,8,10,14,17 m01_plasmid/mobtyper_results.txt | column -t -s $'\t'

sample_id               num_contigs  size    rep_type(s)                       relaxase_type(s)  mpf_type  predicted_mobility  mash_neighbor_identification
m01_genome_final:AE795  2            140403  rep_cluster_1486,rep_cluster_557  MOBC              -         mobilizable         Vibrio parahaemolyticus
m01_genome_final:AD413  1            69340   rep_cluster_1486                  MOBP              -         mobilizable         Vibrio campbellii

conda deactivate
```

> **Comentario (comandos):**
> - `mob_recon`: clasifica cada contig del ensamblaje como cromosoma o plásmido y agrupa los contigs que pertenecen al mismo plásmido.
> - `--infile`: genoma ensamblado en formato FASTA.
> - `--outdir m01_plasmid`: carpeta de salida; `--force` la sobrescribe si ya existe.
> - `--num_threads 10`: número de hilos.
> - `ls -lh m01_plasmid`: lista los archivos generados.
> - `cut -f 2,3,5,6,7,8 ... | column -t -s $'\t'`: extrae de `contig_report.txt` las columnas del tipo de molécula, el identificador del plásmido, el contig, el tamaño, el contenido GC y el código MD5, y las alinea en pantalla.
> - `cut -f 1,2,3,6,8,10,14,17 ...`: extrae de `mobtyper_results.txt` las columnas más informativas de cada plásmido (el archivo completo tiene más de 20).

> **Comentario (resultados):**
> - **Archivos:** `chromosome.fasta` (5,6 MB) contiene los contigs clasificados como cromosoma y cada `plasmid_<código>.fasta`, los de un plásmido; `contig_report.txt` tiene una fila por contig y `mobtyper_results.txt`, una por plásmido; `mge.report.txt` lista los elementos repetitivos y móviles de cada contig, y `biomarkers.blast.txt`, los genes marcadores de plásmidos encontrados (replicasas y relaxasas).
> - **`contig_report.txt`:**
>   - `m01_001` (3 620 046 pb) y `m01_002` (2 216 041 pb) son **cromosomas**: es la organización típica del género *Vibrio*, con un cromosoma grande y uno pequeño.
>   - `m01_003` (73 423 pb), `m01_004` (69 340 pb) y `m01_005` (66 980 pb) son **plásmidos**.
>   - `m01_006` (22 422 pb) aparece como `chromosome`, pero en MOB-suite esa etiqueta significa solo "no asignado a ningún plásmido": no se le encontró replicón ni relaxasa conocidos, ni parecido con plásmidos de la base de datos. Por su tamaño y sus 28 genes, podría ser un plásmido pequeño no descrito o un fago; habría que revisar su anotación para decidirlo.
>   - `primary_cluster_id`: código del grupo de plásmidos de referencia al que se parece cada contig. `m01_003` y `m01_005` tienen el mismo código (`AE795`), por lo que MOB-suite los reporta como **un solo plásmido**; `m01_004` es otro (`AD413`).
>   - `gc`: el contenido GC como proporción (0,456 = 45,6 %). `m01_005` tiene un GC más bajo (42,4 %) que los cromosomas (45,5 %), una señal frecuente de ADN de origen externo.
>   - `md5`: código que identifica de forma única la secuencia de cada contig.
> - **`mobtyper_results.txt`:**
>   - `sample_id`: nombre del genoma y código del plásmido.
>   - `num_contigs` y `size`: el plásmido `AE795` reúne 2 contigs que suman 140 403 pb (73 423 + 66 980); `AD413` tiene 1 contig de 69 340 pb.
>   - `rep_type(s)`: tipo de replicón, es decir, del sistema que permite al plásmido replicarse de forma autónoma. Los dos plásmidos comparten `rep_cluster_1486`.
>   - `relaxase_type(s)`: tipo de relaxasa, la enzima que inicia la transferencia del plásmido (`MOBC` y `MOBP` son familias de relaxasas).
>   - `mpf_type`: sistema de formación del par conjugativo; el guion indica que no se encontró.
>   - `predicted_mobility`: los dos plásmidos son **movilizables** (`mobilizable`): tienen relaxasa, pero no la maquinaria de conjugación, por lo que solo pueden transferirse con ayuda de un plásmido conjugativo presente en la misma célula. Las otras categorías posibles son **conjugativo** (`conjugative`: se transfiere por sí mismo) y **no movilizable** (`non-mobilizable`).
>   - `mash_neighbor_identification`: especie del plásmido más parecido de la base de datos: *Vibrio parahaemolyticus* para `AE795` y *Vibrio campbellii* para `AD413`.
> - **Cruce con la sección 10:** el contig `m01_003`, que porta la toxina PirAB, es un **plásmido movilizable**: la toxina puede transferirse a otras bacterias. En cambio, el gen de resistencia `tet(35)` y el sistema de secreción tipo III están en el cromosoma 1.
> - La agrupación de `m01_003` y `m01_005` en un solo plásmido es una predicción basada en el parecido con plásmidos de referencia. Si en el ensamblaje los dos contigs son circulares (columna `circ.` de Flye), lo más probable es que sean dos plásmidos distintos de la misma familia.

> **Punto de control:** ¿Qué contigs fueron clasificados como cromosoma y cuáles como plásmido? Con su propio genoma, compare con `assembly_info.txt` de Flye (Semana 05): ¿los contigs pequeños y circulares fueron clasificados como plásmidos? Un contig circular pequeño que MOB-suite no reconoce puede ser un plásmido sin replicón conocido en la base de datos, algo frecuente en bacterias ambientales.

### Identificación de elementos genéticos móviles con MobileElementFinder

```bash
cd ~/genomics/annotation/mobile

conda activate resistance

mefinder find --gff --contig ~/genomics/assembly/nanopore/m01_genome_final.fasta m01_mobile

grep -v "^#" m01_mobile.csv | cut -d ',' -f 2,5,9,10,13,14,15 | column -t -s ','

name             type                  identity  coverage  contig   start    end
ISVvu6           insertion sequence    0.743     0.97      m01_001  1504511  1505602
ISVvu6           insertion sequence    0.743     0.97      m01_001  1510904  1511995
ISVa6            insertion sequence    0.972     1.0       m01_001  1499380  1500470
ISVa6            insertion sequence    0.972     1.0       m01_001  1541800  1542890
ISVa15           insertion sequence    0.901     0.995     m01_001  2369754  2370850
ISVa15           insertion sequence    0.902     0.995     m01_001  77463    78559
ISVa15           insertion sequence    0.902     0.995     m01_001  1547967  1549063
ISVa15           insertion sequence    0.903     0.995     m01_001  2773215  2774311
ISVa15           insertion sequence    0.903     0.995     m01_001  2167789  2168885
ISVa15           insertion sequence    0.903     0.995     m01_001  2148498  2149594
ISVa15           insertion sequence    0.903     0.998     m01_001  1550165  1551261
ISVa15           insertion sequence    0.903     0.995     m01_001  1749816  1750912
ISVa15           insertion sequence    0.903     0.998     m01_001  1827697  1828793
ISVa15           insertion sequence    0.903     0.998     m01_001  2213162  2214258
ISVa15           insertion sequence    0.903     0.995     m01_001  2268137  2269233
ISVa15           insertion sequence    0.903     0.995     m01_001  2572762  2573858
ISVa15           insertion sequence    0.903     0.995     m01_001  3561941  3563037
ISVa15           insertion sequence    0.903     0.998     m01_001  1672848  1673944
ISVa15           insertion sequence    0.903     0.998     m01_001  1701733  1702829
ISVa15           insertion sequence    0.903     0.998     m01_001  1316765  1317861
ISVa6            insertion sequence    0.972     1.0       m01_002  481899   482989
ISVa15           insertion sequence    0.901     0.995     m01_002  1982959  1984055
ISVa15           insertion sequence    0.902     0.995     m01_002  502142   503238
ISVa15           insertion sequence    0.902     0.995     m01_002  1354772  1355868
ISVa15           insertion sequence    0.903     0.995     m01_002  1963510  1964606
ISVa15           insertion sequence    0.903     0.995     m01_002  850868   851964
ISVa15           insertion sequence    0.903     0.995     m01_002  1152338  1153434
ISVa15           insertion sequence    0.903     0.998     m01_002  1603366  1604462
Tn6264           composite transposon  1.0       1.0       m01_003  11554    16980
ISVa15           insertion sequence    0.903     0.995     m01_004  29187    30283
ISVa6            insertion sequence    0.972     1.0       m01_005  34385    35475
ISVa15           insertion sequence    0.902     0.995     m01_005  54246    55342
ISVa15           insertion sequence    0.902     0.995     m01_005  56964    58060
ISVa15           insertion sequence    0.903     0.995     m01_005  41718    42814
ISVa15           insertion sequence    0.903     0.995     m01_005  58888    59984
ISVa15           insertion sequence    0.903     0.998     m01_005  46507    47603
cn_6223_ISVvu6   composite transposon  0.743     0.97      m01_001  1499379  1505602
cn_7485_ISVvu6   composite transposon  0.743     0.97      m01_001  1504510  1511995
cn_31987_ISVvu6  composite transposon  0.743     0.97      m01_001  1510903  1542890
cn_6223_ISVa6    composite transposon  0.972     1.0       m01_001  1499379  1505602
cn_31987_ISVa6   composite transposon  0.972     1.0       m01_001  1510903  1542890
cn_3295_ISVa15   composite transposon  0.902     0.995     m01_001  1547966  1551261
cn_29982_ISVa15  composite transposon  0.903     0.998     m01_001  1672847  1702829
cn_49180_ISVa15  composite transposon  0.903     0.998     m01_001  1701732  1750912
cn_20388_ISVa15  composite transposon  0.903     0.995     m01_001  2148497  2168885
cn_46470_ISVa15  composite transposon  0.903     0.995     m01_001  2167788  2214258
cn_20546_ISVa15  composite transposon  0.903     0.995     m01_002  1963509  1984055
cn_5886_ISVa15   composite transposon  0.903     0.995     m01_005  41717    47603
cn_8836_ISVa15   composite transposon  0.903     0.998     m01_005  46506    55342
cn_3815_ISVa15   composite transposon  0.902     0.995     m01_005  54245    58060
cn_3021_ISVa15   composite transposon  0.902     0.995     m01_005  56963    59984

grep -v "^#" m01_mobile.csv | tail -n +2 | cut -d ',' -f 2,5 | sort | uniq -c | sort -k1,1nr

     29 ISVa15,insertion sequence
      4 ISVa6,insertion sequence
      2 ISVvu6,insertion sequence
      1 cn_20388_ISVa15,composite transposon
      1 cn_20546_ISVa15,composite transposon
      1 cn_29982_ISVa15,composite transposon
      1 cn_3021_ISVa15,composite transposon
      1 cn_31987_ISVa6,composite transposon
      1 cn_31987_ISVvu6,composite transposon
      1 cn_3295_ISVa15,composite transposon
      1 cn_3815_ISVa15,composite transposon
      1 cn_46470_ISVa15,composite transposon
      1 cn_49180_ISVa15,composite transposon
      1 cn_5886_ISVa15,composite transposon
      1 cn_6223_ISVa6,composite transposon
      1 cn_6223_ISVvu6,composite transposon
      1 cn_7485_ISVvu6,composite transposon
      1 cn_8836_ISVa15,composite transposon
      1 Tn6264,composite transposon

conda deactivate
```

> **Comentario (comandos):**
> - `mefinder find`: busca elementos genéticos móviles en el genoma.
> - `--contig`: genoma ensamblado en formato FASTA.
> - `--gff`: genera además un archivo GFF con las coordenadas de los elementos, que puede cargarse como pista en Proksee.
> - `m01_mobile`: prefijo de los archivos de salida (`m01_mobile.csv` y `m01_mobile.gff`).
> - **Primer procesamiento de la tabla:** `m01_mobile.csv` está separado por comas, y sus primeras líneas (con `#`) indican la fecha, la muestra y las versiones del programa. `grep -v "^#"` quita esas líneas; `cut -d ',' -f 2,5,9,10,13,14,15` extrae las columnas 2, 5, 9, 10, 13, 14 y 15 indicando que el separador es la coma (`-d ','`); y `column -t -s ','` las alinea en pantalla.
> - **Segundo procesamiento:** `tail -n +2` quita el encabezado; `cut -d ',' -f 2,5` deja el nombre y el tipo de cada elemento; `sort | uniq -c` cuenta cuántas copias hay de cada uno, y `sort -k1,1nr` ordena de mayor a menor.

> **Comentario (resultados):**
> - La tabla completa tiene 16 columnas: `mge_no`, `name`, `synonyms`, `prediction`, `type`, `allele_len`, `depth`, `e_value`, `identity`, `coverage`, `gaps`, `substitution`, `contig`, `start`, `end` y `cigar`. Aquí se muestran el nombre del elemento, su tipo, la identidad y la cobertura respecto a la referencia (de 0 a 1), el contig y las coordenadas.
> - **Secuencias de inserción (IS):** son los elementos móviles más simples: un gen de transposasa flanqueado por repeticiones, de unos 1 100 pb (compare `start` y `end`). El genoma tiene 35 copias de tres IS distintas: **29 de ISVa15**, 4 de ISVa6 y 2 de ISVvu6. Su nombre indica la especie en la que se describieron (`ISVa`: *Vibrio anguillarum*; `ISVvu`: *Vibrio vulnificus*) y un número correlativo.
> - **Identidad:** ISVa6 tiene 97 % de identidad con la referencia; ISVa15, 90 %; e ISVvu6, solo 74 %: las copias de este genoma son variantes divergentes de esa IS, no la misma secuencia.
> - **Distribución:** hay IS en los dos cromosomas y en los plásmidos. El plásmido `m01_005` tiene 6 copias en solo 67 kb, una densidad muy superior a la de los cromosomas: los plásmidos suelen acumular IS.
> - **Transposones compuestos predichos (`cn_...`):** cuando dos copias de la misma IS están cerca, pueden movilizar juntas el ADN que queda entre ellas. MobileElementFinder reporta esas regiones como transposones compuestos **putativos**; el nombre indica su longitud y la IS que los flanquea (`cn_6223_ISVa6`: 6 223 pb entre dos copias de ISVa6). Son predicciones basadas solo en la cercanía, no elementos demostrados; por eso un mismo par de IS puede aparecer en varias filas.
> - **`Tn6264`** es distinto: es un transposón compuesto **conocido**, con identidad y cobertura de 1,0 (100 %), ubicado en el plásmido `m01_003` entre las posiciones 11 554 y 16 980.
> - **Cruce con la sección 10:** los genes de la toxina, `pirB` y `pirA`, están en `m01_003` entre las posiciones 13 218 y 14 882, es decir, **dentro de Tn6264**. La toxina está, entonces, en un transposón, dentro de un plásmido movilizable: dos niveles de movilidad que explican cómo este factor de virulencia se disemina entre cepas y especies de *Vibrio*.
> - La base de datos de MobileElementFinder se construyó sobre todo con bacterias de importancia clínica; en bacterias ambientales es normal obtener pocos resultados o ninguno.

> **Punto de control:** Cruce las coordenadas de las secciones 7, 10 y 11, como se hizo aquí con la toxina PirAB. ¿Los genes de resistencia o de virulencia de su cepa están en un plásmido o cerca de una secuencia de inserción? ¿Algún gen PGP está en un plásmido? Un rasgo codificado en un plásmido puede perderse o transferirse, lo que afecta tanto la estabilidad del bioinoculante como su bioseguridad.

## 12. Genómica comparativa con OrthoVenn3

OrthoVenn3 agrupa las proteínas de varios genomas en clústeres de ortólogos y permite responder: ¿qué genes comparte mi cepa con el resto del género (núcleo) y cuáles son exclusivos de ella?

### Reunir los proteomas

```bash
cd ~/genomics/annotation/orthovenn

cp ~/genomics/annotation/bakta/m01_bakta/m01.faa .

conda activate quality

seqkit stats *.faa

file                     format  type     num_seqs    sum_len  min_len  avg_len  max_len
m01.faa                  FASTA   Protein     5,512  1,733,312       29    314.5    3,828
Vcampbellii_BoB-53.faa   FASTA   Protein     4,836  1,541,149       21    318.7    6,211
Vcampbellii_HJ-2023.faa  FASTA   Protein     5,399  1,758,006       15    325.6    6,211

conda deactivate
```

> **Comentario (comandos):**
> - `cp ... .`: copia el proteoma anotado por Bakta a la carpeta actual (el punto final significa "aquí"), que debe contener además los proteomas de referencia que subió en la sección 7.
> - `seqkit stats *.faa`: calcula estadísticas de todos los archivos terminados en `.faa`.

> **Comentario (resultados):**
> - Para cada archivo se indica el formato (`FASTA`), el tipo de secuencia (`Protein`), el número de secuencias (`num_seqs`), la suma de sus longitudes en aminoácidos (`sum_len`) y la longitud mínima, promedio y máxima.
> - Los tres proteomas son comparables: 5 512 proteínas en `m01`, 4 836 en `Vcampbellii_BoB-53` y 5 399 en `Vcampbellii_HJ-2023`, con una longitud promedio de 315 a 326 aminoácidos (el valor típico en bacterias es de unos 300 a 330).
> - `Vcampbellii_BoB-53` tiene unas 600 proteínas menos que los otros dos: es un genoma más pequeño, por lo que se espera que tenga menos clústeres exclusivos.
> - Este paso sirve de control: un proteoma con muchas menos proteínas que los demás indicaría un archivo incompleto o equivocado.

### Descargar los archivos `.faa` con WinSCP, ir a OrthoVenn3 (https://orthovenn3.bioinfotoolkits.net/), cargar un archivo por cada genoma (*Upload*), asignarle un nombre corto a cada uno y enviar el análisis con el algoritmo OrthoFinder y los parámetros por defecto

<img width="3010" height="1489" alt="image" src="https://github.com/user-attachments/assets/213bccf2-6792-40ae-b923-d5d1ff4cd653" />

**Figura 33.** Formulario de OrthoVenn3 con los tres proteomas cargados, uno por genoma, cada uno con su nombre corto. <!-- VERIFICAR: leyenda escrita sin ver la captura -->

<img width="3007" height="1008" alt="image" src="https://github.com/user-attachments/assets/cdd2340e-53e3-4e5d-9e77-b17861badb62" />

**Figura 34.** Parámetros del análisis: algoritmo de agrupamiento (OrthoFinder), umbral de e-value y valor de inflación, y botón de envío. <!-- VERIFICAR: leyenda escrita sin ver la captura -->

> **Comentario (figuras 33 y 34):**
> - Se carga **un archivo de proteínas por genoma**. Con 4 o 5 genomas el diagrama de Venn todavía es legible; con más, use el gráfico UpSet.
> - `OrthoFinder` es el algoritmo recomendado por su mejor precisión; `OrthoMCL` es la alternativa clásica.
> - El **e-value** es el umbral de significancia de las comparaciones entre proteínas, y el valor de **inflación** controla qué tan finos son los clústeres (a mayor inflación, clústeres más pequeños y numerosos). Deje los valores por defecto.
> - El análisis demora varios minutos; guarde el identificador del trabajo (*job ID*) para volver a consultar los resultados.

### Analizar los resultados obtenidos

<img width="1558" height="1323" alt="image" src="https://github.com/user-attachments/assets/bbaa5e23-8a85-4f35-a413-71e82364d392" />

**Figura 35.** Diagrama de Venn de los clústeres de ortólogos de los tres genomas. El número de la región central corresponde a los clústeres compartidos por todos y los de las regiones externas, a los exclusivos de cada genoma. <!-- VERIFICAR: leyenda escrita sin ver la captura -->

<img width="990" height="1283" alt="image" src="https://github.com/user-attachments/assets/ee4a48f5-e21d-45f5-b188-852cee8ba142" />

**Figura 36.** Resumen por genoma: número de proteínas, de clústeres y de *singletons* (proteínas sin ortólogos en los otros genomas). <!-- VERIFICAR: leyenda escrita sin ver la captura -->

<img width="2512" height="545" alt="image" src="https://github.com/user-attachments/assets/ed8713e5-b9d5-48d2-884f-94b3b04cd141" />

**Figura 37.** Gráfico UpSet: cada barra indica el número de clústeres de una combinación de genomas, señalada por los puntos conectados de la parte inferior. <!-- VERIFICAR: leyenda escrita sin ver la captura -->

<img width="3024" height="765" alt="image" src="https://github.com/user-attachments/assets/cb36d2db-3459-4a96-89a5-9338058539c4" />

**Figura 38.** Tabla de clústeres de la sección seleccionada del diagrama, con su identificador, el número de proteínas y su anotación funcional. <!-- VERIFICAR: leyenda escrita sin ver la captura -->

<img width="3023" height="1357" alt="image" src="https://github.com/user-attachments/assets/86c8d5ab-803b-4d75-9580-7714e32a4e3f" />

**Figura 39.** Enriquecimiento de términos GO en los clústeres seleccionados: funciones que están más representadas de lo esperado por azar. <!-- VERIFICAR: leyenda escrita sin ver la captura -->

<img width="3024" height="1354" alt="image" src="https://github.com/user-attachments/assets/a4b02ca5-e551-4bde-8931-f4d069211ede" />

**Figura 40.** Árbol filogenético de los tres genomas, construido con los genes de copia única. <!-- VERIFICAR: leyenda escrita sin ver la captura -->

> **Comentario (figuras 35 a 40):**
> - **Clúster:** grupo de proteínas ortólogas (y parálogas recientes) de uno o más genomas. Los números del diagrama cuentan **clústeres**, no proteínas: un clúster puede contener varias proteínas del mismo genoma.
> - **Clústeres compartidos por todos los genomas** (región central del Venn): aproximación al genoma núcleo (*core*) del género; suelen ser funciones esenciales. Entre cepas del mismo género, esta es la región con el número más alto.
> - **Clústeres compartidos por algunos genomas:** genoma accesorio.
> - **Clústeres exclusivos de un genoma** y ***singletons***: genes propios de la cepa; pueden reflejar adaptación a su nicho, elementos móviles (fagos, plásmidos) o, a veces, artefactos de la anotación. En `m01`, es de esperar que las proteínas de los plásmidos (incluida la toxina PirAB) estén en este grupo si las cepas de referencia no los tienen.
> - **Gráfico UpSet:** muestra la misma información que el Venn, pero se lee mejor cuando hay muchos genomas.
> - **Tabla de clústeres y enriquecimiento GO:** al hacer clic en una sección del diagrama se listan sus clústeres con su anotación, y se indica qué funciones (términos GO) están enriquecidas, con su valor de p. La tabla se puede descargar.
> - **Árbol filogenético:** se construye con los genes de copia única compartidos; las cepas más cercanas quedan agrupadas en la misma rama. Compárelo con la clasificación por ANI de la Semana 05.

> **Punto de control:** Busque en la tabla de clústeres los locus tags de los genes PGP que encontró en la sección 7. ¿Están en clústeres compartidos por todo el género (rasgo conservado) o son exclusivos de su cepa (rasgo distintivo)? Revise también qué funciones están enriquecidas en los clústeres exclusivos de su cepa.

## 13. Anotación del genoma ensamblado y validado en la Semana 05

Ahora repita todo el proceso con **su propio genoma**: el ensamblaje que eligió como el mejor en la Semana 05 (Raven o Flye pulido), cuya calidad validó con QUAST, CheckM y BUSCO, y cuyo género y especie identificó con 16S y ANI.

### Filtrar y renombrar los contigs de su ensamblaje (ejemplo con el barcode 01 y el ensamblaje de Flye)

```bash
cd ~/genomics/assembly/nanopore

conda activate genome

grep ">" ~/genomics/assembly/nanopore/flye/b01_flye.fasta

prinseq-lite.pl -fasta ~/genomics/assembly/nanopore/flye/b01_flye.fasta -min_len 500 -seq_id "b01_00" -out_good b01_genome_final -out_bad null

grep ">" b01_genome_final.fasta

conda deactivate
```

> **Comentario:**
> - `-fasta`: su mejor ensamblaje de la Semana 05. Si eligió Raven, la ruta es `~/genomics/assembly/nanopore/raven/b01_raven.fasta`.
> - `-seq_id "b01_00"`: los contigs se llamarán `b01_001`, `b01_002`, ... Reemplace `b01` por el código de su barcode.
> - Los dos `grep` muestran los nombres de los contigs antes y después del cambio. Anote a qué contig original corresponde cada nuevo nombre, para relacionarlo con la columna `circ.` de `assembly_info.txt` (Flye) y con lo observado en Bandage.
> - `b01_genome_final.fasta` es la entrada de todos los análisis.

### Repetir las secciones 2 a 12 con su genoma

> - Reemplace el prefijo `m01` por el de su barcode en todos los comandos, y `--locus-tag M01` por su código en mayúsculas (por ejemplo, `B01`).
> - En Bakta, indique el género y la especie que identificó en la Semana 05 (`--genus`, `--species`).
> - En BUSCO, use el mismo linaje que usó en la Semana 05 y ejecute solo el modo `proteins`; para el gráfico comparativo, copie el resumen JSON del modo `genome` desde `~/genomics/validation/busco/`.
> - En la búsqueda dirigida de genes PGP, adapte la lista de genes al género de su cepa.
> - Como genomas de referencia (PGPg_finder y OrthoVenn3), use la cepa tipo de la especie que identificó por ANI y otras especies del mismo género.
> - Envíe los trabajos en línea (eggNOG-mapper y antiSMASH) apenas tenga la anotación de Bakta: recuerde que eggNOG-mapper solo permite un trabajo en ejecución por correo.
> - En cada sección, interprete sus resultados como se hizo aquí con `m01`: no basta con copiar la salida, hay que explicar qué significa para su cepa.

### Estructura de carpetas esperada al finalizar (ejemplo con el barcode 01)

```bash
~/genomics/assembly/nanopore/
└── b01_genome_final.fasta        # Ensamblaje filtrado y con los contigs renombrados (PRINSEQ)

~/genomics/annotation/
├── bakta/
│   └── b01_bakta/                # Salida completa de Bakta (b01.faa, b01.gff3, b01.gbff, b01.tsv, b01.txt, ...)
├── busco/
│   ├── b01_proteins/             # BUSCO en modo proteins (el modo genome está en ~/genomics/validation/busco/)
│   └── b01_busco_summaries/      # Resúmenes JSON y busco_figure.png
├── eggnog/
│   ├── b01.emapper.annotations   # Resultado descargado de eggNOG-mapper
│   ├── b01_eggnog.tsv            # Tabla sin líneas de comentario
│   ├── b01_cog_counts.txt        # Conteo por categoría COG
│   ├── b01_ko.txt                # Gen y KO de eggNOG-mapper, para KEGG Mapper
│   └── b01_ko_bakta.txt          # Gen y KO de Bakta (alternativa directa)
├── pgp/
│   ├── b01_pgp_genes.txt         # Búsqueda dirigida de genes PGP
│   ├── b01_genomes/              # Genoma propio (b01.fasta) + genomas de referencia (.fna)
│   └── b01_pgpg/                 # Salida de PGPg_finder (tables/, figures/)
├── cazymes/
│   ├── b01_dbcan/                # Salida de run_dbcan CAZyme_annotation (overview.tsv)
│   ├── b01_cazymes_filtered.tsv  # CAZymes detectadas por 2 o más métodos
│   ├── b01_dbcan_cgc/            # Salida de run_dbcan easy_CGC (cgc_standard_out.tsv)
│   └── b01_dbcan_substrate/      # Salida de run_dbcan easy_substrate (substrate_prediction.tsv)
├── antismash/                    # Resultados descargados de antiSMASH (opcional)
├── resfinder/
│   └── b01_resfinder/            # Salida de ResFinder (ResFinder_results_tab.txt, pheno_table.txt)
├── virulence/
│   └── b01_vfdb.tab              # Genes de virulencia (ABRicate, VFDB)
├── plasmid/
│   └── b01_plasmid/              # Salida de MOB-suite (contig_report.txt, mobtyper_results.txt)
├── mobile/
│   ├── b01_mobile.csv            # Elementos genéticos móviles (MobileElementFinder)
│   └── b01_mobile.gff
└── orthovenn/
    ├── b01.faa                   # Proteoma propio
    └── *.faa                     # Proteomas de referencia
```

### Bitácora bioinformática:

Debe incluir las siguientes secciones (**solo con el genoma de su propio barcode**, no con el genoma de demostración `m01`). Cada tabla, figura o captura debe estar numerada, tener una leyenda y estar explicada en el texto:

1. **Carátula** (usar la carátula del modelo de bitácora, con los 6 integrantes del grupo)
2. **Título**
3. **Objetivo de la práctica**
4. **Metodología:** flujograma de los análisis realizados (ensamblaje de la Semana 05 → PRINSEQ → Bakta → BUSCO proteins → COG y eggNOG-mapper/KEGG → búsqueda dirigida y PGPg_finder → run_dbcan → antiSMASH → ResFinder y ABRicate → MOB-suite y MobileElementFinder → OrthoVenn3)
5. **Metodología:** estructura de las carpetas
6. **Metodología:** versión de cada programa y base de datos (`programa --version`; en las herramientas en línea, la versión indicada en la página de resultados y la fecha del análisis) y parámetros principales (por ejemplo, `-min_len` y `-seq_id` de PRINSEQ, el género y la especie en Bakta, el linaje en BUSCO, los umbrales de PGPg_finder, el criterio de filtrado de CAZymes, la especie y los umbrales usados en ResFinder y ABRicate, el algoritmo de OrthoVenn3 y los genomas de referencia usados, con su código de acceso)
7. **Resultados:** correspondencia entre los nombres originales y los nuevos nombres de los contigs, tabla resumen de la anotación de Bakta (tamaño, GC, número de CDS totales y por contig, rRNA, tRNA, proteínas hipotéticas y su porcentaje, densidad codificante) y mapa circular del genoma
8. **Resultados:** completitud del proteoma (BUSCO modo `proteins` comparado con el modo `genome`, con el gráfico de `busco --plot`)
9. **Resultados:** porcentaje de proteínas anotadas por eggNOG-mapper y gráfico de barras de categorías COG, con la interpretación de las categorías más abundantes
10. **Resultados:** número de KO obtenidos (eggNOG-mapper y Bakta) y rutas y módulos de KEGG relevantes para la promoción del crecimiento vegetal (indicar si los módulos están completos)
11. **Resultados:** tabla de genes PGP de la búsqueda dirigida (gen, rasgo, número de copias, locus tag) y mapa de calor de PGPg_finder comparando su cepa con los genomas de referencia
12. **Resultados:** número de CAZymes por clase, familias más abundantes, número de clústeres de genes de CAZymes (CGC) y sustratos predichos, indicando qué tan sólida es cada predicción
13. **Resultados:** tabla de regiones de antiSMASH (tipo, clúster conocido más parecido y % de similitud)
14. **Resultados:** genes de resistencia a antimicrobianos detectados por ResFinder (gen, % de identidad, contig y fenotipo predicho) y genes de virulencia detectados por ABRicate/VFDB, distinguiendo toxinas de funciones de colonización
15. **Resultados:** plásmidos identificados por MOB-suite (tamaño, replicón, relaxasa y movilidad predicha) y elementos genéticos móviles (MobileElementFinder), indicando si algún gen de resistencia, virulencia o PGP se ubica en ellos
16. **Resultados:** diagrama de Venn o UpSet de OrthoVenn3, con el número de clústeres del núcleo, accesorios y exclusivos de su cepa
17. **Discusión:** según la evidencia genómica, ¿qué mecanismos de promoción del crecimiento vegetal podría tener su cepa y cuáles no? ¿Coinciden la búsqueda dirigida, PGPg_finder, KEGG y antiSMASH? ¿Los genes PGP pertenecen al núcleo del género o son propios de su cepa? ¿Qué ensayos de laboratorio propondría para confirmar cada rasgo, y qué consideraciones de bioseguridad tendría antes de proponer la cepa como bioinoculante?
