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

* The initial look into the data file shows the first 5 and last 5 lines of the file. From this inspection, it can be gathered that the first row of the file contains column names (`SNP_ID`, `cdv_marker_id`, `Chromosome`, `Position`, `alt_pos`, `mult_positions`,	`amplicon`, etc.). The unique identifiers for the SNP markers in `SNP_ID` appear to match the SNP markers found as column names in the `fang_et_al_genotypes.txt` file. The `Chromosome` includes where the SNP is located. Many of the columns do not contain data in the dataset and spaces are left blank. 
	* There are 10 chromosomes. Some SNPs have multiple positions on multiple chromosomes. Some SNPs have unknown positions. 
* The file details code shows that the `snp_position` file is 84 K; there are 984 lines present; there are 13198 words present; there are 15 columns in the dataset.
* Before data processing begins, the identification of non-ASCII files/characters code was performed, which revealed the file only contained standard text characters



## Part II) Data Processing
*Summary of Workflow*
1. Transpose the genotype data
2. Join the transposed genotype data with snp positions file by `SNP_ID`
3. Filter maize and teosinte data
4. Separate by chromosome (1-10)
5. Sort SNPs by position (increasing & decreasing)
6. Handle missing data (NA) with "?" or "-"
7. Extract SNPs with unknown or multiple positions
8. Repeat for all chromosomes and groups

*Transpose the genotype data*
```
$ awk -f transpose.awk fang_et_al_genotypes.txt > transposed_genotypes.txt	# make transposed format of genotype data
$ awk '{print $1}' transposed_genotypes.txt | sort | uniq	# ensure function was successful (should print unique row names aka SNP IDs)
```
__Explanation__: This function transposed the `fang_et_al_genotypes.txt` file so that instead of the SNPs being columns headers, they were put into the first column as rows. The second line of code ensured that this code worked by printing out the unique rows in the first column, which should be the SNP markers as well as the `Group`, `JG_OTU`, and `Sample_ID` rows.  

*Join the transposed genotype data with snp positions file by `SNP_ID`*
```
# renaming `Sample_ID` to `SNP_ID`
$ sed -i '1s/^Sample_ID/SNP_ID/' transposed_genotypes.txt
$ head -n 1 transposed_genotypes.txt # verify 

# sorting the datasets to ensure proper join
$ sort -k1,1 transposed_genotypes.txt > transposed_genotypes_sorted.txt
$ sort -k1,1 snp_position.txt > snp_position_sorted.txt

# joining files
join -1 1 -2 1 transposed_genotypes_sorted.txt snp_position_sorted.txt > joined_data.txt





``` 








