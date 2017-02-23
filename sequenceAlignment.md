Sequence alignment of the ATAC-seq libraries with BWA.
======================================================

Why BWA ?
---------

While Bowtie 1 and 2 seem to be the usual aligners for ATAC-seq data, I happen
to be more familiar with the use of BWA, which I used extensively in my work on
the "_CAGEscan_" method, which uses paired-end alignment for transcritome
analysis.  Since the ATAC-seq libraries analysed here are paired-end, I am
sticking to BWA (`sampe`).  Also, I am using version 0.5.10 because I happen to
have genome indexes ready in this format; I do not expect major differences
with later versions, where the developmens were mostly focused on the `mem`
algorithm, not used here.

The commands below are to be pasted in the command-line shell of a computer
sufficiently powerful to align the libraries.


Alignment command
-----------------

```
alignOnGenome() {
  SAMPLE=$1
  GENOME=$2
  $BWA aln -t8 $GENOME -f ${SAMPLE}_R1.sai <(zcat ${SAMPLE}_R1.fastq.gz | fastx_trimmer -l 30)
  $BWA aln -t8 $GENOME -f ${SAMPLE}_R2.sai <(zcat ${SAMPLE}_R2.fastq.gz | fastx_trimmer -l 29)
  $BWA sampe   $GENOME    ${SAMPLE}_R1.sai ${SAMPLE}_R2.sai \
                           <(zcat ${SAMPLE}_R1.fastq.gz | fastx_trimmer -l 30) \
                           <(zcat ${SAMPLE}_R2.fastq.gz | fastx_trimmer -l 29) |
    samtools view -uS    |
    samtools sort        |
    tee $SAMPLE.bam      |
    samtools view -u -f2 |
    samtools rmdup - -   |
    samtools sort -n     |
    samtools fixmate - - |
    pairedBamToBed12     |
    awk '{OFS="\t"}{if($6=="+"){$9="0,128,0"};if($6=="-"){$9="128,0,128"};print}' |
    sort -V -k1,1 -k2,2n -k3,3 -k6 > $SAMPLE.bed
  rm ${SAMPLE}_R1.sai ${SAMPLE}_R2.sai
}
```

In brief:

 - The reads are aligned using `bwa aln` and `bwa sampe` with standard
   parameters, after trimming them to 30 bases for Read1 and 29 bases for
   Read2.  This is the read length on the MiSeq runs that we produced, and I want
   to minimise the technical differences between our data and the one from the
   original paper.

 - The output is converted to BAM format and sorted; a copy is kept in the
   filesystems, while the data flows in the next steps through Unix pipes.

 - All the non-properly aligned reads are discarded, and then PCR duplicates are
   removed based on the paired-end coordinates.

 - The pairs are re-sorted by name, and converted to BED12 format using our
   [pairedBamToBed12](https://github.com/Population-Transcriptomics/pairedBamToBed12)
   tool.  (Note that `pairedBamToBed12` generates many errors because of reads
   marked _properly paired_ by BWA on chrM, but where the next mate is maked
   unmapped (0x8).  This was unfortunately not corrected by `samtools fixmate`.

 - The color field is set to green for the positive strand and violet for the
   negative one.

 - The pairs are finally re-sorted by coordinate and saved on the disk.


Alignment on the human (hg19) and mouse (mm9) genomes
-----------------------------------------------------

To search for cross-contaminations, let's align all samples on both genomes,
regardless the nature of the presence of cells.


### Why old version of the genomes ?

As of today, The FANTOM5 data, that defines promoter regions, is still easier
to access on hg19.


### Align on mouse genome

Adjust the paths to your environment, enter the directory containing the
compressed sequence files, and run the loop below.

```
BWA=/home/plessy/snap/bwa_0.5.10/usr/bin/bwa
```

This is alignment of not-so heavy data so I did not spend time to optimise the
speed.  Read a paper, write a grant, take a coffee, go to bed, etc. while it
runs.

```
GENOME=/analysisdata/genomes/mm9_male.fa

for SAMPLE in $(basename -s _R1.fastq.gz *_R1.fastq.gz)
do
  alignOnGenome $SAMPLE $GENOME
done

prename 's/.bam/_mm9.bam/ ; s/.bed/_mm9.bed/' *[A-H][0-9][0-9].b[ae][md] Undetermined_S0_L001.b*
```

### Align on human genome

```
GENOME=/analysisdata/genomes/hg19_male.fa

for SAMPLE in $(basename -s _R1.fastq.gz *_R1.fastq.gz)
do
  alignOnGenome $SAMPLE $GENOME
done

prename 's/.bam/_hg19.bam/ ; s/.bed/_hg19.bed/' *Seq.b[ae][md]
```

### Alignment statistics

#### Proper pairs

```
for bed in *.bed
do
  printf "proper_pairs\t$(basename $bed .bed)\t"
  cat $bed |
  wc -l
done |
  tee ../$LIBRARY.dedupStats.log
```

#### Proper pairs (Q > 20)

```
for bed in *.bed
do
  printf "proper_pairs_Q20\t$(basename $bed .bed)\t"
  awk '$5 > 20' $bed |
  wc -l
done |
  tee ../$LIBRARY.dedupStatsQ20.log
```

#### Redundancy

```
for bam in *.bam
do
  printf "redundancy\t$(basename $bam .bam)\t"
  samtools rmdup $bam  /dev/null 2>&1 |
    grep library |
    cut -f 6 -d ' '
done |
  tee ../$LIBRARY.redundancy.log
```

#### Redundancy (on proper pairs)

``` 
for bam in *.bam
do
  printf "redundancy2\t$(basename $bam .bam)\t"
  samtools view -u -f2 $bam |
  samtools rmdup - /dev/null 2>&1 |
    grep library |
    cut -f 6 -d ' '
done |
  tee ../$LIBRARY.redundancy2.log
```
