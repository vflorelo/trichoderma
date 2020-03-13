Refinamiento global del ensamble Trichoderma_atroviride_LANGEBIO_v3.2 (canu + illumina-polish)
==============================================================================================

Una vez habiendo seleccionado el ensamble v3.2 `(44.49 Mbp; n50=1.58 Mbp)` como una versión mejorada del ensamble de Trichoderma atroviride, se procedió a estructurar los contigs obtenidos con base en alineamientos múltiples

* Selección de contigs con base en su %G+C y profundidad promedio empleando [blobtools](https://github.com/DRL/blobtools) [[ref](https://f1000research.com/articles/6-1287)]
* *Scaffolding* con base en la secuencia de referencia (v1) empleando [ragoo](https://github.com/malonge/RaGOO) [[ref](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-019-1829-6)]
* *Scaffolding* con base en similitud con genomas de especies cercanas usando [minimap2](https://github.com/lh3/minimap2) [[ref](https://academic.oup.com/bioinformatics/article/32/14/2103/1742895)]
* Relleno de gaps empleando lecturas PacBio y [LR_gapcloser](https://github.com/CAFS-bioinformatics/LR_Gapcloser) [[ref](https://academic.oup.com/gigascience/article/8/1/giy157/5256637)]

Selección de contigs con base en %G+C y profundidad promedio
------------------------------------------------------------
```bash
#PBS -q default
#PBS -l nodes=1:ppn=16,mem=48gb,vmem=48gb,walltime=120:00:00
#PBS -N trichoderma_v3.2_blastn
#PBS -V
module load bwa/0.7.15
module load samtools/1.9
module load ncbi-blast+/2.6.0
# v3.2 -> canu + illumina polish
cd /home/vflores/LUSTRE/Trichoderma_atroviride_scaffolding/scaf/run_07/canu_purity
bwa index Trichoderma_atroviride_LANGEBIO_v3.2.ipolish_edited.fa
bwa mem -t 8 Trichoderma_atroviride_LANGEBIO_v3.2.ipolish_edited.fa R1P.fastq.gz R2P.fastq.gz > Trichoderma_atroviride_LANGEBIO_v3.2.ipolish_edited.sam
samtools view -@ 8 -h -b -o Trichoderma_atroviride_LANGEBIO_v3.2.ipolish_edited.tmp.bam Trichoderma_atroviride_LANGEBIO_v3.2.ipolish_edited.sam
samtools sort -@ 8 -l 9 -o Trichoderma_atroviride_LANGEBIO_v3.2.ipolish_edited.bam Trichoderma_atroviride_LANGEBIO_v3.2.ipolish_edited.tmp.bam
samtools index Trichoderma_atroviride_LANGEBIO_v3.2.ipolish_edited.bam


blastn -query Trichoderma_atroviride_LANGEBIO_v3.2.ipolish_edited.fa -db /data/secuencias/NT/nt -out Trichoderma_atroviride_LANGEBIO_v3.2.ipolish_edited.blastn -outfmt "6 qseqid staxids bitscore std" -evalue 1e-25 -num_threads 16 -max_target_seqs 500

blobtools create -i Trichoderma_atroviride_LANGEBIO_v3.2.ipolish_edited.fa -b Trichoderma_atroviride_LANGEBIO_v3.2.ipolish_edited.bam -t Trichoderma_atroviride_LANGEBIO_v3.2.ipolish_edited.blastn -o Trichoderma_atroviride_LANGEBIO_v3.2.ipolish_edited
blobtools view -i Trichoderma_atroviride_LANGEBIO_v3.2.ipolish_edited.blobDB.json
blobtools plot -i Trichoderma_atroviride_LANGEBIO_v3.2.ipolish_edited.blobDB.json
```

Tras la [examinación visual](images/v3.2.blobs.png) se seleccionaron 40 contigs que presentaran entre 38 y 60 %G+C, así como una profundidad promedio mayor a 70 y una longitud mayor a 1000 nt, a esta colección de secuencias se le denominó LANGEBIO_v3.2.1 `(37.01 Mbp; n50=1.94)`
```bash
awk 'BEGIN{FS="\t"}{if( ($3>=38 && $3<=60) && ($4>=70) && ($2>=1000) ){print $1}}' Trichoderma_atroviride_LANGEBIO_v3.2.ipolish_edited.stats.tsv > id_list
for id in $(cat id_list)
do
	seqret Trichoderma_atroviride_LANGEBIO_v3.2.ipolish_edited.fa:$id fasta::stdout
done > Trichoderma_atroviride_LANGEBIO_V3.2.1.fasta
```

*Scaffolding* con base en la secuencia de referencia
----------------------------------------------------
Tras comparar las secuencias obtenidas con el ensamble de referencia, se determinó que los contigs ensamblados en la v1 del genoma podrían ser empleados para ordenar los contigs obtenidos en la v3.2.1. Al final de la corrida se obtuvieron 18 scaffolds con gaps arbitrarios de 100 nts. A esta colección de secuencias se le denominó LANGEBIO_v3.2.2 `(37.01 Mbp; n50=5.68)`
```bash
ragoo.py -m /usr/local/bioinformatics/bin/minimap2 -t 8 Trichoderma_atroviride_LANGEBIO_v3.2.1.fasta Trichoderma_atroviride_IMI_206040.fasta
grep \> ragoo.fasta | perl -pe 's/\>//' > id_list
for i in $(seq 1 18)
do
	id=$(tail -n+$i id_list | head -n1)
	seqret ragoo.fasta:id fasta::stdout | perl -pe "s/\>.*/\>scaffold_$i/"
done > Trichoderma_atroviride_LANGEBIO_v3.2.2.fasta
```

*Scaffolding* con base en similitud con genomas de especies cercanas
--------------------------------------------------------------------
Una vez que se tuvo la versión LANGEBIO_v3.2.2 se compararon los scaffolds obtenidos con las secuencias genómicas de *Trichoderma harzianum* T11-W [[accn](https://www.ncbi.nlm.nih.gov/Traces/wgs/WUWT01)], *Trichoderma koningii* JCM-1883 [[accn](https://www.ncbi.nlm.nih.gov/Traces/wgs/BCGH01)], *Trichoderma reesei* QM6A [[accn](https://www.ncbi.nlm.nih.gov/assembly/GCA_002006585.1)] y *Trichoderma* sp. TW21990_1 [[accn](https://www.ncbi.nlm.nih.gov/Traces/wgs/WXUD01)]; así como con las secuencias genómicas de *Cladobotryum protrusum* CCMJ2080 [[accn](https://www.ncbi.nlm.nih.gov/Traces/wgs/RZGP01)], *Escovopsis weberi* EWB [[accn](https://www.ncbi.nlm.nih.gov/Traces/wgs/NIGB01)] e *Hypomyces perniciosus* HP10 [[accn](https://www.ncbi.nlm.nih.gov/Traces/wgs/SPDT01)].
```bash
cat fof_3.2.2
```
<pre>
#prefix	target_sequence	query_sequence	asm_str
LANGEBIO_v3.2.2_cladobotryum	Cladobotryum_protrusum_CCMJ2080.fasta	Trichoderma_atroviride_LANGEBIO_v3.2.2.fasta	asm20
LANGEBIO_v3.2.2_escovopsis	Escovopsis_weberi_EWB.fasta	Trichoderma_atroviride_LANGEBIO_v3.2.2.fasta	asm20
LANGEBIO_v3.2.2_hypomyces	Hypomyces_perniciosus_HP10.fasta	Trichoderma_atroviride_LANGEBIO_v3.2.2.fasta	asm20
LANGEBIO_v3.2.2_imi	Trichoderma_atroviride_IMI_206040.fasta	Trichoderma_atroviride_LANGEBIO_v3.2.2.fasta	asm5
LANGEBIO_v3.2.2_harz	Trichoderma_harzianum_T11_W.fasta	Trichoderma_atroviride_LANGEBIO_v3.2.2.fasta	asm20
LANGEBIO_v3.2.2_konin	Trichoderma_koningii_JCM_1883.fasta	Trichoderma_atroviride_LANGEBIO_v3.2.2.fasta	asm20
LANGEBIO_v3.2.2_reesei	Trichoderma_reesei_QM6A.full.fasta	Trichoderma_atroviride_LANGEBIO_v3.2.2.fasta	asm20
LANGEBIO_v3.2.2_sp	Trichoderma_sp_TW21990_1.fasta	Trichoderma_atroviride_LANGEBIO_v3.2.2.fasta	asm20
</pre>

```bash
for i in $(seq 2 9)
do
	prefix=$(tail -n+$i fof_3.2.2 | head -n1 | cut -f1)
	target_sequence=$(tail -n+$i fof_3.2.2 | head -n1 | cut -f2)
	query_sequence=$(tail -n+$i fof_3.2.2 | head -n1 | cut -f3)
	asm_str=$(tail -n+$i fof_3.2.2 | head -n1 | cut -f4)
	minimap2 -t 8 -x $asm_str $target_sequence $query_sequence > $prefix.paf
	pafCoordsDotPlotly.R --input $prefix.paf --output $prefix --min-query-length 200 --min-alignment-length 100 --plot-size 15 --show-horizontal-lines --identity --identity-on-target --interactive-plot-off
done
```
Tras examinar visualmente las [comparaciones genómicas](LANGEBIO_v3.2.2_dotplots.md) se construyó una tabla con las orientaciones adecuadas de cada scaffold para formar nuevas secuencias contiguas

```bash
cat order
```
<pre>
scaffold_1	scaffold_1:forward	scaffold_11:reverse	scaffold_15:reverse
scaffold_3	scaffold_12:forward	scaffold_4:reverse	scaffold_3:reverse
scaffold_2	scaffold_5:forward	scaffold_2:forward
scaffold_4	scaffold_6:forward	scaffold_8:forward
scaffold_5	scaffold_17:reverse	scaffold_14:forward	scaffold_18:reverse
scaffold_6	scaffold_7:forward
scaffold_7	scaffold_9:forward
scaffold_8	scaffold_10:forward
scaffold_9	scaffold_13:forward
scaffold_10	scaffold_16:forward
</pre>

De modo tal que para la versión LANGEBIO_v3.2.3, el scaffold 1 está conformado por los scaffolds 1,11 y 15 de la versión LANGEBIO_v3.2.2; el scaffold 5 (v3.2.3) está conformado por los scaffolds 17, 14 y 18 (v3.2.2). Los scaffolds 6-10 únicamente cuentan con una secuencia proveniente de la versión anterior. Este orden fue establecido considerando que dos scaffolds deberían estar contiguos en por lo menos dos comparaciones genómicas distintas, por ejemplo, los scaffolds 3 y 4 estuvieron contiguos en las comparaciones con las secuencias de [Trichoderma reesei](images/LANGEBIO_v3.2.2_reesei.png), [Trichoderma koningii](images/LANGEBIO_v3.2.2_konin.png), [Trichoderma harzianum](images/LANGEBIO_v3.2.2_harz.png)

* Codigo para obtener los nuevos scaffolds con gaps de 120 nucleótidos
```bash
#!/bin/bash
sequence_file="$1"
order_file="$2"
num_scaffolds=$(cat "$order_file"  | wc -l)
for i in $(seq 1 $num_scaffolds)
do
  scaf_datablock=$(tail -n+$i ${order_file} | head -n1)
  scaf_name=$(echo "$scaf_datablock" | cut -f1)
  subscaf_num=$(echo "$scaf_datablock" | cut -f1 --complement | perl -pe 's/\t/\n/g' | wc -l)
  echo ">$scaf_name"
  for j in $(seq 1 $subscaf_num)
  do
    scaf_id=$(echo "$scaf_datablock" | cut -f1 --complement | cut -f$j | cut -d\: -f1)
    scaf_dir=$(echo "$scaf_datablock" | cut -f1 --complement | cut -f$j | cut -d\: -f2)
    if [ "$scaf_dir" == "forward" ]
    then
      revcomp_str=""
    elif [ "$scaf_dir" == "reverse" ]
    then
      revcomp_str=" -sreverse "
    fi
    seqret ${sequence_file}:${scaf_id} ${revcomp_str} raw::stdout | perl -pe 's/\n//'
    printf 'N%.0s' {1..120}
  done
  echo
done
```

* Con el código anterior obtenemos la nueva versión LANGEBIO_v3.2.3 `(37.01 Mbp; n50=6.69)`
```bash
get_scaffolds.sh Trichoderma_atroviride_LANGEBIO_v3.2.2.fasta order > Trichoderma_atroviride_LANGEBIO_v3.2.3.fasta
```

* Finalmente hacemos la examinación de los scaffolds construidos comparandolos con los genomas mencionados anteriormente

```bash
cat fof_3.2.3

```
<pre>
#prefix	target_sequence	query_sequence	asm_str
LANGEBIO_v3.2.3_cladobotryum	Cladobotryum_protrusum_CCMJ2080.fasta	Trichoderma_atroviride_LANGEBIO_v3.2.3.fasta	asm20
LANGEBIO_v3.2.3_escovopsis	Escovopsis_weberi_EWB.fasta	Trichoderma_atroviride_LANGEBIO_v3.2.3.fasta	asm20
LANGEBIO_v3.2.3_hypomyces	Hypomyces_perniciosus_HP10.fasta	Trichoderma_atroviride_LANGEBIO_v3.2.3.fasta	asm20
LANGEBIO_v3.2.3_imi	Trichoderma_atroviride_IMI_206040.fasta	Trichoderma_atroviride_LANGEBIO_v3.2.3.fasta	asm5
LANGEBIO_v3.2.3_harz	Trichoderma_harzianum_T11_W.fasta	Trichoderma_atroviride_LANGEBIO_v3.2.3.fasta	asm20
LANGEBIO_v3.2.3_konin	Trichoderma_koningii_JCM_1883.fasta	Trichoderma_atroviride_LANGEBIO_v3.2.3.fasta	asm20
LANGEBIO_v3.2.3_reesei	Trichoderma_reesei_QM6A.full.fasta	Trichoderma_atroviride_LANGEBIO_v3.2.3.fasta	asm20
LANGEBIO_v3.2.3_sp	Trichoderma_sp_TW21990_1.fasta	Trichoderma_atroviride_LANGEBIO_v3.2.3.fasta	asm20
</pre>

```bash
for i in $(seq 2 9)
do
	prefix=$(tail -n+$i fof_3.2.3 | head -n1 | cut -f1)
	target_sequence=$(tail -n+$i fof_3.2.3 | head -n1 | cut -f2)
	query_sequence=$(tail -n+$i fof_3.2.3 | head -n1 | cut -f3)
	asm_str=$(tail -n+$i fof_3.2.3 | head -n1 | cut -f4)
	minimap2 -t 8 -x $asm_str $target_sequence $query_sequence > $prefix.paf
	pafCoordsDotPlotly.R --input $prefix.paf --output $prefix --min-query-length 200 --min-alignment-length 100 --plot-size 15 --show-horizontal-lines --identity --identity-on-target --interactive-plot-off
done
```

Obteniendo las siguientes [comparaciones genómicas](LANGEBIO_v3.2.3_dotplots.md)
