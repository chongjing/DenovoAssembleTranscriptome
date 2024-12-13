# DenovoAssembleTranscriptome

# 

This project is to de novo assemble a transcriptome for samples of nematode at three different stages - ppJ2, pJ2 and pJ3, using `Trinity`. Detailed usage is [here](https://github.com/trinityrnaseq/trinityrnaseq/wiki).

## 0.Preparation

Trinity installation. Luckily we have `docker` in Hydrogen node13. 

```bash
cd /data/pathology/program/Trinity
docker pull trinityrnaseq/trinityrnaseq
```

## 1.pJ2

Raw reads were filtered and trimmed to get high quality reads using `Trimmomatic (v0.39)`. Briefly, the bases with quality less than 20 at the start or end of a read were cut off, and reads with length shorter than 60bp were dropped.

```bash
cd /data/pathology/cxia/projects/Sebastian/03.DenovoTranscriptome/01.TrimmedData/02.pJ2/

for sample in Hs1 Hs2 Hs3; do
    # extract sample name
    sample_name="001.pJ2_${sample}"
    fq1="/home/bm522/Hsc_gland_cell_data_all_stages/pJ2/merged_reads_clem/${sample}_1_final.fq.gz"
    fq2="/home/bm522/Hsc_gland_cell_data_all_stages/pJ2/merged_reads_clem/${sample}_2_final.fq.gz"

    # set output files
    output_fwd_paired="${sample_name}_1P.fq.gz"
    output_fwd_unpaired="${sample_name}_1U.fq.gz"
    output_rev_paired="${sample_name}_2P.fq.gz"
    output_rev_unpaired="${sample_name}_2U.fq.gz"

    # Run Trimmomatic
    echo "processing ${sample_name}"
    java -jar /data/pathology/program/Trimmomatic/Trimmomatic-0.39/trimmomatic-0.39.jar PE -threads 16 -summary ${sample_name}.summary "$fq1" "$fq2" \
        "$output_fwd_paired" "$output_fwd_unpaired" "$output_rev_paired" "$output_rev_unpaired" \
        LEADING:20 TRAILING:20 SLIDINGWINDOW:4:20 MINLEN:60
    echo "${sample_name} finished"
done


```

Summary of `Trimmomatic` can be found in each folder. In the same directory as the trimmed reads, run Trinity. Otherwise, it's hard to designate input files in `Docker` container.

```bash
for sample in pJ2_Hs1 pJ2_Hs2 pJ2_Hs3; do
        echo "Trinity Processing ${sample}"
        docker run --rm -v `pwd`:`pwd` -w `pwd` trinityrnaseq/trinityrnaseq Trinity --seqType fq --max_memory 64G --left 001.${sample}_1P.fq.gz --right 001.${sample}_2P.fq.gz --CPU 16 --output 003.trinity.${sample} 1>002.${sample}.Trinity.o 2>&1
done
```

## 2.pJ3

```bash
# run Trimmomatic and Trinity together iterate through each sample
for sample in Hs1 Hs2 Hs3 Hs4; do
    # extract sample name
    sample_name="001.pJ3_${sample}"
    fq1=(/home/bm522/Hsc_gland_cell_data_all_stages/pJ3/raw_data/${sample}/*_L2_1.fq.gz)
    fq2=(/home/bm522/Hsc_gland_cell_data_all_stages/pJ3/raw_data/${sample}/*_L2_2.fq.gz)

    # set output files
    output_fwd_paired="${sample_name}_1P.fq.gz"
    output_fwd_unpaired="${sample_name}_1U.fq.gz"
    output_rev_paired="${sample_name}_2P.fq.gz"
    output_rev_unpaired="${sample_name}_2U.fq.gz"

    # Run Trimmomatic
    echo "Trimmomatic processing ${sample_name}"
    java -jar /data/pathology/program/Trimmomatic/Trimmomatic-0.39/trimmomatic-0.39.jar PE -threads 16 -summary ${sample_name}.summary "$fq1" "$fq2" \
        "$output_fwd_paired" "$output_fwd_unpaired" "$output_rev_paired" "$output_rev_unpaired" \
        LEADING:20 TRAILING:20 SLIDINGWINDOW:4:20 MINLEN:60
    echo "Trimmomatic ${sample_name} finished"

    # Run Trinity
    echo "Trinity Processing ${sample}"
    docker run --rm -v `pwd`:`pwd` -w `pwd` trinityrnaseq/trinityrnaseq Trinity --seqType fq --max_memory 64G --left ${sample_name}_1P.fq.gz --right ${sample_name}_2P.fq.gz --CPU 16 --output 003.trinity.${sample} 1>002.${sample}.Trinity.o 2>&1
    echo "Trinity ${sample} finished"
done
```

## 3.ppJ2

```bash
cd /data/pathology/cxia/projects/Sebastian/03.DenovoTranscriptome/01.TrimmedData/01.ppJ2/
# run trimmomatic iterate through each sample
for sample in Hs3 Hs4 Hs5; do
    # extract sample name
    sample_name="001.ppJ2_${sample}"
    fq1=(/home/bm522/Hsc_gland_cell_data_all_stages/ppJ2/01.RawData/${sample}/trimmed_reads/${sample}_1_trim.fq.gz)
    fq2=(/home/bm522/Hsc_gland_cell_data_all_stages/ppJ2/01.RawData/${sample}/trimmed_reads/${sample}_2_trim.fq.gz)

    # set output files
    output_fwd_paired="${sample_name}_1P.fq.gz"
    output_fwd_unpaired="${sample_name}_1U.fq.gz"
    output_rev_paired="${sample_name}_2P.fq.gz"
    output_rev_unpaired="${sample_name}_2U.fq.gz"

    # Run Trimmomatic
    echo "Trimmomatic processing ${sample_name}"
    java -jar /data/pathology/program/Trimmomatic/Trimmomatic-0.39/trimmomatic-0.39.jar PE -threads 16 -summary ${sample_name}.summary "$fq1" "$fq2" \
        "$output_fwd_paired" "$output_fwd_unpaired" "$output_rev_paired" "$output_rev_unpaired" \
        LEADING:20 TRAILING:20 SLIDINGWINDOW:4:20 MINLEN:60
    echo "Trimmomatic ${sample_name} finished"

    # Run Trinity
    echo "Trinity Processing ${sample}"
    docker run --rm -v `pwd`:`pwd` -w `pwd` trinityrnaseq/trinityrnaseq Trinity --seqType fq --max_memory 64G --left ${sample_name}_1P.fq.gz --right ${sample_name}_2P.fq.gz --CPU 16 --output 003.trinity.${sample} 1>002.${sample}.Trinity.o 2>&1
    echo "Trinity ${sample} finished"
done
```

## 4.Results

```bash
## pJ2 assembled transcriptomes:
/data/pathology/cxia/projects/Sebastian/03.DenovoTranscriptome/01.TrimmedData/02.pJ2/003.trinity.pJ2_Hs(1|2|3).Trinity.fasta

## pJ3 assembled transcriptomes:
/data/pathology/cxia/projects/Sebastian/03.DenovoTranscriptome/01.TrimmedData/03.pJ3/003.trinity.Hs(1|2|3).Trinity.fasta

## ppJ2 assembled transcriptomes
/data/pathology/cxia/projects/Sebastian/03.DenovoTranscriptome/02.Trinity/01.ppJ2/01.trinity.ppJ2.Hs(1|2).Trinity.fasta
/data/pathology/cxia/projects/Sebastian/03.DenovoTranscriptome/01.TrimmedData/01.ppJ2/003.trinity.Hs(3|4|5).Trinity.fasta
```

