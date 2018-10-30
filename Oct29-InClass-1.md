---
title: "October 29 In Class Assignment A"
author: "Emma Mulholland"
date: "October 29, 2018"
output: 
  html_document: 
    keep_md: yes
---
---
title: "In class assignment week 2, part 2. A worked example using control flow (for loops, if statements, etc)"
author: "Ian Dworkin"
output: 
  html_document:
    keep_md: yes
    number_sections: yes
---

# Introduction
Let's do a little exercise integrating some of the things we have learned. Here are some Illumina HiSeq reads for one of our recent projects:


```r
read_1 <- "CGCGCAGTAGGGCACATGCCAGGTGTCCGCCACTTGGTGGGCACACAGCCGATGACGAACGGGCTCCTTGACTATAATCTGACCCGTTTGCGTTTGGGTGACCAGGGAGAACTGGTGCTCCTGC"

read_2 <- "AAAAAGCCAACCGAGAAATCCGCCAAGCCTGGCGACAAGAAGCCAGAGCAGAAGAAGACTGCTGCGGCTCCCGCTGCCGGCAAGAAGGAGGCTGCTCCCTCGGCTGCCAAGCCAGCTGCCGCTG"

read_3  <- "CAGCACGGACTGGGGCTTCTTGCCGGCGAGGACCTTCTTCTTGGCATCCTTGCTCTTGGCCTTGGCGGCCGCGGTCGTCTTTACGGCCGCGGGCTTCTTGGCAGCAGCACCGGCGGTCGCTGGC"
```

Question 1. what species are these sequences from?
All three sequences are from *D. melanogaster* (determined using NCBI BLAST).

Question 2. Put al?l three of these reads into a single object (a vector).  What class will the vector `reads` be? Check to make sure! How many characters are in each read (and why does `length()` not give you what you want.. try...)

```r
reads<-c(read_1, read_2, read_3)
class(reads)
```

```
## [1] "character"
```

```r
nchar(reads)
```

```
## [1] 124 124 124
```
`length()` doesn't give you the number of characters in each string, it will give you the number of objects in the vector.

Question 3. Say we wanted to print each character (not the full string) from read_1, how do we do this using a for loop? You may wish to look at a function like `strsplit()` to accomplish this (there are other ways.)

Replace the blanks.

```r
read_1_split <- strsplit(read_1, split = "", fixed = T)
```

Question 4. What kind of object does this return? How might we make it a character vector again?

This returns a list. You can make it a character vector with the `unlist()` function and by assigning the output to a variable.

```r
read_1_split<-unlist(read_1_split)
str(read_1_split)
```

```
##  chr [1:124] "C" "G" "C" "G" "C" "A" "G" "T" "A" "G" "G" "G" "C" "A" ...
```

Question 5. How about if we wanted the number of occurrences of each base? Or better yet, their frequencies? You could write a loop, but I suggest looking at the help for the `table()` function... Also keep in mind that for for most objects `length()` tells you how many elements there are in a vector. For lists use `lengths()` (so you can either do this on a character vector or a list, your choice)

```r
#for a character vector
base_count<-table(read_1_split)
base_freq<-base_count/length(read_1_split)
print(base_count)
```

```
## read_1_split
##  A  C  G  T 
## 23 35 40 26
```

```r
print(base_freq)
```

```
## read_1_split
##         A         C         G         T 
## 0.1854839 0.2822581 0.3225806 0.2096774
```

Question 6. How would you make this into a nice looking function that can work on either a list or vectors of characters? (Still just for a single read)

```r
base_freq<-function(x){
  if (class(x)=="list"){
    x<-unlist(x)
  }
  return(table(x)/length(x))
}
```
Question 7. Now how can you modify your approach to do it for an arbitrary numbers of reads? You could use a loop or use one of the apply like functions (which one)?


Question 8. Can you revise your function so that it can handle the input of either a string as a single multicharacter vector, **or** a vector of individual characters **or** a list? Try it out with the vector of three sequence reads (`reads`).  

```r
#I copied out the answer here when we went through it; on our own we only made it to Q6.
BaseFrequencies<-function(x){
  if(length(x)==1 & mode(x)=="character"){
    x<-strsplit(x, split = "", fixed = TRUE)
    x<-as.character(unlist(x))
  }
  if (mode(x)=="list"){
    tab<-table(x)/lengths(x)
  }
  else{
    tab<-table(x)/length(x)
  }
  return(tab)
}
```
