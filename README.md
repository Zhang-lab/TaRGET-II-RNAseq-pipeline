# TaRGET-II-RNA-seq-pipeline

### Overview
The TaRGET II RNA-seq pipeline is designed for standardized data processing of mouse RNA-seq samples. It incorporates automatic quality control, generates user friendly files for computational analysis and outputs genome browser tracks for data visualization. To ensure consistent and reproducible data processing, the entire workflow, associated software and libraries are built into a singularity image, which can be run on computational clusters with job submission as well as on stand-alone machines. Pipeline installation requires minimal user input. All the software and genome references used for TaRGET II RNA-seq data processing are included in the pipeline image. The pipeline supports both single- and paired-end data, it accepts FASTQ files, performs alignments, gene features summary and data visualization. 
### Software Used in the Pipeline

**cutadapt (v1.16)** was used to find and remove adapter sequence, and other types of unwanted sequence from high-throughput sequencing reads.

**fastqc (v0.11.7)** was used to provide quality control check s on raw sequence data.

**star (v2.5.4b)** was used to map sequence reads against the mouse reference genome.

**RSeQC** was used to comprehensively evaluate high throughput sequence data.

**featureCounts** (v1.5.1) was used to count reads to genomic features such as genes.

**RSEM (v1.3.0)** was used to estimate gene and isoform expression levels from RNA-Seq data.

**samtools (v1.9)** and **bedtools (v2.29)** were used to manipulate the sequence alignments and genome features.

### Genomic References Used in the Pipeline 

The following mouse genome and gene references were built into the singularity image and used for TaRGET RNA-seq data processing:
1)	STAR indexes of mouse genome (mm10).
2)	Gtf file of vM20 gene annotation from GENCODE.
3)	RSEM references using GENCODE vM20 annotations.
4)	Chromosome size file of mm10.

# Usage:

**2-step guideline for pipeline usage:**

**step.1** Download the RNA-seq singularity image [here](http://brc.wustl.edu/SPACE/shaopengliu/Singularity_image/rna-seq/rna-seq_mm10_v4.simg):
```
# download image from local server:
wget http://brc.wustl.edu/SPACE/shaopengliu/Singularity_image/rna-seq/rna-seq_mm10_v4.simg  
```
**step.2** Process RNA-seq data with the image: 

Please Run at same directory with your data OR the soft link of your data
```
singularity run -B ./:/process <path-to-image> -r <SE/PE> -g <mm10 > -o <read_file1> -p <read_file2>
```
**soft link introduction:** For soft link of data, need to add one bind option for singularity, which is ```-B <full-path-of-original-position>:<full-path-of-original-position>```. 
For example:
soft link the data from /scratch to run on folder /home/example. **Please make sure you use the absolute path.**
```
ln -s /scrach/mydata.fastq.gz /home/example;
cd /home/example
singularity run -B ./:/process -B /scratch:/scratch /home/image/RNA_IAP_v1.00.simg -r PE -g mm10 -o read1.fastq.gz -p read2.fastq.gz
```
**parameters:**

`-h`: help information.

`-r`: SE for single-end, PE for paired-end.

`-g`: genome reference, one simg is designed for ONLY one species due to the file size. For now the supported genoms are mm10.

`-o`: reads file 1 or the SE reads file, must be ended by .fastq or .fastq.gz or .sra (for both SE and PE). 

`-p`: reads file 2 if input PE data, must be ended by .fastq or .fastq.gz.

`-a`: ADAPT1 for cutadapt

`-b`: ADAPT2 for cutadapt, if not specified there will be ONLY quality trimming rather than adapter trimming

`-t`: (optional) specify number of threads to use, default 24

# Output files

After running the pipeline, there will be a folder called Processed_${name}, all intermediate files and final output files are stored there. The major outputs produced by the pipeline are:
  1)	JSON file with quality control measurement of user supplied dataset. 
  2)	Log file with processing status of each step.
  3)	Processed files after alignment (.bam) and summary files of gene features. 
  4)	Bigwig file for visualization purpose.

# Quailty Control Parameters
The JSON output file contains the QC parameters that were used to check the quality of RNA-seq data.

**Key QC parameters:**
 
**uniq_ratio**: The ratio of uniquely mapped reads number after alignment against reference genome to the total input reads number for alignment.

**assign_reads**: The number of reads that were assigned to genes provided by GENCODE vM20 gene annotation file.

**genes_w_CPM_gt1**: The number of expressed genes with CPM value larger than 1.

|QC parameter|Value Cutoff|Score: >= cutoff|Score: <cutoff|
| :---: | :---: | :---: | :---: |
|**uniq_ratio**|0.35|1|0|
|**assign_reads**|10000000|1|0|
|**genes_w_CPM_gt1**|9000|1|0|

The QC score used to check the quality of data:
| Type |QC cutoff| >= cutoff | < cutoff |
| :---: | :---: | :---: | :---: |
| **Sum of QC score** | 3 | good quality | bad quaility |
