# ngs-course
UNIX course final exercise task

Preparing working directories

	mkdir data
	
	mkdir scripts
	
	mkdir -p plot/plot_data
	
	echo 'data*' >> .gitignore
  
Copying source data to the new directory "data"

	cd data
  
	cp home/data-shared/vcf_examples/luscinia_vars.vcf.gz .

Task 1: Distribution of PHRED qualities over the whole genome and by chromosome

Exploring the source data.

	cd ..
	
	 < data/luscinia_vars.vcf.gz zcat|less -S
	 
Comment lines begin with ##, header begins with #

Chromosome number is in the 1st column

Position is in the 2nd column

Quality score is in the 6th column

Wrokflow:

Storing source data into a variable for easier calling

	IN=data/luscinia_vars.vcf.gz
	
Subsetting the data

	<$IN zcat |cut -f1-2,6|less -S
	
	<$IN zcat |  cut -f1-2,6  > plot/plot_data/luscinia_vars.vcf
	
Storing new dataset into a variable
	
	IN=plot/plot_data/luscinia_vars.vcf
	
Getting rid of comment lines and # in the header
	
	<$IN grep -v '##' | less -S
	
	<$IN grep -v '##' | tail -c +2 | less -S
	
Storing in tab separated file

	<$IN grep -v '##' | tail -c +2 > plot/plot_data/luscinia_vars.tsv
	
	less -S plot/plot_data/luscinia_vars.tsv
	
Continue with plot construction in R Studio

	setwd('~/ngs-course/plot/plot_data')
	
	read_tsv('luscinia_vars.tsv') -> l
	
	read_tsv('luscinia_vars.tsv') %>%
	mutate(CHROM = as.factor(CHROM)) -> l

	
