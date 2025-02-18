# EEOB 4560 Unix Assignment


## Part I) Data Inspection

###Attributes of `fang_et_al_genotypes`

```
# initial look
(head -n 5; tail -n 5) < fang_et_al_genotypes.txt

# file details (file size, # of lines, # of words, # of columns)
du -h fang_et_al_genotypes.txt				# file size
wc -l fang_et_al_genotypes.txt          		# number of lines
wc -w fang_et_al_genotypes.txt                   	# number of words
awk '{print NF}' fang_et_al_genotypes.txt | head -1  	# number of columns

# identification of non-ASCII files and characters
file fang_et_al_genotypes.txt 

```

By inspecting this file I learned that:

* The initial look into the data file shows the first 5 and last 5 lines of the file. From this inspection, it can be gathered that the first row of the file contains column names (e.g. ` Sample_ID`, `JG_OTU`, `Group`, and a list of gene/marker names (`abph1.20`, `ae1.3`, `PZA00003.11`, etc.)). The content of the dataset include genotype calls at SNP positions using biallelic notation (allele/allele), when in some cases where an allele is missing, a "?" is put in place.  
* The file details code shows that the `fang_et_al_genotypes` file is 11 MB; there are 2783 lines present; there are 2744038 words present; there are 986 columns in the dataset.  
* Before data processing begins, the identification of non-ASCII files/characters code was performed, which revealed the file only contained standard text characters, and that the dataset had "very long lines (10846)"

###Attributes of `snp_position.txt`

```
# initial look
(head -n 5; tail -n 5) < snp_position.txt



```

By inspecting this file I learned that:

* This file 














