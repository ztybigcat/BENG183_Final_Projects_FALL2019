# Introducing DeepVariant with performance comparing with GATK and SpeedSeq

Zoe Li, Tianyi Zhang, Ningxin Kang
<div style="float:right">Dec 9 2019</div>

### Abstract
We are unique genetically because we have variations in our DNA. Genetic variants can be grouped into three categories by size: single nucleotide variant (SNV), insertion and deletion (indel), and structural variant (SV, including copy number variation, duplication, translocation, etc.). With the advancement of NGS (Next-generation Sequencing), tons of genetic data is produced for analysis. To identify mutation, the first important step is to detect candidate variants. Currently, there are a few tools available for variant calling: DeepVariant, GATK, and SpeedSeq. We will briefly introduce all three of these tools and then compare them in terms of their performance, runtime, device requirement, and potential possibilities in the following chapter. Among these three tools, Deep Variant is one of the newest TensorFlow based deep learning variant callers, which was first published in 2017 by winning the PrecisionFDA Truth Challenge. Comparing with the other tools, DeepVariant has the highest precision, highest true positive calling, highest F1 score, and lowest false negative calling,  although it also requires a higher computer hardware setting. Since DeepVarinat is currently one of the most valuable technology to learn for bioinformaticians, we will include a deeper explanation and analysis of how DeepVariant works with a step by step tutorial.

### Introduction
Next-generation sequencing (NGS) allows people to use genetic testing for a better understanding of their own genetic make up. The whole-genome sequencing (WGS) allows people to detect diseases causing variant in both protein encoding and non-coding regions of the genome[5]. With genomic detection, a wide range of diseases such as sporadic cancer[6], heart diseases[7], respiratory tract diseases[8], diabetes[9] and psychiatric conditions[10] can be detected. Since the genetic disease is directed related to people’s life, the precision and accuracy of the detection are essential. Currently, DeepVariant is one of the best tools to use. 

During this detecting process, the current algorithm aligns the individual’s gene with the reference gene to find the difference in between. Usually, the difference is called genetic variation. Currently, a few different types of genetic variations are analyzed. The single nucleotide variants (SNVs) and short indels are commonly called, whereas structural variants (SVs) and copy number variants (CNVs) have proven more challenging to detect in WGS data[11]. DeepVariant mostly focus on Indel in its variants calling and achieved an outstanding performance.
<p align="center">
<img src="https://github.com/ztybigcat/BENG183_Final_Projects_FALL2019/blob/master/images_group4/image7.png", width = "600"> <br/>
[Figure 1: Difference between SNPs, Indels, and SV. [20]]<br/>
</p>

Based on Dr. Supernat’s review on DeepVariant, we chose the top two competitors, GATK and SpeedSeq, to do a deeper analysis in terms of their performance compared with the new technology DeepVariant[4]. We are first introducing the mechanisms of GATK, SpeedSeq, and DeepVariant followed by a comparison in terms of their false negative, precision, F1 score, true positive, and their computing hardware requirements. In the second half of this chapter, we also have a step by step tutorial for running the DeepVariant with the sample data from its official GitHub repo[21]. The sample commands and outputs are included in the tutorial for your reference.

#### GATK Methods:
GATK(Genome Analysis Toolkit) is a framework initially released in 2011 to “discover and genotype variation[1]. After years of development, the current variant call software in GATK is called HaplotypeCaller. It can call SNPs and indel.[2] The procedure generally contains the following steps:
<p align="center">
<img src="https://github.com/ztybigcat/BENG183_Final_Projects_FALL2019/blob/master/images_group4/image8.jpg", width = "800"><br/>
[Figure 2: GATK Workflow[2]]<br/>
</p>

##### 1. Define active regions 
Basically in this step the software finds the region where it needs to work on (Where there are differences) and ignore the parts that match perfectly. During this step, the software uses the original mapping output and counts mismatches to calculate a “active probability” . Using the active probability an active area of 50bps to 300bps is defined[2].
##### 2. Determine haplotypes by assembly of the active region
 The program generates a De Brujin graph based on the alignment. In active region, both the aligned and reference genome is separated into k-mers of 10-25 bps. Initialization of graph involved in linking the overlapped reference k-mers into a single path. Based on number of reads containing a particular k-mer, a weight is assigned to the edge. If a pair of paths contains entire k-mers  in common, they are merged together. Edges that do not support many k-mers are discarded. From the graph, it predicts possible haplotypes from the data. A haplotype is a set of k-mers that represent possible sequences. Then the program redo a local alignment using the Smith-Waterman algorithm to identify possible sites of variant[2].
##### 3. Determine likelihoods of the haplotypes given the read data 
Using PairHMM algorithm, the program aligns each possible haplotype(different k-mers) with the reference. The core of the algorithm is a mathematical model called paired Hidden Markov Model(pair-HMM). It is used to eliminate differences due to instrument, like PCR error. The result is recorded in a matrix form that contains score for each read and candidate haplotype. Then the matrix is marginalized, producing the probability each allele[2].
##### 4. Assign sample genotypes 
Bayes' rule is used to calculate the possibility of genotypes per sample using the results generated from the last step. Finally the program picks the highest possible genotype.
After running HaplotypeCaller, the user will need to run separate commands in GATK to generate VCFs [2].
We will not discuss those commands here, rather we will discuss VCF file generated by deep variant in tutorial section. 

