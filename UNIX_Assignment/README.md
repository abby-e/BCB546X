#This is my document about my UNIX assignment.


###Step 1: File Inspection


```
ls -lh fang_et_al_genotypes.txt
```

I used this command to inspect the fang_genotypes file; the result was that the file is 11MB in size.

```
head fang_et_al_genotypes.txt
```

It seemed to me that this command was not working as I expected, because it returned a lot of information. Let's see what the awk program results.

```
awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt 
```

I used this command to try to figure out how many columns are in this file. The result is 986 columns, which could be why head seems to be printing too much to standard out.

```
wc fang_et_al_genotypes.txt
```

This command told me there are 2783 lines,  2744038 words, and 11051939 bytes (characters) in the file.

```
less -S fang_et_al_genotypes.txt
```

This command turns off the text-wrapping in the less command, and it is clear that the long rows of text is probably what was affecting the head command previously.

I used all of these commands on the snp_position.txt file as well. I was able to determine that this file is 91KB in size, it has 15 columns, 984 lines, 13198 words, and 82763 characters. 

Additionally, we care about the SNP ID, Chromosome, and Position information from each file. Chromosome is column 3 in snp_position.txt and position is column 4 in the same file.

###Step 2: Maize Data Processing

For the maize genes, I want one file for each chromosome that contains the SNPs ordered based on increasing position and another file that contains the SNPs ordered based on decreasing position. I first need to figure out how to make a file with just the information based on the maize samples.

```
grep "Group" fang_et_al_genotypes.txt > maize_genotypes.txt 
```

I wanted to preserve the line with the headers in it that match the snp_position file. Now I need to append the rows I want to that file. The double greater-thans let me do that.


```
grep "ZMM[IL,LR,MR]" fang_et_al_genotypes.txt >> maize_genotypes.txt
```

Awesome, now I have a file with the information that I want and also some metadata.

I think I will next need to join the information on the two files, and then use a cut command piped to a command to create a new file from the standard out.

```
awk -f transpose.awk maize_genotypes.txt > transposed_maize_genotypes.txt
```

I used the transpose.awk function to transpose the maize\_genotypes.txt file to create transposed\_maize_genotypes.txt. This new file now has 1574 columns, which was previously the number of rows of the maize\_genotypes.txt file, +1 for the metadata line.

The column that the join will be based on for the transposed\_maize\_genotypes.txt file is column 1. I will be joining the entire contents of this file to selected columns in the snp\_positions.txt file. Before doing this, I used vi to change the column header from group to SNP\_ID in the transposed\_maize\_genotypes.txt file so that the first line can be included in the join.

```
cut -f 1,3,4 snp_position.txt > maize_snp_position.txt
```

I cut just the columns that we care about from the snp\_position.txt file and created a new file to use in the join.

```
join -1 1 -2 1 maize_snp_position.txt transposed_maize_genotypes.txt > maize_join.txt
```

I used less -S to inspect the file, and YAY, I have all the info that I wanted included in this file. Now I will be able to cut just the info that I want.

```
awk '$2 ==1' maize_join.txt > maize_chr1.txt
```

I will use this command to create the files for the maize data separated by chromosome. Now that I have the files i want, the data that I want to sort on is in column 3 (position data). I have to tell the sort command that these are numeric data.

```
sort -k3,3n maize_chr1.txt > maize_chr1_for.txt
```

I will do this for each file, making a forward and backwards sort.

```
sort -k3,3nr maize_chr1.txt > maize_chr1_rev.txt
```

Great. Now I have things sorted forwards and backwards based on snp\_position. Next I will need to figure out which SNPs have unknown positions and which are found at more than one position.

```
awk '$2 =="unknown"' maize_join.txt > maize_unk.txt
```

This made the file with the SNPs with unknown positions.

```
awk '$2 =="multiple"' maize_join.txt > maize_multiple.txt
```

This made the file with the SNPs with multiple positions. Now I should clean up my directory by removing my intermediate files created from pulling out the data from maize\_join.txt into individual files. For teosinte individuals I will try to use a pipe to eliminate making these intermediate files.

###Step 3: Teosinte Data Processing

The teosinte genotypes are groups ZMPBA, ZMPIL, and ZMPJA.

```
grep "Group" fang_et_al_genotypes.txt > teosinte_genotypes.txt
```

```
grep "ZMP[JA,IL,BA]" fang_et_al_genotypes.txt >> teosinte_genotypes.txt
```


```
awk -f transpose.awk teosinte_genotypes.txt > transposed_teosinte_genotypes.txt
```


I made a teosinte directory for housing my files.

```
mkdir teosinte
```

```
cut -f 1,3,4 snp_position.txt > teosinte_snp_position.txt
```

I again used the join command to join the information in the teosinte\_snp\_position.txt file and the teosinte\_genotypes.txt file. I then used the following command to avoid creating the intermediate files that I made last time.

```
awk '$2 ==1' teosinte_join.txt | sort -k3,3n > teosinte_chr1_for.txt
```

I did the same thing to make the reverse files.

```
awk '$2 ==1' teosinte_join.txt | sort -k3,3nr > teosinte_chr1_rev.txt
```

I used the same syntax as the maize situation to make the files with SNPs from unknown or multiple positions.

```
awk '$2 =="multiple"' teosinte_join.txt > teosinte_multiple.txt
```

```
awk '$2 =="unknown"' teosinte_join.txt > teosinte_unk.txt
```

Lastly, I remembered to change the way missing data was encoded in the reverse sorted files. This probably could have been part of my pipe, but I didn't do this until the end and had to also go back and do it for the maize files.

```
sed -i 's/?/-/g' teosinte_chr10_rev.txt
```

