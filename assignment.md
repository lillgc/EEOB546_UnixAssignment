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
awk 'NR==1 || $3 ~ /ZMMIL|ZMMLR|ZMMMR/' fang_et_al_genotypes.txt > maize_genotypes.txt
wc -l maize_genotypes.txt  #1573 
cut -d $'\t' --complement -f2,3 maize_genotypes.txt > maize_clean_genotype.txt
```
*Explanation*: I decided to filter the maize groups and put the data into a separate file so I wouldn't have to filter after transposing. The first line keeps the first row (the column headers), looks through the 3rd column for the group name strings in the `fang_et_al_genotypes.txt` file, and makes a new file `maize_genotypes.txt`. The third line removes column 2 & 3, since those are no longer necessary for this particular analysis. 


__Filter for teosinte (Group = ZMPBA, ZMPIL, and ZMPJA) & create new file__

```
awk 'NR==1 || $3 ~ /ZMPBA|ZMPIL|ZMPJA/' fang_et_al_genotypes.txt > teosinte_genotypes.txt
wc -l teosinte_genotypes.txt  #975
cut -d $'\t' --complement -f2,3 teosinte_genotypes.txt > teosinte_clean_genotype.txt
```
*Explanation*: Same explanation as filtering for maize, but with the teosinte groups.


__Select for `SNP_ID`, `Chromosome`, and `Position` in `snp_position.txt` file__

```
awk -F'\t' 'BEGIN {OFS="\t"} { print $1, $3, $4 }' snp_position.txt > snp_position_clean.txt
```
*Explanation*: Since the `snp_position.txt` has a lot of extra information, I filtered for only the necessary columns: `SNP_ID`, `Chromosome`, `Position`. The `'BEGIN {OFS="\t"}` code ensures the new file `snp_position_clean.txt` stays in tab delimited format (or the join command will break)


__Transpose the genotype data__

```
# maize
awk -F'\t' -f transpose.awk maize_clean_genotype.txt > transposed_maize_genotypes.txt

# teosinte
awk -F'\t' -f transpose.awk teosinte_clean_genotype.txt > transposed_teosinte_genotypes.txt
```
*Explanation*: These commands transpose the filtered genotype files using the transpose.awk script. 


__Sort the data so join is aligned__

```
# snp
head -n 1 snp_position_clean.txt > snp_position_clean_header.txt                #extracting header
tail -n +2 snp_position_clean.txt | sort -k1,1 > snp_position_clean_sorted.txt  #sorting data w/o headers
cat snp_position_clean_header.txt snp_position_clean_sorted.txt > snp_position_for_join.txt     #reassembly

# maize
head -n 1 transposed_maize_genotypes.txt > transposed_maize_genotype_header.txt                  #extracting header
tail -n +2 transposed_maize_genotypes.txt | sort -k1,1 > transposed_maize_genotype_sorted.txt    #sorting data w/o headers
cat transposed_maize_genotype_header.txt transposed_maize_genotype_sorted.txt > maize_for_join.txt

# teosinte
head -n 1 transposed_teosinte_genotypes.txt > transposed_teosinte_genotype_header.txt   # Extracting header
tail -n +2 transposed_teosinte_genotypes.txt | sort -k1,1 > transposed_teosinte_genotype_sorted.txt   # Sorting data w/o headers
cat transposed_teosinte_genotype_header.txt transposed_teosinte_genotype_sorted.txt > teosinte_for_join.txt   # Combining header with sorted data

# `Sample_ID` to `SNP_ID`
sed -i '1s/^Sample_ID/SNP_ID/' maize_for_join.txt 
sed -i '1s/^Sample_ID/SNP_ID/' teosinte_for_join.txt
```
*Explanation*: These commands remove and save the headers of the files, sort the SNPs, and reassemble the file. For a clean join, the `Sample_ID` is renamed to `SNP_ID` to represent the rows underneath the column. 


__Join the transposed genotype data with snp positions file by first column__

```
# maize
join -t $'\t' -1 1 -2 1 -a 1 snp_position_for_join.txt maize_for_join.txt > merged_maize_data.txt

# teosinte
join -t $'\t' -1 1 -2 1 -a 1 snp_position_for_join.txt teosinte_for_join.txt > merged_teosinte_data.txt
```
*Explanation*: These commands join the `snp_position.txt` file with the genotype files so that the merged files will contain SNP IDs with chromosome, position, and sample genotype data.


