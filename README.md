Reporte del ensamble y andamiaje del genoma de Trichoderma atroviride IMI206040 con lecturas PacBio e Illumina
==============================================================================================================

[![Join the chat at https://gitter.im/vflorelo-trichoderma/community](https://badges.gitter.im/vflorelo-trichoderma/community.svg)](https://gitter.im/vflorelo-trichoderma/community?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

* Se inició el proceso de re-secuenciación del genoma de *Trichoderma atroviride* IMI206040 con 12 bibliotecas PacBio.
* Se determinó la longitud y %G+C de las lecturas de cada biblioteca, observando que no todas las reacciones de secuenciación contaban con rendimientos adecuados, estimado como número de lecturas y longitud de las mismas

|Biblioteca|Reads|Longitud máxima|Longitud media|Información|
|-|-|-|-|-|
|24D1|6005713|87962|1893.66|[plot](images/24D1.png)|
|24D2|1421683|116707|7015.09|[plot](images/24D2.png)|
|24D3|1645852|104870|6897.54|[plot](images/24D3.png)|
|30D1|8177858|83868|1756.11|[plot](images/30D1.png)|
|30D2-long|1046360|107468|7064.06|[plot](images/30D2-long.png)|
|30D2-short|1396560|62193|2323.17|[plot](images/30D2-short.png)|
|30D3-1|11020|117565|3284.33|[plot](images/30D3-1.png)|
|30D3-2|4087|72870|2410.4|[plot](images/30D3-2.png)|
|CR1_short|359360|76353|4172.8|[plot](images/CR1_short.png)|
|CR2_long|1304985|117056|6919.88|[plot](images/CR2_long.png)|
|CR2_short|2804001|99115|3242.37|[plot](images/CR2_short.png)|
|CR3_long|1509190|115974|6718.48|[plot](images/CR3_long.png)|

Para el ensamble inicial se consideraron unicamente las lecturas de las bibliotecas CR1, CR2 y CR3, se empleó el programa Canu v1.8 [[ref](https://www.ncbi.nlm.nih.gov/pubmed/28298431)], obteniendo un ensamble que fue ordenado manualmente con base en similitud con el genoma de *Trichoderma reesei* QM6A [[ref](https://www.ncbi.nlm.nih.gov/nuccore/?term=CP016232%3ACP016238+%5Bpacc%5D)]. Este ensamble (referido como scf) mostró una contigüidad adecuada, logrando obtener 8 scaffolds correspondientes a los cromosomas de *Trichoderma atroviride*. No obstante, al mapear datos provenientes de RNAseq, se obtuvo una fracción elevada de lecturas no mapeadas que si lograban ser alineadas al genoma de referencia, específicamente a la secuencia correspondiente al contig_18.

Con base en esta información, se procedió a reensamblar el genoma de *Trichoderma atroviride* IMI 206040 y de pulir dichos ensambles con lecturas Illumina y PacBio.

Preparación de un set de lecturas PacBio de alta calidad para el pulido de los ensambles obtenidos
--------------------------------------------------------------------------------------------------------------
Se emplearon únicamente las lecturas provenientes de las bibliotecas [24D2](images/24D2.png), [24D3](images/24D3.png), [30D2-long](images/30D2-long.png), [CR2_long](images/CR2_long.png) y [CR3_long](images/CR3_long.png), mismas que fueron concatenadas, obteniendo ~ 45 Gbp para una profundidad aproximada de 1000X.

Posteriormente las lecturas seleccionadas fueron sometidas a un proceso de corrección empleando Canu [[ref](https://www.ncbi.nlm.nih.gov/pubmed/28298431)].

Generación de ensambles alternativos
------------------------------------
Una vez que se obtuvo el dataset de lecturas corregidas, se procedió a la construcción de 3 ensambles adicionales, v3 empleando wtdbg2 [[ref](https://www.biorxiv.org/content/10.1101/530972v1)], v3.2 empleando Canu [[ref](https://www.ncbi.nlm.nih.gov/pubmed/28298431)] y v3.3 empleando racon [[ref](https://www.ncbi.nlm.nih.gov/pubmed/28100585)]. Las estadísticas de dichos ensambles son las siguientes:
|Metric|v1|scf|v3|v3.2|v3.3|
|------|--|---|--|----|----|
|Assembly length|36.1437|36.3064|45.2236|36.3359|43.4588|
|Longest contig|5.62178|6.6475|4.69221|5.64224|3.58511|
|Contig count|29|23|482|24|322|
|Assembly n50|2.0079|5.65082|1.5878|2.135|1.50516|
|Contigs >1e6 bp|15|8|12|14|16|
|Length  >1e6 bp|33.1471|36.1388|27.4887|33.4079|29.6941|
|Contigs >1e5 bp|20|8|29|18|31|
|Length  >1e5 bp|36.061|36.1388|36.6339|36.2242|36.419|
|Contigs >1e4 bp|22|19|363|24|320|
|Length  >1e4 bp|36.0967|36.2807|44.5593|36.3359|43.4453|

Pulido de los ensambles generados
=================================
Los ensambles generados fueron pulidos con lecturas PacBio empleando racon [[ref](https://www.ncbi.nlm.nih.gov/pubmed/28100585)], y con lecturas Illumina empleando ntedit [[ref](https://www.ncbi.nlm.nih.gov/pubmed/31095290)].

Los ensambles obtenidos tras pulir con lecturas PacBio fueron nombrados pb-polish, mientras que los ensambles pulidos con lecturas Illumina fueron nombrados ilm-polish, adicionalmente se realizaron pulidos dobles, con lecturas PacBio e Illumina denominados pb-ilm-polish.

Evaluación de los ensambles usando lecturas Illumina de DNAseq
--------------------------------------------------------------
Para determinar el efecto de las distintas estrategias de pulido, así como la calidad propia de los nuevos ensambles obtenidos, se emplearon lecturas Illumina de DNAseq, las cuales fueron alineadas a las secuencias generadas. Posteriormente se determinó que fracción de dichas lecturas alineaba adecuadamente a las secuencias (`samtools view -f3`) y que fracción de las lecturas no fue alineada (`samtools view -f12`). Dichas fracciones fueron calculadas tanto para los ensambles generados como para el ensamble de referencia. Asimismo, se determinó que lecturas había en común entre los alineamientos de cada secuencia generada y los alineamientos del genoma de referencia, es decir, cuanto mayor fuera el porcentaje de lecturas adecuadamente pareadas entre el alineamiento de los nuevos ensambles y el alineamiento del genoma de referencia, menos información se estaría perdiendo para dicho ensamble. Del mismo modo, cuanto menor fuera el porcentaje de lecturas compartidas no adecuadamente pareadas, menos información se estaría perdiendo en los nuevos ensambles.


|Sequence|Status|Unmapped read pairs|Shared|Unique|%Shared|%Unique|Properly mapped read pairs|Shared|Unique|%Shared|%Unique|
|-|-|-|-|-|-|-|-|-|-|-|-|
|Trichoderma atroviride v1|raw|1747356|-|-|-|-|-|-|-|-|-|
|Trichoderma atroviride scf|raw|1887134|1730276|156858|99.02|8.31|14674710|14654290|20420|97.82|0.14|
||Pb-polish|1887852|1730252|157600|99.02|8.35|14673524|14652466|21058|97.81|0.14|
||Ilm-polish|1887236|1730660|156576|99.04|8.30|14674120|14654272|19848|97.82|0.14|
||Pb-ilm-polish|1887830|1730424|157406|99.03|8.34|14673194|14652436|20758|97.81|0.14|
|Trichoderma atroviride v3|raw|1880944|1729154|151790|98.96|8.07|14685178|14663290|21888|97.88|0.15|
||Pb-polish|1877492|1723216|154276|98.62|8.22|14688178|14660288|27890|97.86|0.19|
||Ilm-polish|1882324|1730530|151794|99.04|8.06|14684120|14663406|20714|97.88|0.14|
||Pb-ilm-polish|1878088|1723746|154342|98.65|8.22|14687756|14660338|27418|97.86|0.19|
|Trichoderma atroviride v3.2|raw|1534676|1534292|384|87.81|0.03|15219110|14978430|240680|99.98|1.58|
||Pb-polish|1535208|1534812|396|87.84|0.03|15218650|14978414|240236|99.98|1.58|
||Ilm-polish|1535076|1534754|322|87.83|0.02|15219546|14979136|240410|99.99|1.58|
||Pb-ilm-polish|1535124|1534982|142|87.85|0.01|15219434|14979214|240220|99.99|1.58|
|Trichoderma atroviride v3.3|raw|1554104|1535082|19022|87.85|1.22|15194030|14955302|238728|99.83|1.57|
||Pb-polish|1554076|1535072|19004|87.85|1.22|15194452|14955874|238578|99.83|1.57|
||Ilm-polish|1553850|1535104|18746|87.85|1.21|15193558|14955598|237960|99.83|1.57|
||Pb-ilm-polish|1553740|1535008|18732|87.85|1.21|15194692|14956034|238658|99.83|1.57|


Evaluación de los ensambles usando lecturas Illumina de DNAseq
--------------------------------------------------------------
Para evaluar el efecto de las rondas de pulido, se procedió a determinar el número de lecturas RNAseq (Illumina) que no lograron ser alineadas a los ensambles obtenidos pero que si podían ser alineadas a los contigs presentes en el ensamble de referencia. A continuación se muestran los resultados obtenidos:

|Contig|Length|scf||||v3||||v3.2||||v3.3||||
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
|||raw|pb-polish|ilm-polish|pb-ilm-polish|raw|pb-polish|ilm-polish|pb-ilm-polish|raw|pb-polish|ilm-polish|pb-ilm-polish|raw|pb-polish|ilm-polish|pb-ilm-polish|
|contig_1|4710|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|
|contig_2|3162|116|83|116|83|0|1|0|0|116|0|0|0|0|0|0|0|
|contig_3|9774|330|330|330|330|330|330|330|330|330|330|330|330|330|330|330|330|
|contig_4|4483|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|
|contig_5|8316|618|618|618|618|618|618|618|618|2996|4|4|4|4|4|4|4|
|contig_6|7936|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|0|
|contig_7|8555|42|42|42|42|42|42|42|42|208|0|0|0|0|0|0|0|
|contig_8|10219|1374|15|852|15|16|16|16|16|10182|5|5|5|9|9|9|9|
|contig_9|25523|100|101|100|101|29|31|29|31|362|0|0|0|362|362|362|362|
|contig_10|205695|2|3|2|3|41|41|41|41|12721|6|7|6|89|85|84|85|
|contig_11|311306|42|39|40|39|226|223|223|223|99862|224|225|224|7659|7660|7658|7659|
|contig_12|487615|40|38|38|38|447|375|375|375|130055|365|365|365|81|44|42|43|
|contig_13|911962|960|978|952|973|971|967|961|961|319584|178|178|173|1007|1057|954|961|
|contig_14|997338|239|245|239|240|102|117|100|100|269672|129|112|112|818|876|791|834|
|contig_15|1127681|205|223|204|205|181|526|165|166|404067|1015|996|992|237|165|155|148|
|contig_16|1160020|1192|1273|1189|1197|159|247|157|155|403023|233|209|209|274|289|271|285|
|contig_17|1474880|243|288|238|262|301|234|227|226|569309|396|308|316|1717|1746|1711|1724|
|contig_18|1417455|567707|527567|567700|526548|567048|527110|567042|526971|2475587|357415|357539|357447|2160|2226|1822|2100|
|contig_19|1567605|286|268|260|258|414|479|387|410|618119|2531|1867|2502|459|431|374|355|
|contig_20|1588798|1681|1580|1573|1573|1596|1660|1588|1592|538456|1208|1170|1196|1632|1621|1616|1615|
|contig_21|1650603|151|201|147|151|1243|1250|1238|1238|397650|1206|1171|1183|378|364|304|326|
|contig_22|1899297|599|628|793|793|340|368|519|516|701998|506|647|649|494|487|677|676|
|contig_23|2124193|404|295|391|284|1460|1507|1449|1453|504712|288|271|266|1522|1490|1481|1482|
|contig_24|2790066|489|497|465|467|2887|3001|2861|2909|961547|638|582|578|673|581|603|533|
|contig_25|3049622|613|760|545|593|3379|3408|3330|3326|1040010|3271|3240|3243|707|783|646|688|
|contig_26|4122911|2171|2155|2094|2097|2298|2375|2214|2289|1166844|6337|6187|6177|707|809|570|558|
|contig_27|5621775|5873|5965|5815|5894|1112|1116|1049|1054|1948214|2501|2252|2309|5686|5581|5384|5496|
|contig_28|2007903|1985|2057|1968|1967|5216|5313|5189|5215|618354|434|365|359|1881|1967|1849|1864|
|contig_29|1544261|2120|2487|2115|2144|2129|2130|2117|2117|717281|2052|2085|2089|2095|1634|2076|1617|

Como puede apreciarse, las rondas de pulido mejoraron significativamente la calidad del ensamble, con lo que puede proceder el andamiaje de la secuencia que vaya a ser seleccionada para su análisis.

Anexos
======

Comandos para los ensambles obtenidos
-------------------------------------

* Ensamble scf
```bash
#PBS -q ensam
#PBS -l nodes=1:ppn=20,mem=250gb,vmem=250gb,walltime=120:00:00
#PBS -N tricho_canu_2_200_ctrl
#PBS -V
module load canu/1.6
module load minimap2/2.12
cd /LUSTRE/usuario/atriztan/PacBioTricho/pacbio/control
canu -d . -p Trichoderma_atroviride_IMI_ctrl genomeSize=35M corOutCoverage=400 corMinCoverage=2 useGrid=false -pacbio-raw CR1_short.fasta  CR2_long.fasta  CR2_short.fasta  CR3_long.fasta
```

* Obtención de lecturas corregidas
```bash
#PBS -q ensam
#PBS -l nodes=1:ppn=20,mem=250gb,vmem=250gb,walltime=720:00:00
#PBS -N trichoderma_canu_cor_10_1000
#PBS -V
module load canu/1.8
module load java/1.8
module load minimap2/2.12
cd /home/vflores/LUSTRE/Trichoderma_atroviride_scaffolding/scaf/run_07
canu -correct -d . -p Trichoderma_atroviride_LANGEBIO genomeSize=35M corOutCoverage=1000 corMinCoverage=10 useGrid=false -pacbio-raw reads.fasta
```

* Ensamble v3
```bash
#PBS -q ensam
#PBS -l nodes=1:ppn=20,mem=250gb,vmem=250gb,walltime=120:00:00
#PBS -N trichoderma_wtdbg2_cor
#PBS -V
module load wtdbg2/2.1
cd /home/vflores/LUSTRE/Trichoderma_atroviride_scaffolding/scaf/run_07/wtdbg2_assembly
wtdbg2 --cpu 20 --input reads.fasta.gz --force --prefix Trichoderma_atroviride --kmer-fsize 0 --kmer-psize 21 --kmer-depth-max 1000 --kmer-depth-min 10 --kmer-subsampling 4 --aln-kmer-sampling 256 --dp-max-gap 4 --dp-max-var 4 --dp-penalty-gap -7 --dp-penalty-var -21 --aln-min-length 2048 --aln-min-match 200 --aln-max-var 0.2 --aln-dovetail 256 --aln-strand 3 --aln-maxhit 1000 --aln-bestn 500 --verbose --tidy-reads 2000 --node-len 1024 --node-ovl 256 --node-matched-bins 1 --node-drop 0.25 --edge-min 5 --node-min 5 --node-max 1000 --ttr-cutoff-depth 0 --ttr-cutoff-ratio 0.5 --dump-kbm Vanilla_SAGARPA_cor --bubble-step 40 --tip-step 10 --ctg-min-length 1000 --ctg-min-nodes 3 --bin-complexity-cutoff 2
wtpoa-cns -t 20 -i Trichoderma_atroviride.ctg.lay -f -o Trichoderma_atroviride.ctg.fasta -M 2 -X -5 -I -2 -D -4 -B 96 -W 200 -w 100 -A -R 16 -C 3 -F 0.5 -N 20 -v
```

* Ensamble v3.2
```bash
#PBS -q ensam
#PBS -l nodes=1:ppn=16,mem=720gb,vmem=720gb,walltime=72:00:00
#PBS -N trichoderma_canu_10_1000
#PBS -V
module load canu/1.8
module load java/1.8
module load minimap2/2.12
cd /home/vflores/LUSTRE/Trichoderma_atroviride_scaffolding/scaf/run_07/canu_assembly
canu -assemble -d . -p Trichoderma_atroviride_LANGEBIO genomeSize=35M corOutCoverage=1000 corMinCoverage=10 useGrid=false -pacbio-corrected reads.fasta.gz
```

* Ensamble v3.3 (nota, el ensamble v3.3.polish se obtiene directamente en este paso)
```bash
#PBS -q ensam
#PBS -l nodes=1:ppn=20,mem=250gb,vmem=250gb,walltime=120:00:00
#PBS -N trichoderma_minimap2_miniasm_racon_polish
#PBS -V
module load minimap2/2.12
module load miniasm/0.3
module load racon/1.3.1
cd /home/vflores/LUSTRE/Trichoderma_atroviride_scaffolding/scaf/run_07/racon_polish_assembly
minimap2 -x ava-pb -t 20 reads.fasta.gz reads.fasta.gz > minimap2_alignments.paf
miniasm -f reads.fasta.gz minimap2_alignments.paf > fragment_assembly.gfa
awk '$1 ~/S/ {print ">"$2"\n"$3}' fragment_assembly.gfa > fragment_reads.fasta
minimap2 -t 20 fragment_reads.fasta reads.fasta.gz > minimap2_fragment_assembly.paf
racon -t 20 reads.fasta.gz minimap2_fragment_assembly.paf fragment_reads.fasta > racon_contigs.fasta
minimap2 -t 20 racon_contigs.fasta reads.fasta.gz > minimap2_polish_assembly.paf
racon -t 20 reads.fasta.gz minimap2_polish_assembly.paf racon_contigs.fasta > racon_polished_contigs.fasta
```

* Pulido de secuencias con lecturas PacBio
```bash
#PBS -q ensam
#PBS -l nodes=1:ppn=20,mem=250gb,vmem=250gb,walltime=120:00:00
#PBS -N trichoderma_racon_polish
#PBS -V
module load minimap2/2.12
module load racon/1.3.1
cd /home/vflores/LUSTRE/Trichoderma_atroviride_scaffolding/scaf/run_07/racon_polish
for assembly in Trichoderma_atroviride_LANGEBIO_scf.fasta Trichoderma_atroviride_LANGEBIO_v3.fasta Trichoderma_atroviride_LANGEBIO_v3.2.fasta Trichoderma_atroviride_LANGEBIO_v3.3.fasta
do
	base_name=$(echo $assembly | perl -pe 's/\.fasta//')
  minimap2 -t 20 ${assembly} reads.fasta.gz > ${base_name}.paf
  racon    -t 20 reads.fasta.gz ${base_name}.paf ${assembly} > ${base_name}.polish.fasta
done
```

* Pulido de secuencias con lecturas Illumina
```bash
#PBS -q ensam
#PBS -l nodes=1:ppn=16,mem=240gb,vmem=240gb,walltime=120:00:00
#PBS -N trichoderma_ntedit
#PBS -V
cd /home/vflores/LUSTRE/Trichoderma_atroviride_scaffolding/scaf/run_07/illumina_polish
/home/vflores/bin/nthits -p tatro -b 36 -k 40 -t 16 --outbloom --solid R1P.fastq.gz R2P.fastq.gz
/home/vflores/bin/ntedit -t 16 -f Trichoderma_atroviride_LANGEBIO_scf.fasta -r tatro_k40.bf -b Trichoderma_atroviride_LANGEBIO_scf.ipolish -z 10000
/home/vflores/bin/ntedit -t 16 -f Trichoderma_atroviride_LANGEBIO_scf.polish.fasta -r tatro_k40.bf -b Trichoderma_atroviride_LANGEBIO_scf.polish.ipolish -z 10000
/home/vflores/bin/ntedit -t 16 -f Trichoderma_atroviride_LANGEBIO_v3.fasta -r tatro_k40.bf -b Trichoderma_atroviride_LANGEBIO_v3.ipolish -z 10000
/home/vflores/bin/ntedit -t 16 -f Trichoderma_atroviride_LANGEBIO_v3.polish.fasta -r tatro_k40.bf -b Trichoderma_atroviride_LANGEBIO_v3.polish.ipolish -z 10000
/home/vflores/bin/ntedit -t 16 -f Trichoderma_atroviride_LANGEBIO_v3.2.fasta -r tatro_k40.bf -b Trichoderma_atroviride_LANGEBIO_v3.2.ipolish -z 10000
/home/vflores/bin/ntedit -t 16 -f Trichoderma_atroviride_LANGEBIO_v3.2.polish.fasta -r tatro_k40.bf -b Trichoderma_atroviride_LANGEBIO_v3.2.polish.ipolish -z 10000
/home/vflores/bin/ntedit -t 16 -f Trichoderma_atroviride_LANGEBIO_v3.3.fasta -r tatro_k40.bf -b Trichoderma_atroviride_LANGEBIO_v3.3.ipolish -z 10000
/home/vflores/bin/ntedit -t 16 -f Trichoderma_atroviride_LANGEBIO_v3.3.polish.fasta -r tatro_k40.bf -b Trichoderma_atroviride_LANGEBIO_v3.3.polish.ipolish -z 10000
```

* Alineamiento de lecturas Illumina DNAseq sobre los ensambles construidos
```bash
for file_name in $(ls | grep fai$ | perl -pe 's/\.fai//')
do
  base_name=$(echo $file_name | perl -pe 's/\.fa$//;s/\.fasta$//')
  bwa index $file_name
  bwa mem -t 8 $file_name R1P.fastq.gz R2P.fastq.gz > ${base_name}.sam
	samtools view -@ 8 -h -b -o ${base_name}_tmp.bam ${base_name}.sam
	samtools sort -@ 8 -o ${base_name}.bam ${base_name}_tmp.bam
	samtools index ${base_name}.bam
	samtools view -f 12 -@ 8 -h -b -o ${base_name}_unmapped.bam ${base_name}.bam
	samtools view -f 3 -@ 8 -h -b -o ${base_name}_properly_paired.bam ${base_name}.bam
	samtools view ${base_name}_properly_paired.bam | cut -f1 | sort -V | uniq > ${base_name}_properly_paired.id_list
	samtools view ${base_name}_unmapped.bam | cut -f1 | sort -V | uniq > ${base_name}_unmapped.id_list
	total_reads=$(cat $base_name.id_list | wc -l | awk 'print $1*2')
	proper_shared_reads=$(grep -wFf ${base_name}_properly_paired.id_list Trichoderma_atroviride_IMI_206040_properly_paired.id_list | wc -l | awk '{print $1*2}')
	proper_exclusive_reads=$(echo -e "$total_reads\t$proper_shared_reads" | awk 'BEGIN{FS="\t"}{print $1-$2}')
	unmapped_shared_reads=$(grep -wFf ${base_name}_unmapped.id_list Trichoderma_atroviride_IMI_206040_unmapped.id_list | wc -l | awk '{print $1*2}')
	unmapped_exclusive_reads=$(echo -e "$total_reads\t$unmapped_shared_reads" | awk 'BEGIN{FS="\t"}{print $1-$2}')
	echo -e "$base_name\t$unmapped_shared_reads\t$unmapped_exclusive_reads\t$proper_shared_reads\t$proper_exclusive_reads" >> mapping_summary.tsv
done
```

* Alineamiento de secuencias de RNAseq usando hisat2
```bash
#PBS -q default
#PBS -N trichoderma_hisat2_mappings
#PBS -l nodes=1:ppn=16,mem=32gb,vmem=32gb,walltime=24:00:00
module load  hisat2/2.1.0
module load samtools/1.9
cd /home/vflores/LUSTRE/Trichoderma_atroviride_scaffolding/transcriptome
for base_name in
Trichoderma_atroviride_LANGEBIO_scf Trichoderma_atroviride_LANGEBIO_scf.polish Trichoderma_atroviride_LANGEBIO_scf.ipolish_edited Trichoderma_atroviride_LANGEBIO_scf.polish.ipolish_edited Trichoderma_atroviride_LANGEBIO_v3 Trichoderma_atroviride_LANGEBIO_v3.polish Trichoderma_atroviride_LANGEBIO_v3.2 Trichoderma_atroviride_LANGEBIO_v3.2.polish Trichoderma_atroviride_LANGEBIO_v3.2.ipolish_edited Trichoderma_atroviride_LANGEBIO_v3.2.polish.ipolish_edited Trichoderma_atroviride_LANGEBIO_v3.3 Trichoderma_atroviride_LANGEBIO_v3.3.polish Trichoderma_atroviride_LANGEBIO_v3.3.ipolish_edited Trichoderma_atroviride_LANGEBIO_v3.3.polish.ipolish_edited Trichoderma_atroviride_LANGEBIO_v3.4 Trichoderma_atroviride_LANGEBIO_v3.ipolish_edited Trichoderma_atroviride_LANGEBIO_v3.polish.ipolish_edited
do
	hisat2-build ${base_name}.fasta ${base_name}
  hisat2 -p 16 -q -x ${base_name} -U CT01.fastq.gz -S ${base_name}_CT01.sam
  samtools view -h -b -o ${base_name}_CT01_tmp.bam -@ 16 ${base_name}_CT01.sam
  samtools sort -l 9 -@ 16 -o ${base_name}_CT01.bam ${base_name}_CT01_tmp.bam
  samtools index ${base_name}_CT01.bam
done
rm -rf *.sam *_tmp.bam
```

* Selección de lecturas RNAseq que no alinean a los ensambles construidos
```bash
module load samtools/1.9
cd /home/vflores/LUSTRE/Trichoderma_atroviride_scaffolding/transcriptome
for base_name in
Trichoderma_atroviride_LANGEBIO_scf Trichoderma_atroviride_LANGEBIO_scf.polish Trichoderma_atroviride_LANGEBIO_scf.ipolish_edited Trichoderma_atroviride_LANGEBIO_scf.polish.ipolish_edited Trichoderma_atroviride_LANGEBIO_v3 Trichoderma_atroviride_LANGEBIO_v3.polish Trichoderma_atroviride_LANGEBIO_v3.2 Trichoderma_atroviride_LANGEBIO_v3.2.polish Trichoderma_atroviride_LANGEBIO_v3.2.ipolish_edited Trichoderma_atroviride_LANGEBIO_v3.2.polish.ipolish_edited Trichoderma_atroviride_LANGEBIO_v3.3 Trichoderma_atroviride_LANGEBIO_v3.3.polish Trichoderma_atroviride_LANGEBIO_v3.3.ipolish_edited Trichoderma_atroviride_LANGEBIO_v3.3.polish.ipolish_edited Trichoderma_atroviride_LANGEBIO_v3.4 Trichoderma_atroviride_LANGEBIO_v3.ipolish_edited Trichoderma_atroviride_LANGEBIO_v3.polish.ipolish_edited
do
	outbase_name=$(echo "$base_name" | perl -pe 's/Trichoderma_atroviride_LANGEBIO_/unmapped_/')
  samtools view -b -h -o ${outbase_name}_hisat2.bam -f4 ${base_name}_CT01.bam
	bamToFastq -i ${outbase_name}_hisat2.bam -fq ${outbase_name}_hisat2.fastq
	gzip -9 ${outbase_name}_hisat2.fastq
done
rm -rf *.sam *_tmp.bam
```

* Alineamiento de lecturas RNAseq sobre el genoma de referencia usando lecturas no mapeadas a los ensambles construidos
```bash
#PBS -q default
#PBS -N trichoderma_hisat2_mappings
#PBS -l nodes=1:ppn=16,mem=32gb,vmem=32gb,walltime=24:00:00
module load  hisat2/2.1.0
module load samtools/1.9
cd /home/vflores/LUSTRE/Trichoderma_atroviride_scaffolding/transcriptome
echo > base_table
for base_name in
scf scf.polish scf.ipolish_edited scf.polish.ipolish_edited v3 v3.polish v3.2 v3.2.polish v3.2.ipolish_edited v3.2.polish.ipolish_edited v3.3 v3.3.polish v3.3.ipolish_edited v3.3.polish.ipolish_edited v3.4 v3.ipolish_edited v3.polish.ipolish_edited
do
	hisat2-build Trichoderma_atroviride_IMI_206040.fasta Trichoderma_atroviride_IMI_206040
  hisat2 -p 16 -q -x Trichoderma_atroviride_IMI_206040 -U unmapped_${base_name}_hisat2.fastq.gz -S Trichoderma_atroviride_IMI_unmapped_${base_name}_hisat2.sam
  samtools view -h -b -o Trichoderma_atroviride_IMI_unmapped_${base_name}_hisat2_tmp.bam -@ 16 Trichoderma_atroviride_IMI_unmapped_${base_name}_hisat2.sam
  samtools sort -l 9 -@ 16 -o Trichoderma_atroviride_IMI_unmapped_${base_name}_hisat2.bam Trichoderma_atroviride_IMI_unmapped_${base_name}_hisat2_tmp.bam
  samtools index Trichoderma_atroviride_IMI_unmapped_${base_name}_hisat2.bam
	samtools idxstats Trichoderma_atroviride_IMI_unmapped_${base_name}_hisat2.bam > tmp_table
	echo "$(paste base_table tmp_table)" > base_table
done
rm -rf *.sam *_tmp.bam
```



## Datos de calidad de las secuencias Illumina con las que se realizaron las rondas de pulido
|Dataset|Forward|Reverse|
|-|-|-|
|Full|[link](https://agrogenomica.ddns.net/qc/raw/AH1FT1SS01_S1_L001_R1_001_fastqc.html)|[link](https://agrogenomica.ddns.net/qc/raw/AH1FT1SS01_S1_L001_R2_001_fastqc.html)|
|TRAILING:25 CROP:250|[link](https://agrogenomica.ddns.net/qc/trailing_25_crop_250/R1P_fastqc.html)|[link](https://agrogenomica.ddns.net/qc/trailing_25_crop_250/R2P_fastqc.html)|
|TRAILING:25 MINLEN:200 CROP:250|[link](https://agrogenomica.ddns.net/qc/trailing_25_minlen_200_crop_250/R1P_fastqc.html)|[link](https://agrogenomica.ddns.net/qc/trailing_25_minlen_200_crop_250/R2P_fastqc.html)|
|TRAILING:25 MINLEN:250|[link](https://agrogenomica.ddns.net/qc/trailing_25_minlen_250/R1P_fastqc.html)|[link](https://agrogenomica.ddns.net/qc/trailing_25_minlen_250/R2P_fastqc.html)|
