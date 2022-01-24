# ngs-course
UNIX course final exercise task

Preparing working directories

	mkdir data
	
	mkdir -p plot/plot_data
	
	echo 'data*' >> .gitignore
  
Copying source data to the new directory "data"

	cd data
  
	cp /data-shared/vcf_examples/luscinia_vars.vcf.gz .

Task 1: Distribution of PHRED qualities over the whole genome and by chromosome

Exploring the source data.

	cd ..
	
	 < data/luscinia_vars.vcf.gz zcat|less -S
	 
Comment lines begin with ##, header begins with #

Chromosome number is in the 1st column

Position is in the 2nd column

Quality score is in the 6th column

Workflow:

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
		
	  
Creating bins and labeles that fit positions in my data

	c(0:9,
	  seq(14, 50, by = 5),
	  seq(59, 100, by = 10),
	  seq(149, 300, by = 50),
	  seq(400, 1000, by=100),
	  seq(11000, 91000,by = 10000),
	  seq(1091000, 12000000, by = 1000000)) -> breaks
	  
	data.frame(
	    l = breaks %>% head(-1),
	    r = breaks %>% tail(-1)) %>%
	  mutate(
	    diff = r - l,
	    lab = ifelse(diff > 1, paste0(l + 1, "-", r), as.character(r))) ->
	  labs
  
Adding coloured vertical quality zones

	data.frame(
	  ymin = c(0, 50, 250),
	  ymax = c(50, 250, 1000),
	  colour=c("red", "orange", "green")) ->
	  quals

	
Creating table with new column containing bin information

	l %>%
	  mutate(bin=cut(POS, breaks, labels = labs$lab)) ->
	  lm
	  
Trial plotting of the qualities in the bins
  
	ggplot(lm, aes(bin, QUAL)) +
	  geom_boxplot(outlier.colour = NA) +
	  ylim(c(0, 1000))
	  
Final plot containing both boxplots and zones
  
	ggplot(lm) +
	  geom_rect(aes(ymin = ymin, ymax = ymax, fill = colour),
            xmin = -Inf,
            xmax = Inf,
            alpha=0.3,
            data = quals) +
	  scale_fill_identity() +
	  geom_boxplot(aes(bin, QUAL), outlier.colour = NA, fill = "yellow") +
	  geom_smooth(aes(bin, QUAL, group = 1), colour = "blue") +
	  theme(axis.text.x = element_text(angle = 40, hjust = 1))
	  
Distribution of qualities over the genome:
	
![image](https://user-images.githubusercontent.com/95357905/147915986-25394ac4-faa8-4614-8163-37370619936c.png)


Distribution of qualities by chromozome - I demonstrate the whole process on chromozome 1 (Chr1)

While subsetting the data in UNIX, I chose only chromozome 1 and stored it in separate tsv file:

	<$IN grep -E $'^chr1\t'| less -S
	<$IN grep -E $'^chr1\t' > plot/plot_data/luscinia_vars_chr1.tsv
	
The rest is similar to the previous part.

R workflow:

	read_tsv('luscinia_vars_chr1.tsv',header=FALSE) -> chr1
	names(chr1) <- c("CHROM","POS","QUAL")

Creating bins and labeles that fit positions in my data

	c(seq(0, 11000000, by=200000)) -> breaks
	  
	data.frame(
	    l = breaks %>% head(-1),
	    r = breaks %>% tail(-1)) %>%
	  mutate(
	    diff = r - l,
	    lab = ifelse(diff > 1, paste0(l + 1, "-", r), as.character(r))) ->
	  labs
  
Adding coloured vertical quality zones

	data.frame(
	  ymin = c(0, 50, 250),
	  ymax = c(50, 250, 1000),
	  colour=c("red", "orange", "green")) ->
	  quals
	
Creating table with new column containing bin information

	chr1 %>%
	  mutate(bin=cut(POS, breaks, labels = labs$lab)) ->
	  chr1m
	  
Trial plotting of the qualities in the bins
  
	ggplot(chr1m, aes(bin, QUAL)) +
	  geom_boxplot(outlier.colour = NA) +
	  ylim(c(0, 1000))
	  
Final plot containing both boxplots and zones
  
	ggplot(chr1m) +
	  geom_rect(aes(ymin = ymin, ymax = ymax, fill = colour),
            xmin = -Inf,
            xmax = Inf,
            alpha=0.3,
            data = quals) +
	  scale_fill_identity() +
	  geom_boxplot(aes(bin, QUAL), outlier.colour = NA, fill = "yellow") +
	  geom_smooth(aes(bin, QUAL, group = 1), colour = "blue") +
	  theme(axis.text.x = element_text(angle = 40, hjust = 1))
	  
Distribution of qualities over the chr1
	  
![chr1_plot](https://user-images.githubusercontent.com/95357905/150776227-263020aa-b954-4d09-ad70-dbbfe0bd6003.png)

