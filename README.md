Duplex-Sequencing
=================
README

These programs are meant to be run in order, and result in the transformation of two input .FASTQ files containing data from two reads of an Ilumina sequencing run (or another next-generation sequencer) into a paired-end BAM file containing duplex concensus sequences (DCS).  They will also generate a file containing each different tag present, and how many times it occured, as well as a file, called extraConsensus.bam containing single strand concensus sequences (SSCSs) that, for one reason or another, didn't have a mate (This is possible even if all reads origionaly had mates if the mate had too few or too many reads).  

Instructions: 

First, run PE_BASH_MAKER.py, then make the bash script (.sh file) exicutable using the command below.  

chmod +x PE_DCS_CALC.*.*.sh

Run the bash script.  This should run the rest of the process through to an output paired-end BAM file.  It is strongly sugested that the final sorted BAM file undergo post-processing with picard-tools-1.70/AddOrReplaceReadGroups.jar and GATK/GenomeAnalysisTK.jar, before generating statistics.  Do not run PE_BASH_MAKER.py in a folder containing pre-existing SAM files, as it will delete them.  

Inputs:
	read-1-raw-data.fq
	read-2-raw-data.fq

Outputs:
	SSCS-output-file.bam
		PE SSCSs

	DCS-output-file.bam
		PE DCSs

	tag-counts-file.tagcounts
		Number of members in each group, regardless of weather or not a SSCS was created from that group.

	extraConsensus.bam
		SSCSs that didn't have a mate

	extraDCS.bam
		DCSs for which the switchtag resulted in too many Ns.  


PE_BASH_MAKER.py

Synopsis:
		python PE_BASH_MAKER.py --ref reference-genome.fasta--r1src read-1-raw-data.fq --r2src read-2-raw-data.fq [--min min-group-size] [--max max-group-size] [--cut min-%-identity] [--Ncut max-%-Ns] [--rlength read-length]

Arguments:
	--ref
		Location on disk of the reference genome to which reads should be alligned.

	--r1src
		First reads as a fastq file (.fq), probably should be in same the same folder as this program.

	--r2src
		Second readsas a fastq file (.fq), probably should be in same the same folder as this program.

	--min
		Minimum number of sequences with a single barcode at a given positon required to make a SSCS. 

	--max 
		Maximum number of sequences with a single barcode at a given positon allowed to make a SSCS.

	--cut
		Percent identity at a single base to place it in a SSCS

	--Ncut
		Maximum % Ns allowed in a SSCS

	--rlength
		Length of a single read.  Must be accurate, or else  the consensus maker (and the duplex maker) programs will crash.  


ConsensusMaker2.py

Synopsis:
	python ConsensusMaker2.py --infile in-PE.bam --tagfile  tag-file.tagcounts --outfile out-PE.bam --readlength read-length [--minmem min-mem] [--maxmem max-mem] [--cutoff cutoff] [--Ncutoff N-cutoff]

Arguments:
	--infile 
		Paired end BAM file containing all reads. 

	--tagfile 
		Name for the output tagcounts file

	--outfile 
		Name for the output paired ends BAM file

	--readlength
		Length of a single read.  Must be accurate, or else program will crash.  

	--minmem  
		Minimum number of reads in order to initiate creation of a SSCS.  Any group with less reads than this will just be thrown out.  

	--maxmem  
		Maximum number of reads that should be used in the creation of a SSCS.  Any group with more reads than this will just be thrown out.  

	--cutoff 	
		Minimum percent identity reqired at a position to place a base in the consensus sequence.  

	--Ncutoff
		Maximum percentage of Ns allowed in a SSCS.  

DuplexMaker2.py

Synopsis:
	python DuplexMaker2.py --infile in-file.bam --outfile out-file.ban --readlength read-length [--Ncutoff N-cutoff]

Arguments:
	--infile
		Sorted paired-end BAM file containing SSCSs.  Must have already come through ConsensusMaker2

	--outfile
		Output file in which to store paired-end DCS sequences.  

	--readlength
		Length of a single read.  Must be accurate, or else program will crash.  

	--Ncutoff
		Maximum percentage of Ns allowed in a DCS.