#### SpeedSeq Methods:
SpeedSeq is a genome analysis tool that aims to help with alignment, insertions and deletions (indel) variant detection and functional detection. The characteristics of the SpeedSeq, compared to other variant calling method, is its efficiency and low-cost property. As indicated in the SpeedSeq: ultra-fast personal genome analysis and interpretation. written by Chiang, Colby, et al, SpeedSeq achieves superior processing efficiency through rapid duplicate marking with SAMBLASTER5, through balanced parallelization of external applications and by executing nondependent pipeline components simultaneously. The speed is also obtained from processing by SAMBLASTER due to the addition of mate CIGAR and mate mapping quality tags. [3]
<p align="center">
<img src="https://github.com/ztybigcat/BENG183_Final_Projects_FALL2019/blob/master/images_group4/image9.png", width = "800"><br/>
[Figure 3: SpeedSeq Workflow[3]]<br/>
</p>

SpeedSeq workflow:
##### 1. SpeedSeq data alignments:
```
speedseq align
```

SpeedSeq aligns paired-end FASTQ file to the human genome, which produces three sorted, indexed BAM files. These aligned reads are than streamed directly into SMABLASTER, a separate genomic tool which allows simultaneous extraction of discordant read pairs and split-read alignments.

##### 2. Run FreeBayes on one or more BAM files:
```
speedseq var
``` 
FreeBayes is a genetic variant detector used by SpeedSeq, it is a seperate program and needed to be installed separately to enable the functionality of the SpeedSeq. This step will help to detect single-nucleotide variants (SNVs) and indels. It produces a single indexed VCF (variant call format) file that stores gene sequence variation, and is optionally annotated with Variant Effect Predictor (VEP), which determines the effect of the variants on the genes. 
##### 3. Run FreeBayes on a tumor/normal pair of BAM files:
```
speedseq somatic 
```
This step will help in detecting structural variants. It produces a single indexed VCF file that is optionally annotated with VEP.
 ##### 4. Run LUMPY:
 ```
 speedseq sv
 ```
LUMPY is a seperate program that uses probabilistic framework to discover genomic variant. LUMPY is needed to be installed separately to enable the functionality of the SpeedSeq. This step runs LUMPY on one or more BAM files, with optional breakend genotyping by SVTyper (a Bayesian genotyper for structural variants), and optional read-depth analysis by CNVnator (a tool for copy number variation discovery and genotyping from depth-of-coverage by mapped reads), this step produces a bgzipped, indexed VCF file.
##### 5.Realign from a BAM file:
```
speedseq realign
``` 
This step allows alignment from one or more BAM files, rather than FASTQ inputs. It automatically parses read group information from the BAM header to label duplicates based on the library. It produces three sorted, indexed BAM files

#### Deep Variant Methods:
##### 1. Summary of DeepVariant’s mechanism:
DeepVariant converts BAM files into images similar to genome browser snapshots and then use TensorFlow framework to classify reads’ positions in image into variants or non-variants[4];
##### 2. Summary of DeepVariant’s mechanism:
DeepVariant workflow (Referenced from the official paper):

<p align="center">
  <img src="https://github.com/ztybigcat/BENG183_Final_Projects_FALL2019/blob/master/images_group4/image5.png" width="600" class='center'/><br/>
[Figure 4: Workflow of DeepVarant[4]]<br/>
  <img src="https://github.com/ztybigcat/BENG183_Final_Projects_FALL2019/blob/master/images_group4/image1.png" width="300" /><br/>
[Figure 5: Sample pileup image: A: a true SNP on one chromosome pair, B: a deletion on one chromosome, C: a deletion on both chromosomes, D: a false variant caused by errors[22].]<br/>
</p>




i. Left Box in Figure 4 :

	Step 1: Preparation for variant calling.
	File: Make_examples.py
Purpose: Convert BAM files into images (as shown in Figure 2) that can be used in CNN in step 2.
Method: The Make_examples.py use a very sensitive caller to find all positions that might be a variant. Then it uses a type of local reassembly as a more thorough version of indel realignment. Finally, it pileup all reads into different images for different positions mentioned above[24].   

Step 2: Call variants
File: Call_variants.py
Purpose: Classify images from step 1 as variant (Homozygous with different sequence, Heterozygote) or non-variant (Homozygous with same sequence).
Method: Call_variants.py use a pre-trained convolutional neural network (CNN, or ConvNet) to predict which category does the image belong to. After the classification, if the most likely genotype is heterozygous or homozygous non-reference, a variant call is emitted[24].

ii. Middle box in Figure 4 :

