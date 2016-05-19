# Cytoruler

## Introduction


## Normalization

### Normalization with spike-in control

#### 1. Map the reads to reference genome (Preprocessing)

As a preprocessing step, fasta files from TSA-seq should be first mapped to reference genome. For read mapping, any well-known mapping tools such as Bowtie, BWA or SOAP2 can be used. We do suggest to remove PCR duplicates after the mapping process and bam file should be sorted and indexed. For example:
```shell
# Since K562 is derived from a female, we remove chromosome Y from human reference genome hg19 and named it hg19F
# Use bowtie2 to map the reads and link with to get bam file
# bowtie2 -p 8 -x hg19F -U SON_TSA-seq_pulldown.fastq | samtools view -bS - > SON_TSA-seq_pulldown.bam
# Sort bam file
samtools sort SON_TSA-seq_pulldown.bam SON_TSA-seq_pulldown_sort
# remove pcr duplicate from bam file
# skip pcr duplication removal step if do not the step2 (map reads to spike-in reference)
samtools rmdup SON_TSA-seq_pulldown_sort SON_TSA-seq_pulldown_rmdup.bam
# Index bam file
samtools index SON_TSA-seq_pulldown_rmdup.bam
```
#### 2. Map the reads to spike-in control sequence (Preprocessing)
Depend on what kind of spike-in sequence is used, this step is either a required or optional. i) If the spike-in sequence can not be found in any place of human reference genome, fastq file should be mapped to spike-in sequence. ii) If the spike-in sequence can only be found in single position in human reference genome, this step is optional. iii) If the spike-in sequence (even only a small part of the whole sequence) can be mapped to multiple position in human reference genome, this step is optional but highly suggested to do.
```shell
# Use bowtie2 to map the reads and link with to get bam file
# bowtie2 -p 8 -x spike-in -U SON_TSA-seq_pulldown.fastq | samtools view -bS - > SON_TSA-seq_pulldown_spike-in.bam
# Sort bam file
samtools sort SON_TSA-seq_pulldown_spike-in.bam SON_TSA-seq_pulldown_spike-in_sort
# do not remove pcr duplication for spike-in since many read will be mapped to the same position
# Index bam file
samtools index SON_TSA-seq_pulldown_spike-in_sort

```

#### 3. Count spike-in read (Preprocessing)
This following code will count number of mapped reads in spike-in sequence and get the normalization factor R
```shell
python TSA-seq_spike_in_get_ratio.py --pulldown SON_TSA-seq_pulldown_spike-in_sort --control NoPrimaryCtrl_TSA-seq_pulldown_spike-in_sort -m spike 
```
You can also run the same code for input samples of primary antibody pulldown and no primary control pulldown. The purpose is to confirm that spike-in added in two input sample are around same
```shell
python TSA-seq_spike_in_get_ratio.py --pulldown SON_TSA-seq_input_spike-in_sort --control NoPrimaryCtrl_TSA-seq_input_spike-in_sort -m spike 
```

#### 4. Normalization
Support the normalization factor we got is 8.334721, run the normalization with spike-ind correction like this.
```shell
python TSA-seq_normalize.py -r 100 -w 20000 -g genome/hg19/hg19F.genome -ep data/SON_TSA-seq/SON_TSA-seq_pulldown_rmdup.bam -cp data/SON_TSA-seq/SON_TSA-seq_input_rmdup.bam -en SON_TSA-seq/NoPrimary_TSA-seq_pulldown_rmdup.bam -cn SON_TSA-seq/NoPrimary_TSA-seq_input_rmdup.bam -R 8.334721 -o result/1118_normalize/SON_TSA-seq_Sucrose_Score --wig2bw sys
```

### Normalization without spike-in control

