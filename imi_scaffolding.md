*Scaffolding* con base en las secuencias de referencia de otras especies de Trichoderma

Se tomó la secuencia genómica de Trichoderma atroviride IMI 206040 y se comparó con las secuencias genómicas de *Trichoderma harzianum* T11-W [[accn](https://www.ncbi.nlm.nih.gov/Traces/wgs/WUWT01)], *Trichoderma koningii* JCM-1883 [[accn](https://www.ncbi.nlm.nih.gov/Traces/wgs/BCGH01)], *Trichoderma reesei* QM6A [[accn](https://www.ncbi.nlm.nih.gov/assembly/GCA_002006585.1)] y *Trichoderma* sp. TW21990_1 [[accn](https://www.ncbi.nlm.nih.gov/Traces/wgs/WXUD01)]; así como con las secuencias genómicas de *Cladobotryum protrusum* CCMJ2080 [[accn](https://www.ncbi.nlm.nih.gov/Traces/wgs/RZGP01)], *Escovopsis weberi* EWB [[accn](https://www.ncbi.nlm.nih.gov/Traces/wgs/NIGB01)] e *Hypomyces perniciosus* HP10 [[accn](https://www.ncbi.nlm.nih.gov/Traces/wgs/SPDT01)].

```bash
cat fof_imi_v1
#prefix	target_sequence	query_sequence	asm_str
IMI_cladobotryum	Cladobotryum_protrusum_CCMJ2080.fasta	Trichoderma_atroviride_IMI_206040.fasta	asm20
IMI_escovopsis	Escovopsis_weberi_EWB.fasta	Trichoderma_atroviride_IMI_206040.fasta	asm20
IMI_hypomyces	Hypomyces_perniciosus_HP10.fasta	Trichoderma_atroviride_IMI_206040.fasta	asm20
IMI_harz	Trichoderma_harzianum_T11_W.fasta	Trichoderma_atroviride_IMI_206040.fasta	asm20
IMI_konin	Trichoderma_koningii_JCM_1883.fasta	Trichoderma_atroviride_IMI_206040.fasta	asm20
IMI_reesei	Trichoderma_reesei_QM6A.full.fasta	Trichoderma_atroviride_IMI_206040.fasta	asm20
IMI_sp	Trichoderma_sp_TW21990_1.fasta	Trichoderma_atroviride_IMI_206040.fasta	asm20
for i in $(seq 2 8)
do
	prefix=$(tail -n+$i fof_imi_v1 | head -n1 | cut -f1)
	target_sequence=$(tail -n+$i fof_imi_v1 | head -n1 | cut -f2)
	query_sequence=$(tail -n+$i fof_imi_v1 | head -n1 | cut -f3)
	asm_str=$(tail -n+$i fof_imi_v1 | head -n1 | cut -f4)
	minimap2 -t 8 -x $asm_str $target_sequence $query_sequence > $prefix.paf
	pafCoordsDotPlotly.R --input $prefix.paf --output $prefix --min-query-length 200 --min-alignment-length 100 --plot-size 15 --show-horizontal-lines --identity --identity-on-target --interactive-plot-off
done
```

Tras examinar visualmente las comparaciones genómicas se construyó una tabla con las orientaciones adecuadas de cada scaffold para formar nuevas secuencias contiguas. El orden de cada contig dentro de los scaffolds construidos debio presentarse en por lo menos dos comparaciones distintas. En caso de que hubiera datos contradictorios, es decir que un contig se observara junto a distintos contigs en distintas comparaciones, se consideró como incierta la agrupación y se mandaron como scaffolds individuales.

```bash
cat order_file
scaffold_1	ABDG02000012.1:forward	ABDG02000022.1:reverse	ABDG02000026.1:reverse
scaffold_2	ABDG02000016.1:forward	ABDG02000013.1:forward
scaffold_3	ABDG02000002.1:reverse	ABDG02000023.1:forward	ABDG02000015.1:reverse	ABDG02000014.1:reverse
scaffold_4	ABDG02000007.1:forward	ABDG02000017.1:forward	ABDG02000019.1:forward	ABDG02000008.1:reverse
scaffold_5	ABDG02000028.1:reverse	ABDG02000025.1:forward	ABDG02000005.1:reverse	ABDG02000009.1:reverse	ABDG02000029.1:reverse
scaffold_6	ABDG02000018.1:forward
scaffold_7	ABDG02000020.1:forward
scaffold_8	ABDG02000021.1:forward	ABDG02000006.1:reverse	ABDG02000004.1:reverse	ABDG02000011.1:reverse
scaffold_9	ABDG02000010.1:forward	ABDG02000024.1:forward
scaffold_10	ABDG02000027.1:forward
scaffold_11	ABDG02000001.1:forward
scaffold_12	ABDG02000003.1:forward
```

Habiendo construido la tabla del orden de los scaffolds se obtienen las secuencias con gaps arbitrarios de 1200 nucleótidos

```bash
get_scaffolds.sh Trichoderma_atroviride_IMI_206040.fasta order_file > Trichoderma_atroviride_IMI_206040_v2.fasta
```

*get_scaffolds.sh*
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
    printf 'N%.0s' {1..1200}
  done
  echo
done
```

Posteriormente se rellenan los gaps con las secuencias consenso provenientes de lecturas pacbio con el programa [LR_Gapcloser](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6324547/)

```bash
LR_Gapcloser.sh -i Trichoderma_atroviride_IMI_206040_v2.fasta -l reads.fasta -s p -t 16 -n 10 -r 5 -o LR_gapcloser
```