Purpose: Show how to train a CNN model, a deep learning model to classify images with different genotypes.
Steps:
1. Use paired pileup images as training set images (as shown in Figure 2) and known genotypes as training set labels for the CNN model.
2. Use stochastic gradient descent algorithm during training and find the best model during training.
3. Along with other CNN models to optimize the CNN parameters to maximize accuracy. Other CNN models includes random CNN model, CNN models trained for other image classifications, and prior DeepVariant models[12]. 
4. Finalize the final trained model and save it for the variant calling.

iii. Right box in Figure 4 :

Purpose: Shows how the CNN model is used.
Method: 
1. Encoded reference reads into a red-green-blue(RGB) pileup image (as shown in Figure 2) at a candidate variant [12]. 
2. Use CNN to classify the images into three diploid genotype states of homozygous reference (hom-ref), heterozygous (het), or homozygous alternate (hom-alt)[12].

<p align="center">
<img src="https://github.com/ztybigcat/BENG183_Final_Projects_FALL2019/blob/master/images_group4/image10.png", width = "800"><br/>
Table 1: Comparison of variant calling pipelines using false negative SNV calls. Data comes from Comparison of three variant callers for human whole genome sequencing(2018) written by Supernat, A., Vidarsson, O. V., Steen, V. M., & Stokowy, T. . The variants are called from 30x, 15x and 10x coverage of the NA12878 sample (HiSeq4000, Genomics Core Facility, Bergen, Norway) and compared to GIAB NISTv3.3.2 (fp://fp-trace.ncbi.nlm.nih.gov/giab/fp/release/NA12878_HG001/NISTv3.3.2/GRCh38/). <br/>
</p>

<p align="center">
<img src="https://github.com/ztybigcat/BENG183_Final_Projects_FALL2019/blob/master/images_group4/image2.png", width = "800"><br/>
Table 2: Comparison of variant calling pipelines using single-nucleotide variant (SNV) calling precisions. Data source is the same as Table 1, using the same coverage comparison.<br/>
</p>


<p align="center">
<img src="https://github.com/ztybigcat/BENG183_Final_Projects_FALL2019/blob/master/images_group4/image11.png", width = "800"><br/>
Table 3: Comparison of variant calling pipelines using F1 score to test the accuracy of three systems, it considers both the precision and the recall of the test to calculate. Data source is the same as Table 1, using the same coverage comparison.
<br/>
</p>

<p align="center">
<img src="https://github.com/ztybigcat/BENG183_Final_Projects_FALL2019/blob/master/images_group4/image6.png", width = "800"><br/>
Table 4: Comparison of variant calling pipelines using true positive (sensitivity) SNV calls. Data source is the same as Table 1, using the same coverage comparison.
<br/>
</p>
<br/>

These four tables are comparison of variant calling pipelines using four different types of scores that specifically deals with precision, sensitivity, and accuracy. Data from each table is called from 30x, 15x and 10x coverage, meant to test the influences of the sequencing depth (the number of unique reads that include a given nucleotide in the reconstructed sequence). Therefore, statistics in each table are presented in 3 groups with decreasing coverages, and compare three tools within each coverage scale respectively. Bar chart is chosen as the presentation format, as it can clearly visualize the differences of scores between three tools.
<br/>
From these four tables, we can see that DeepVariant was clearly more precise (F-Score of 0.94) in indel calling as compared to GATK and SpeedSeq (F-Scores 0.90 and 0.84, respectively) (Table 3). Obviously, DeepVariant is also the most sensitive and accurate tool with the highest number of true positive indel calls (460,271) (Table 4) as well as the lowest number of false negative (39,426) (Table 1)  With respect to the performance on the data with coverage of 15x and 10x, we observed that reduced coverage resulted in a marked drop of the quality of variant calling for all tools. Among these three tools, DeepVariant keep the most stable score among all three coverages, it does not change much with the reduced coverage. Indeed, the F-Scores of DeepVariant for 15x data were almost similar to SpeedSeq at 30x (Table 3). By comparing the false negative score (Table 1), we see that out of the three tested variant callers, GATK was most prone to errors in low coverage regions, while DeepVariant was most robust in such region. 
<br/>

<p align="center">
<img src="https://github.com/ztybigcat/BENG183_Final_Projects_FALL2019/blob/master/images_group4/image3.png", width = "800"><br/>
Table 5: Comparison of amount of hours running DeepVariant and GATK with 35X WGS sample with a 8 core machine 16GB RAM computer.
<br/>
</p>
<br/>
As showed in Table 5, running DeepVariant requires more hours if running on CPU. To speed up this process, a better computer setting like GPU, a specialized electronic circuit designed to rapidly manipulate and alter memory for faster computing, is recommended. Without GPU, DeepVariant requires 830 hours to run a 35X WGS sample, while GATK only requires 430 hours[23]. Even with GPU, to run the complete WGS with a 8 core machine 16GB RAM hardware, it need 24 to 48 hours to complete, as the trade off of good performance[4].
<br/>





###**Reference**
