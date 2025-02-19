# EEOB 4560 Unix Assignment


## Part I) Data Inspection

*Attributes of `fang_et_al_genotypes`*

```
# initial look
$ (head -n 5; tail -n 5) < fang_et_al_genotypes.txt

# unique groups
$ cut -f 3 fang_et_al_genotypes.txt | sort | uniq -c

# file details (file size, # of lines, # of words, # of columns)
$ du -h fang_et_al_genotypes.txt				# file size
$ wc -l fang_et_al_genotypes.txt          		# number of lines
$ wc -w fang_et_al_genotypes.txt                   	# number of words
$ awk '{print NF}' fang_et_al_genotypes.txt | head -1  	# number of columns

# identification of non-ASCII files and characters
$ file fang_et_al_genotypes.txt 
```

By inspecting this file I learned that:

* The initial look into the data file shows the first 5 and last 5 lines of the file. From this inspection, it can be gathered that the first row of the file contains column names (e.g. ` Sample_ID`, `JG_OTU`, `Group`, and a list of gene/marker names (`abph1.20`, `ae1.3`, `PZA00003.11`, etc.)). The content of the dataset mostly include genotype calls at SNP positions using biallelic notation (allele/allele), when in some cases where an allele is missing, a "?" is put in place.
	* The `Sample_ID`, `JG_OTU`, and `Group` columns contain additional information (example of content [S0398	Zmm-IL-W64A_a	ZMMIL] where the sample ID is a unique identifier for each sample; the JG OTU is potentially an operational taxonomic unit ID, and group differentiates between maize, teosinte, and tripsacum individuals. 
	* There are 16 unique groups with groups ZMPBA, ZMMLR, ZMMIL having a significantly higher count than others.  
* The file details code shows that the `fang_et_al_genotypes` file is 11 MB; there are 2783 lines present; there are 2744038 words present; there are 986 columns in the dataset.  
* Before data processing begins, the identification of non-ASCII files/characters code was performed, which revealed the file only contained standard text characters, and that the dataset had "very long lines (10846)"

*Attributes of `snp_position.txt`*

```
# initial look
$ (head -n 5; tail -n 5) < snp_position.txt

# unique chromosomes
$ cut -f 3 snp_position.txt | sort | uniq -c

# file details (file size, # of lines, # of words, # of columns)
$ du -h snp_position.txt                          # file size
$ wc -l snp_position.txt                          # number of lines
$ wc -w snp_position.txt                          # number of words
$ awk '{print NF}' snp_position.txt | head -1     # number of columns

# identification of non-ASCII files and characters
$ file snp_position.txt
```

By inspecting this file I learned that:

* The initial look into the data file shows the first 5 and last 5 lines of the file. From this inspection, it can be gathered that the first row of the file contains column names (`SNP_ID`, `cdv_marker_id`, `Chromosome`, `Position`, `alt_pos`, `mult_positions`,	`amplicon`, etc.). The unique identifiers for the SNP markers in `SNP_ID` appear to match the SNP markers found as column names in the `fang_et_al_genotypes.txt` file. The `Chromosome` includes which chromosome the SNP is located on. The `Position` column is the position of the SNP on the chromosome. There are multiple other columns containing data that do not appear relevant to this analysis. 
	* There are 10 chromosomes. Some SNPs have multiple positions on multiple chromosomes. Some SNPs have unknown positions. 
* The file details code shows that the `snp_position` file is 84 K; there are 984 lines present; there are 13198 words present; there are 15 columns in the dataset.
* Before data processing begins, the identification of non-ASCII files/characters code was performed, which revealed the file only contained standard text characters



## Part II) Data Processing
*Summary of Workflow*
1. Filter for maize (Group = ZMMIL, ZMMLR, and ZMMMR) & create new file
→ take out `JG_OTU` column
2. Filter for teosinte (Group = ZMPBA, ZMPIL, and ZMPJA) & create new file
→ take out `JG_OTU` column
3. Select for `SNP_ID`, `Chromosome`, and `Position` in `snp_position.txt` file

The following applies to both of the filtered files:

4. Sort the data so join is aligned
5. Transpose the genotype data
6. Join the transposed genotype data with snp positions file by first column (want to keep extra columns ex: `Chromosome` & `Position`)
7. Filter by chromosome (create a new file for each)
8. Filter for SNPs w/ unknown positions (create 1 file)
9. Filter for SNPs w/ multiple positions (create 1 file)

The following applies to all chromosome files:
 
10. Ensure all missing data is formatted as “?/?”
11. Sort SNPs by increasing position (create file) [10 files in total]
12. Ensure all missing data is formatted as “-/-”
13. Sort SNPs by decreasing position (create file) [10 files in total] 


__Filter for maize (Group = ZMMIL, ZMMLR, and ZMMMR) & create new file__

```
awk '$3 ~ /ZMMIL|ZMMLR|ZMMMR/' fang_et_al_genotypes.txt > maize_genotypes.txt
```
*Explanation*: 


__Filter for teosinte (Group = ZMPBA, ZMPIL, and ZMPJA) & create new file__

```
awk '$3 ~ /ZMPBA|ZMPIL|ZMPJA/' fang_et_al_genotypes.txt > teosinte_genotypes.txt
```
*Explanation*: 


__Select for `SNP_ID`, `Chromosome`, and `Position` in `snp_position.txt` file__

```

```
*Explanation*: 


__Transpose the genotype data__

```

```
*Explanation*: 


__Sort the data so join is aligned__

```

```
*Explanation*: 


__Join the transposed genotype data with snp positions file by first column__

```

```
*Explanation*: 


__Filter by chromosome__

```

```
*Explanation*: 


__Filter for SNPs w/ unknown positions__

```

```
*Explanation*: 


__Filter for SNPs w/ multiple positions (create 1 file)__

```

```
*Explanation*: 


__Ensure all missing data is formatted as “?/?” (Incr. position)__

```

```
*Explanation*: 


__Sort SNPs by increasing position (create file) [10 files in total]__

```

```
*Explanation*: 


__Ensure all missing data is formatted as “-/-” (Decr. position)__

```

```
*Explanation*: 


__Sort SNPs by decreasing position (create file) [10 files in total]__

```

```
*Explanation*: 