__Filter by chromosome__

```
# maize ascending
awk -F'\t' 'NR==1 || $2 == "1" {gsub(/NA/, "?/?"); gsub(/\?\/\?/, "?/?"); print}' merged_maize_data.txt | sort -k3,3n > maize_chr1_asc.txt #1
awk '{print $2, $3}' maize_chr1_asc.txt #check

for i in {2..10}; do
    awk -F'\t' 'NR==1 || $2 == "'$i'" {gsub(/NA/, "?/?"); gsub(/\?\/\?/, "?/?"); print}' merged_maize_data.txt | sort -k3,3n > maize_chr"${i}"_asc.txt
done
awk '{print $2, $3}' maize_chr4_asc.txt #check

# maize descending
awk -F'\t' 'NR==1 || $2 == "1" {gsub(/NA/, "?/?"); gsub(/\?\/\?/, "-/-"); print}' merged_maize_data.txt | sort -k3,3nr > maize_chr1_desc.txt #1
awk '{print $2, $3}' maize_chr1_desc.txt #check

for i in {2..10}; do
    awk -F'\t' 'NR==1 || $2 == "'$i'" {gsub(/NA/, "?/?"); gsub(/\?\/\?/, "-/-"); print}' merged_maize_data.txt | sort -k3,3nr > maize_chr"${i}"_desc.txt
done
awk '{print $2, $3}' maize_chr4_desc.txt #check


# teosinte ascending
(head -n 1 merged_teosinte_data.txt && tail -n +2 merged_teosinte_data.txt | awk -F'\t' '$2 == "1" {gsub(/NA/, "?/?"); gsub(/\?\/\?/, "-/-"); print}' | sort -k3,3nr) > teosinte_chr1_desc.txt #1
awk '{print $2, $3}' teosinte_chr1_desc.txt #check

for i in {2..10}; do
    (head -n 1 merged_teosinte_data.txt && tail -n +2 merged_teosinte_data.txt | awk -F'\t' '$2 == "'$i'" {gsub(/NA/, "?/?"); gsub(/\?\/\?/, "-/-"); print}' | sort -k3,3nr) > teosinte_chr"${i}"_desc.txt
done
awk '{print $2, $3}' teosinte_chr6_desc.txt #check

# teosinte descending
awk -F'\t' 'NR==1 || $2 == "1" {gsub(/NA/, "?/?"); gsub(/\?\/\?/, "?/?"); print}' merged_teosinte_data.txt | sort -k3,3n > teosinte_chr1_asc.txt #1
awk '{print $2, $3}' teosinte_chr1_asc.txt #check

for i in {2..10}; do
    awk -F'\t' 'NR==1 || $2 == "'$i'" {gsub(/NA/, "?/?"); gsub(/\?\/\?/, "?/?"); print}' merged_teosinte_data.txt | sort -k3,3n > teosinte_chr"${i}"_asc.txt
Done
awk '{print $2, $3}' teosinte_chr10_asc.txt #check
```
*Explanation*: The first commands (for maize and teosinte) filter by chromosome (column 2), convert the missing data to their proper form (?/? for ascending and -/- for descending), sort in either ascending or descending order (header maintained at top for descending through additional code), and finally save work as a new file. The second commands check success. The for loop then completes the same functions for all remaining chromosomes.   


__Filter for SNPs w/ unknown positions__

```
# maize
awk -F'\t' 'NR==1 || $3 == "unknown"' merged_maize_data.txt > maize_unknown_positions.txt

# teosinte
awk -F'\t' 'NR==1 || $3 == "unknown"' merged_teosinte_data.txt > teosinte_unknown_positions.txt
```
*Explanation*: These commands keep the column heads and then create a new file with rows where column 3 (`Position`) is equal to unknown, representing unknown positions in the genome.


__Filter for SNPs w/ multiple positions__

```
# maize
awk -F'\t' 'NR==1 || $3 == "multiple"' merged_maize_data.txt > maize_multiple_positions.txt

# teosinte
awk -F'\t' 'NR==1 || $3 == "multiple"' merged_teosinte_data.txt > teosinte_multiple_positions.txt
```
*Explanation*: These commands keep the column heads and then create a new file with rows where column 3 (`Position`) is equal to mulitple, representing multiple positions in the genome.


