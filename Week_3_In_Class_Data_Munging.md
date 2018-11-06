---
title: "Bio720_DataMunging"
author: "Ian Dworkin"
date: "November 5th, 2018"
output: 
  html_document: 
    keep_md: yes
    number_sections: yes
    toc: yes
---


# Data parsing and munging in Base R

## Brief introduction
One of the most common "programming" activities you will do in genomics and bioinfomatics is to get the data into a format that facilitates (and makes possible) the analyses you are doing. This activity can often occupy very large amounts of your time

These activities have many names: *wrangling data, munging data, cleaning data, data manipulation*. Given how important this is, it will come as no surprise that there are entire R libraries devoted to it. There are several lists that are worth looking at like [here](http://datascienceplus.com/best-packages-for-data-manipulation-in-r/) and [here](https://www.analyticsvidhya.com/blog/2015/12/faster-data-manipulation-7-packages/). Also, books like [Data Manipulation with R](https://www.amazon.ca/Data-Manipulation-R-Phil-Spector/dp/0387747303)!

## Why no tidyverse in this tutorial
You have already been introduced to a little bit of the use of the *dplyr* library and verbs within it like `filter()`, `arrange()`, `group_by()` and `summarize()` (one other important one is `select()`). You will have more opportunities to learn aspects of the tidyverse (more on dplyr and ggplot2 in particular). However, today I am not going to show you data munging this way, instead relying on base R techniques.

Why? tidyverse libraries are excellent in many ways, *but* they do have some issues. First and foremost, there are lots of dependencies (so if you load library "a", you also need libraries "b", "c", "d", etc). That can add a lot of libraries potentially. Second, unlike base R where the code is very (probably too) conservative, and those that maintain and alter the code do so in a manner that is very unlikely to alter downstream behaviour (i.e. code in other libraries), tidyverse libraries change more rapidly (and thus, while still very unlikely are more likely to break existing code). 

Currently, *[Bioconductor](http://bioconductor.org/)* objects don't always "play nice" with tidyverse tibbles and approaches. These are essentially two independently evolving "ecosystems of syntax" (really macro languages) in *R*. There is a move to integrate these better. But at this time, if you are planning on using libraries from *Bioconductor* for genomic and bioinformatic analysis it is important to know how to work with data in base R.

It is worth thinking about when to use tidyverse, and when not to.

If you are writing scripts that are just for your own data analysis (i.e. you are not writing a general purpose library) and are not interacting with *Bioconductor* objects too much (or don't mind altering objects from one to the other), then the *tidyverse* is an extremely smart choice. Fast to learn, relatively consistent (in comparison to Base *R*), and often faster than base *R*.

## *data.table* library for fast import and execution of large data sets.
Finally it is worth pointing out that the *data.table* library is not only the most efficient for importing large data sets (by a wide margin usually), but data munging (sorting, subsetting, etc..) can also be done via this library with its own syntax. For large datasets, this is usually the fastest. Datacamp has a course on this [here](https://www.datacamp.com/courses/data-table-data-manipulation-r-tutorial). Like `readr` and *tibbles* in tidyverse, the `data.table` is a slightly different object from a standard `data.frame`. If you are working with large datasets in `R`, knowing this library is essential! 

## for your own review

We have already spent considerable time in previous tutorials making data frames using `cbind`, `rbind`, `data.frame`, etc.. As we have limited time in class I leave review of that to you from the screencasts and activites at datacamp.

## install packages.
You may wish to install certain `R` libraries that get used a lot. While R-studio has a number of easy ways to install libraries I still prefer having it in the script. You only need to install the libraries once, although it is worth updating them regularly, and I re-install them everytime I update `R`.


```r
# remove hash to install.
#install.packages("data.table")
```

## Data we are using.

We will use a number of different data sets to illustrate each of the points, but for the main one, we will use an old *Drosophila melanogaster* data set measuring several traits (lengths) and the number of sex comb teeth (a structure used to clasp females during copulation) for different wild type strains (line) reared at different developmental temperatures, with and without a mutation that effects proximal-distal axis development in limbs (genotype). Note that we are downloading the data from a data repository (Dryad). If you really care, it is from [this](https://www.ncbi.nlm.nih.gov/pubmed/15733306) paper.



```r
dll_data = read.csv("http://datadryad.org/bitstream/handle/10255/dryad.8377/dll.csv", header=TRUE)
```


Before we go on, how should we look at the data to make sure it imported correctly, and the structure (and other information) about the object we have just created?
 

```r
summary(dll_data)
```

```
##    replicate         line      genotype        temp          femur     
##  Min.   :1.00   line-7 : 132   Dll: 871   Min.   :25.0   Min.   :0.21  
##  1st Qu.:1.00   line-18: 121   wt :1102   1st Qu.:25.0   1st Qu.:0.53  
##  Median :1.00   line-4 : 112              Median :25.0   Median :0.55  
##  Mean   :1.18   line-8 : 110              Mean   :27.4   Mean   :0.55  
##  3rd Qu.:1.00   line-2 : 104              3rd Qu.:30.0   3rd Qu.:0.57  
##  Max.   :2.00   line-11: 100              Max.   :30.0   Max.   :0.70  
##                 (Other):1294                             NA's   :24    
##      tibia          tarsus          SCT      
##  Min.   :0.34   Min.   :0.11   Min.   : 6.0  
##  1st Qu.:0.46   1st Qu.:0.18   1st Qu.:10.0  
##  Median :0.48   Median :0.19   Median :11.0  
##  Mean   :0.48   Mean   :0.19   Mean   :11.2  
##  3rd Qu.:0.50   3rd Qu.:0.20   3rd Qu.:12.0  
##  Max.   :0.61   Max.   :0.26   Max.   :32.0  
##  NA's   :19     NA's   :17     NA's   :25
```

```r
str(dll_data)
```

```
## 'data.frame':	1973 obs. of  8 variables:
##  $ replicate: int  1 1 1 1 1 1 1 1 1 1 ...
##  $ line     : Factor w/ 27 levels "line-1","line-11",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ genotype : Factor w/ 2 levels "Dll","wt": 1 1 1 1 1 1 1 1 1 1 ...
##  $ temp     : int  25 25 25 25 25 25 25 25 25 25 ...
##  $ femur    : num  0.59 0.55 0.588 0.588 0.596 ...
##  $ tibia    : num  0.499 0.501 0.488 0.515 0.502 ...
##  $ tarsus   : num  0.219 0.214 0.211 0.211 0.207 ...
##  $ SCT      : int  9 13 11 NA 12 14 11 12 10 12 ...
```

```r
dim(dll_data)
```

```
## [1] 1973    8
```

```r
head(dll_data)
```

```
##   replicate   line genotype temp femur tibia tarsus SCT
## 1         1 line-1      Dll   25 0.590 0.499  0.219   9
## 2         1 line-1      Dll   25 0.550 0.501  0.214  13
## 3         1 line-1      Dll   25 0.588 0.488  0.211  11
## 4         1 line-1      Dll   25 0.588 0.515  0.211  NA
## 5         1 line-1      Dll   25 0.596 0.502  0.207  12
## 6         1 line-1      Dll   25 0.577 0.499  0.207  14
```

```r
tail(dll_data)
```

```
##      replicate   line genotype temp femur tibia tarsus SCT
## 1968         1 line-w       wt   30 0.507 0.480  0.181  10
## 1969         1 line-w       wt   30 0.528 0.485  0.178   9
## 1970         1 line-w       wt   30    NA    NA  0.175  11
## 1971         1 line-w       wt   30 0.545 0.486  0.174  12
## 1972         1 line-w       wt   30 0.542 0.504  0.167  12
## 1973         1 line-w       wt   30 0.511 0.469  0.165  11
```
- Since we are using base *R*, this is a regular data.frame.

## Cleaning data
### removing missing data

Sometimes your data set has missing data, i.e. for some reason you could not measure one of your variables on a particular object. How you decide to deal with missing data can be a big topic, but for the moment we are going to assume you want to delete rows that contain missing data. 

First let's check if there is any missing data. Use `is.na` to see if there is any? How many observations have missing data

```r
sum(is.na(dll_data))
```

```
## [1] 85
```

```r
#Gives us all the rows with missing data
```

More generally we can use `anyNA()`. Try it on this data:

```r
anyNA(dll_data)
```

```
## [1] TRUE
```

```r
# Function asks are there any NAs in this data set?
```

### Why missing data can cause headaches

```r
mean(dll_data$femur)
```

```
## [1] NA
```
- Since there is some missing data, the `mean()` function defaults to NA, to warn you. This is a useful behaviour.


```r
mean(dll_data$femur, na.rm = TRUE)
```

```
## [1] 0.546
```


### What to do with this rows with missing data
We need to decide what to do with the missing data. If you have lots of data, and do not mind removing all cases (rows) that contain **any** missing data then `na.omit` is a useful solution. How does using `na.omit()` change the number of observations for *dll.data*? Don't erase the original data!


There are numerous options about how `R` should handle operations if an `NA` is found. You can look at the help under `?NA`. 


For the moment, we are going to accept the approach above, and use it. However, I don't generally recommend this. Instead, you can first subset the variables you are going to use, and then decide how to deal with the missing data. Or you can allow them into your analysis, but make sure to set the right flags in functions that enable them to deal with missing data.


```r
dll_data <- na.omit(dll_data)
mean(dll_data$femur)
```

```
## [1] 0.546
```

```r
#na.omit removes the entire row with an NA
```
- Note that the mean now is not the same as before. This is because additional variables (not just femur) had missing data for different observations, but we removed any observation with any missing data!

### finding and removing duplicate rows in a data frame.
The `duplicated()` allows you to see which, if any rows are perfect duplicates of one another.

We can first ask if any rows are duplicates.  Use `duplicated` to check.

```r
sum(duplicated(dll_data))
```

```
## [1] 0
```


How might we find how many duplicated?


Or if you want it as a Boolean (hint use `any`)


`R` has a convenience function for the above:


```r
any(duplicated(dll_data))
```

```
## [1] FALSE
```
So for this example there was one duplicate rows.

How can we use anyDuplicated to pull out the duplicated rows?

However let's say an accident happened in generating the data set and a few rows were duplicated. I am going to use the `sample()` function to extract 5 random rows.


```r
new_rows <- dll_data[sample(nrow(dll_data), size = 5, replace = T ),]

dll_data2 <- rbind(dll_data, new_rows)
str(dll_data2)
```

```
## 'data.frame':	1923 obs. of  8 variables:
##  $ replicate: int  1 1 1 1 1 1 1 1 1 1 ...
##  $ line     : Factor w/ 27 levels "line-1","line-11",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ genotype : Factor w/ 2 levels "Dll","wt": 1 1 1 1 1 1 1 1 1 1 ...
##  $ temp     : int  25 25 25 25 25 25 25 25 25 25 ...
##  $ femur    : num  0.59 0.55 0.588 0.596 0.577 ...
##  $ tibia    : num  0.499 0.501 0.488 0.502 0.499 ...
##  $ tarsus   : num  0.219 0.214 0.211 0.207 0.207 ...
##  $ SCT      : int  9 13 11 12 14 11 12 10 12 13 ...
##  - attr(*, "na.action")= 'omit' Named int  4 61 73 92 93 142 207 268 315 319 ...
##   ..- attr(*, "names")= chr  "4" "61" "73" "92" ...
```

Now you don't know which rows, they are.  Please show how you would find whether there are any duplicated rows, and which ones they are using `duplicated()`

```r
any(duplicated(dll_data2))
```

```
## [1] TRUE
```

```r
dll_data2[any(duplicated(dll_data2))] #I'm not sure this is right?
```

```
##       replicate      line genotype temp femur tibia tarsus SCT
## 1             1    line-1      Dll   25 0.590 0.499  0.219   9
## 2             1    line-1      Dll   25 0.550 0.501  0.214  13
## 3             1    line-1      Dll   25 0.588 0.488  0.211  11
## 5             1    line-1      Dll   25 0.596 0.502  0.207  12
## 6             1    line-1      Dll   25 0.577 0.499  0.207  14
## 7             1    line-1      Dll   25 0.618 0.494  0.204  11
## 8             1    line-1      Dll   25 0.582 0.528  0.203  12
## 9             1    line-1      Dll   25 0.572 0.479  0.202  10
## 10            1    line-1      Dll   25 0.568 0.479  0.202  12
## 11            1    line-1      Dll   25 0.555 0.518  0.200  13
## 12            1    line-1      Dll   25 0.596 0.505  0.200  12
## 13            1    line-1      Dll   25 0.570 0.511  0.200  11
## 14            1    line-1      Dll   25 0.606 0.504  0.196  13
## 15            1    line-1      Dll   25 0.544 0.504  0.191  12
## 16            1    line-1      Dll   25 0.578 0.528  0.190  12
## 17            1    line-1      Dll   25 0.607 0.508  0.190  12
## 18            1    line-1      Dll   25 0.583 0.499  0.188  10
## 19            1    line-1      Dll   25 0.571 0.513  0.185  14
## 20            1    line-1      Dll   25 0.451 0.342  0.165  16
## 21            1   line-11      Dll   25 0.590 0.496  0.214  14
## 22            1   line-11      Dll   25 0.519 0.482  0.209  13
## 23            1   line-11      Dll   25 0.597 0.496  0.204  11
## 24            1   line-11      Dll   25 0.544 0.511  0.202  15
## 25            1   line-11      Dll   25 0.541 0.499  0.200  12
## 26            1   line-11      Dll   25 0.572 0.489  0.198  14
## 27            1   line-11      Dll   25 0.551 0.500  0.198  14
## 28            1   line-11      Dll   25 0.565 0.490  0.198  14
## 29            1   line-11      Dll   25 0.584 0.500  0.197  13
## 30            1   line-11      Dll   25 0.563 0.515  0.195  14
## 31            1   line-11      Dll   25 0.578 0.507  0.191  13
## 32            1   line-11      Dll   25 0.581 0.488  0.190  13
## 33            1   line-11      Dll   25 0.533 0.496  0.187  13
## 34            1   line-11      Dll   25 0.564 0.494  0.176  13
## 35            1   line-11      Dll   25 0.564 0.505  0.172  13
## 36            1   line-11      Dll   25 0.561 0.482  0.166  13
## 37            1   line-12      Dll   25 0.586 0.506  0.219  11
## 38            1   line-12      Dll   25 0.557 0.488  0.217  12
## 39            1   line-12      Dll   25 0.575 0.527  0.215  11
## 40            1   line-12      Dll   25 0.546 0.497  0.211  11
## 41            1   line-12      Dll   25 0.541 0.514  0.207  11
## 42            1   line-12      Dll   25 0.544 0.494  0.206  10
## 43            1   line-12      Dll   25 0.575 0.515  0.205  12
## 44            1   line-12      Dll   25 0.569 0.506  0.205  14
## 45            1   line-12      Dll   25 0.556 0.498  0.204  10
## 46            1   line-12      Dll   25 0.556 0.490  0.203  11
## 47            1   line-12      Dll   25 0.569 0.509  0.203  13
## 48            1   line-12      Dll   25 0.557 0.502  0.200  11
## 49            1   line-12      Dll   25 0.494 0.496  0.199  11
## 50            1   line-12      Dll   25 0.558 0.516  0.190   9
## 51            1   line-12      Dll   25 0.570 0.500  0.187  10
## 52            1   line-12      Dll   25 0.567 0.523  0.185  11
## 53            1   line-12      Dll   25 0.552 0.531  0.185  11
## 54            1   line-12      Dll   25 0.560 0.484  0.184  11
## 55            1   line-12      Dll   25 0.569 0.520  0.183  13
## 56            1   line-12      Dll   25 0.552 0.521  0.183  11
## 57            1   line-13      Dll   25 0.557 0.483  0.225  14
## 58            1   line-13      Dll   25 0.570 0.506  0.223  16
## 59            1   line-13      Dll   25 0.585 0.495  0.223  14
## 60            1   line-13      Dll   25 0.561 0.499  0.220  12
## 62            1   line-13      Dll   25 0.583 0.526  0.217  20
## 63            1   line-13      Dll   25 0.555 0.482  0.215  12
## 64            1   line-13      Dll   25 0.578 0.478  0.214  14
## 65            1   line-13      Dll   25 0.546 0.464  0.205  13
## 66            1   line-13      Dll   25 0.540 0.483  0.200  13
## 67            1   line-13      Dll   25 0.592 0.493  0.198  15
## 68            1   line-13      Dll   25 0.531 0.487  0.195  12
## 69            1   line-13      Dll   25 0.560 0.485  0.192  11
## 70            1   line-13      Dll   25 0.569 0.512  0.191  12
## 71            1   line-13      Dll   25 0.570 0.480  0.190  13
## 72            1   line-13      Dll   25 0.570 0.508  0.186  13
## 74            1   line-15      Dll   25 0.552 0.523  0.231  12
## 75            1   line-15      Dll   25 0.590 0.519  0.224  13
## 76            1   line-15      Dll   25 0.572 0.503  0.213  12
## 77            1   line-15      Dll   25 0.578 0.532  0.211  13
## 78            1   line-15      Dll   25 0.580 0.532  0.210  14
## 79            1   line-15      Dll   25 0.559 0.521  0.209  15
## 80            1   line-15      Dll   25 0.562 0.515  0.207  12
## 81            1   line-15      Dll   25 0.612 0.533  0.203  14
## 82            1   line-15      Dll   25 0.568 0.535  0.201  14
## 83            1   line-15      Dll   25 0.580 0.503  0.200  12
## 84            1   line-15      Dll   25 0.575 0.517  0.200  14
## 85            1   line-15      Dll   25 0.588 0.522  0.198  15
## 86            1   line-15      Dll   25 0.561 0.527  0.197  13
## 87            1   line-15      Dll   25 0.582 0.527  0.194  13
## 88            1   line-15      Dll   25 0.553 0.487  0.194  12
## 89            1   line-15      Dll   25 0.588 0.530  0.190  12
## 90            1   line-15      Dll   25 0.600 0.551  0.187  10
## 91            1   line-15      Dll   25 0.577 0.528  0.183  12
## 94            1   line-16      Dll   25 0.592 0.521  0.216  16
## 95            1   line-16      Dll   25 0.580 0.498  0.210  13
## 96            1   line-16      Dll   25 0.550 0.483  0.202  10
## 97            1   line-16      Dll   25 0.558 0.484  0.196  11
## 98            1   line-16      Dll   25 0.575 0.521  0.194  14
## 99            1   line-16      Dll   25 0.566 0.514  0.194   9
## 100           1   line-16      Dll   25 0.555 0.472  0.193  15
## 101           1   line-16      Dll   25 0.560 0.487  0.191   9
## 102           1   line-16      Dll   25 0.558 0.479  0.190  17
## 103           1   line-16      Dll   25 0.556 0.473  0.189  15
## 104           1   line-16      Dll   25 0.570 0.513  0.185  12
## 105           1   line-17      Dll   25 0.548 0.489  0.203  14
## 106           1   line-17      Dll   25 0.554 0.509  0.195  12
## 107           1   line-17      Dll   25 0.544 0.488  0.191  14
## 108           1   line-17      Dll   25 0.557 0.510  0.188  14
## 109           1   line-17      Dll   25 0.551 0.477  0.176  10
## 110           1   line-17      Dll   25 0.538 0.465  0.175  11
## 111           1   line-17      Dll   25 0.597 0.529  0.175  13
## 112           1   line-17      Dll   25 0.509 0.474  0.172  16
## 113           1   line-17      Dll   25 0.532 0.492  0.171  11
## 114           1   line-17      Dll   25 0.539 0.504  0.161  11
## 115           1   line-18      Dll   25 0.547 0.465  0.209  11
## 116           1   line-18      Dll   25 0.576 0.485  0.208  11
## 117           1   line-18      Dll   25 0.542 0.489  0.199  13
## 118           1   line-18      Dll   25 0.561 0.485  0.194  13
## 119           1   line-18      Dll   25 0.571 0.499  0.194  14
## 120           1   line-18      Dll   25 0.570 0.458  0.192  13
## 121           1   line-18      Dll   25 0.564 0.509  0.190  10
## 122           1   line-18      Dll   25 0.555 0.510  0.189  10
## 123           1   line-18      Dll   25 0.503 0.490  0.188  11
## 124           1   line-18      Dll   25 0.554 0.517  0.185  12
## 125           1   line-18      Dll   25 0.573 0.479  0.182  11
## 126           1   line-18      Dll   25 0.536 0.482  0.182  11
## 127           1   line-18      Dll   25 0.516 0.459  0.182  11
## 128           1   line-18      Dll   25 0.555 0.492  0.176  11
## 129           1   line-18      Dll   25 0.551 0.480  0.174  13
## 130           1   line-18      Dll   25 0.592 0.501  0.170  12
## 131           1   line-18      Dll   25 0.548 0.445  0.170  10
## 132           1   line-18      Dll   25 0.566 0.487  0.166  11
## 133           1   line-18      Dll   25 0.557 0.511  0.161  12
## 134           1   line-18      Dll   25 0.558 0.497  0.157  11
## 135           1   line-18      Dll   25 0.570 0.492  0.152  10
## 136           1   line-19      Dll   25 0.560 0.494  0.205  12
## 137           1   line-19      Dll   25 0.557 0.494  0.188  11
## 138           1   line-19      Dll   25 0.545 0.465  0.180  13
## 139           1   line-19      Dll   25 0.542 0.485  0.174  12
## 140           2    line-2      Dll   25 0.553 0.519  0.208  14
## 141           1    line-2      Dll   25 0.545 0.496  0.203  13
## 143           2    line-2      Dll   25 0.563 0.509  0.202  13
## 144           1    line-2      Dll   25 0.573 0.483  0.196  11
## 145           1    line-2      Dll   25 0.558 0.488  0.196  16
## 146           1    line-2      Dll   25 0.594 0.471  0.196  10
## 147           2    line-2      Dll   25 0.545 0.462  0.193  16
## 148           2    line-2      Dll   25 0.566 0.511  0.192  16
## 149           1    line-2      Dll   25 0.532 0.473  0.189  13
## 150           1    line-2      Dll   25 0.558 0.492  0.188  11
## 151           1    line-2      Dll   25 0.548 0.505  0.186  12
## 152           1    line-2      Dll   25 0.551 0.487  0.185  14
## 153           1    line-2      Dll   25 0.536 0.488  0.185  12
## 154           2    line-2      Dll   25 0.556 0.526  0.184  14
## 155           2    line-2      Dll   25 0.575 0.514  0.176  10
## 156           1    line-2      Dll   25 0.565 0.451  0.176  12
## 157           1    line-2      Dll   25 0.530 0.491  0.173  11
## 158           1    line-2      Dll   25 0.565 0.482  0.171  12
## 159           1    line-2      Dll   25 0.550 0.506  0.170  12
## 160           1    line-2      Dll   25 0.545 0.502  0.166  12
## 161           1   line-20      Dll   25 0.575 0.482  0.201  12
## 162           1   line-20      Dll   25 0.538 0.477  0.201  11
## 163           1   line-20      Dll   25 0.548 0.505  0.199  12
## 164           1   line-20      Dll   25 0.511 0.506  0.195  13
## 165           1   line-20      Dll   25 0.543 0.487  0.193  12
## 166           1   line-20      Dll   25 0.491 0.487  0.190  11
## 167           1   line-20      Dll   25 0.525 0.453  0.187  13
## 168           1   line-20      Dll   25 0.571 0.536  0.186  11
## 169           1   line-20      Dll   25 0.555 0.514  0.185  15
## 170           1   line-20      Dll   25 0.527 0.460  0.183  12
## 171           1   line-20      Dll   25 0.551 0.486  0.182  12
## 172           1   line-20      Dll   25 0.549 0.498  0.180  11
## 173           1   line-20      Dll   25 0.525 0.435  0.180  11
## 174           1   line-20      Dll   25 0.565 0.513  0.177  12
## 175           1   line-20      Dll   25 0.478 0.514  0.173  12
## 176           1   line-20      Dll   25 0.545 0.488  0.172  11
## 177           1   line-20      Dll   25 0.498 0.446  0.162  12
## 178           1   line-21      Dll   25 0.557 0.486  0.210  12
## 179           1   line-21      Dll   25 0.588 0.510  0.201   9
## 180           1   line-21      Dll   25 0.574 0.513  0.200  10
## 181           1   line-21      Dll   25 0.559 0.459  0.200  11
## 182           1   line-21      Dll   25 0.588 0.490  0.195  11
## 183           1   line-21      Dll   25 0.555 0.499  0.194  11
## 184           1   line-21      Dll   25 0.591 0.495  0.194  13
## 185           1   line-21      Dll   25 0.558 0.517  0.193  11
## 186           1   line-21      Dll   25 0.567 0.513  0.193  10
## 187           1   line-21      Dll   25 0.565 0.508  0.190  12
## 188           1   line-21      Dll   25 0.571 0.472  0.190  12
## 189           1   line-21      Dll   25 0.562 0.499  0.189  11
## 190           1   line-21      Dll   25 0.547 0.498  0.187  10
## 191           1   line-21      Dll   25 0.560 0.492  0.187  11
## 192           1   line-21      Dll   25 0.574 0.496  0.187  10
## 193           1   line-21      Dll   25 0.492 0.492  0.185  11
## 194           1   line-21      Dll   25 0.569 0.521  0.185  13
## 195           1   line-21      Dll   25 0.602 0.521  0.184  12
## 196           1   line-21      Dll   25 0.545 0.516  0.180  10
## 197           1   line-21      Dll   25 0.580 0.506  0.176  10
## 198           1   line-22      Dll   25 0.552 0.480  0.220  10
## 199           1   line-22      Dll   25 0.575 0.458  0.218  11
## 200           1   line-22      Dll   25 0.570 0.489  0.209  15
## 201           1   line-22      Dll   25 0.570 0.472  0.196  13
## 202           1   line-22      Dll   25 0.560 0.508  0.192  12
## 203           1   line-22      Dll   25 0.580 0.505  0.190  12
## 204           1   line-22      Dll   25 0.565 0.485  0.188  13
## 205           1   line-22      Dll   25 0.546 0.507  0.188  13
## 206           1   line-22      Dll   25 0.566 0.489  0.188  12
## 208           1   line-22      Dll   25 0.568 0.494  0.186  13
## 209           1   line-22      Dll   25 0.558 0.508  0.186  13
## 210           1   line-22      Dll   25 0.549 0.484  0.183  14
## 211           1   line-22      Dll   25 0.537 0.491  0.176  12
## 212           1   line-22      Dll   25 0.536 0.495  0.175  13
## 213           1   line-23      Dll   25 0.531 0.501  0.220  11
## 214           1   line-23      Dll   25 0.549 0.498  0.199  13
## 215           1   line-23      Dll   25 0.535 0.469  0.197  13
## 216           1   line-23      Dll   25 0.602 0.529  0.194  10
## 217           1   line-23      Dll   25 0.540 0.476  0.192  14
## 218           1   line-23      Dll   25 0.547 0.487  0.187  14
## 219           1   line-23      Dll   25 0.530 0.467  0.178   9
## 220           1   line-23      Dll   25 0.540 0.477  0.176  12
## 221           1   line-26      Dll   25 0.584 0.504  0.250  11
## 222           1   line-26      Dll   25 0.559 0.506  0.228  12
## 223           1   line-26      Dll   25 0.581 0.501  0.221  13
## 224           1   line-26      Dll   25 0.574 0.527  0.220   8
## 225           1   line-26      Dll   25 0.560 0.463  0.214  11
## 226           1   line-26      Dll   25 0.562 0.505  0.212  12
## 227           1   line-26      Dll   25 0.568 0.505  0.212  11
## 228           1   line-26      Dll   25 0.534 0.485  0.210  13
## 229           1   line-26      Dll   25 0.506 0.477  0.203  11
## 230           1   line-26      Dll   25 0.574 0.526  0.201  12
## 231           1   line-26      Dll   25 0.548 0.491  0.200  13
## 232           1   line-26      Dll   25 0.575 0.503  0.194   9
## 233           1   line-26      Dll   25 0.556 0.508  0.194  12
## 234           1   line-26      Dll   25 0.574 0.510  0.191  12
## 235           1   line-26      Dll   25 0.558 0.501  0.191  10
## 236           1   line-26      Dll   25 0.534 0.475  0.162  11
## 237           1   line-26      Dll   25 0.491 0.453  0.159  13
## 238           1   line-27      Dll   25 0.593 0.544  0.222  14
## 239           1   line-27      Dll   25 0.574 0.554  0.221  12
## 240           1   line-27      Dll   25 0.594 0.534  0.219  16
## 241           1   line-27      Dll   25 0.591 0.531  0.217  14
## 242           1   line-27      Dll   25 0.578 0.499  0.211  13
## 243           1   line-27      Dll   25 0.583 0.482  0.210  14
## 244           1   line-27      Dll   25 0.598 0.529  0.209  12
## 245           1   line-27      Dll   25 0.605 0.540  0.208  10
## 246           1   line-27      Dll   25 0.563 0.515  0.208  13
## 247           1   line-27      Dll   25 0.580 0.529  0.207  12
## 248           1   line-27      Dll   25 0.588 0.527  0.205  12
## 249           1   line-27      Dll   25 0.594 0.535  0.204  11
## 250           1   line-27      Dll   25 0.612 0.492  0.203  12
## 251           1   line-27      Dll   25 0.560 0.538  0.202  11
## 252           1   line-27      Dll   25 0.595 0.555  0.202  12
## 253           1   line-27      Dll   25 0.567 0.501  0.202  12
## 254           1   line-27      Dll   25 0.576 0.545  0.198  12
## 255           1   line-27      Dll   25 0.575 0.533  0.198  13
## 256           1   line-27      Dll   25 0.572 0.490  0.197  14
## 257           1   line-27      Dll   25 0.585 0.536  0.189  13
## 258           2    line-3      Dll   25 0.577 0.525  0.242  15
## 259           2    line-3      Dll   25 0.589 0.537  0.236  15
## 260           2    line-3      Dll   25 0.583 0.523  0.235  17
## 261           2    line-3      Dll   25 0.571 0.511  0.233  16
## 262           2    line-3      Dll   25 0.596 0.526  0.233  15
## 263           2    line-3      Dll   25 0.565 0.530  0.233  15
## 264           2    line-3      Dll   25 0.559 0.562  0.230  14
## 265           2    line-3      Dll   25 0.593 0.516  0.226  15
## 266           2    line-3      Dll   25 0.575 0.529  0.225  16
## 267           2    line-3      Dll   25 0.609 0.553  0.220  15
## 269           2    line-3      Dll   25 0.595 0.552  0.217  14
## 270           2    line-3      Dll   25 0.588 0.528  0.215  12
## 271           2    line-3      Dll   25 0.538 0.494  0.213  16
## 272           2    line-3      Dll   25 0.590 0.541  0.211  15
## 273           2    line-3      Dll   25 0.544 0.496  0.211  16
## 274           2    line-3      Dll   25 0.582 0.527  0.206  14
## 275           2    line-3      Dll   25 0.604 0.553  0.205  14
## 276           2    line-3      Dll   25 0.573 0.542  0.204  16
## 277           2    line-3      Dll   25 0.594 0.537  0.202  15
## 278           2    line-3      Dll   25 0.586 0.522  0.196  14
## 279           1    line-4      Dll   25 0.601 0.492  0.224  12
## 280           1    line-4      Dll   25 0.581 0.491  0.210  13
## 281           2    line-4      Dll   25 0.571 0.522  0.206  12
## 282           1    line-4      Dll   25 0.569 0.511  0.205  12
## 283           2    line-4      Dll   25 0.575 0.497  0.204  10
## 284           1    line-4      Dll   25 0.565 0.512  0.203   9
## 285           1    line-4      Dll   25 0.591 0.521  0.201  11
## 286           2    line-4      Dll   25 0.524 0.535  0.200  11
## 287           2    line-4      Dll   25 0.576 0.518  0.200  10
## 288           2    line-4      Dll   25 0.578 0.529  0.199  11
## 289           1    line-4      Dll   25 0.554 0.485  0.199  11
## 290           2    line-4      Dll   25 0.549 0.517  0.199  11
## 291           2    line-4      Dll   25 0.576 0.541  0.198  12
## 292           1    line-4      Dll   25 0.539 0.486  0.197  10
## 293           1    line-4      Dll   25 0.517 0.510  0.196  13
## 294           2    line-4      Dll   25 0.575 0.514  0.192  13
## 295           1    line-4      Dll   25 0.568 0.499  0.192  12
## 296           2    line-4      Dll   25 0.591 0.552  0.191  11
## 297           1    line-4      Dll   25 0.550 0.488  0.190  12
## 298           1    line-4      Dll   25 0.553 0.503  0.190  10
## 299           1    line-4      Dll   25 0.538 0.474  0.190  11
## 300           1    line-4      Dll   25 0.537 0.502  0.188  11
## 301           1    line-4      Dll   25 0.548 0.488  0.188  11
## 302           2    line-4      Dll   25 0.575 0.543  0.186  14
## 303           2    line-4      Dll   25 0.572 0.519  0.186  10
## 304           1    line-4      Dll   25 0.520 0.510  0.183  11
## 305           1    line-4      Dll   25 0.521 0.484  0.179  12
## 306           1    line-4      Dll   25 0.507 0.504  0.179  11
## 307           1    line-4      Dll   25 0.532 0.477  0.178  13
## 308           1    line-4      Dll   25 0.524 0.478  0.175  12
## 309           1    line-4      Dll   25 0.514 0.483  0.166  11
## 310           2    line-4      Dll   25 0.522 0.492  0.163  11
## 311           1    line-6      Dll   25 0.609 0.514  0.235  13
## 312           1    line-6      Dll   25 0.577 0.535  0.213  15
## 313           1    line-6      Dll   25 0.579 0.506  0.205  15
## 314           1    line-6      Dll   25 0.559 0.503  0.204  15
## 316           2    line-7      Dll   25 0.574 0.518  0.226  14
## 317           1    line-7      Dll   25 0.561 0.487  0.219  14
## 318           2    line-7      Dll   25 0.582 0.513  0.217  11
## 320           2    line-7      Dll   25 0.566 0.498  0.216  13
## 321           1    line-7      Dll   25 0.569 0.514  0.215  13
## 322           2    line-7      Dll   25 0.585 0.535  0.214  11
## 323           1    line-7      Dll   25 0.593 0.519  0.213  10
## 324           2    line-7      Dll   25 0.594 0.506  0.213  13
## 325           2    line-7      Dll   25 0.575 0.528  0.212  12
## 326           1    line-7      Dll   25 0.577 0.510  0.211  14
## 327           2    line-7      Dll   25 0.570 0.499  0.211  15
## 328           2    line-7      Dll   25 0.585 0.520  0.209  12
## 329           2    line-7      Dll   25 0.572 0.513  0.207  12
## 330           2    line-7      Dll   25 0.578 0.497  0.207  12
## 331           1    line-7      Dll   25 0.563 0.526  0.206  16
## 332           2    line-7      Dll   25 0.571 0.496  0.205  13
## 333           2    line-7      Dll   25 0.575 0.490  0.205  15
## 334           1    line-7      Dll   25 0.522 0.499  0.205  11
## 335           2    line-7      Dll   25 0.564 0.510  0.203  11
## 336           1    line-7      Dll   25 0.494 0.518  0.203  15
## 337           1    line-7      Dll   25 0.577 0.499  0.203  12
## 338           2    line-7      Dll   25 0.562 0.512  0.202  12
## 339           1    line-7      Dll   25 0.521 0.526  0.201  14
## 340           1    line-7      Dll   25 0.599 0.546  0.200  11
## 341           2    line-7      Dll   25 0.562 0.528  0.199  13
## 342           2    line-7      Dll   25 0.558 0.506  0.198  14
## 343           1    line-7      Dll   25 0.561 0.518  0.197  11
## 344           2    line-7      Dll   25 0.570 0.534  0.197  13
## 345           1    line-7      Dll   25 0.538 0.472  0.196  14
## 346           1    line-7      Dll   25 0.541 0.485  0.196  12
## 347           2    line-7      Dll   25 0.559 0.531  0.195  12
## 348           2    line-7      Dll   25 0.583 0.534  0.192  13
## 349           2    line-7      Dll   25 0.580 0.547  0.192  14
## 350           1    line-7      Dll   25 0.597 0.516  0.191  11
## 351           1    line-7      Dll   25 0.532 0.474  0.191  14
## 353           1    line-7      Dll   25 0.569 0.510  0.186  12
## 354           2    line-7      Dll   25 0.568 0.513  0.184  12
## 355           1    line-7      Dll   25 0.546 0.495  0.182  11
## 356           1    line-7      Dll   25 0.591 0.490  0.180  12
## 357           1    line-7      Dll   25 0.573 0.537  0.167  13
## 358           1    line-8      Dll   25 0.604 0.548  0.224  10
## 359           2    line-8      Dll   25 0.541 0.499  0.220   9
## 360           1    line-8      Dll   25 0.571 0.497  0.219   9
## 361           2    line-8      Dll   25 0.581 0.506  0.218  11
## 362           2    line-8      Dll   25 0.557 0.491  0.216  13
## 363           2    line-8      Dll   25 0.560 0.493  0.212   9
## 364           2    line-8      Dll   25 0.576 0.474  0.211  10
## 365           2    line-8      Dll   25 0.562 0.510  0.210  10
## 366           1    line-8      Dll   25 0.574 0.481  0.209  15
## 368           2    line-8      Dll   25 0.555 0.488  0.208  12
## 369           1    line-8      Dll   25 0.554 0.521  0.207   9
## 370           1    line-8      Dll   25 0.586 0.487  0.207  10
## 371           2    line-8      Dll   25 0.539 0.468  0.203  11
## 372           1    line-8      Dll   25 0.579 0.516  0.200  10
## 373           1    line-8      Dll   25 0.562 0.490  0.200  13
## 374           2    line-8      Dll   25 0.534 0.523  0.200  12
## 375           2    line-8      Dll   25 0.570 0.495  0.200  12
## 376           2    line-8      Dll   25 0.516 0.489  0.199   9
## 377           1    line-8      Dll   25 0.584 0.496  0.199  12
## 378           2    line-8      Dll   25 0.570 0.481  0.197  11
## 379           2    line-8      Dll   25 0.519 0.514  0.197  14
## 380           1    line-8      Dll   25 0.566 0.480  0.196  11
## 381           2    line-8      Dll   25 0.569 0.489  0.194   9
## 382           2    line-8      Dll   25 0.554 0.517  0.194  10
## 383           1    line-8      Dll   25 0.519 0.468  0.193  14
## 384           1    line-8      Dll   25 0.562 0.483  0.190  11
## 385           1    line-8      Dll   25 0.576 0.477  0.189  12
## 386           2    line-8      Dll   25 0.573 0.506  0.189  10
## 387           2    line-8      Dll   25 0.565 0.496  0.188  10
## 388           1    line-8      Dll   25 0.576 0.469  0.178  11
## 389           1    line-9      Dll   25 0.611 0.507  0.258  14
## 390           1    line-9      Dll   25 0.562 0.549  0.253  15
## 391           1    line-9      Dll   25 0.579 0.526  0.246  15
## 392           1    line-9      Dll   25 0.593 0.534  0.244  16
## 393           1    line-9      Dll   25 0.614 0.510  0.242  12
## 394           1    line-9      Dll   25 0.600 0.537  0.239  14
## 395           1    line-9      Dll   25 0.604 0.514  0.237  15
## 396           1    line-9      Dll   25 0.556 0.500  0.234  16
## 397           1    line-9      Dll   25 0.609 0.520  0.231  16
## 398           1    line-9      Dll   25 0.570 0.534  0.226  13
## 399           1    line-9      Dll   25 0.569 0.516  0.223  12
## 400           1    line-9      Dll   25 0.599 0.525  0.220  15
## 401           1    line-9      Dll   25 0.572 0.490  0.214  13
## 402           1    line-9      Dll   25 0.558 0.515  0.214  12
## 403           1    line-9      Dll   25 0.597 0.497  0.208  10
## 404           1    line-9      Dll   25 0.572 0.497  0.205  16
## 405           1 line-OreR      Dll   25 0.549 0.439  0.231  12
## 406           1 line-OreR      Dll   25 0.537 0.483  0.214   9
## 407           1 line-OreR      Dll   25 0.578 0.497  0.204  11
## 408           1 line-OreR      Dll   25 0.592 0.514  0.200  11
## 409           1 line-OreR      Dll   25 0.565 0.513  0.198   9
## 410           1 line-OreR      Dll   25 0.565 0.520  0.194  11
## 411           1 line-OreR      Dll   25 0.548 0.526  0.193  11
## 412           1 line-OreR      Dll   25 0.586 0.458  0.192  11
## 413           1 line-OreR      Dll   25 0.593 0.515  0.190  10
## 414           1 line-OreR      Dll   25 0.582 0.481  0.190  10
## 415           1 line-OreR      Dll   25 0.540 0.505  0.189   9
## 416           1 line-OreR      Dll   25 0.564 0.472  0.185  11
## 418           1    line-w      Dll   25 0.555 0.507  0.218  13
## 419           1    line-w      Dll   25 0.595 0.475  0.217  13
## 420           1    line-w      Dll   25 0.560 0.496  0.214  13
## 421           1    line-w      Dll   25 0.566 0.517  0.205  12
## 422           1    line-w      Dll   25 0.567 0.463  0.200  11
## 423           1    line-w      Dll   25 0.546 0.530  0.196  10
## 424           1    line-w      Dll   25 0.567 0.521  0.193  10
## 425           1    line-w      Dll   25 0.556 0.500  0.184  13
## 426           2    line-1       wt   25 0.582 0.514  0.219  13
## 427           1    line-1       wt   25 0.539 0.480  0.211  12
## 428           1    line-1       wt   25 0.560 0.508  0.211  11
## 429           2    line-1       wt   25 0.540 0.503  0.210  12
## 430           1    line-1       wt   25 0.593 0.474  0.209  12
## 431           2    line-1       wt   25 0.581 0.480  0.208  10
## 432           2    line-1       wt   25 0.538 0.478  0.207  10
## 433           1    line-1       wt   25 0.569 0.461  0.206  12
## 434           1    line-1       wt   25 0.538 0.510  0.205  12
## 435           1    line-1       wt   25 0.556 0.502  0.205  12
## 436           1    line-1       wt   25 0.584 0.488  0.203  11
## 437           2    line-1       wt   25 0.537 0.464  0.202  11
## 438           1    line-1       wt   25 0.597 0.483  0.200  11
## 439           2    line-1       wt   25 0.528 0.484  0.200  10
## 440           1    line-1       wt   25 0.579 0.472  0.199  11
## 441           2    line-1       wt   25 0.533 0.483  0.196  11
## 442           1    line-1       wt   25 0.565 0.474  0.195  11
## 444           2    line-1       wt   25 0.564 0.502  0.195  11
## 445           2    line-1       wt   25 0.530 0.463  0.195  10
## 446           2    line-1       wt   25 0.552 0.479  0.194  10
## 447           2    line-1       wt   25 0.553 0.465  0.193  12
## 448           2    line-1       wt   25 0.540 0.505  0.193  11
## 449           1    line-1       wt   25 0.560 0.490  0.192  12
## 450           1    line-1       wt   25 0.581 0.480  0.190  12
## 451           2    line-1       wt   25 0.541 0.478  0.189  11
## 452           2    line-1       wt   25 0.562 0.498  0.188   9
## 453           1    line-1       wt   25 0.557 0.484  0.187  11
## 456           2    line-1       wt   25 0.551 0.488  0.183  13
## 457           1    line-1       wt   25 0.560 0.526  0.181  11
## 458           1    line-1       wt   25 0.590 0.490  0.179  11
## 459           2    line-1       wt   25 0.566 0.507  0.179  11
## 460           2    line-1       wt   25 0.551 0.497  0.177  10
## 461           2    line-1       wt   25 0.547 0.494  0.177  10
## 462           2    line-1       wt   25 0.557 0.491  0.175  10
## 463           1    line-1       wt   25 0.569 0.458  0.175  10
## 464           2    line-1       wt   25 0.562 0.486  0.170   9
## 468           1   line-11       wt   25 0.555 0.464  0.229  12
## 469           1   line-11       wt   25 0.578 0.485  0.227  13
## 470           1   line-11       wt   25 0.561 0.495  0.216  14
## 471           1   line-11       wt   25 0.559 0.469  0.213  11
## 472           1   line-11       wt   25 0.609 0.524  0.209  14
## 473           1   line-11       wt   25 0.557 0.442  0.205  10
## 474           1   line-11       wt   25 0.536 0.470  0.203  12
## 475           1   line-11       wt   25 0.565 0.464  0.201  13
## 476           1   line-11       wt   25 0.524 0.472  0.200  12
## 477           1   line-11       wt   25 0.575 0.504  0.200  11
## 478           1   line-11       wt   25 0.560 0.505  0.199  12
## 479           1   line-11       wt   25 0.563 0.488  0.199  13
## 480           1   line-11       wt   25 0.547 0.462  0.199  12
## 481           1   line-11       wt   25 0.555 0.494  0.196  13
## 482           1   line-11       wt   25 0.562 0.479  0.194  14
## 483           1   line-11       wt   25 0.518 0.484  0.193  12
## 484           1   line-11       wt   25 0.530 0.475  0.190  13
## 485           1   line-11       wt   25 0.583 0.507  0.190  14
## 486           1   line-11       wt   25 0.570 0.482  0.189  14
## 487           1   line-11       wt   25 0.554 0.479  0.187  13
## 488           1   line-11       wt   25 0.555 0.494  0.186  11
## 489           1   line-11       wt   25 0.563 0.486  0.182  12
## 490           1   line-12       wt   25 0.576 0.459  0.222  10
## 491           1   line-12       wt   25 0.556 0.510  0.220  12
## 492           1   line-12       wt   25 0.553 0.483  0.219  11
## 493           1   line-12       wt   25 0.589 0.497  0.216  11
## 495           1   line-12       wt   25 0.565 0.475  0.211  13
## 496           1   line-12       wt   25 0.581 0.508  0.210  12
## 497           1   line-12       wt   25 0.568 0.503  0.208  12
## 498           1   line-12       wt   25 0.570 0.524  0.207  10
## 499           1   line-12       wt   25 0.537 0.503  0.207  10
## 500           1   line-12       wt   25 0.562 0.479  0.205  11
## 501           1   line-12       wt   25 0.577 0.496  0.205  11
## 502           1   line-12       wt   25 0.584 0.523  0.203  10
## 503           1   line-12       wt   25 0.546 0.485  0.203  13
## 504           1   line-12       wt   25 0.547 0.483  0.196  11
## 505           1   line-12       wt   25 0.518 0.489  0.196  12
## 506           1   line-12       wt   25 0.562 0.496  0.194  11
## 507           1   line-12       wt   25 0.555 0.498  0.194  12
## 508           1   line-12       wt   25 0.586 0.523  0.186  12
## 509           1   line-12       wt   25 0.532 0.475  0.182  11
## 510           1   line-12       wt   25 0.545 0.479  0.178  11
## 511           1   line-12       wt   25 0.560 0.472  0.177  12
## 512           1   line-13       wt   25 0.571 0.464  0.227  13
## 513           1   line-13       wt   25 0.542 0.445  0.216  12
## 514           1   line-13       wt   25 0.591 0.475  0.204  11
## 515           1   line-13       wt   25 0.568 0.482  0.204  12
## 516           1   line-13       wt   25 0.551 0.477  0.204  11
## 517           1   line-13       wt   25 0.530 0.419  0.203  12
## 518           1   line-13       wt   25 0.582 0.507  0.201  11
## 519           1   line-13       wt   25 0.567 0.462  0.200  11
## 520           1   line-13       wt   25 0.562 0.470  0.200  12
## 521           1   line-13       wt   25 0.556 0.474  0.199  12
## 522           1   line-13       wt   25 0.494 0.463  0.199  14
## 523           1   line-13       wt   25 0.565 0.472  0.198  11
## 524           1   line-13       wt   25 0.568 0.507  0.196  12
## 525           1   line-13       wt   25 0.555 0.487  0.196  12
## 526           1   line-13       wt   25 0.569 0.492  0.195  11
## 527           1   line-13       wt   25 0.584 0.494  0.194  12
## 528           1   line-13       wt   25 0.562 0.497  0.190  11
## 529           1   line-13       wt   25 0.548 0.457  0.187  10
## 530           1   line-13       wt   25 0.522 0.461  0.172  11
## 531           1   line-15       wt   25 0.588 0.507  0.211  13
## 532           1   line-15       wt   25 0.555 0.481  0.203   9
## 533           1   line-15       wt   25 0.583 0.496  0.203  12
## 534           1   line-15       wt   25 0.576 0.498  0.202  10
## 535           1   line-15       wt   25 0.553 0.514  0.200  12
## 536           1   line-15       wt   25 0.569 0.511  0.200  11
## 537           1   line-15       wt   25 0.586 0.487  0.200  11
## 538           1   line-15       wt   25 0.578 0.508  0.200  10
## 539           1   line-15       wt   25 0.561 0.481  0.196  11
## 540           1   line-15       wt   25 0.558 0.523  0.193  11
## 541           1   line-15       wt   25 0.547 0.467  0.190  12
## 542           1   line-15       wt   25 0.576 0.521  0.188  11
## 543           1   line-15       wt   25 0.564 0.491  0.187  11
## 544           1   line-15       wt   25 0.550 0.483  0.187  12
## 545           1   line-15       wt   25 0.562 0.525  0.183  12
## 546           1   line-15       wt   25 0.530 0.465  0.183  11
## 547           1   line-15       wt   25 0.536 0.473  0.176  10
## 548           1   line-15       wt   25 0.531 0.464  0.176  10
## 549           1   line-15       wt   25 0.553 0.483  0.169  10
## 550           1   line-15       wt   25 0.585 0.564  0.163   9
## 551           1   line-16       wt   25 0.539 0.476  0.219  10
## 552           1   line-16       wt   25 0.585 0.506  0.213  15
## 553           1   line-16       wt   25 0.565 0.461  0.213  11
## 554           1   line-16       wt   25 0.564 0.470  0.211  10
## 555           1   line-16       wt   25 0.540 0.476  0.210   9
## 556           1   line-16       wt   25 0.563 0.458  0.206  10
## 557           1   line-16       wt   25 0.541 0.464  0.205   9
## 558           1   line-16       wt   25 0.545 0.483  0.201   9
## 559           1   line-16       wt   25 0.548 0.466  0.196  10
## 560           1   line-16       wt   25 0.550 0.459  0.195  12
## 561           1   line-16       wt   25 0.598 0.509  0.194  14
## 562           1   line-16       wt   25 0.548 0.488  0.193  11
## 563           1   line-16       wt   25 0.541 0.464  0.191  10
## 564           1   line-16       wt   25 0.560 0.483  0.190   9
## 566           1   line-16       wt   25 0.520 0.453  0.189  10
## 567           1   line-16       wt   25 0.557 0.421  0.186  10
## 568           1   line-16       wt   25 0.577 0.502  0.186  10
## 569           1   line-16       wt   25 0.527 0.458  0.186  12
## 570           1   line-16       wt   25 0.516 0.441  0.178  11
## 571           1   line-17       wt   25 0.556 0.471  0.212  12
## 572           1   line-17       wt   25 0.552 0.437  0.211   9
## 573           1   line-17       wt   25 0.544 0.526  0.207  11
## 574           1   line-17       wt   25 0.571 0.499  0.205  10
## 575           1   line-17       wt   25 0.557 0.492  0.200  12
## 576           1   line-17       wt   25 0.548 0.481  0.198  11
## 577           1   line-17       wt   25 0.557 0.467  0.197  11
## 578           1   line-17       wt   25 0.546 0.491  0.197  10
## 579           1   line-17       wt   25 0.543 0.474  0.195  11
## 580           1   line-17       wt   25 0.559 0.456  0.193  11
## 581           1   line-17       wt   25 0.535 0.495  0.192  11
## 582           1   line-17       wt   25 0.565 0.502  0.190  10
## 583           1   line-17       wt   25 0.532 0.470  0.186   9
## 584           1   line-17       wt   25 0.532 0.483  0.186  10
## 585           1   line-17       wt   25 0.557 0.495  0.185  11
## 586           1   line-17       wt   25 0.554 0.482  0.183  12
## 587           1   line-17       wt   25 0.562 0.492  0.180  12
## 588           1   line-17       wt   25 0.584 0.488  0.180  11
## 589           1   line-17       wt   25 0.555 0.481  0.175  11
## 590           1   line-18       wt   25 0.556 0.503  0.198   9
## 591           1   line-18       wt   25 0.548 0.477  0.194  10
## 592           1   line-18       wt   25 0.547 0.462  0.194  10
## 593           1   line-18       wt   25 0.535 0.473  0.194   9
## 594           1   line-18       wt   25 0.527 0.455  0.185   9
## 595           1   line-18       wt   25 0.552 0.467  0.180  10
## 596           1   line-18       wt   25 0.517 0.456  0.180  11
## 597           1   line-18       wt   25 0.536 0.459  0.178  11
## 598           1   line-18       wt   25 0.551 0.461  0.176   9
## 599           1   line-18       wt   25 0.552 0.485  0.174  10
## 600           1   line-18       wt   25 0.542 0.471  0.174  10
## 601           1   line-18       wt   25 0.565 0.500  0.173  10
## 602           1   line-18       wt   25 0.526 0.480  0.162  12
## 603           1   line-18       wt   25 0.560 0.482  0.162  10
## 604           1   line-18       wt   25 0.453 0.408  0.161  11
## 605           1   line-18       wt   25 0.517 0.466  0.161  12
## 606           1   line-18       wt   25 0.529 0.479  0.158   9
## 607           1   line-18       wt   25 0.560 0.481  0.158  10
## 608           1   line-18       wt   25 0.471 0.421  0.157  11
## 609           1   line-18       wt   25 0.571 0.463  0.156  11
## 610           1   line-18       wt   25 0.488 0.403  0.146   9
## 611           1   line-18       wt   25 0.525 0.465  0.136  10
## 612           1   line-19       wt   25 0.565 0.457  0.210  11
## 613           1   line-19       wt   25 0.519 0.419  0.199  12
## 614           1   line-19       wt   25 0.560 0.505  0.198  12
## 615           1   line-19       wt   25 0.521 0.444  0.196  10
## 616           1   line-19       wt   25 0.570 0.454  0.196  12
## 617           1   line-19       wt   25 0.549 0.512  0.196  12
## 618           1   line-19       wt   25 0.540 0.483  0.192  10
## 619           1   line-19       wt   25 0.531 0.435  0.191  12
## 620           1   line-19       wt   25 0.530 0.426  0.190  10
## 621           1   line-19       wt   25 0.572 0.470  0.186  12
## 622           1   line-19       wt   25 0.525 0.447  0.186  10
## 623           1   line-19       wt   25 0.554 0.489  0.185  12
## 624           1   line-19       wt   25 0.499 0.453  0.185  10
## 625           1   line-19       wt   25 0.530 0.460  0.179  12
## 626           1   line-19       wt   25 0.536 0.493  0.174  12
## 627           1   line-19       wt   25 0.526 0.465  0.174  11
## 628           1   line-19       wt   25 0.526 0.473  0.169  10
## 630           1   line-19       wt   25 0.552 0.472  0.159  13
## 631           1   line-19       wt   25 0.518 0.449  0.157  10
## 632           1    line-2       wt   25 0.528 0.469  0.242  11
## 633           2    line-2       wt   25 0.564 0.463  0.225  12
## 634           1    line-2       wt   25 0.533 0.474  0.211  11
## 635           2    line-2       wt   25 0.558 0.493  0.210  11
## 636           1    line-2       wt   25 0.532 0.454  0.209  13
## 637           2    line-2       wt   25 0.507 0.451  0.206  12
## 639           2    line-2       wt   25 0.549 0.485  0.205  11
## 640           2    line-2       wt   25 0.544 0.448  0.205  10
## 641           2    line-2       wt   25 0.567 0.504  0.203  13
## 642           1    line-2       wt   25 0.505 0.437  0.202   9
## 643           2    line-2       wt   25 0.548 0.471  0.201  11
## 644           1    line-2       wt   25 0.534 0.473  0.200  13
## 645           2    line-2       wt   25 0.549 0.485  0.200  11
## 646           2    line-2       wt   25 0.567 0.472  0.199  13
## 647           2    line-2       wt   25 0.549 0.490  0.199  12
## 648           1    line-2       wt   25 0.540 0.464  0.197  11
## 649           1    line-2       wt   25 0.543 0.493  0.196  11
## 650           1    line-2       wt   25 0.554 0.480  0.195  13
## 651           1    line-2       wt   25 0.537 0.471  0.194  14
## 652           1    line-2       wt   25 0.525 0.503  0.194  12
## 653           2    line-2       wt   25 0.553 0.487  0.193  12
## 654           2    line-2       wt   25 0.562 0.506  0.192  11
## 655           2    line-2       wt   25 0.550 0.504  0.192  11
## 656           1    line-2       wt   25 0.553 0.437  0.192  13
## 657           1    line-2       wt   25 0.495 0.452  0.191  10
## 658           1    line-2       wt   25 0.555 0.453  0.190  11
## 659           1    line-2       wt   25 0.546 0.468  0.190  11
## 660           2    line-2       wt   25 0.550 0.468  0.189  10
## 661           2    line-2       wt   25 0.563 0.493  0.189  12
## 662           2    line-2       wt   25 0.575 0.489  0.188  13
## 663           2    line-2       wt   25 0.526 0.468  0.185  13
## 664           2    line-2       wt   25 0.525 0.468  0.185  11
## 665           2    line-2       wt   25 0.538 0.483  0.185  11
## 666           2    line-2       wt   25 0.550 0.468  0.183  11
## 667           1    line-2       wt   25 0.541 0.455  0.181  11
## 668           1    line-2       wt   25 0.514 0.432  0.175  13
## 669           1    line-2       wt   25 0.547 0.506  0.173  12
## 670           1    line-2       wt   25 0.503 0.448  0.170  13
## 671           1   line-20       wt   25 0.537 0.469  0.205  11
## 672           1   line-20       wt   25 0.521 0.478  0.200  11
## 673           1   line-20       wt   25 0.547 0.479  0.197  11
## 674           1   line-20       wt   25 0.564 0.513  0.196  12
## 675           1   line-20       wt   25 0.558 0.468  0.193  10
## 676           1   line-20       wt   25 0.559 0.489  0.186  11
## 677           1   line-20       wt   25 0.531 0.509  0.185  11
## 678           1   line-20       wt   25 0.550 0.480  0.185  11
## 679           1   line-20       wt   25 0.499 0.451  0.185  11
## 680           1   line-20       wt   25 0.553 0.412  0.182  12
## 681           1   line-20       wt   25 0.535 0.475  0.181  11
## 682           1   line-20       wt   25 0.568 0.520  0.180  13
## 683           1   line-20       wt   25 0.515 0.464  0.180  11
## 684           1   line-20       wt   25 0.520 0.481  0.179   9
## 685           1   line-20       wt   25 0.538 0.490  0.179  12
## 686           1   line-20       wt   25 0.534 0.462  0.177  12
## 687           1   line-20       wt   25 0.513 0.459  0.177  10
## 688           1   line-20       wt   25 0.512 0.413  0.171  12
## 689           1   line-20       wt   25 0.526 0.453  0.167  12
## 690           1   line-20       wt   25 0.556 0.433  0.166  10
## 691           1   line-21       wt   25 0.492 0.514  0.210  11
## 692           1   line-21       wt   25 0.570 0.482  0.203  11
## 693           1   line-21       wt   25 0.548 0.498  0.202  11
## 694           1   line-21       wt   25 0.539 0.464  0.201   9
## 695           1   line-21       wt   25 0.575 0.513  0.200   9
## 696           1   line-21       wt   25 0.566 0.514  0.197  11
## 697           1   line-21       wt   25 0.573 0.496  0.196  11
## 698           1   line-21       wt   25 0.559 0.499  0.196  11
## 699           1   line-21       wt   25 0.540 0.468  0.196  11
## 700           1   line-21       wt   25 0.496 0.463  0.195  11
## 701           1   line-21       wt   25 0.570 0.507  0.193  11
## 702           1   line-21       wt   25 0.555 0.472  0.190  11
## 703           1   line-21       wt   25 0.517 0.439  0.185  11
## 704           1   line-21       wt   25 0.523 0.431  0.185   9
## 705           1   line-21       wt   25 0.508 0.468  0.184  12
## 706           1   line-21       wt   25 0.557 0.479  0.183  10
## 707           1   line-21       wt   25 0.535 0.438  0.182  10
## 708           1   line-21       wt   25 0.562 0.491  0.181  10
## 709           1   line-21       wt   25 0.509 0.443  0.180  10
## 710           1   line-21       wt   25 0.537 0.460  0.179   9
## 711           1   line-22       wt   25 0.568 0.473  0.213  10
## 712           1   line-22       wt   25 0.549 0.464  0.209  11
## 713           1   line-22       wt   25 0.524 0.453  0.200  12
## 714           1   line-22       wt   25 0.530 0.472  0.199  13
## 715           1   line-22       wt   25 0.551 0.475  0.197  10
## 716           1   line-22       wt   25 0.531 0.454  0.196  11
## 717           1   line-22       wt   25 0.532 0.450  0.195  12
## 718           1   line-22       wt   25 0.540 0.448  0.194  12
## 719           1   line-22       wt   25 0.516 0.455  0.192   9
## 720           1   line-22       wt   25 0.559 0.487  0.186  12
## 721           1   line-22       wt   25 0.557 0.450  0.185  11
## 722           1   line-22       wt   25 0.532 0.480  0.183  12
## 723           1   line-22       wt   25 0.531 0.444  0.182  14
## 724           1   line-22       wt   25 0.542 0.468  0.182  12
## 725           1   line-22       wt   25 0.528 0.471  0.181  11
## 726           1   line-22       wt   25 0.560 0.497  0.180  10
## 727           1   line-22       wt   25 0.538 0.430  0.179  12
## 728           1   line-22       wt   25 0.511 0.443  0.177  12
## 729           1   line-22       wt   25 0.540 0.481  0.175  10
## 730           1   line-23       wt   25 0.539 0.459  0.209  10
## 731           1   line-23       wt   25 0.531 0.465  0.207  12
## 732           1   line-23       wt   25 0.522 0.443  0.206   9
## 733           1   line-23       wt   25 0.552 0.478  0.199  13
## 734           1   line-23       wt   25 0.535 0.483  0.199  11
## 735           1   line-23       wt   25 0.527 0.456  0.197  13
## 736           1   line-23       wt   25 0.551 0.472  0.196   9
## 737           1   line-23       wt   25 0.512 0.465  0.191  12
## 738           1   line-23       wt   25 0.500 0.445  0.189  11
## 739           1   line-23       wt   25 0.558 0.484  0.185   9
## 740           1   line-23       wt   25 0.513 0.457  0.185  12
## 741           1   line-23       wt   25 0.522 0.464  0.183   9
## 742           1   line-23       wt   25 0.539 0.483  0.182  11
## 743           1   line-23       wt   25 0.522 0.459  0.180  13
## 744           1   line-23       wt   25 0.521 0.451  0.178  11
## 745           1   line-23       wt   25 0.526 0.451  0.177  10
## 746           1   line-23       wt   25 0.515 0.430  0.174  11
## 747           1   line-23       wt   25 0.503 0.456  0.173  12
## 748           1   line-23       wt   25 0.514 0.488  0.172  12
## 749           1   line-23       wt   25 0.526 0.478  0.171  11
## 750           1   line-24       wt   25 0.545 0.480  0.219  11
## 751           1   line-24       wt   25 0.542 0.497  0.210  11
## 752           1   line-24       wt   25 0.533 0.492  0.208  11
## 753           1   line-24       wt   25 0.553 0.497  0.207   9
## 754           1   line-24       wt   25 0.466 0.471  0.205  11
## 755           1   line-24       wt   25 0.506 0.473  0.202  11
## 756           1   line-24       wt   25 0.547 0.496  0.200  12
## 757           1   line-24       wt   25 0.570 0.496  0.200  11
## 758           1   line-24       wt   25 0.560 0.491  0.200  11
## 759           1   line-24       wt   25 0.529 0.484  0.196  11
## 760           1   line-24       wt   25 0.542 0.493  0.194  10
## 761           1   line-24       wt   25 0.482 0.440  0.191  10
## 762           1   line-24       wt   25 0.535 0.482  0.190  11
## 763           1   line-24       wt   25 0.517 0.456  0.189  12
## 764           1   line-24       wt   25 0.516 0.461  0.187  11
## 765           1   line-24       wt   25 0.480 0.443  0.186   9
## 766           1   line-24       wt   25 0.494 0.455  0.185  11
## 767           1   line-24       wt   25 0.521 0.458  0.182  10
## 768           1   line-24       wt   25 0.503 0.453  0.181  10
## 769           1   line-24       wt   25 0.506 0.448  0.179  10
## 770           1   line-26       wt   25 0.569 0.487  0.226  12
## 771           1   line-26       wt   25 0.530 0.485  0.224  11
## 772           1   line-26       wt   25 0.546 0.424  0.216  11
## 773           1   line-26       wt   25 0.553 0.513  0.215  13
## 774           1   line-26       wt   25 0.512 0.472  0.209  11
## 775           1   line-26       wt   25 0.565 0.494  0.209  12
## 776           1   line-26       wt   25 0.554 0.489  0.209  11
## 777           1   line-26       wt   25 0.556 0.514  0.208  11
## 778           1   line-26       wt   25 0.547 0.485  0.205   9
## 779           1   line-26       wt   25 0.559 0.511  0.204  10
## 780           1   line-26       wt   25 0.548 0.511  0.201  10
## 781           1   line-26       wt   25 0.561 0.506  0.200  12
## 782           1   line-26       wt   25 0.505 0.453  0.199  11
## 783           1   line-26       wt   25 0.556 0.494  0.197  11
## 784           1   line-26       wt   25 0.554 0.495  0.197  12
## 785           1   line-26       wt   25 0.520 0.455  0.196  11
## 786           1   line-26       wt   25 0.561 0.492  0.196  10
## 787           1   line-26       wt   25 0.537 0.505  0.190   8
## 788           1   line-26       wt   25 0.537 0.486  0.185  11
## 789           1   line-26       wt   25 0.523 0.464  0.176  11
## 790           1   line-27       wt   25 0.568 0.493  0.226  10
## 791           1   line-27       wt   25 0.564 0.489  0.221  10
## 792           1   line-27       wt   25 0.566 0.516  0.217  10
## 793           1   line-27       wt   25 0.550 0.496  0.211  11
## 794           1   line-27       wt   25 0.537 0.485  0.210  10
## 795           1   line-27       wt   25 0.554 0.482  0.207  10
## 796           1   line-27       wt   25 0.570 0.497  0.205  10
## 797           1   line-27       wt   25 0.530 0.486  0.204   8
## 798           1   line-27       wt   25 0.537 0.493  0.203   9
## 799           1   line-27       wt   25 0.549 0.460  0.201   9
## 800           1   line-27       wt   25 0.564 0.499  0.200   8
## 801           1   line-27       wt   25 0.550 0.494  0.191  10
## 802           1   line-27       wt   25 0.585 0.503  0.191   9
## 803           1   line-27       wt   25 0.564 0.496  0.186  10
## 804           1   line-27       wt   25 0.546 0.483  0.185  10
## 805           1   line-27       wt   25 0.533 0.474  0.184  10
## 806           1   line-27       wt   25 0.537 0.466  0.183  10
## 807           1   line-27       wt   25 0.528 0.460  0.182  11
## 808           1   line-27       wt   25 0.549 0.474  0.182  10
## 809           1   line-27       wt   25 0.545 0.500  0.178   9
## 810           2    line-3       wt   25 0.574 0.473  0.226  11
## 811           2    line-3       wt   25 0.581 0.497  0.217  12
## 812           2    line-3       wt   25 0.580 0.495  0.215  12
## 813           2    line-3       wt   25 0.582 0.515  0.211  10
## 814           2    line-3       wt   25 0.582 0.528  0.211  11
## 815           2    line-3       wt   25 0.581 0.513  0.210  12
## 816           2    line-3       wt   25 0.564 0.486  0.210  12
## 817           2    line-3       wt   25 0.566 0.491  0.208  11
## 818           2    line-3       wt   25 0.592 0.504  0.206  12
## 819           2    line-3       wt   25 0.558 0.521  0.205   9
## 820           2    line-3       wt   25 0.561 0.476  0.205  11
## 821           2    line-3       wt   25 0.572 0.510  0.203  11
## 822           2    line-3       wt   25 0.565 0.522  0.200  12
## 823           2    line-3       wt   25 0.564 0.464  0.200  10
## 824           2    line-3       wt   25 0.567 0.510  0.200  13
## 825           2    line-3       wt   25 0.568 0.512  0.195  10
## 826           2    line-3       wt   25 0.579 0.526  0.192  10
## 827           2    line-3       wt   25 0.552 0.471  0.190  10
## 828           2    line-3       wt   25 0.572 0.531  0.190   9
## 829           2    line-3       wt   25 0.571 0.505  0.190  11
## 830           2    line-3       wt   25 0.564 0.497  0.188  12
## 831           2    line-3       wt   25 0.551 0.480  0.188  11
## 832           2    line-3       wt   25 0.554 0.477  0.182  13
## 833           2    line-3       wt   25 0.536 0.474  0.181  11
## 834           1    line-4       wt   25 0.574 0.497  0.226  12
## 835           1    line-4       wt   25 0.577 0.509  0.215  11
## 836           2    line-4       wt   25 0.575 0.505  0.215  12
## 837           2    line-4       wt   25 0.559 0.500  0.213  10
## 838           2    line-4       wt   25 0.572 0.502  0.210  11
## 839           1    line-4       wt   25 0.538 0.458  0.205  11
## 840           2    line-4       wt   25 0.581 0.530  0.205  11
## 841           2    line-4       wt   25 0.585 0.506  0.203  11
## 842           2    line-4       wt   25 0.564 0.489  0.202  10
## 843           2    line-4       wt   25 0.573 0.524  0.201  11
## 844           1    line-4       wt   25 0.545 0.462  0.200  11
## 845           1    line-4       wt   25 0.568 0.483  0.200  11
## 846           1    line-4       wt   25 0.538 0.467  0.199  11
## 847           2    line-4       wt   25 0.573 0.502  0.198  11
## 848           2    line-4       wt   25 0.576 0.520  0.196  11
## 849           2    line-4       wt   25 0.533 0.484  0.196  10
## 850           2    line-4       wt   25 0.573 0.500  0.195   9
## 851           2    line-4       wt   25 0.576 0.533  0.193  11
## 852           1    line-4       wt   25 0.544 0.460  0.193  12
## 853           2    line-4       wt   25 0.562 0.498  0.192  10
## 854           1    line-4       wt   25 0.517 0.440  0.192  11
## 855           2    line-4       wt   25 0.556 0.492  0.191  10
## 856           1    line-4       wt   25 0.535 0.448  0.190  11
## 857           1    line-4       wt   25 0.559 0.475  0.190  12
## 858           2    line-4       wt   25 0.541 0.478  0.189  11
## 859           1    line-4       wt   25 0.556 0.478  0.188  13
## 860           1    line-4       wt   25 0.545 0.472  0.185  13
## 861           1    line-4       wt   25 0.550 0.486  0.185  11
## 862           2    line-4       wt   25 0.521 0.455  0.184   9
## 863           2    line-4       wt   25 0.540 0.485  0.184  10
## 864           1    line-4       wt   25 0.536 0.474  0.182  12
## 865           1    line-4       wt   25 0.560 0.499  0.181   9
## 866           1    line-4       wt   25 0.484 0.451  0.181  11
## 867           2    line-4       wt   25 0.543 0.510  0.181  11
## 868           2    line-4       wt   25 0.548 0.468  0.180  10
## 869           2    line-4       wt   25 0.567 0.460  0.173  12
## 870           1    line-4       wt   25 0.521 0.482  0.172  12
## 871           1    line-4       wt   25 0.535 0.479  0.168  11
## 872           1    line-6       wt   25 0.571 0.496  0.211  13
## 873           1    line-6       wt   25 0.568 0.502  0.207  11
## 874           1    line-6       wt   25 0.557 0.474  0.205  12
## 875           1    line-6       wt   25 0.570 0.494  0.204  14
## 876           1    line-6       wt   25 0.540 0.467  0.203  12
## 877           1    line-6       wt   25 0.551 0.479  0.201   9
## 878           1    line-6       wt   25 0.582 0.508  0.201  12
## 879           1    line-6       wt   25 0.580 0.496  0.200  12
## 880           1    line-6       wt   25 0.564 0.487  0.198  12
## 881           1    line-6       wt   25 0.552 0.474  0.197  11
## 882           1    line-6       wt   25 0.558 0.492  0.195  13
## 883           1    line-6       wt   25 0.578 0.483  0.192   9
## 884           1    line-6       wt   25 0.560 0.495  0.185  13
## 885           1    line-6       wt   25 0.506 0.457  0.183  12
## 886           1    line-6       wt   25 0.493 0.456  0.181  13
## 887           1    line-6       wt   25 0.540 0.459  0.178  12
## 888           1    line-6       wt   25 0.491 0.422  0.172  12
## 892           1    line-7       wt   25 0.555 0.509  0.235  10
## 893           1    line-7       wt   25 0.571 0.507  0.220  11
## 894           1    line-7       wt   25 0.516 0.429  0.214  12
## 895           2    line-7       wt   25 0.567 0.507  0.208  11
## 896           2    line-7       wt   25 0.550 0.483  0.208  11
## 897           2    line-7       wt   25 0.550 0.497  0.208   9
## 898           2    line-7       wt   25 0.557 0.456  0.207  11
## 899           2    line-7       wt   25 0.527 0.461  0.206  10
## 900           2    line-7       wt   25 0.558 0.511  0.205  12
## 901           2    line-7       wt   25 0.566 0.479  0.203  10
## 902           2    line-7       wt   25 0.573 0.501  0.203  12
## 903           1    line-7       wt   25 0.558 0.487  0.203   9
## 904           2    line-7       wt   25 0.552 0.491  0.202  12
## 905           2    line-7       wt   25 0.568 0.498  0.201  11
## 906           1    line-7       wt   25 0.600 0.485  0.200  11
## 907           1    line-7       wt   25 0.574 0.496  0.199  11
## 908           2    line-7       wt   25 0.569 0.503  0.199   9
## 909           2    line-7       wt   25 0.533 0.474  0.199  10
## 910           1    line-7       wt   25 0.569 0.500  0.197   9
## 911           1    line-7       wt   25 0.590 0.528  0.196  14
## 912           1    line-7       wt   25 0.546 0.462  0.196  10
## 913           2    line-7       wt   25 0.553 0.506  0.196  11
## 914           1    line-7       wt   25 0.536 0.460  0.195  11
## 915           1    line-7       wt   25 0.542 0.478  0.195  11
## 916           2    line-7       wt   25 0.563 0.502  0.194  11
## 917           1    line-7       wt   25 0.527 0.463  0.193  10
## 918           2    line-7       wt   25 0.558 0.492  0.193  11
## 919           2    line-7       wt   25 0.526 0.493  0.192  11
## 920           2    line-7       wt   25 0.532 0.475  0.191  11
## 921           1    line-7       wt   25 0.536 0.491  0.190  11
## 922           2    line-7       wt   25 0.570 0.519  0.190  12
## 923           1    line-7       wt   25 0.577 0.502  0.189  12
## 924           1    line-7       wt   25 0.586 0.496  0.187  10
## 925           2    line-7       wt   25 0.560 0.491  0.187  13
## 926           2    line-7       wt   25 0.559 0.476  0.186  11
## 927           1    line-7       wt   25 0.538 0.474  0.186  12
## 928           2    line-7       wt   25 0.487 0.408  0.185   9
## 929           2    line-7       wt   25 0.577 0.523  0.185  11
## 930           1    line-7       wt   25 0.545 0.491  0.184  12
## 931           1    line-7       wt   25 0.575 0.486  0.182  10
## 932           2    line-7       wt   25 0.528 0.453  0.180  10
## 933           2    line-7       wt   25 0.548 0.475  0.180  11
## 934           2    line-7       wt   25 0.494 0.431  0.173  10
## 935           1    line-7       wt   25 0.588 0.512  0.171  10
## 936           1    line-8       wt   25 0.591 0.532  0.225   9
## 937           1    line-8       wt   25 0.558 0.509  0.224   9
## 938           2    line-8       wt   25 0.569 0.515  0.220  10
## 939           1    line-8       wt   25 0.538 0.524  0.217  12
## 940           2    line-8       wt   25 0.578 0.509  0.217  11
## 941           1    line-8       wt   25 0.572 0.532  0.216  12
## 942           1    line-8       wt   25 0.583 0.534  0.215  13
## 943           2    line-8       wt   25 0.555 0.504  0.215  13
## 944           2    line-8       wt   25 0.570 0.502  0.215  11
## 945           2    line-8       wt   25 0.544 0.450  0.214   9
## 946           1    line-8       wt   25 0.575 0.498  0.213   9
## 947           2    line-8       wt   25 0.590 0.510  0.213  11
## 948           1    line-8       wt   25 0.580 0.537  0.213  11
## 949           1    line-8       wt   25 0.565 0.515  0.211  12
## 950           1    line-8       wt   25 0.568 0.504  0.211   9
## 951           2    line-8       wt   25 0.593 0.498  0.209  10
## 952           1    line-8       wt   25 0.544 0.501  0.207  12
## 953           2    line-8       wt   25 0.570 0.512  0.206  12
## 954           1    line-8       wt   25 0.580 0.519  0.205  11
## 955           2    line-8       wt   25 0.575 0.478  0.203  12
## 956           1    line-8       wt   25 0.574 0.527  0.203  10
## 957           2    line-8       wt   25 0.527 0.499  0.202  11
## 958           2    line-8       wt   25 0.577 0.509  0.201  11
## 959           1    line-8       wt   25 0.554 0.477  0.201   9
## 960           1    line-8       wt   25 0.547 0.509  0.200  12
## 961           2    line-8       wt   25 0.529 0.473  0.200  11
## 962           2    line-8       wt   25 0.569 0.480  0.199  10
## 963           2    line-8       wt   25 0.574 0.499  0.197  11
## 964           2    line-8       wt   25 0.539 0.504  0.197  13
## 965           2    line-8       wt   25 0.567 0.476  0.197  11
## 966           2    line-8       wt   25 0.556 0.496  0.196  12
## 967           2    line-8       wt   25 0.569 0.481  0.196   9
## 968           2    line-8       wt   25 0.570 0.506  0.195   9
## 969           2    line-8       wt   25 0.542 0.467  0.192  10
## 970           1    line-8       wt   25 0.603 0.532  0.192  10
## 971           1    line-8       wt   25 0.540 0.485  0.189  10
## 972           1    line-8       wt   25 0.566 0.516  0.188  10
## 973           2    line-8       wt   25 0.558 0.489  0.187  11
## 974           1    line-8       wt   25 0.580 0.520  0.187  10
## 975           1    line-8       wt   25 0.519 0.507  0.186   9
## 976           2    line-8       wt   25 0.566 0.498  0.185   9
## 977           2    line-8       wt   25 0.538 0.473  0.184  10
## 978           1    line-8       wt   25 0.522 0.472  0.174  12
## 980           1    line-9       wt   25 0.573 0.505  0.240   9
## 981           1    line-9       wt   25 0.593 0.503  0.238  11
## 982           1    line-9       wt   25 0.576 0.529  0.234  10
## 983           1    line-9       wt   25 0.575 0.497  0.228  11
## 984           1    line-9       wt   25 0.591 0.520  0.225  11
## 985           1    line-9       wt   25 0.591 0.494  0.219   8
## 986           1    line-9       wt   25 0.593 0.494  0.217  10
## 987           1    line-9       wt   25 0.580 0.523  0.213  11
## 988           1    line-9       wt   25 0.572 0.501  0.210  10
## 989           1    line-9       wt   25 0.604 0.511  0.210  11
## 990           1    line-9       wt   25 0.581 0.499  0.210  11
## 991           1    line-9       wt   25 0.552 0.478  0.209  11
## 992           1    line-9       wt   25 0.572 0.497  0.206  11
## 993           1    line-9       wt   25 0.563 0.508  0.199  10
## 994           1    line-9       wt   25 0.570 0.507  0.190  10
## 995           1 line-OreR       wt   25 0.570 0.505  0.207  10
## 996           1 line-OreR       wt   25 0.591 0.500  0.207  11
## 997           1 line-OreR       wt   25 0.562 0.465  0.205  10
## 998           1 line-OreR       wt   25 0.585 0.468  0.204  11
## 999           1 line-OreR       wt   25 0.610 0.455  0.200  10
## 1000          1 line-OreR       wt   25 0.582 0.476  0.200   9
## 1001          1 line-OreR       wt   25 0.588 0.482  0.198   9
## 1002          1 line-OreR       wt   25 0.596 0.451  0.197  11
## 1003          1 line-OreR       wt   25 0.583 0.493  0.195   9
## 1004          1 line-OreR       wt   25 0.558 0.483  0.193  10
## 1005          1 line-OreR       wt   25 0.618 0.511  0.193  10
## 1006          1 line-OreR       wt   25 0.583 0.490  0.190  10
## 1007          1 line-OreR       wt   25 0.561 0.489  0.190  12
## 1008          1 line-OreR       wt   25 0.581 0.496  0.189   9
## 1009          1 line-OreR       wt   25 0.566 0.492  0.189  11
## 1010          1 line-OreR       wt   25 0.590 0.520  0.187  11
## 1011          1 line-OreR       wt   25 0.560 0.478  0.185  10
## 1012          1 line-OreR       wt   25 0.571 0.506  0.181   9
## 1013          1 line-OreR       wt   25 0.573 0.509  0.178  11
## 1014          1 line-OreR       wt   25 0.575 0.485  0.169   9
## 1015          1    line-w       wt   25 0.550 0.486  0.220  11
## 1016          1    line-w       wt   25 0.561 0.508  0.220  10
## 1017          1    line-w       wt   25 0.576 0.505  0.217  13
## 1018          1    line-w       wt   25 0.540 0.498  0.210  11
## 1019          1    line-w       wt   25 0.545 0.495  0.209  10
## 1020          1    line-w       wt   25 0.557 0.489  0.205  13
## 1021          1    line-w       wt   25 0.563 0.484  0.204  12
## 1022          1    line-w       wt   25 0.566 0.503  0.200  11
## 1023          1    line-w       wt   25 0.544 0.499  0.198  13
## 1024          1    line-w       wt   25 0.562 0.491  0.197  13
## 1025          1    line-w       wt   25 0.547 0.484  0.197  11
## 1026          1    line-w       wt   25 0.535 0.487  0.197  11
## 1028          1    line-w       wt   25 0.543 0.485  0.196  11
## 1029          1    line-w       wt   25 0.571 0.499  0.194  13
## 1030          1    line-w       wt   25 0.571 0.473  0.194  12
## 1031          1    line-w       wt   25 0.531 0.470  0.187  10
## 1032          1    line-w       wt   25 0.537 0.493  0.187  11
## 1033          1    line-w       wt   25 0.573 0.512  0.187  12
## 1034          1    line-w       wt   25 0.567 0.517  0.184  11
## 1035          1    line-1      Dll   30 0.520 0.434  0.184  11
## 1036          1    line-1      Dll   30 0.589 0.472  0.178   9
## 1037          1    line-1      Dll   30 0.525 0.411  0.175  10
## 1038          1    line-1      Dll   30 0.579 0.460  0.175  11
## 1039          1    line-1      Dll   30 0.547 0.443  0.168   9
## 1040          1    line-1      Dll   30 0.563 0.438  0.162  10
## 1041          1    line-1      Dll   30 0.507 0.415  0.141   8
## 1042          1    line-1      Dll   30 0.532 0.463  0.134  11
## 1044          1   line-11      Dll   30 0.561 0.502  0.207  11
## 1045          1   line-11      Dll   30 0.606 0.533  0.205  12
## 1046          2   line-11      Dll   30 0.540 0.508  0.205  11
## 1047          2   line-11      Dll   30 0.590 0.490  0.201  10
## 1048          2   line-11      Dll   30 0.563 0.477  0.197  11
## 1049          1   line-11      Dll   30 0.577 0.513  0.195  12
## 1050          2   line-11      Dll   30 0.570 0.514  0.192  13
## 1051          2   line-11      Dll   30 0.564 0.476  0.192   9
## 1052          1   line-11      Dll   30 0.570 0.530  0.190  10
## 1053          2   line-11      Dll   30 0.576 0.505  0.190  11
## 1054          2   line-11      Dll   30 0.581 0.503  0.185  14
## 1055          2   line-11      Dll   30 0.577 0.485  0.185  12
## 1056          2   line-11      Dll   30 0.583 0.541  0.184  11
## 1057          2   line-11      Dll   30 0.524 0.495  0.184  11
## 1058          1   line-11      Dll   30 0.585 0.539  0.183  10
## 1059          2   line-11      Dll   30 0.543 0.459  0.182  15
## 1060          2   line-11      Dll   30 0.564 0.497  0.181  11
## 1061          1   line-11      Dll   30 0.569 0.501  0.181  12
## 1062          2   line-11      Dll   30 0.554 0.479  0.181  13
## 1063          2   line-11      Dll   30 0.530 0.489  0.180  11
## 1064          1   line-11      Dll   30 0.591 0.519  0.180  12
## 1065          2   line-11      Dll   30 0.557 0.518  0.178  13
## 1066          1   line-11      Dll   30 0.560 0.502  0.178  12
## 1067          1   line-11      Dll   30 0.571 0.505  0.178  12
## 1068          2   line-11      Dll   30 0.565 0.468  0.177  11
## 1070          1   line-11      Dll   30 0.568 0.487  0.175  12
## 1071          2   line-11      Dll   30 0.508 0.465  0.175  11
## 1072          2   line-11      Dll   30 0.551 0.484  0.172  12
## 1074          1   line-12      Dll   30 0.543 0.522  0.203  12
## 1075          1   line-12      Dll   30 0.570 0.530  0.200   9
## 1076          1   line-12      Dll   30 0.548 0.510  0.190  12
## 1077          1   line-12      Dll   30 0.539 0.514  0.189   9
## 1078          1   line-12      Dll   30 0.583 0.498  0.188   9
## 1079          1   line-12      Dll   30 0.562 0.520  0.188  11
## 1080          1   line-12      Dll   30 0.552 0.472  0.186   8
## 1081          1   line-12      Dll   30 0.555 0.508  0.185  11
## 1082          1   line-12      Dll   30 0.586 0.530  0.185   9
## 1083          1   line-12      Dll   30 0.552 0.508  0.183   9
## 1084          1   line-12      Dll   30 0.555 0.497  0.176  10
## 1085          1   line-12      Dll   30 0.558 0.503  0.176  10
## 1086          1   line-12      Dll   30 0.578 0.516  0.174  10
## 1087          1   line-12      Dll   30 0.543 0.479  0.172  11
## 1088          1   line-12      Dll   30 0.492 0.422  0.166  10
## 1089          1   line-12      Dll   30 0.497 0.442  0.165  11
## 1090          1   line-12      Dll   30 0.535 0.512  0.163   9
## 1091          1   line-12      Dll   30 0.534 0.490  0.155   9
## 1092          1   line-12      Dll   30 0.541 0.446  0.132  10
## 1093          1   line-12      Dll   30 0.551 0.484  0.130   9
## 1094          2   line-13      Dll   30 0.536 0.441  0.193  12
## 1095          2   line-13      Dll   30 0.563 0.507  0.192  15
## 1096          2   line-13      Dll   30 0.554 0.505  0.178  10
## 1097          2   line-13      Dll   30 0.517 0.477  0.174  10
## 1098          2   line-13      Dll   30 0.548 0.511  0.173  12
## 1099          2   line-13      Dll   30 0.589 0.535  0.169  18
## 1100          1   line-15      Dll   30 0.551 0.487  0.203  10
## 1101          1   line-15      Dll   30 0.546 0.507  0.200  14
## 1102          1   line-15      Dll   30 0.559 0.507  0.192  13
## 1103          1   line-15      Dll   30 0.567 0.562  0.190  15
## 1104          1   line-15      Dll   30 0.542 0.493  0.186  10
## 1105          1   line-15      Dll   30 0.547 0.477  0.185  11
## 1106          1   line-15      Dll   30 0.536 0.479  0.184   9
## 1107          1   line-15      Dll   30 0.526 0.494  0.184  12
## 1108          1   line-15      Dll   30 0.554 0.500  0.181  11
## 1109          1   line-15      Dll   30 0.556 0.497  0.175  10
## 1110          1   line-15      Dll   30 0.549 0.492  0.174  12
## 1111          1   line-15      Dll   30 0.554 0.532  0.172  10
## 1112          1   line-15      Dll   30 0.545 0.493  0.165  10
## 1113          1   line-15      Dll   30 0.515 0.496  0.162   8
## 1114          1   line-16      Dll   30 0.556 0.496  0.206  10
## 1115          1   line-16      Dll   30 0.522 0.458  0.200  11
## 1116          1   line-16      Dll   30 0.551 0.486  0.193  13
## 1117          1   line-16      Dll   30 0.554 0.482  0.191  10
## 1118          1   line-16      Dll   30 0.560 0.476  0.190  10
## 1119          1   line-16      Dll   30 0.549 0.470  0.189  10
## 1120          1   line-16      Dll   30 0.542 0.467  0.188  10
## 1121          1   line-16      Dll   30 0.538 0.452  0.187   8
## 1122          1   line-16      Dll   30 0.529 0.471  0.181  12
## 1123          1   line-16      Dll   30 0.531 0.426  0.180  13
## 1124          1   line-16      Dll   30 0.535 0.485  0.180  11
## 1125          1   line-16      Dll   30 0.554 0.492  0.179  10
## 1126          1   line-16      Dll   30 0.540 0.443  0.179  13
## 1127          1   line-16      Dll   30 0.521 0.449  0.178  11
## 1128          1   line-16      Dll   30 0.519 0.462  0.177  14
## 1129          1   line-16      Dll   30 0.543 0.460  0.171  15
## 1130          1   line-16      Dll   30 0.501 0.467  0.157  14
## 1131          2   line-17      Dll   30 0.541 0.485  0.190  10
## 1132          2   line-17      Dll   30 0.548 0.489  0.181  10
## 1133          2   line-17      Dll   30 0.511 0.460  0.179  10
## 1134          2   line-17      Dll   30 0.492 0.472  0.179   9
## 1135          2   line-17      Dll   30 0.514 0.473  0.179  10
## 1136          2   line-17      Dll   30 0.529 0.479  0.175  10
## 1137          2   line-17      Dll   30 0.514 0.465  0.174  11
## 1138          2   line-17      Dll   30 0.536 0.479  0.173  12
## 1139          2   line-17      Dll   30 0.539 0.475  0.173  11
## 1140          2   line-17      Dll   30 0.494 0.442  0.172   9
## 1141          2   line-17      Dll   30 0.510 0.434  0.171   8
## 1142          2   line-17      Dll   30 0.539 0.466  0.170  10
## 1143          2   line-17      Dll   30 0.537 0.453  0.170  11
## 1144          2   line-17      Dll   30 0.554 0.501  0.163  10
## 1145          2   line-17      Dll   30 0.550 0.497  0.160   9
## 1146          2   line-17      Dll   30 0.531 0.495  0.159  11
## 1147          2   line-17      Dll   30 0.525 0.466  0.157  10
## 1148          2   line-17      Dll   30 0.472 0.436  0.154   9
## 1149          2   line-17      Dll   30 0.499 0.462  0.153  10
## 1150          2   line-17      Dll   30 0.509 0.466  0.149  10
## 1151          1   line-18      Dll   30 0.537 0.460  0.201  11
## 1152          1   line-18      Dll   30 0.551 0.483  0.191  10
## 1153          2   line-18      Dll   30 0.529 0.464  0.185  11
## 1154          1   line-18      Dll   30 0.527 0.479  0.184  10
## 1155          2   line-18      Dll   30 0.549 0.486  0.179   8
## 1156          1   line-18      Dll   30 0.503 0.456  0.179   9
## 1157          2   line-18      Dll   30 0.524 0.475  0.176   9
## 1158          2   line-18      Dll   30 0.517 0.470  0.176  12
## 1159          1   line-18      Dll   30 0.531 0.452  0.176  11
## 1160          2   line-18      Dll   30 0.512 0.474  0.175  12
## 1161          2   line-18      Dll   30 0.542 0.480  0.175   9
## 1162          2   line-18      Dll   30 0.531 0.492  0.174  10
## 1163          2   line-18      Dll   30 0.520 0.453  0.173  10
## 1164          1   line-18      Dll   30 0.515 0.456  0.173  11
## 1165          2   line-18      Dll   30 0.512 0.463  0.172   8
## 1166          1   line-18      Dll   30 0.535 0.478  0.172   9
## 1167          1   line-18      Dll   30 0.501 0.467  0.171  10
## 1168          2   line-18      Dll   30 0.496 0.446  0.171  11
## 1169          1   line-18      Dll   30 0.543 0.480  0.171   9
## 1170          1   line-18      Dll   30 0.518 0.452  0.170  10
## 1171          2   line-18      Dll   30 0.525 0.476  0.169  11
## 1172          1   line-18      Dll   30 0.503 0.427  0.169   9
## 1173          2   line-18      Dll   30 0.523 0.478  0.169  10
## 1174          2   line-18      Dll   30 0.518 0.470  0.168   8
## 1175          1   line-18      Dll   30 0.505 0.442  0.168  11
## 1176          1   line-18      Dll   30 0.529 0.458  0.168  10
## 1177          1   line-18      Dll   30 0.502 0.470  0.167  11
## 1178          1   line-18      Dll   30 0.496 0.440  0.165   8
## 1179          1   line-18      Dll   30 0.527 0.435  0.164   7
## 1180          2   line-18      Dll   30 0.528 0.475  0.163  11
## 1181          1   line-18      Dll   30 0.523 0.433  0.163  10
## 1182          1   line-18      Dll   30 0.513 0.458  0.161   8
## 1183          1   line-18      Dll   30 0.504 0.429  0.159   8
## 1184          2   line-18      Dll   30 0.506 0.450  0.158  11
## 1185          2   line-18      Dll   30 0.535 0.445  0.152   8
## 1186          2   line-18      Dll   30 0.478 0.425  0.144   9
## 1187          2   line-18      Dll   30 0.534 0.472  0.144  11
## 1188          2   line-18      Dll   30 0.531 0.391  0.106  10
## 1189          1   line-19      Dll   30 0.525 0.479  0.180  11
## 1190          1   line-19      Dll   30 0.534 0.470  0.179   9
## 1191          1   line-19      Dll   30 0.519 0.463  0.176  10
## 1192          1   line-19      Dll   30 0.527 0.452  0.174  11
## 1193          1   line-19      Dll   30 0.535 0.461  0.173  11
## 1194          1   line-19      Dll   30 0.520 0.481  0.168  10
## 1195          1   line-19      Dll   30 0.536 0.448  0.167  10
## 1196          1   line-19      Dll   30 0.551 0.465  0.166  14
## 1197          1   line-19      Dll   30 0.539 0.461  0.165  11
## 1198          1   line-19      Dll   30 0.531 0.440  0.164  10
## 1199          1   line-19      Dll   30 0.535 0.453  0.161  10
## 1200          1   line-19      Dll   30 0.533 0.450  0.161  12
## 1201          1   line-19      Dll   30 0.520 0.465  0.160  11
## 1202          1   line-19      Dll   30 0.545 0.457  0.159  13
## 1203          1   line-19      Dll   30 0.537 0.455  0.159  10
## 1204          1   line-19      Dll   30 0.530 0.438  0.158  10
## 1205          1   line-19      Dll   30 0.530 0.440  0.158  10
## 1206          1   line-19      Dll   30 0.513 0.422  0.151  10
## 1207          1   line-19      Dll   30 0.511 0.446  0.149  11
## 1208          1   line-19      Dll   30 0.494 0.422  0.149  10
## 1210          1    line-2      Dll   30 0.508 0.476  0.193  10
## 1211          1    line-2      Dll   30 0.528 0.469  0.193  14
## 1212          1    line-2      Dll   30 0.527 0.471  0.187  11
## 1213          1    line-2      Dll   30 0.536 0.490  0.186  12
## 1214          1    line-2      Dll   30 0.535 0.461  0.185  12
## 1215          1    line-2      Dll   30 0.520 0.486  0.185  11
## 1216          1    line-2      Dll   30 0.537 0.435  0.185  11
## 1217          1    line-2      Dll   30 0.519 0.466  0.185  11
## 1218          1    line-2      Dll   30 0.528 0.485  0.181  13
## 1219          1    line-2      Dll   30 0.511 0.459  0.181  11
## 1220          1    line-2      Dll   30 0.522 0.465  0.180  10
## 1221          1    line-2      Dll   30 0.509 0.455  0.179  11
## 1222          1    line-2      Dll   30 0.544 0.464  0.178  10
## 1223          1    line-2      Dll   30 0.525 0.445  0.173  10
## 1224          1    line-2      Dll   30 0.531 0.430  0.173  13
## 1225          1    line-2      Dll   30 0.538 0.460  0.173  11
## 1226          1    line-2      Dll   30 0.537 0.479  0.172  11
## 1227          1    line-2      Dll   30 0.524 0.437  0.169  11
## 1228          1    line-2      Dll   30 0.490 0.423  0.166   8
## 1229          1    line-2      Dll   30 0.518 0.472  0.157  12
## 1230          1   line-20      Dll   30 0.500 0.451  0.183  10
## 1231          1   line-20      Dll   30 0.535 0.483  0.182  10
## 1232          1   line-20      Dll   30 0.527 0.487  0.179  10
## 1233          1   line-20      Dll   30 0.482 0.440  0.177  11
## 1234          1   line-20      Dll   30 0.477 0.450  0.174  12
## 1235          1   line-20      Dll   30 0.524 0.470  0.171  10
## 1236          1   line-20      Dll   30 0.502 0.425  0.170  11
## 1237          1   line-20      Dll   30 0.485 0.444  0.165  14
## 1238          1   line-20      Dll   30 0.506 0.480  0.163  17
## 1239          1   line-20      Dll   30 0.488 0.449  0.162  11
## 1240          1   line-20      Dll   30 0.489 0.454  0.161   9
## 1241          1   line-20      Dll   30 0.490 0.452  0.160   9
## 1242          1   line-20      Dll   30 0.494 0.454  0.159  10
## 1243          1   line-20      Dll   30 0.477 0.449  0.157  10
## 1244          1   line-20      Dll   30 0.489 0.458  0.153   8
## 1245          1   line-20      Dll   30 0.477 0.452  0.151  10
## 1246          1   line-20      Dll   30 0.493 0.447  0.148   9
## 1247          1   line-20      Dll   30 0.502 0.464  0.143  11
## 1248          1   line-20      Dll   30 0.478 0.432  0.142   9
## 1249          1   line-20      Dll   30 0.475 0.422  0.142  10
## 1250          1   line-20      Dll   30 0.478 0.448  0.141  10
## 1251          1   line-20      Dll   30 0.476 0.431  0.141   8
## 1252          1   line-21      Dll   30 0.546 0.477  0.197  10
## 1253          1   line-21      Dll   30 0.539 0.504  0.191  12
## 1254          1   line-21      Dll   30 0.523 0.493  0.191  13
## 1255          1   line-21      Dll   30 0.537 0.485  0.189   9
## 1256          1   line-21      Dll   30 0.543 0.484  0.187  11
## 1257          1   line-21      Dll   30 0.558 0.489  0.184  13
## 1258          1   line-21      Dll   30 0.567 0.501  0.183   9
## 1259          1   line-21      Dll   30 0.554 0.495  0.180  10
## 1260          1   line-21      Dll   30 0.556 0.499  0.180   9
## 1261          1   line-21      Dll   30 0.545 0.477  0.178  10
## 1262          1   line-21      Dll   30 0.554 0.490  0.177   9
## 1263          1   line-21      Dll   30 0.539 0.481  0.176  12
## 1264          1   line-21      Dll   30 0.544 0.489  0.175  11
## 1265          1   line-21      Dll   30 0.533 0.491  0.174  10
## 1266          1   line-21      Dll   30 0.493 0.492  0.174   9
## 1267          1   line-21      Dll   30 0.528 0.466  0.171  11
## 1268          1   line-21      Dll   30 0.517 0.469  0.167   9
## 1269          1   line-21      Dll   30 0.547 0.511  0.166  13
## 1270          1   line-21      Dll   30 0.510 0.484  0.144   9
## 1271          1   line-21      Dll   30 0.545 0.474  0.141   9
## 1272          1   line-21      Dll   30 0.542 0.465  0.137  11
## 1273          1   line-22      Dll   30 0.508 0.456  0.199  12
## 1274          1   line-22      Dll   30 0.528 0.472  0.190  10
## 1275          1   line-22      Dll   30 0.530 0.471  0.190  10
## 1276          1   line-22      Dll   30 0.517 0.469  0.184  10
## 1277          1   line-22      Dll   30 0.536 0.474  0.180  10
## 1278          1   line-22      Dll   30 0.527 0.454  0.180  11
## 1279          1   line-22      Dll   30 0.524 0.470  0.174  11
## 1280          1   line-22      Dll   30 0.527 0.464  0.173  10
## 1281          1   line-22      Dll   30 0.532 0.450  0.172  10
## 1282          1   line-22      Dll   30 0.516 0.466  0.171   9
## 1283          1   line-22      Dll   30 0.523 0.472  0.170  11
## 1284          1   line-22      Dll   30 0.515 0.449  0.170  11
## 1285          1   line-22      Dll   30 0.519 0.460  0.170  11
## 1286          1   line-22      Dll   30 0.527 0.440  0.168  11
## 1287          1   line-22      Dll   30 0.498 0.425  0.168   9
## 1288          1   line-22      Dll   30 0.514 0.461  0.166   9
## 1289          1   line-22      Dll   30 0.534 0.454  0.164  12
## 1290          1   line-22      Dll   30 0.521 0.451  0.162   9
## 1291          1   line-22      Dll   30 0.532 0.462  0.162  12
## 1292          1   line-22      Dll   30 0.520 0.484  0.157   9
## 1293          1   line-23      Dll   30 0.519 0.460  0.182  12
## 1294          1   line-23      Dll   30 0.539 0.497  0.178  10
## 1295          1   line-23      Dll   30 0.526 0.477  0.174   9
## 1296          1   line-23      Dll   30 0.521 0.427  0.172  11
## 1297          1   line-23      Dll   30 0.505 0.433  0.171  10
## 1298          1   line-23      Dll   30 0.503 0.444  0.171  10
## 1299          1   line-23      Dll   30 0.532 0.463  0.170   8
## 1300          1   line-23      Dll   30 0.508 0.454  0.170   7
## 1301          1   line-23      Dll   30 0.484 0.422  0.168   7
## 1302          1   line-23      Dll   30 0.533 0.467  0.165  11
## 1303          1   line-23      Dll   30 0.534 0.480  0.164   9
## 1304          1   line-23      Dll   30 0.529 0.482  0.164  12
## 1305          1   line-23      Dll   30 0.508 0.453  0.163   8
## 1306          1   line-23      Dll   30 0.521 0.445  0.162  13
## 1307          1   line-23      Dll   30 0.529 0.484  0.160   9
## 1308          1   line-23      Dll   30 0.531 0.487  0.158   9
## 1309          1   line-23      Dll   30 0.509 0.454  0.156   9
## 1310          1   line-23      Dll   30 0.473 0.433  0.156   9
## 1311          1   line-23      Dll   30 0.489 0.438  0.152   9
## 1312          1   line-23      Dll   30 0.503 0.474  0.140  11
## 1313          1   line-24      Dll   30 0.533 0.483  0.200  13
## 1314          1   line-24      Dll   30 0.538 0.473  0.199  11
## 1315          1   line-24      Dll   30 0.540 0.470  0.191  18
## 1316          1   line-24      Dll   30 0.534 0.480  0.187  18
## 1317          1   line-27      Dll   30 0.560 0.507  0.198  11
## 1318          1   line-27      Dll   30 0.571 0.488  0.197  12
## 1319          1   line-27      Dll   30 0.560 0.484  0.195  10
## 1320          1   line-27      Dll   30 0.558 0.485  0.195   9
## 1321          1   line-27      Dll   30 0.571 0.509  0.192  10
## 1322          1   line-27      Dll   30 0.600 0.545  0.189   8
## 1323          1   line-27      Dll   30 0.577 0.506  0.188   8
## 1324          1   line-27      Dll   30 0.585 0.507  0.184   9
## 1325          1   line-27      Dll   30 0.549 0.504  0.183  10
## 1326          1   line-27      Dll   30 0.565 0.497  0.183  11
## 1327          1   line-27      Dll   30 0.551 0.473  0.183  10
## 1328          1   line-27      Dll   30 0.554 0.518  0.181  11
## 1329          1   line-27      Dll   30 0.536 0.493  0.178  11
## 1330          1   line-27      Dll   30 0.560 0.487  0.175   8
## 1331          1   line-27      Dll   30 0.554 0.510  0.170  12
## 1332          1   line-27      Dll   30 0.552 0.491  0.169  11
## 1333          1   line-27      Dll   30 0.555 0.486  0.168  12
## 1334          1   line-27      Dll   30 0.550 0.474  0.160  10
## 1335          1    line-3      Dll   30 0.536 0.493  0.207  14
## 1336          1    line-3      Dll   30 0.554 0.506  0.179   9
## 1337          1    line-3      Dll   30 0.530 0.470  0.177   9
## 1338          1    line-3      Dll   30 0.534 0.481  0.164  12
## 1339          1    line-3      Dll   30 0.533 0.465  0.157  16
## 1340          1    line-3      Dll   30 0.501 0.455  0.155  12
## 1341          1    line-4      Dll   30 0.556 0.507  0.195  12
## 1343          1    line-4      Dll   30 0.570 0.493  0.183  10
## 1344          1    line-4      Dll   30 0.544 0.474  0.178  12
## 1345          1    line-4      Dll   30 0.512 0.462  0.178  11
## 1346          1    line-4      Dll   30 0.525 0.481  0.176   9
## 1347          1    line-4      Dll   30 0.540 0.460  0.174   9
## 1348          1    line-4      Dll   30 0.557 0.485  0.172  11
## 1349          1    line-4      Dll   30 0.523 0.450  0.170  10
## 1350          1    line-4      Dll   30 0.537 0.472  0.169  10
## 1351          1    line-4      Dll   30 0.551 0.493  0.168  10
## 1352          1    line-4      Dll   30 0.560 0.468  0.167  10
## 1353          1    line-4      Dll   30 0.505 0.448  0.166  10
## 1354          1    line-4      Dll   30 0.506 0.457  0.165   9
## 1355          1    line-4      Dll   30 0.519 0.458  0.165  12
## 1356          1    line-4      Dll   30 0.552 0.481  0.165  11
## 1357          1    line-4      Dll   30 0.523 0.458  0.162  11
## 1358          1    line-4      Dll   30 0.541 0.470  0.158   7
## 1359          1    line-4      Dll   30 0.541 0.447  0.156  10
## 1360          1    line-4      Dll   30 0.548 0.495  0.154  10
## 1361          1    line-4      Dll   30 0.513 0.471  0.152  10
## 1362          1    line-4      Dll   30 0.483 0.396  0.151   9
## 1363          1    line-6      Dll   30 0.557 0.494  0.196  11
## 1364          1    line-6      Dll   30 0.560 0.482  0.195  13
## 1365          1    line-6      Dll   30 0.534 0.476  0.187  11
## 1366          1    line-6      Dll   30 0.543 0.512  0.179  10
## 1367          1    line-6      Dll   30 0.553 0.516  0.176  12
## 1368          1    line-6      Dll   30 0.539 0.482  0.176  11
## 1369          1    line-6      Dll   30 0.504 0.509  0.175  18
## 1370          1    line-6      Dll   30 0.539 0.478  0.174  11
## 1371          1    line-6      Dll   30 0.572 0.535  0.173  12
## 1372          1    line-6      Dll   30 0.516 0.451  0.169  10
## 1373          1    line-6      Dll   30 0.558 0.505  0.169  11
## 1374          1    line-7      Dll   30 0.531 0.439  0.186  11
## 1375          1    line-7      Dll   30 0.550 0.474  0.186  11
## 1376          1    line-7      Dll   30 0.551 0.498  0.186  12
## 1377          1    line-7      Dll   30 0.558 0.497  0.183  11
## 1378          1    line-7      Dll   30 0.542 0.472  0.181  10
## 1379          1    line-7      Dll   30 0.524 0.473  0.180   8
## 1380          1    line-7      Dll   30 0.534 0.473  0.178  10
## 1381          1    line-7      Dll   30 0.494 0.441  0.178  10
## 1382          1    line-7      Dll   30 0.536 0.491  0.175  14
## 1383          1    line-7      Dll   30 0.533 0.449  0.175   9
## 1384          1    line-7      Dll   30 0.552 0.486  0.175   9
## 1385          1    line-7      Dll   30 0.551 0.488  0.175  11
## 1386          1    line-7      Dll   30 0.550 0.479  0.174   9
## 1387          1    line-7      Dll   30 0.563 0.462  0.173  11
## 1388          1    line-7      Dll   30 0.565 0.496  0.172  10
## 1389          1    line-7      Dll   30 0.500 0.456  0.171   8
## 1390          1    line-7      Dll   30 0.491 0.463  0.168  10
## 1391          1    line-7      Dll   30 0.548 0.493  0.167  11
## 1392          1    line-7      Dll   30 0.543 0.486  0.167  10
## 1394          1    line-7      Dll   30 0.561 0.489  0.164  11
## 1395          1    line-7      Dll   30 0.516 0.460  0.162  10
## 1396          1    line-7      Dll   30 0.532 0.501  0.157   9
## 1397          1    line-8      Dll   30 0.598 0.531  0.206  12
## 1398          1    line-8      Dll   30 0.569 0.515  0.199  11
## 1399          1    line-8      Dll   30 0.554 0.472  0.196   8
## 1400          1    line-8      Dll   30 0.552 0.506  0.192  10
## 1401          1    line-8      Dll   30 0.548 0.522  0.189  10
## 1402          1    line-8      Dll   30 0.537 0.523  0.187  13
## 1403          1    line-8      Dll   30 0.568 0.521  0.186   9
## 1404          1    line-8      Dll   30 0.540 0.498  0.184   9
## 1405          1    line-8      Dll   30 0.556 0.506  0.183   9
## 1406          1    line-8      Dll   30 0.536 0.501  0.176   8
## 1407          1    line-8      Dll   30 0.537 0.500  0.173  11
## 1408          1    line-8      Dll   30 0.558 0.545  0.170  11
## 1409          1    line-8      Dll   30 0.512 0.468  0.169   6
## 1410          1    line-8      Dll   30 0.521 0.497  0.169  10
## 1411          1    line-8      Dll   30 0.554 0.504  0.165  10
## 1412          1    line-8      Dll   30 0.536 0.505  0.161   9
## 1413          1    line-8      Dll   30 0.504 0.410  0.143   9
## 1414          1    line-9      Dll   30 0.579 0.515  0.208  10
## 1415          1    line-9      Dll   30 0.567 0.509  0.197  10
## 1416          1    line-9      Dll   30 0.576 0.500  0.193  10
## 1417          1    line-9      Dll   30 0.587 0.525  0.185  11
## 1418          1    line-9      Dll   30 0.563 0.493  0.183  11
## 1419          1    line-9      Dll   30 0.537 0.466  0.172   9
## 1420          1 line-CanS      Dll   30 0.569 0.492  0.196  11
## 1421          1 line-CanS      Dll   30 0.549 0.482  0.190  10
## 1422          1 line-CanS      Dll   30 0.517 0.464  0.190  10
## 1423          1 line-CanS      Dll   30 0.541 0.475  0.187  11
## 1424          1 line-CanS      Dll   30 0.532 0.457  0.183  10
## 1425          1 line-CanS      Dll   30 0.546 0.472  0.183  11
## 1426          1 line-CanS      Dll   30 0.556 0.476  0.178  10
## 1427          1 line-CanS      Dll   30 0.544 0.476  0.176  11
## 1428          1 line-OreR      Dll   30 0.510 0.494  0.190   9
## 1429          1 line-OreR      Dll   30 0.541 0.517  0.190   9
## 1430          1 line-OreR      Dll   30 0.551 0.502  0.185  11
## 1431          1 line-OreR      Dll   30 0.546 0.480  0.174   9
## 1432          1 line-OreR      Dll   30 0.527 0.501  0.172   9
## 1433          1 line-OreR      Dll   30 0.553 0.479  0.168   8
## 1434          1 line-OreR      Dll   30 0.560 0.460  0.167  10
## 1435          1 line-OreR      Dll   30 0.559 0.474  0.166   8
## 1436          1 line-OreR      Dll   30 0.515 0.483  0.159  10
## 1437          1 line-OreR      Dll   30 0.538 0.464  0.159   9
## 1438          1 line-OreR      Dll   30 0.501 0.454  0.156  11
## 1439          1 line-OreR      Dll   30 0.537 0.436  0.152  10
## 1440          1 line-OreR      Dll   30 0.554 0.498  0.151  10
## 1441          1 line-OreR      Dll   30 0.543 0.472  0.149  10
## 1442          1 line-OreR      Dll   30 0.548 0.467  0.140  10
## 1443          1 line-OreR      Dll   30 0.542 0.452  0.134  10
## 1444          1 line-OreR      Dll   30 0.536 0.439  0.121   8
## 1445          1  line-Sam      Dll   30 0.570 0.463  0.206  11
## 1446          1  line-Sam      Dll   30 0.569 0.473  0.189  12
## 1447          1  line-Sam      Dll   30 0.564 0.495  0.187  12
## 1448          1  line-Sam      Dll   30 0.569 0.500  0.186  11
## 1449          1  line-Sam      Dll   30 0.533 0.467  0.186  12
## 1450          1  line-Sam      Dll   30 0.561 0.487  0.186  10
## 1451          1  line-Sam      Dll   30 0.564 0.522  0.186  11
## 1452          1  line-Sam      Dll   30 0.543 0.504  0.184  10
## 1453          1  line-Sam      Dll   30 0.523 0.451  0.182  12
## 1454          1  line-Sam      Dll   30 0.557 0.477  0.181  12
## 1455          1  line-Sam      Dll   30 0.550 0.472  0.180  12
## 1456          1  line-Sam      Dll   30 0.558 0.507  0.177  14
## 1457          1  line-Sam      Dll   30 0.533 0.492  0.175  13
## 1458          1  line-Sam      Dll   30 0.538 0.459  0.174  11
## 1459          1  line-Sam      Dll   30 0.524 0.495  0.169  14
## 1460          1  line-Sam      Dll   30 0.564 0.486  0.169  10
## 1461          1  line-Sam      Dll   30 0.487 0.432  0.157   9
## 1463          1    line-w      Dll   30 0.539 0.501  0.192  12
## 1464          1    line-w      Dll   30 0.494 0.472  0.191  16
## 1469          1    line-w      Dll   30 0.506 0.451  0.174  22
## 1470          1    line-w      Dll   30 0.552 0.501  0.171  16
## 1471          1    line-w      Dll   30 0.514 0.477  0.171  15
## 1473          1    line-w      Dll   30 0.552 0.494  0.167  13
## 1474          1    line-w      Dll   30 0.481 0.456  0.150  15
## 1475          1    line-w      Dll   30 0.489 0.467  0.150  13
## 1481          1    line-1       wt   30 0.544 0.485  0.208  10
## 1482          1    line-1       wt   30 0.562 0.456  0.199  12
## 1483          1    line-1       wt   30 0.576 0.480  0.191  10
## 1484          1    line-1       wt   30 0.548 0.465  0.187   9
## 1485          1    line-1       wt   30 0.530 0.454  0.186  11
## 1486          1    line-1       wt   30 0.532 0.468  0.186   9
## 1487          1    line-1       wt   30 0.554 0.470  0.185  10
## 1488          1    line-1       wt   30 0.523 0.469  0.184  10
## 1489          1    line-1       wt   30 0.569 0.510  0.182   9
## 1490          1    line-1       wt   30 0.572 0.473  0.175   9
## 1491          1    line-1       wt   30 0.488 0.443  0.173  10
## 1492          1    line-1       wt   30 0.548 0.460  0.172  11
## 1493          1    line-1       wt   30 0.551 0.467  0.171  10
## 1494          1    line-1       wt   30 0.525 0.428  0.171   9
## 1495          1    line-1       wt   30 0.526 0.461  0.170  10
## 1496          1    line-1       wt   30 0.549 0.480  0.170   9
## 1497          1    line-1       wt   30 0.526 0.430  0.169  11
## 1498          1    line-1       wt   30 0.502 0.420  0.169  10
## 1499          1    line-1       wt   30 0.532 0.464  0.169  10
## 1500          1    line-1       wt   30 0.500 0.425  0.162   9
## 1501          1    line-1       wt   30 0.476 0.373  0.150  10
## 1502          2   line-11       wt   30 0.526 0.461  0.202  12
## 1503          2   line-11       wt   30 0.546 0.491  0.200  15
## 1505          2   line-11       wt   30 0.548 0.454  0.193  13
## 1506          2   line-11       wt   30 0.574 0.514  0.193  13
## 1507          1   line-11       wt   30 0.586 0.514  0.191  13
## 1508          2   line-11       wt   30 0.571 0.500  0.191  13
## 1509          2   line-11       wt   30 0.544 0.491  0.190  11
## 1510          2   line-11       wt   30 0.568 0.497  0.190  12
## 1511          2   line-11       wt   30 0.545 0.457  0.190  13
## 1512          2   line-11       wt   30 0.551 0.470  0.190  13
## 1513          1   line-11       wt   30 0.563 0.500  0.190  11
## 1514          1   line-11       wt   30 0.539 0.490  0.190  14
## 1515          2   line-11       wt   30 0.563 0.492  0.189  13
## 1516          2   line-11       wt   30 0.540 0.454  0.189  12
## 1517          1   line-11       wt   30 0.588 0.535  0.189  13
## 1518          2   line-11       wt   30 0.568 0.501  0.188  12
## 1519          2   line-11       wt   30 0.527 0.471  0.188  12
## 1520          1   line-11       wt   30 0.563 0.499  0.185  14
## 1521          2   line-11       wt   30 0.539 0.492  0.185  13
## 1522          1   line-11       wt   30 0.562 0.480  0.185  13
## 1523          1   line-11       wt   30 0.530 0.468  0.184  14
## 1524          2   line-11       wt   30 0.539 0.446  0.184  12
## 1525          1   line-11       wt   30 0.550 0.502  0.183  13
## 1526          2   line-11       wt   30 0.566 0.494  0.182  12
## 1527          2   line-11       wt   30 0.570 0.490  0.178  14
## 1528          1   line-11       wt   30 0.567 0.495  0.178  10
## 1529          2   line-11       wt   30 0.537 0.466  0.174  10
## 1530          1   line-11       wt   30 0.532 0.453  0.173  12
## 1531          1   line-11       wt   30 0.539 0.499  0.173  12
## 1532          2   line-11       wt   30 0.571 0.499  0.168  10
## 1533          1   line-11       wt   30 0.530 0.462  0.164  11
## 1534          1   line-12       wt   30 0.553 0.476  0.209  10
## 1535          1   line-12       wt   30 0.537 0.476  0.204  10
## 1536          1   line-12       wt   30 0.571 0.486  0.202  11
## 1537          1   line-12       wt   30 0.567 0.509  0.199  10
## 1538          1   line-12       wt   30 0.505 0.523  0.198  10
## 1539          1   line-12       wt   30 0.573 0.491  0.190  10
## 1540          1   line-12       wt   30 0.526 0.471  0.189  10
## 1541          1   line-12       wt   30 0.501 0.443  0.184  11
## 1542          1   line-12       wt   30 0.580 0.509  0.177  12
## 1543          1   line-12       wt   30 0.548 0.483  0.172   9
## 1544          1   line-12       wt   30 0.561 0.487  0.172  10
## 1545          1   line-12       wt   30 0.499 0.412  0.165   9
## 1546          1   line-12       wt   30 0.566 0.540  0.162  10
## 1547          1   line-12       wt   30 0.496 0.442  0.161  10
## 1548          2   line-13       wt   30 0.555 0.481  0.212  10
## 1549          2   line-13       wt   30 0.562 0.486  0.200  13
## 1550          2   line-13       wt   30 0.560 0.495  0.194  12
## 1551          2   line-13       wt   30 0.519 0.499  0.194  13
## 1552          2   line-13       wt   30 0.569 0.470  0.193  11
## 1553          2   line-13       wt   30 0.559 0.499  0.193  11
## 1554          2   line-13       wt   30 0.562 0.497  0.193  13
## 1555          2   line-13       wt   30 0.531 0.496  0.191  11
## 1556          2   line-13       wt   30 0.542 0.461  0.190  12
## 1557          2   line-13       wt   30 0.546 0.483  0.190  12
## 1558          2   line-13       wt   30 0.530 0.446  0.188  11
## 1559          2   line-13       wt   30 0.559 0.496  0.182  11
## 1560          2   line-13       wt   30 0.531 0.457  0.182  11
## 1561          2   line-13       wt   30 0.557 0.483  0.182  11
## 1562          2   line-13       wt   30 0.555 0.500  0.181  12
## 1563          2   line-13       wt   30 0.558 0.495  0.179  12
## 1564          2   line-13       wt   30 0.560 0.506  0.179  11
## 1565          2   line-13       wt   30 0.559 0.500  0.172  11
## 1566          2   line-13       wt   30 0.551 0.498  0.170  10
## 1567          2   line-13       wt   30 0.540 0.493  0.165  11
## 1568          2   line-13       wt   30 0.531 0.473  0.155  10
## 1569          1   line-15       wt   30 0.569 0.509  0.195  10
## 1570          1   line-15       wt   30 0.546 0.484  0.192  10
## 1571          1   line-15       wt   30 0.545 0.494  0.186  10
## 1572          1   line-15       wt   30 0.564 0.498  0.184  12
## 1573          1   line-15       wt   30 0.543 0.501  0.184  11
## 1574          1   line-15       wt   30 0.562 0.502  0.183  11
## 1575          1   line-15       wt   30 0.541 0.469  0.183  10
## 1576          1   line-15       wt   30 0.557 0.480  0.181  10
## 1577          1   line-15       wt   30 0.530 0.476  0.179  12
## 1578          1   line-15       wt   30 0.532 0.476  0.179  11
## 1579          1   line-15       wt   30 0.503 0.470  0.178  11
## 1580          1   line-15       wt   30 0.548 0.507  0.175  13
## 1581          1   line-15       wt   30 0.534 0.492  0.174  11
## 1582          1   line-15       wt   30 0.516 0.480  0.174  12
## 1583          1   line-15       wt   30 0.527 0.460  0.173  12
## 1584          1   line-15       wt   30 0.523 0.462  0.172  11
## 1585          1   line-16       wt   30 0.586 0.507  0.221  10
## 1586          1   line-16       wt   30 0.570 0.467  0.207  13
## 1587          1   line-16       wt   30 0.560 0.462  0.207  10
## 1588          1   line-16       wt   30 0.551 0.468  0.200  11
## 1589          1   line-16       wt   30 0.578 0.477  0.199  10
## 1590          1   line-16       wt   30 0.551 0.442  0.198  13
## 1591          1   line-16       wt   30 0.557 0.452  0.198  12
## 1592          1   line-16       wt   30 0.577 0.501  0.196  13
## 1593          1   line-16       wt   30 0.523 0.480  0.196  11
## 1594          1   line-16       wt   30 0.564 0.455  0.194  13
## 1595          1   line-16       wt   30 0.549 0.454  0.192  12
## 1596          1   line-16       wt   30 0.554 0.479  0.191  13
## 1597          1   line-16       wt   30 0.514 0.446  0.190  10
## 1598          1   line-16       wt   30 0.547 0.456  0.189  12
## 1599          1   line-16       wt   30 0.573 0.460  0.187  15
## 1600          1   line-16       wt   30 0.555 0.506  0.183  10
## 1601          1   line-16       wt   30 0.559 0.508  0.179   7
## 1602          1   line-16       wt   30 0.532 0.436  0.175  11
## 1603          2   line-17       wt   30 0.554 0.477  0.198  11
## 1604          2   line-17       wt   30 0.548 0.459  0.196  10
## 1605          2   line-17       wt   30 0.541 0.475  0.192  12
## 1606          2   line-17       wt   30 0.532 0.491  0.187  10
## 1607          2   line-17       wt   30 0.536 0.473  0.182  10
## 1608          2   line-17       wt   30 0.528 0.459  0.180   8
## 1609          2   line-17       wt   30 0.513 0.476  0.178  12
## 1610          2   line-17       wt   30 0.526 0.452  0.175  10
## 1611          2   line-17       wt   30 0.531 0.450  0.175  11
## 1612          2   line-17       wt   30 0.545 0.465  0.174   9
## 1613          2   line-17       wt   30 0.542 0.465  0.174  11
## 1614          2   line-17       wt   30 0.495 0.413  0.172   9
## 1615          2   line-17       wt   30 0.519 0.450  0.172  11
## 1616          2   line-17       wt   30 0.517 0.449  0.168  10
## 1617          2   line-17       wt   30 0.539 0.482  0.168  10
## 1618          2   line-17       wt   30 0.514 0.465  0.167  13
## 1619          2   line-17       wt   30 0.510 0.451  0.163   8
## 1620          2   line-17       wt   30 0.516 0.472  0.162  12
## 1621          2   line-18       wt   30 0.546 0.489  0.195  10
## 1622          1   line-18       wt   30 0.553 0.476  0.193  10
## 1623          1   line-18       wt   30 0.543 0.475  0.190   9
## 1624          1   line-18       wt   30 0.556 0.481  0.187  10
## 1625          2   line-18       wt   30 0.553 0.490  0.187  10
## 1626          1   line-18       wt   30 0.515 0.433  0.186  11
## 1627          2   line-18       wt   30 0.550 0.483  0.186  10
## 1628          1   line-18       wt   30 0.519 0.480  0.184  11
## 1629          1   line-18       wt   30 0.528 0.478  0.183  10
## 1630          1   line-18       wt   30 0.545 0.477  0.182  10
## 1631          1   line-18       wt   30 0.550 0.447  0.181  10
## 1632          1   line-18       wt   30 0.534 0.458  0.181   8
## 1633          1   line-18       wt   30 0.527 0.474  0.181   9
## 1634          2   line-18       wt   30 0.526 0.442  0.180  10
## 1635          1   line-18       wt   30 0.556 0.467  0.180  10
## 1636          2   line-18       wt   30 0.519 0.492  0.180  10
## 1637          2   line-18       wt   30 0.532 0.484  0.180   9
## 1638          1   line-18       wt   30 0.527 0.452  0.179  10
## 1639          1   line-18       wt   30 0.521 0.459  0.178   9
## 1640          2   line-18       wt   30 0.541 0.473  0.178  10
## 1641          1   line-18       wt   30 0.488 0.501  0.178   9
## 1642          2   line-18       wt   30 0.539 0.481  0.178  11
## 1643          2   line-18       wt   30 0.537 0.460  0.175  10
## 1644          2   line-18       wt   30 0.523 0.477  0.173  11
## 1645          1   line-18       wt   30 0.525 0.448  0.172   9
## 1646          1   line-18       wt   30 0.509 0.442  0.172   9
## 1647          1   line-18       wt   30 0.538 0.465  0.171   9
## 1648          2   line-18       wt   30 0.523 0.473  0.169  12
## 1649          1   line-18       wt   30 0.517 0.445  0.168   8
## 1650          1   line-18       wt   30 0.505 0.433  0.168   8
## 1651          1   line-18       wt   30 0.496 0.445  0.167  10
## 1652          2   line-18       wt   30 0.558 0.494  0.167   9
## 1653          2   line-18       wt   30 0.492 0.427  0.165   8
## 1654          1   line-18       wt   30 0.512 0.441  0.163  11
## 1655          2   line-18       wt   30 0.496 0.406  0.163   9
## 1656          2   line-18       wt   30 0.509 0.433  0.161  11
## 1657          2   line-18       wt   30 0.493 0.451  0.160  11
## 1658          2   line-18       wt   30 0.541 0.485  0.159  11
## 1661          1   line-19       wt   30 0.483 0.431  0.186  11
## 1662          1   line-19       wt   30 0.518 0.467  0.183  12
## 1663          1   line-19       wt   30 0.514 0.443  0.180  13
## 1664          1   line-19       wt   30 0.523 0.420  0.178  11
## 1665          1   line-19       wt   30 0.490 0.422  0.175   9
## 1666          1   line-19       wt   30 0.493 0.427  0.174  12
## 1667          1   line-19       wt   30 0.495 0.445  0.173  13
## 1668          1   line-19       wt   30 0.531 0.417  0.171  11
## 1669          1   line-19       wt   30 0.509 0.444  0.168  13
## 1670          1   line-19       wt   30 0.525 0.484  0.167   9
## 1671          1   line-19       wt   30 0.555 0.500  0.165   9
## 1672          1   line-19       wt   30 0.492 0.423  0.165  12
## 1673          1   line-19       wt   30 0.473 0.409  0.164  11
## 1674          1   line-19       wt   30 0.506 0.437  0.163  10
## 1675          1   line-19       wt   30 0.512 0.426  0.162  10
## 1676          1   line-19       wt   30 0.506 0.458  0.159  11
## 1677          1   line-19       wt   30 0.527 0.465  0.159  11
## 1678          1   line-19       wt   30 0.512 0.437  0.157  12
## 1679          1   line-19       wt   30 0.509 0.441  0.145  11
## 1680          1   line-19       wt   30 0.503 0.445  0.144  13
## 1681          1    line-2       wt   30 0.522 0.483  0.206  13
## 1682          1    line-2       wt   30 0.496 0.442  0.202  13
## 1683          1    line-2       wt   30 0.509 0.462  0.200  12
## 1684          1    line-2       wt   30 0.511 0.452  0.193  13
## 1685          1    line-2       wt   30 0.518 0.454  0.193  11
## 1686          1    line-2       wt   30 0.521 0.459  0.188  12
## 1687          1    line-2       wt   30 0.535 0.494  0.186  12
## 1688          1    line-2       wt   30 0.546 0.506  0.185  11
## 1689          1    line-2       wt   30 0.549 0.449  0.183  11
## 1690          1    line-2       wt   30 0.508 0.421  0.182  11
## 1691          1    line-2       wt   30 0.532 0.457  0.181  11
## 1692          1    line-2       wt   30 0.524 0.458  0.180  10
## 1693          1    line-2       wt   30 0.510 0.463  0.180  13
## 1694          1    line-2       wt   30 0.536 0.443  0.179  12
## 1695          1    line-2       wt   30 0.535 0.472  0.175  11
## 1696          1    line-2       wt   30 0.507 0.442  0.174  12
## 1697          1    line-2       wt   30 0.527 0.460  0.174  12
## 1698          1    line-2       wt   30 0.481 0.421  0.165  10
## 1699          1    line-2       wt   30 0.506 0.439  0.164  11
## 1700          1    line-2       wt   30 0.510 0.424  0.164  10
## 1702          1    line-2       wt   30 0.511 0.445  0.158  11
## 1703          1    line-2       wt   30 0.519 0.454  0.156  11
## 1704          1   line-20       wt   30 0.489 0.433  0.179  11
## 1705          1   line-20       wt   30 0.491 0.451  0.174  11
## 1706          1   line-20       wt   30 0.491 0.440  0.172  10
## 1707          1   line-20       wt   30 0.441 0.441  0.167  11
## 1708          1   line-20       wt   30 0.480 0.419  0.167  11
## 1709          1   line-20       wt   30 0.482 0.445  0.167  10
## 1710          1   line-20       wt   30 0.509 0.471  0.167  10
## 1711          1   line-20       wt   30 0.501 0.477  0.165  10
## 1712          1   line-20       wt   30 0.485 0.437  0.164   9
## 1713          1   line-20       wt   30 0.485 0.448  0.160  10
## 1714          1   line-20       wt   30 0.495 0.434  0.159  12
## 1715          1   line-20       wt   30 0.501 0.465  0.159  11
## 1716          1   line-20       wt   30 0.490 0.445  0.157   9
## 1717          1   line-20       wt   30 0.513 0.478  0.149  11
## 1718          1   line-20       wt   30 0.450 0.412  0.149  10
## 1719          1   line-20       wt   30 0.439 0.396  0.149  10
## 1720          1   line-20       wt   30 0.488 0.428  0.148  10
## 1721          1   line-20       wt   30 0.423 0.383  0.147  10
## 1722          1   line-20       wt   30 0.437 0.408  0.139   9
## 1723          1   line-21       wt   30 0.570 0.510  0.204  11
## 1724          1   line-21       wt   30 0.540 0.475  0.201   8
## 1725          1   line-21       wt   30 0.590 0.536  0.200  10
## 1726          1   line-21       wt   30 0.537 0.479  0.200  12
## 1727          1   line-21       wt   30 0.548 0.473  0.199  10
## 1728          1   line-21       wt   30 0.549 0.498  0.196  12
## 1729          1   line-21       wt   30 0.538 0.474  0.196  12
## 1730          1   line-21       wt   30 0.566 0.497  0.192  12
## 1731          1   line-21       wt   30 0.554 0.496  0.191  12
## 1732          1   line-21       wt   30 0.562 0.499  0.191  10
## 1733          1   line-21       wt   30 0.536 0.479  0.191  13
## 1734          1   line-21       wt   30 0.534 0.496  0.186  11
## 1735          1   line-21       wt   30 0.540 0.496  0.184  11
## 1736          1   line-21       wt   30 0.541 0.465  0.184  10
## 1737          1   line-21       wt   30 0.521 0.481  0.181  12
## 1738          1   line-21       wt   30 0.523 0.474  0.180  11
## 1739          1   line-21       wt   30 0.514 0.467  0.178  11
## 1740          1   line-21       wt   30 0.530 0.467  0.168  13
## 1741          1   line-22       wt   30 0.554 0.491  0.201  12
## 1742          1   line-22       wt   30 0.550 0.487  0.200  12
## 1743          1   line-22       wt   30 0.496 0.437  0.195  13
## 1744          1   line-22       wt   30 0.567 0.479  0.191  13
## 1745          1   line-22       wt   30 0.535 0.448  0.187  11
## 1746          1   line-22       wt   30 0.517 0.475  0.184  12
## 1747          1   line-22       wt   30 0.519 0.443  0.181  11
## 1748          1   line-22       wt   30 0.522 0.430  0.181  10
## 1749          1   line-22       wt   30 0.517 0.467  0.180  11
## 1750          1   line-22       wt   30 0.501 0.444  0.176  12
## 1751          1   line-22       wt   30 0.520 0.442  0.175  11
## 1752          1   line-22       wt   30 0.507 0.409  0.175   9
## 1753          1   line-22       wt   30 0.527 0.459  0.174  11
## 1754          1   line-22       wt   30 0.503 0.452  0.174  11
## 1755          1   line-22       wt   30 0.509 0.427  0.173   9
## 1756          1   line-22       wt   30 0.513 0.444  0.168  10
## 1757          1   line-22       wt   30 0.501 0.451  0.168  11
## 1758          1   line-22       wt   30 0.500 0.447  0.168  10
## 1759          1   line-22       wt   30 0.486 0.428  0.167  10
## 1760          1   line-22       wt   30 0.518 0.447  0.167  11
## 1761          1   line-22       wt   30 0.524 0.463  0.167  11
## 1762          1   line-22       wt   30 0.507 0.436  0.165  10
## 1763          1   line-22       wt   30 0.503 0.429  0.161  10
## 1764          1   line-22       wt   30 0.513 0.447  0.159  11
## 1765          1   line-22       wt   30 0.510 0.423  0.154  11
## 1766          1   line-23       wt   30 0.515 0.445  0.180  10
## 1767          1   line-23       wt   30 0.511 0.449  0.176  12
## 1768          1   line-23       wt   30 0.534 0.460  0.175   9
## 1769          1   line-23       wt   30 0.531 0.480  0.174  12
## 1770          1   line-23       wt   30 0.514 0.458  0.173  10
## 1771          1   line-23       wt   30 0.523 0.467  0.171  12
## 1772          1   line-23       wt   30 0.495 0.411  0.171  11
## 1773          1   line-23       wt   30 0.531 0.462  0.170   8
## 1774          1   line-23       wt   30 0.502 0.451  0.170  10
## 1775          1   line-23       wt   30 0.472 0.412  0.169  12
## 1776          1   line-23       wt   30 0.477 0.405  0.166  12
## 1777          1   line-23       wt   30 0.491 0.417  0.165  10
## 1778          1   line-23       wt   30 0.508 0.423  0.162  12
## 1779          1   line-23       wt   30 0.498 0.444  0.161  10
## 1780          1   line-23       wt   30 0.499 0.435  0.160  10
## 1781          1   line-23       wt   30 0.504 0.453  0.159  10
## 1782          1   line-23       wt   30 0.513 0.421  0.159  11
## 1783          1   line-23       wt   30 0.490 0.406  0.159  11
## 1785          1   line-23       wt   30 0.488 0.414  0.154  11
## 1786          1   line-23       wt   30 0.525 0.458  0.153  10
## 1787          1   line-24       wt   30 0.558 0.492  0.207  10
## 1788          1   line-24       wt   30 0.546 0.458  0.202  11
## 1789          1   line-24       wt   30 0.530 0.468  0.202  11
## 1790          1   line-24       wt   30 0.556 0.499  0.200  13
## 1791          1   line-24       wt   30 0.536 0.481  0.199  11
## 1792          1   line-24       wt   30 0.537 0.487  0.196   9
## 1793          1   line-24       wt   30 0.540 0.466  0.195  12
## 1794          1   line-24       wt   30 0.547 0.469  0.194  10
## 1795          1   line-24       wt   30 0.545 0.467  0.192  11
## 1796          1   line-24       wt   30 0.515 0.460  0.191  10
## 1797          1   line-24       wt   30 0.556 0.499  0.188  12
## 1798          1   line-24       wt   30 0.555 0.512  0.187  10
## 1799          1   line-24       wt   30 0.547 0.484  0.181  12
## 1800          1   line-24       wt   30 0.536 0.505  0.180  11
## 1801          1   line-24       wt   30 0.562 0.493  0.178  13
## 1802          1   line-24       wt   30 0.513 0.450  0.173  11
## 1803          1   line-27       wt   30 0.573 0.492  0.203  11
## 1804          1   line-27       wt   30 0.544 0.510  0.202   9
## 1805          1   line-27       wt   30 0.558 0.483  0.200   9
## 1806          1   line-27       wt   30 0.558 0.480  0.196  10
## 1807          1   line-27       wt   30 0.554 0.489  0.195  10
## 1808          1   line-27       wt   30 0.582 0.483  0.195  10
## 1809          1   line-27       wt   30 0.556 0.450  0.192   9
## 1810          1   line-27       wt   30 0.530 0.448  0.190  11
## 1811          1   line-27       wt   30 0.584 0.495  0.189  10
## 1812          1   line-27       wt   30 0.561 0.502  0.189   9
## 1813          1   line-27       wt   30 0.565 0.500  0.189  10
## 1814          1   line-27       wt   30 0.553 0.504  0.183  11
## 1815          1   line-27       wt   30 0.564 0.497  0.183   9
## 1816          1   line-27       wt   30 0.566 0.486  0.181  11
## 1817          1   line-27       wt   30 0.545 0.477  0.181  10
## 1818          1   line-27       wt   30 0.556 0.469  0.181  11
## 1819          1   line-27       wt   30 0.549 0.478  0.180  11
## 1820          1   line-27       wt   30 0.554 0.467  0.177  10
## 1821          1   line-27       wt   30 0.552 0.485  0.177  10
## 1822          1   line-27       wt   30 0.540 0.466  0.177  10
## 1823          1   line-27       wt   30 0.543 0.467  0.169  10
## 1824          1   line-27       wt   30 0.546 0.484  0.166  10
## 1825          1    line-3       wt   30 0.551 0.495  0.193  10
## 1826          1    line-3       wt   30 0.565 0.486  0.187  10
## 1827          1    line-3       wt   30 0.543 0.501  0.174  10
## 1828          1    line-3       wt   30 0.541 0.488  0.173  10
## 1829          1    line-3       wt   30 0.451 0.413  0.157  10
## 1830          1    line-4       wt   30 0.575 0.504  0.203  11
## 1831          1    line-4       wt   30 0.570 0.514  0.202  13
## 1832          1    line-4       wt   30 0.572 0.491  0.201  11
## 1833          1    line-4       wt   30 0.584 0.521  0.195  11
## 1834          1    line-4       wt   30 0.595 0.500  0.193  12
## 1835          1    line-4       wt   30 0.551 0.490  0.191  11
## 1836          1    line-4       wt   30 0.562 0.492  0.188  11
## 1837          1    line-4       wt   30 0.564 0.507  0.186  11
## 1838          1    line-4       wt   30 0.518 0.424  0.185  11
## 1839          1    line-4       wt   30 0.582 0.512  0.185  11
## 1840          1    line-4       wt   30 0.558 0.497  0.183  10
## 1841          1    line-4       wt   30 0.560 0.466  0.177  12
## 1842          1    line-4       wt   30 0.575 0.506  0.177  10
## 1843          1    line-4       wt   30 0.539 0.484  0.175  11
## 1844          1    line-4       wt   30 0.528 0.459  0.174  13
## 1845          1    line-4       wt   30 0.544 0.498  0.172  12
## 1847          1    line-4       wt   30 0.567 0.490  0.167  12
## 1848          1    line-4       wt   30 0.539 0.464  0.167  10
## 1849          1    line-4       wt   30 0.524 0.481  0.165  12
## 1850          1    line-6       wt   30 0.559 0.484  0.195  11
## 1851          1    line-6       wt   30 0.554 0.484  0.194  12
## 1852          1    line-6       wt   30 0.566 0.493  0.194  12
## 1853          1    line-6       wt   30 0.560 0.509  0.192  12
## 1854          1    line-6       wt   30 0.571 0.502  0.190  11
## 1855          1    line-6       wt   30 0.578 0.512  0.187  14
## 1856          1    line-6       wt   30 0.550 0.478  0.185  11
## 1857          1    line-6       wt   30 0.555 0.480  0.184  12
## 1858          1    line-6       wt   30 0.564 0.506  0.184  14
## 1859          1    line-6       wt   30 0.565 0.500  0.178  11
## 1860          1    line-6       wt   30 0.502 0.468  0.176  15
## 1861          1    line-6       wt   30 0.567 0.504  0.174  11
## 1862          1    line-6       wt   30 0.560 0.498  0.171  11
## 1863          1    line-6       wt   30 0.539 0.484  0.163  10
## 1864          1    line-7       wt   30 0.548 0.485  0.198  10
## 1865          1    line-7       wt   30 0.542 0.459  0.194   9
## 1866          1    line-7       wt   30 0.534 0.439  0.191  11
## 1868          1    line-7       wt   30 0.554 0.470  0.186  13
## 1869          1    line-7       wt   30 0.542 0.475  0.185   9
## 1870          1    line-7       wt   30 0.534 0.469  0.184  11
## 1871          1    line-7       wt   30 0.538 0.488  0.183   9
## 1872          1    line-7       wt   30 0.572 0.460  0.183  11
## 1873          1    line-7       wt   30 0.574 0.496  0.181   9
## 1874          1    line-7       wt   30 0.580 0.501  0.178  10
## 1876          1    line-7       wt   30 0.557 0.487  0.177   9
## 1877          1    line-7       wt   30 0.496 0.440  0.175  11
## 1878          1    line-7       wt   30 0.558 0.473  0.172  10
## 1879          1    line-7       wt   30 0.548 0.460  0.170  11
## 1880          1    line-7       wt   30 0.546 0.460  0.169  12
## 1881          1    line-7       wt   30 0.554 0.467  0.168  11
## 1882          1    line-7       wt   30 0.520 0.453  0.168  12
## 1883          1    line-7       wt   30 0.483 0.396  0.167  10
## 1884          1    line-7       wt   30 0.521 0.446  0.166   8
## 1885          1    line-7       wt   30 0.525 0.467  0.163  11
## 1886          1    line-7       wt   30 0.535 0.439  0.162  10
## 1887          1    line-8       wt   30 0.548 0.470  0.203  10
## 1888          1    line-8       wt   30 0.537 0.521  0.203  11
## 1889          1    line-8       wt   30 0.590 0.509  0.200   9
## 1890          1    line-8       wt   30 0.556 0.499  0.198  11
## 1891          1    line-8       wt   30 0.563 0.489  0.198  12
## 1892          1    line-8       wt   30 0.542 0.488  0.193  10
## 1893          1    line-8       wt   30 0.528 0.497  0.190  10
## 1894          1    line-8       wt   30 0.539 0.513  0.189  12
## 1895          1    line-8       wt   30 0.558 0.470  0.188  10
## 1896          1    line-8       wt   30 0.566 0.509  0.186  10
## 1897          1    line-8       wt   30 0.510 0.458  0.183   8
## 1898          1    line-8       wt   30 0.553 0.513  0.181   8
## 1899          1    line-8       wt   30 0.565 0.501  0.180  11
## 1900          1    line-8       wt   30 0.531 0.474  0.177   9
## 1901          1    line-8       wt   30 0.563 0.485  0.171  12
## 1902          1    line-8       wt   30 0.566 0.493  0.170  10
## 1903          1    line-8       wt   30 0.514 0.468  0.166  11
## 1904          1    line-8       wt   30 0.510 0.476  0.155  10
## 1905          1    line-9       wt   30 0.698 0.609  0.253  11
## 1906          1    line-9       wt   30 0.669 0.578  0.250  10
## 1907          1    line-9       wt   30 0.636 0.593  0.220  11
## 1908          1 line-CanS       wt   30 0.551 0.505  0.185  11
## 1909          1 line-CanS       wt   30 0.526 0.478  0.179  11
## 1910          1 line-CanS       wt   30 0.486 0.439  0.167  10
## 1911          1 line-CanS       wt   30 0.538 0.461  0.167   9
## 1913          1 line-OreR       wt   30 0.587 0.499  0.202  10
## 1914          1 line-OreR       wt   30 0.601 0.531  0.190  10
## 1915          1 line-OreR       wt   30 0.569 0.505  0.189  12
## 1916          1 line-OreR       wt   30 0.563 0.481  0.189  11
## 1917          1 line-OreR       wt   30 0.559 0.487  0.188  11
## 1918          1 line-OreR       wt   30 0.564 0.496  0.187  11
## 1919          1 line-OreR       wt   30 0.574 0.507  0.185  11
## 1920          1 line-OreR       wt   30 0.559 0.500  0.185  10
## 1921          1 line-OreR       wt   30 0.548 0.474  0.184  12
## 1922          1 line-OreR       wt   30 0.551 0.468  0.183  10
## 1923          1 line-OreR       wt   30 0.563 0.503  0.180  11
## 1924          1 line-OreR       wt   30 0.551 0.500  0.179  12
## 1925          1 line-OreR       wt   30 0.522 0.469  0.179  10
## 1926          1 line-OreR       wt   30 0.547 0.474  0.177  11
## 1927          1 line-OreR       wt   30 0.540 0.480  0.174  10
## 1928          1 line-OreR       wt   30 0.553 0.488  0.172  10
## 1929          1 line-OreR       wt   30 0.538 0.479  0.171  11
## 1930          1 line-OreR       wt   30 0.546 0.482  0.167  10
## 1931          1 line-OreR       wt   30 0.473 0.417  0.161   9
## 1932          1 line-OreR       wt   30 0.551 0.473  0.157  10
## 1933          1  line-Sam       wt   30 0.552 0.492  0.202  12
## 1934          1  line-Sam       wt   30 0.547 0.503  0.197   9
## 1935          1  line-Sam       wt   30 0.572 0.509  0.195  10
## 1936          1  line-Sam       wt   30 0.539 0.479  0.195  10
## 1937          1  line-Sam       wt   30 0.549 0.505  0.194  11
## 1938          1  line-Sam       wt   30 0.515 0.500  0.193  12
## 1939          1  line-Sam       wt   30 0.531 0.495  0.191  12
## 1940          1  line-Sam       wt   30 0.540 0.475  0.191  10
## 1941          1  line-Sam       wt   30 0.543 0.506  0.190  10
## 1942          1  line-Sam       wt   30 0.551 0.490  0.189  10
## 1943          1  line-Sam       wt   30 0.542 0.454  0.186  11
## 1944          1  line-Sam       wt   30 0.553 0.461  0.185  11
## 1945          1  line-Sam       wt   30 0.555 0.490  0.183  12
## 1946          1  line-Sam       wt   30 0.515 0.495  0.183  10
## 1947          1  line-Sam       wt   30 0.507 0.461  0.182  10
## 1948          1  line-Sam       wt   30 0.536 0.492  0.178  10
## 1949          1  line-Sam       wt   30 0.538 0.468  0.177  11
## 1950          1  line-Sam       wt   30 0.511 0.473  0.175  11
## 1951          1  line-Sam       wt   30 0.545 0.460  0.175  11
## 1952          1  line-Sam       wt   30 0.527 0.465  0.171  10
## 1953          1  line-Sam       wt   30 0.517 0.463  0.167  11
## 1954          1    line-w       wt   30 0.531 0.478  0.207  13
## 1955          1    line-w       wt   30 0.543 0.495  0.202  10
## 1956          1    line-w       wt   30 0.536 0.472  0.201  11
## 1957          1    line-w       wt   30 0.525 0.464  0.198  12
## 1958          1    line-w       wt   30 0.539 0.489  0.192  10
## 1959          1    line-w       wt   30 0.548 0.474  0.192  11
## 1960          1    line-w       wt   30 0.503 0.465  0.190  10
## 1961          1    line-w       wt   30 0.543 0.485  0.190  11
## 1962          1    line-w       wt   30 0.541 0.487  0.186  10
## 1963          1    line-w       wt   30 0.544 0.496  0.185  11
## 1964          1    line-w       wt   30 0.482 0.430  0.185  10
## 1965          1    line-w       wt   30 0.516 0.473  0.184  10
## 1966          1    line-w       wt   30 0.537 0.484  0.183  10
## 1967          1    line-w       wt   30 0.544 0.495  0.181  12
## 1968          1    line-w       wt   30 0.507 0.480  0.181  10
## 1969          1    line-w       wt   30 0.528 0.485  0.178   9
## 1971          1    line-w       wt   30 0.545 0.486  0.174  12
## 1972          1    line-w       wt   30 0.542 0.504  0.167  12
## 1973          1    line-w       wt   30 0.511 0.469  0.165  11
## 7001          1   line-21       wt   25 0.496 0.463  0.195  11
## 16871         1    line-2       wt   30 0.535 0.494  0.186  12
## 4031          1    line-9      Dll   25 0.597 0.497  0.208  10
## 17271         1   line-21       wt   30 0.548 0.473  0.199  10
## 5041          1   line-12       wt   25 0.547 0.483  0.196  11
```



So what should we do? We can use the `unique()` to remove these duplicated rows if we know for certain that they do not belong in the data set (i.e. they are duplicated in error). 


```r
dll_data_unique <- unique(dll_data2)

dim(dll_data_unique)
```

```
## [1] 1918    8
```

```r
dim(dll_data2)
```

```
## [1] 1923    8
```

```r
dim(dll_data)
```

```
## [1] 1918    8
```

Let's just do a bit of clean-up.


```r
rm(dll_data_complete, dll_data_unique, dll_data2)
```

```
## Warning in rm(dll_data_complete, dll_data_unique, dll_data2): object
## 'dll_data_complete' not found
```

## Subsetting data sets (review)

In some sense this is a review of what we learned during the first week, but there are a few details that are worth considering.

Let's look at the data again:


```r
str(dll_data)
```

```
## 'data.frame':	1918 obs. of  8 variables:
##  $ replicate: int  1 1 1 1 1 1 1 1 1 1 ...
##  $ line     : Factor w/ 27 levels "line-1","line-11",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ genotype : Factor w/ 2 levels "Dll","wt": 1 1 1 1 1 1 1 1 1 1 ...
##  $ temp     : int  25 25 25 25 25 25 25 25 25 25 ...
##  $ femur    : num  0.59 0.55 0.588 0.596 0.577 ...
##  $ tibia    : num  0.499 0.501 0.488 0.502 0.499 ...
##  $ tarsus   : num  0.219 0.214 0.211 0.207 0.207 ...
##  $ SCT      : int  9 13 11 12 14 11 12 10 12 13 ...
##  - attr(*, "na.action")= 'omit' Named int  4 61 73 92 93 142 207 268 315 319 ...
##   ..- attr(*, "names")= chr  "4" "61" "73" "92" ...
```

As we can see there are two genotypes. Let's say we wanted to make a data frame (`dll_data_wt`) that was a subset with only the wild type (`wt`) data? How would you do this with the index?

### using the index

```r
dll_data_wt <- dll_data[dll_data$genotype == "wt",] #take out all rows where the genotype element equals wt
#we still have two levels, even though we've subsetted to have only wt (retaining meta-data)
```

Has this created the appropriate subset of data? How would you check?


### subsetting on factors and removing unused levels.

So that seems to match. We should only have one level now in `genotype`. Let's check.


```r
levels(dll_data_wt$genotype)
```

```
## [1] "Dll" "wt"
```

So what is going on? 

We know we only have the correct number of observations associated, but it still has the level `Dll` in the factor genotype, why? Because of the way it has been stored, each of these levels is still associated as part of `genotype`. Thankfully it is very easy to fix. Use the `droplevels()` to drop unused levels. 


```r
dll_data_wt <- droplevels(dll_data_wt) #This removes all unused variable levels
levels(dll_data_wt$genotype)
```

```
## [1] "wt"
```

### using the subset function

Of course the easiest alternative is to use the `subset` function we introduced a few weeks back. Please use this to make a data frame (`dll_data_Dll`) for the *Dll* factor level of genotype and check what has happened to the *wt* factor level. Try that here.



```r
dll_data_Dll <- subset(dll_data, genotype == "Dll" )
dim(dll_data_Dll)
```

```
## [1] 841   8
```

```r
with(dll_data, table(genotype))
```

```
## genotype
##  Dll   wt 
##  841 1077
```

```r
levels(dll_data_Dll$genotype)
```

```
## [1] "Dll" "wt"
```

```r
dll_data_Dll <- droplevels(dll_data_Dll)
```

`subset()` like using the index allows you to select specific columns as well. Create a new version of dll_data_Dll where the only columns that remain are line, genotype, temp and SCT.


```r
dll_data_Dll <- subset(dll_data_Dll, select = c(line, genotype, temp, SCT))
dim(dll_data_Dll)
```

```
## [1] 841   4
```

### the `%in%` matching operator 

`%in%` can be very useful for finding things in your data set to work on.

The `%in%` is a matching operator (also see `match()`) that returns a boolean (TRUE or FALSE). This can be really useful to find things. Go ahead and find all instances where a couple of the lines `c("line-Sam", "line-1")` are in the data set `dll_data$line`. I recommend first creating a dummy logical variable that can be used to filter via the index`[]`.


```r
matched_set <- c("line-Sam", "line-1") %in% dll_data$line
#matched_set<- dll_data$line %in% c("line-Sam", "line-1") 
#we're matching everything in the dataset to the table 
dll_data_new_subset <- dll_data[matched_set]
dim(dll_data_new_subset)
```

```
## [1] 1918    8
```


### Clean up before moving on.
let's remove the data frames we are not using.


```r
rm(dll_data_Dll, dll_data_wt, dll_data_new_subset, matched_set)
```

## Cleaning variable names
Let's take a closer look at the names of the different fly strains used.


```r
levels(dll_data$line)
```

```
##  [1] "line-1"    "line-11"   "line-12"   "line-13"   "line-15"  
##  [6] "line-16"   "line-17"   "line-18"   "line-19"   "line-2"   
## [11] "line-20"   "line-21"   "line-22"   "line-23"   "line-24"  
## [16] "line-26"   "line-27"   "line-3"    "line-4"    "line-6"   
## [21] "line-7"    "line-8"    "line-9"    "line-CanS" "line-OreR"
## [26] "line-Sam"  "line-w"
```

Some are numbers, some are letters. We decide, that we need to clean this up a bit. First off, we have no need to have "line-" as a prefix for each of these labels, and may make things confusing down the road. So we want to get rid of them. However we **don't** want to edit the spreadsheet, as it is most importantly bad scientific practice, and would be a lot of work. So how do we do it efficiently?

We actually have a couple of options.

First thing to keep in mind though is that `dll_data$line` is currently stored as an object of class `factor`. Why is this important?
-Factors are stored as an integer
-To do string manipulation, will have to coerce factors into characters

### `substr()`
Let's try to do some simple string manipulation. There are a few functions that could be useful here. Let's start with `substr()` (substring). **This will extract or replace substrings within a character vector**. So what do we need to do first to this variable (stored as a factor)


```r
line_str <- substr(as.character(dll_data$line), start = 6, stop = 1000000L) #large number as stop is essentially #"stop when it ends" start at 6, because that is the number that we want to extract
str(line_str)
```

```
##  chr [1:1918] "1" "1" "1" "1" "1" "1" "1" "1" "1" "1" "1" "1" "1" "1" ...
```

```r
head(line_str)
```

```
## [1] "1" "1" "1" "1" "1" "1"
```

Now we can use `substr()`. It is by position. Look up `?substr`, in the function you will want to use `stop = 1000000L`


```r
line_names <- ?

head(line_names)
tail(line_names)
```

```
## [1] "C:/Program Files/R/R-3.5.1/library/utils/help/head"
```

In this case, since the strings we want to keep start at different positions in the original string, we can't use an arbitrary string length (i.e. try `stop  = 7`). 


### `strsplit()`
An alternative way to do this is to use the fact that we (purposefully) used a delimiter character to seperate parts of the name. In this case we used the "-" to seperate the word "line" from the actual line name we car about. As such we can use a second function that can be very valuable, `strsplit()`. Use `strsplit` to create a new object splitting on the "-"


```r
line_names2 <- ?
head(line_names2)
```
Here it has created a list, where each list contains the 2 elements ("line" and the name of the line which we want). We are going to convert this list to a matrix so we can extract what we need.


```r
line_names2_mat <- ?

head(line_names2_mat)
```

And the second column contains what we want (line names) so we could make a new variable.


```r
dll_data$line_names <- ?
str(dll_data$line_names)
```

As an aside, in my lab I enforce an approach for imaging where all images, videos, or sequence files have a consistent (and identical within project) naming convention. For instance our wing images have a naming convention like initials_project_line_sex_genotype_rep. So for images it may be something like:

`ID_GBE_SAM_F_WT_01.tif`

Which would be ID = Ian Dworkin (initials), GBE = project name, SAM is line name, etc... This enables me to import the file names as variables and use `strsplit()` like we did above and automatically populate the variables for the experiment. No need to try to seperate things in a spreadsheet program!

### `gsub`

Many times however, there is no consistent delimiter like "-", nor can we assume the variable name we want is in a particular part of the string. So sometimes we need to utilize some simple regular expressions. `R` has a number of useful regular expression tools in the `base` library (`gsub`, `grep`, `grepl`, etc ). I suggest using `?grep` to take a deeper look. Here we will encounter just a few.

`sub()` and `gsub()` are functions that perform replacements for the first match or globally (g) for all matches respectively. If we go back to the names of the original lines, we will see they all have `line-` in common, so perhaps we can search and replace based on that pattern. This is quite easy as well. use `gsub` to extract just the line name without the first part of the string, "line-"


```r
str(line_str)
```

```
##  chr [1:1918] "1" "1" "1" "1" "1" "1" "1" "1" "1" "1" "1" "1" "1" "1" ...
```

```r
line_names3 <- gsub("line-", "", line_str)
head(line_names3)
```

```
## [1] "1" "1" "1" "1" "1" "1"
```

```r
tail(line_names3)
```

```
## [1] "w" "w" "w" "w" "w" "w"
```


### Some other things to help "clean" up names, etc..

You may have noticed that for the line names which are short words, some are lower case and some are upper case. It could be that this inconsistentcy make cause an issue down the road. So let's say we wanted to take `line_names3` and make all of the words lower case. How would we do that? `R` has some nice functions to help, namely `tolower()` (and `toupper()`) and the more general `chartr()` for character translation.


```r
line_names3 <- tolower(line_names3)

head(line_names3)
```

```
## [1] "1" "1" "1" "1" "1" "1"
```

```r
tail(line_names3, n = 25)
```

```
##  [1] "sam" "sam" "sam" "sam" "sam" "sam" "w"   "w"   "w"   "w"   "w"  
## [12] "w"   "w"   "w"   "w"   "w"   "w"   "w"   "w"   "w"   "w"   "w"  
## [23] "w"   "w"   "w"
```

Say we wanted to call the "sam" line by its proper name "Samarkand". How could we do this? with `sub`?


```r
line_names3 <- sub("sam", "SAMARKAND", line_names3)
#sub will replace first occurence in each line
```

#### Clean up before moving on.

```r
rm(line_names2, line_names3, line_names, line_names2_mat, line_str, new_rows)
```

### renaming variables, factors etc..

For this experiment, all flies were reared during their development at two temperatures, 25C and 30C. Currently this is kept as an integer. 


```r
str(dll_data$temp)
```

```
##  int [1:1918] 25 25 25 25 25 25 25 25 25 25 ...
```

However we might want to treat this as a factor. Also your PI has decided they want you to encode it at "HighTemp" and "LowTemp". So you have decided to create a new variable to do so. Not surprisingly, you have a few different ways to do this. The most obvious (and what you have done before) is to generate a factor. You can make the variable part of the data frame object, but I am going to keep it seperate for now.

Use the `factor()` function to generate a new variable `temp_as_factor` with factor labels "LowTemp", "HighTemp".


```r
temp_as_factor <- factor(dll_data$temp)
#<- with(dll_data, factor(temp, labels - c("LowTemp", "HighTemp")))
#without the labels, factors are called 25, 30 

str(temp_as_factor)
```

```
##  Factor w/ 2 levels "25","30": 1 1 1 1 1 1 1 1 1 1 ...
```

```r
head(temp_as_factor)
```

```
## [1] 25 25 25 25 25 25
## Levels: 25 30
```

```r
tail(temp_as_factor)
```

```
## [1] 30 30 30 30 30 30
## Levels: 25 30
```

### Using ifelse()

An alternative approach would be to use the `ifelse()` function. Create a new variable (that achieves the same thing) using `ifelse()`


```r
temp_as_factor2 <- ifelse(dll_data$temp == 25, "LowTemp", "HighTemp")
#ifelse(temp == 24, "LowTemp"", ifelse(temp == 30, "HighTemp", NA))
temp_as_factor2 <- factor(temp_as_factor2)
str(temp_as_factor2)
```

```
##  Factor w/ 2 levels "HighTemp","LowTemp": 2 2 2 2 2 2 2 2 2 2 ...
```

One issue with this is nested `if` statements, like `for` loops can get messy. Plus there is an internal limit on how much nesting can occur (I think a hierarchy of 5 or 6), so you want to take some care in using this approach.

## Sorting data frames

Sometimes you just want to sort your data frame. While this is rarely necessary for statistical analyses or plotting of the data, it is important to look at the raw data as a double (triple!!!) check that there are no typos, or issues you were not aware of. I would say in genomic analysis this is most important when you are looking at the summary of statistical analyses and you want to sort your output by genes with greatest effects or something else. While there is a `sort()` function in `R` this is not generally the way you want to approach it.


```r
head(sort(dll_data$SCT))
```

```
## [1] 6 7 7 7 7 7
```

```r
tail(sort(dll_data$SCT))
```

```
## [1] 18 18 18 18 20 22
```
This just sorts the vector, which does not help us much. Instead we are going to use the `order()` function which provides information on the index for each observation. So 


```r
head(order(dll_data$SCT))
```

```
## [1] 1375 1148 1268 1269 1325 1555
```

```r
tail(order(dll_data$SCT))
```

```
## [1] 1068 1283 1284 1336   60 1430
```
Provides the row numbers. We can use this to sort via the *index*. Use the `order()` to sort the dll_data frame into a new object (dll_data_sorted) based on dll_data$SCT.


```r
dll_data_sorted <- dll_data[order(dll_data$SCT),]
#Reorder dll_data based on the index created by order(dll_data$SCT)

head(dll_data_sorted)
```

```
##      replicate    line genotype temp femur tibia tarsus SCT
## 1409         1  line-8      Dll   30 0.512 0.468  0.169   6
## 1179         1 line-18      Dll   30 0.527 0.435  0.164   7
## 1300         1 line-23      Dll   30 0.508 0.454  0.170   7
## 1301         1 line-23      Dll   30 0.484 0.422  0.168   7
## 1358         1  line-4      Dll   30 0.541 0.470  0.158   7
## 1601         1 line-16       wt   30 0.559 0.508  0.179   7
##                                             line_names
## 1409 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 1179 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 1300 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 1301 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 1358 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 1601 C:/Program Files/R/R-3.5.1/library/utils/help/str
```

How would you sort on tibia?

```r
dll_data_sorted_t <- dll_data[order(dll_data$tibia),]
head(dll_data_sorted_t)
```

```
##      replicate    line genotype temp femur tibia tarsus SCT
## 20           1  line-1      Dll   25 0.451 0.342  0.165  16
## 1501         1  line-1       wt   30 0.476 0.373  0.150  10
## 1721         1 line-20       wt   30 0.423 0.383  0.147  10
## 1188         2 line-18      Dll   30 0.531 0.391  0.106  10
## 1883         1  line-7       wt   30 0.483 0.396  0.167  10
## 1362         1  line-4      Dll   30 0.483 0.396  0.151   9
##                                             line_names
## 20   C:/Program Files/R/R-3.5.1/library/utils/help/str
## 1501 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 1721 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 1188 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 1883 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 1362 C:/Program Files/R/R-3.5.1/library/utils/help/str
```

### sorting from highest to lowest

So this by default sorts from lowest to highest. Use the help function for `order` and sort the data from highest to lowest.


```r
dll_data_sorted <- dll_data[order(dll_data$tibia, decreasing = TRUE),]
head(dll_data_sorted)
```

```
##      replicate    line genotype temp femur tibia tarsus SCT
## 1905         1  line-9       wt   30 0.698 0.609  0.253  11
## 1907         1  line-9       wt   30 0.636 0.593  0.220  11
## 1906         1  line-9       wt   30 0.669 0.578  0.250  10
## 550          1 line-15       wt   25 0.585 0.564  0.163   9
## 264          2  line-3      Dll   25 0.559 0.562  0.230  14
## 1103         1 line-15      Dll   30 0.567 0.562  0.190  15
##                                             line_names
## 1905 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 1907 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 1906 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 550  C:/Program Files/R/R-3.5.1/library/utils/help/str
## 264  C:/Program Files/R/R-3.5.1/library/utils/help/str
## 1103 C:/Program Files/R/R-3.5.1/library/utils/help/str
```

### Sorting on multiple different columns. 

This is a natural extension of what we have already done. Sort first on *SCT*, then on *temp*


```r
dll_data_sorted <- dll_data[order(dll_data$SCT, dll_data$temp),]
#if there is a tie in the SCT column, it will then sort on temp
head(dll_data_sorted)
```

```
##      replicate    line genotype temp femur tibia tarsus SCT
## 1409         1  line-8      Dll   30 0.512 0.468  0.169   6
## 1179         1 line-18      Dll   30 0.527 0.435  0.164   7
## 1300         1 line-23      Dll   30 0.508 0.454  0.170   7
## 1301         1 line-23      Dll   30 0.484 0.422  0.168   7
## 1358         1  line-4      Dll   30 0.541 0.470  0.158   7
## 1601         1 line-16       wt   30 0.559 0.508  0.179   7
##                                             line_names
## 1409 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 1179 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 1300 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 1301 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 1358 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 1601 C:/Program Files/R/R-3.5.1/library/utils/help/str
```

```r
head(dll_data)
```

```
##   replicate   line genotype temp femur tibia tarsus SCT
## 1         1 line-1      Dll   25 0.590 0.499  0.219   9
## 2         1 line-1      Dll   25 0.550 0.501  0.214  13
## 3         1 line-1      Dll   25 0.588 0.488  0.211  11
## 5         1 line-1      Dll   25 0.596 0.502  0.207  12
## 6         1 line-1      Dll   25 0.577 0.499  0.207  14
## 7         1 line-1      Dll   25 0.618 0.494  0.204  11
##                                          line_names
## 1 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 2 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 3 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 5 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 6 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 7 C:/Program Files/R/R-3.5.1/library/utils/help/str
```

## Merging data frames

We often end up in a situation where we collect more data on the same set of samples (or in the same locales) that we need to include in our analyses. Again, we do not want to go back and edit the spreadsheets, but instead merge the data frames during the analysis. For instance, let's say we had this new data that related the elevation that each of the 27 lines were originally collected at, and we thought this may explain some important component of the variation in the traits (in this case size). So our collaborator collected this data:


```r
line_names <- dll_data$line
levels(line_names)
```

```
##  [1] "line-1"    "line-11"   "line-12"   "line-13"   "line-15"  
##  [6] "line-16"   "line-17"   "line-18"   "line-19"   "line-2"   
## [11] "line-20"   "line-21"   "line-22"   "line-23"   "line-24"  
## [16] "line-26"   "line-27"   "line-3"    "line-4"    "line-6"   
## [21] "line-7"    "line-8"    "line-9"    "line-CanS" "line-OreR"
## [26] "line-Sam"  "line-w"
```

```r
elevations <- c(100, 300, 270, 250, 500, 900, 500, 1100, 500,
                3000,500, 570, 150, 800, 600, 500, 1900, 100,
                300, 270, 250, 500, 900, 500, 1100, 500, 600)

MeanDayTimeTemp <- c(rnorm(27, mean = 20, sd = 5))
elevation_data <- data.frame(levels(line_names), 
                             elevations,
                             MeanDayTimeTemp)
```
So how do we combine these?

Using a for loop or ifelse is possible, but would be messy. There are two easier options. First using merge.

### `merge()`

The `merge()` function is designed for exactly this. It works on matching variable names (exactly is best) from variables with the same name, or where the matching pairs are specified.

In our case


```r
names(dll_data)
```

```
## [1] "replicate"  "line"       "genotype"   "temp"       "femur"     
## [6] "tibia"      "tarsus"     "SCT"        "line_names"
```

```r
names(elevation_data)
```

```
## [1] "levels.line_names." "elevations"         "MeanDayTimeTemp"
```

The easiest thing to do would be to rename the variable in `elevation_data` so that it matches that in `dll_data`


```r
names(elevation_data)[1] <- "line"
str(elevation_data)
```

```
## 'data.frame':	27 obs. of  3 variables:
##  $ line           : Factor w/ 27 levels "line-1","line-11",..: 1 2 3 4 5 6 7 8 9 10 ...
##  $ elevations     : num  100 300 270 250 500 900 500 1100 500 3000 ...
##  $ MeanDayTimeTemp: num  21.81 19.01 23.66 23.82 9.52 ...
```

Now we can merge the data


```r
merged_data <- merge(x = elevation_data, 
                     y = dll_data)
```

How can we check if it merged correctly?


```r
head(merged_data)
```

```
##     line elevations MeanDayTimeTemp replicate genotype temp femur tibia
## 1 line-1        100            21.8         1      Dll   25 0.590 0.499
## 2 line-1        100            21.8         1      Dll   25 0.550 0.501
## 3 line-1        100            21.8         1      Dll   25 0.588 0.488
## 4 line-1        100            21.8         1      Dll   25 0.596 0.502
## 5 line-1        100            21.8         1      Dll   25 0.577 0.499
## 6 line-1        100            21.8         1      Dll   25 0.618 0.494
##   tarsus SCT                                        line_names
## 1  0.219   9 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 2  0.214  13 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 3  0.211  11 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 4  0.207  12 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 5  0.207  14 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 6  0.204  11 C:/Program Files/R/R-3.5.1/library/utils/help/str
```

```r
tail(merged_data)
```

```
##        line elevations MeanDayTimeTemp replicate genotype temp femur tibia
## 1913 line-w        600            15.8         1       wt   30 0.544 0.495
## 1914 line-w        600            15.8         1       wt   30 0.507 0.480
## 1915 line-w        600            15.8         1       wt   30 0.528 0.485
## 1916 line-w        600            15.8         1       wt   30 0.545 0.486
## 1917 line-w        600            15.8         1       wt   30 0.542 0.504
## 1918 line-w        600            15.8         1       wt   30 0.511 0.469
##      tarsus SCT                                        line_names
## 1913  0.181  12 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 1914  0.181  10 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 1915  0.178   9 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 1916  0.174  12 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 1917  0.167  12 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 1918  0.165  11 C:/Program Files/R/R-3.5.1/library/utils/help/str
```

The `merge()` function has a great deal more flexibility, and it is worth looking at the examples in the help function! A word of warning with `merge()` and `reshape()` (below). The functions can result in some odd behaviour when there are non-unique rows which create conflicts. They can also stumble with missing data sometimes. So be on the look out for such things when you are using these functions.

### the power of the index

We can also use the index to do something similar.

Say we measured gene expression of a particular transcript for both genotypes *Dll* and *wt*. We wanted to include that. It would be very easy to use an ifelse, but we can also use the index


```r
geneXpression <- c(Dll=1121, wt = 2051)
merged_data$geneExp <- geneXpression[dll_data$genotype]

head(dll_data)
```

```
##   replicate   line genotype temp femur tibia tarsus SCT
## 1         1 line-1      Dll   25 0.590 0.499  0.219   9
## 2         1 line-1      Dll   25 0.550 0.501  0.214  13
## 3         1 line-1      Dll   25 0.588 0.488  0.211  11
## 5         1 line-1      Dll   25 0.596 0.502  0.207  12
## 6         1 line-1      Dll   25 0.577 0.499  0.207  14
## 7         1 line-1      Dll   25 0.618 0.494  0.204  11
##                                          line_names
## 1 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 2 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 3 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 5 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 6 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 7 C:/Program Files/R/R-3.5.1/library/utils/help/str
```

Make a new variable based on genotype for `geneY = c(200, 500)`.

#### Clean up for next part.

```r
rm(elevation_data, elevations, MeanDayTimeTemp, temp_as_factor, dll_data_sorted, geneXpression, temp_as_factor2, line_names, stuff, merged_data)
```

```
## Warning in rm(elevation_data, elevations, MeanDayTimeTemp,
## temp_as_factor, : object 'stuff' not found
```

## reshaping data

There are often times when your data is in a format that is not appropriate for the analyses you need to do. In particular sometimes you need each variable (say each gene whose expression you are monitoring) in its own column (so called *wide* format). Sometimes it is best to have a single column for gene expression, with a second column indexing which gene the measure of expression is for. i.e. all expression data is in a single column, and thus each row has a single measure, but the number of rows will be equal to the number of samples multiplied by the number of genes (lots of rows!!). This is called the *long* format.

This is such a common operation that `R` has a function `reshape()` to do this. Unfortunately (but perhaps well deserved), the arguments and argument names for the `reshape()` function are not so intuitive. Thus while I give an example of it below, this is one of those cases that using an additional package (like `reshape2` or `tidyr`) may be a worthwhile investment in time. Indeed `gather` and `spread` are really straightforward in *tidyr*

### `reshape()` function in `R`
Let's take a quick look at our data again:

```r
head(dll_data)
```

```
##   replicate   line genotype temp femur tibia tarsus SCT
## 1         1 line-1      Dll   25 0.590 0.499  0.219   9
## 2         1 line-1      Dll   25 0.550 0.501  0.214  13
## 3         1 line-1      Dll   25 0.588 0.488  0.211  11
## 5         1 line-1      Dll   25 0.596 0.502  0.207  12
## 6         1 line-1      Dll   25 0.577 0.499  0.207  14
## 7         1 line-1      Dll   25 0.618 0.494  0.204  11
##                                          line_names
## 1 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 2 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 3 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 5 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 6 C:/Program Files/R/R-3.5.1/library/utils/help/str
## 7 C:/Program Files/R/R-3.5.1/library/utils/help/str
```

It is clear that we have three variables (femur, tibia and tarsus) which are all measures of size (3 segments of the leg). While we currently have these in the *wide* format, for some analysis (such as in mixed models) we may wish to put them in a *long* format. 

First let me show you the code (and that it worked), and then we will go through the various arguments to make sense of what we have done.

```r
long_data <- reshape(dll_data,
                     varying = list(names(dll_data)[5:7]),
                     direction = "long",
                     ids = row.names(dll_data),
                     times = c("femur", "tibia", "tarsus"),
                     timevar = "leg_segments",
                     v.names = "length")
```

Let's check that this worked. How might we do this? What do we expect in terms of number of rows in `long_data`?


```r
# It should have 3 times the number of rows as our original data
(3*nrow(dll_data)) == nrow(long_data)
```

```
## [1] TRUE
```

```r
head(long_data, n = 10)
```

```
##          replicate   line genotype temp SCT
## 1.femur          1 line-1      Dll   25   9
## 2.femur          1 line-1      Dll   25  13
## 3.femur          1 line-1      Dll   25  11
## 5.femur          1 line-1      Dll   25  12
## 6.femur          1 line-1      Dll   25  14
## 7.femur          1 line-1      Dll   25  11
## 8.femur          1 line-1      Dll   25  12
## 9.femur          1 line-1      Dll   25  10
## 10.femur         1 line-1      Dll   25  12
## 11.femur         1 line-1      Dll   25  13
##                                                 line_names leg_segments
## 1.femur  C:/Program Files/R/R-3.5.1/library/utils/help/str        femur
## 2.femur  C:/Program Files/R/R-3.5.1/library/utils/help/str        femur
## 3.femur  C:/Program Files/R/R-3.5.1/library/utils/help/str        femur
## 5.femur  C:/Program Files/R/R-3.5.1/library/utils/help/str        femur
## 6.femur  C:/Program Files/R/R-3.5.1/library/utils/help/str        femur
## 7.femur  C:/Program Files/R/R-3.5.1/library/utils/help/str        femur
## 8.femur  C:/Program Files/R/R-3.5.1/library/utils/help/str        femur
## 9.femur  C:/Program Files/R/R-3.5.1/library/utils/help/str        femur
## 10.femur C:/Program Files/R/R-3.5.1/library/utils/help/str        femur
## 11.femur C:/Program Files/R/R-3.5.1/library/utils/help/str        femur
##          length id
## 1.femur   0.590  1
## 2.femur   0.550  2
## 3.femur   0.588  3
## 5.femur   0.596  5
## 6.femur   0.577  6
## 7.femur   0.618  7
## 8.femur   0.582  8
## 9.femur   0.572  9
## 10.femur  0.568 10
## 11.femur  0.555 11
```

```r
tail(long_data, n = 10)
```

```
##             replicate   line genotype temp SCT
## 1963.tarsus         1 line-w       wt   30  11
## 1964.tarsus         1 line-w       wt   30  10
## 1965.tarsus         1 line-w       wt   30  10
## 1966.tarsus         1 line-w       wt   30  10
## 1967.tarsus         1 line-w       wt   30  12
## 1968.tarsus         1 line-w       wt   30  10
## 1969.tarsus         1 line-w       wt   30   9
## 1971.tarsus         1 line-w       wt   30  12
## 1972.tarsus         1 line-w       wt   30  12
## 1973.tarsus         1 line-w       wt   30  11
##                                                    line_names leg_segments
## 1963.tarsus C:/Program Files/R/R-3.5.1/library/utils/help/str       tarsus
## 1964.tarsus C:/Program Files/R/R-3.5.1/library/utils/help/str       tarsus
## 1965.tarsus C:/Program Files/R/R-3.5.1/library/utils/help/str       tarsus
## 1966.tarsus C:/Program Files/R/R-3.5.1/library/utils/help/str       tarsus
## 1967.tarsus C:/Program Files/R/R-3.5.1/library/utils/help/str       tarsus
## 1968.tarsus C:/Program Files/R/R-3.5.1/library/utils/help/str       tarsus
## 1969.tarsus C:/Program Files/R/R-3.5.1/library/utils/help/str       tarsus
## 1971.tarsus C:/Program Files/R/R-3.5.1/library/utils/help/str       tarsus
## 1972.tarsus C:/Program Files/R/R-3.5.1/library/utils/help/str       tarsus
## 1973.tarsus C:/Program Files/R/R-3.5.1/library/utils/help/str       tarsus
##             length   id
## 1963.tarsus  0.185 1963
## 1964.tarsus  0.185 1964
## 1965.tarsus  0.184 1965
## 1966.tarsus  0.183 1966
## 1967.tarsus  0.181 1967
## 1968.tarsus  0.181 1968
## 1969.tarsus  0.178 1969
## 1971.tarsus  0.174 1971
## 1972.tarsus  0.167 1972
## 1973.tarsus  0.165 1973
```



### some other approaches to using `reshape()`

Sometimes we actually have unique subject identifiers already in the data frame, and we don't need to just use row numbers. These unique identifiers can be a combination of several variables if it generates a unique combination for each row (for data currently in wide format). See the section below on combining names. In turns out that even if we combined line, genotype, temp we would not have unique names, so that strategy does not work here. So we can make a fake identifer

```r
dll_data$subject <- as.character(1:nrow(dll_data))
```


```r
long_data2 <- reshape(dll_data,
                     varying = list(names(dll_data)[5:7]),
                     direction = "long",
                     idvar = "subject",
                     #ids = row.names(dll_data),
                     times = c("femur", "tibia", "tarsus"),
                     timevar = "leg_segments",
                     v.names = "length"
                     )
```

Which we can check again


```r
3*nrow(dll_data) == nrow(long_data2)
```

```
## [1] TRUE
```

```r
head(long_data2)
```

```
##         replicate   line genotype temp SCT
## 1.femur         1 line-1      Dll   25   9
## 2.femur         1 line-1      Dll   25  13
## 3.femur         1 line-1      Dll   25  11
## 4.femur         1 line-1      Dll   25  12
## 5.femur         1 line-1      Dll   25  14
## 6.femur         1 line-1      Dll   25  11
##                                                line_names subject
## 1.femur C:/Program Files/R/R-3.5.1/library/utils/help/str       1
## 2.femur C:/Program Files/R/R-3.5.1/library/utils/help/str       2
## 3.femur C:/Program Files/R/R-3.5.1/library/utils/help/str       3
## 4.femur C:/Program Files/R/R-3.5.1/library/utils/help/str       4
## 5.femur C:/Program Files/R/R-3.5.1/library/utils/help/str       5
## 6.femur C:/Program Files/R/R-3.5.1/library/utils/help/str       6
##         leg_segments length
## 1.femur        femur  0.590
## 2.femur        femur  0.550
## 3.femur        femur  0.588
## 4.femur        femur  0.596
## 5.femur        femur  0.577
## 6.femur        femur  0.618
```

In this case, this is really similar to the approach above as we used the same fundamental IDs.

## Combining variables together.
Sometimes you need to combine several variables together. For instance, we may need to look at combinations of both *genotype* and *temp*


```r
str(dll_data)
```

```
## 'data.frame':	1918 obs. of  10 variables:
##  $ replicate : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ line      : Factor w/ 27 levels "line-1","line-11",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ genotype  : Factor w/ 2 levels "Dll","wt": 1 1 1 1 1 1 1 1 1 1 ...
##  $ temp      : int  25 25 25 25 25 25 25 25 25 25 ...
##  $ femur     : num  0.59 0.55 0.588 0.596 0.577 ...
##  $ tibia     : num  0.499 0.501 0.488 0.502 0.499 ...
##  $ tarsus    : num  0.219 0.214 0.211 0.207 0.207 ...
##  $ SCT       : int  9 13 11 12 14 11 12 10 12 13 ...
##  $ line_names: chr  "C:/Program Files/R/R-3.5.1/library/utils/help/str" "C:/Program Files/R/R-3.5.1/library/utils/help/str" "C:/Program Files/R/R-3.5.1/library/utils/help/str" "C:/Program Files/R/R-3.5.1/library/utils/help/str" ...
##  $ subject   : chr  "1" "2" "3" "4" ...
##  - attr(*, "na.action")= 'omit' Named int  4 61 73 92 93 142 207 268 315 319 ...
##   ..- attr(*, "names")= chr  "4" "61" "73" "92" ...
```

What if we wanted to combine these two variables together into a meta-variable? There are a few good options. One is to use paste of course


```r
meta_variable <- with(dll_data,
                      paste(genotype, temp, 
                            sep = ":"))

head(meta_variable, n = 20)
```

```
##  [1] "Dll:25" "Dll:25" "Dll:25" "Dll:25" "Dll:25" "Dll:25" "Dll:25"
##  [8] "Dll:25" "Dll:25" "Dll:25" "Dll:25" "Dll:25" "Dll:25" "Dll:25"
## [15] "Dll:25" "Dll:25" "Dll:25" "Dll:25" "Dll:25" "Dll:25"
```

```r
str(meta_variable)
```

```
##  chr [1:1918] "Dll:25" "Dll:25" "Dll:25" "Dll:25" "Dll:25" "Dll:25" ...
```
Note: you get to choose the character to use as a seperator. ":", "." and "_" are very common!

The only problem is this is now a character vector not a factor, although this is easily fixed.


```r
meta_variable <- as.factor(meta_variable)
str(meta_variable)
```

```
##  Factor w/ 4 levels "Dll:25","Dll:30",..: 1 1 1 1 1 1 1 1 1 1 ...
```
Great!

The other pretty easy way, especially for factors is to use the `interaction` function.


```r
meta_variable2 <- with(dll_data,
     interaction(as.factor(temp), genotype,
                 drop = TRUE,
                 sep = ":"))

head(meta_variable2, n = 20)
```

```
##  [1] 25:Dll 25:Dll 25:Dll 25:Dll 25:Dll 25:Dll 25:Dll 25:Dll 25:Dll 25:Dll
## [11] 25:Dll 25:Dll 25:Dll 25:Dll 25:Dll 25:Dll 25:Dll 25:Dll 25:Dll 25:Dll
## Levels: 25:Dll 30:Dll 25:wt 30:wt
```

```r
str(meta_variable2)
```

```
##  Factor w/ 4 levels "25:Dll","30:Dll",..: 1 1 1 1 1 1 1 1 1 1 ...
```

Please note the `drop = TRUE` flag. You need this if there are combinations of factors that do not exist in the data, and it is almost always the safer choice unless you need all possible combinations (including those that don't occur).

## Please ignore
In class we are going to go through just a few activities that are important. combining levels in a factor (grepl factor, ), transform, numeric to factor (cut),   creating a dataset from multiple external files (RNAseq example)....
