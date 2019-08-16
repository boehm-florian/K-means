---
title:   "Script for generating dummy data"
author:  "Dietmar H. and Fabian P."
date:    "Aug 2019"
---



# Introduction
Still to fill

# General Settings
The pacman package is an R package management tool for loading and installing
packages if necessary. The following packages are used for the data generation: 
 
- **data.table:** For filtering, grouping and transforming the data as well
as fast read and write opterations. This package is particularly suitable for
the fast processing of large amounts of data.
- **dplyr:** This package is a grammar of data manipulation, providing a consistent
set of verbs that help you solve the most common data manipulation challenges.
We need it especially for the use of the pipe operator.
- **rmarkdown:** R Markdown provides an authoring framework for data science.
One can use a single R Markdown file to save and execute code and generate high
quality reports that can be shared with an audience.
- **ggplot2:** ggplot2 is a system for declaratively creating graphics, based
on The Grammar of Graphics. 
We suppress the warning for the *pacman* library because the message that the 
package was built under a different R-versions should not bother the user by
beeing displayed on the console in red. Then the required packages are loaded
and installed if necessary. 



```r
suppressWarnings(if (!require("pacman")) install.packages("pacman"))
pacman::p_load(data.table, dplyr, rmarkdown, ggplot2)
```

For the analysis of the data functions are used which work with random numbers.
To make the results reproducible the random number generator is initialized 
with an arbitrary number (here 100) to ensure that the random numbers are the
same at every program run. 


```r
set.seed(100)
```

# Data preparation
The data is read in and a first check is performed to evaluate the structure 
of the data. It turns out that the data set contains 1200 observations with 4
variables each, namely sex, age, sum assured and ID. The variable ID is a 
consecutive number that uniquely identifies each contract. 


```r
raw_data <- fread("./data/insurance_portfolio.txt")
raw_data %>% str()
```

```
## Classes 'data.table' and 'data.frame':	1200 obs. of  4 variables:
##  $ sex        : chr  "m" "m" "m" "f" ...
##  $ age        : int  72 22 28 30 73 37 57 49 48 44 ...
##  $ sum_assured: int  43000 38601 11589 8962 8942 28284 10702 8335 43964 7414 ...
##  $ ID         : int  1 2 3 4 5 6 7 8 9 10 ...
##  - attr(*, ".internal.selfref")=<externalptr>
```

Since there can be only 2 different values for the variable sex, namely male 
and female, this variable is defined as a factor. 


```r
raw_data[, sex := as.factor(sex)]
```

A more detailed analysis of the input data using the summary function shows that:
1. The portfolio consists of 600 female and 600 male policyholders.
2. The age is between 8 and 87 years, which suggests that there are no 
implausible data points.
3. There are 10 policyholders whose age is indicated with NA. This indicates 
missing or incomplete data.
4. The sums insured are between 5119 and 61485. If one looks at the quantiles,
the minimum and maximum values, one can see that at first glance there are no
extreme outliers. Another good way to analyze the portfolio is the visual 
inspection that is performed in one of the next steps.


```r
raw_data %>% summary()
```

```
##  sex          age         sum_assured          ID        
##  f:600   Min.   : 8.00   Min.   : 5119   Min.   :   1.0  
##  m:600   1st Qu.:28.00   1st Qu.: 9498   1st Qu.: 300.8  
##          Median :46.00   Median :17183   Median : 600.5  
##          Mean   :47.42   Mean   :23625   Mean   : 600.5  
##          3rd Qu.:68.00   3rd Qu.:37735   3rd Qu.: 900.2  
##          Max.   :87.00   Max.   :61485   Max.   :1200.0  
##          NA's   :10
```

As already stated, there are 10 data points that have no values for age. These
10 data points will be removed for further analysis in order to work with a
cleaned dataset. 


```r
clean_data <- raw_data[!is.na(age),]
```

## Visual inspection
After a first overview of the data has already been given, the visual inspection
is carried out. For this purpose, the sum insured is plotted against age in a
point plot. The previous assumption that there are no outliers can be confirmed 
visually and new knowledge about the data structure can be gained. A total of
4 different clusters can be identified as seen in the plot below. A cluster shows
policies with a sum insured of about 10.000 and ages in the entire range of 
observations. Another cluster are those policies that have an insurance sum
of approximately 30.000 to 50.000 and an age of approximately 22. The next 
cluster covers policies with an sum insured in the range of 20.000 up to 
50.000 and an age of 40 to 50.The last cluster is a little more inhomogeneous
than the others in terms of both age and sum insured. It covers ages from 60 
to 80 and sums insured from 25.000 to 60.000.


```r
ggplot(data = clean_data, aes(x = age, y = sum_assured)) + geom_point()
```

![](k-means_files/figure-html/plot-all-1.png)<!-- --> 

In the previous plot, all data points were viewed in a single graph. In the 
next step a point plot is created, separated into men and women. It is important
to note that the scaling of the two plots is the same. It becomes apparent that
there are now 6 clusters for both men and women. The single cluster with
an insured sum of approximately 10000 is now divided into 3 different age groups. 


```r
ggplot(data = clean_data, aes(x = age, y = sum_assured)) +
  geom_point() +
  facet_grid(cols = vars(sex))
```

![](k-means_files/figure-html/plot-facets-1.png)<!-- --> 

The same information can also be obtained by coloring the data points according
to their sex with men displayed in blue and women in red.


```r
ggplot(data = clean_data, aes(x = age, y = sum_assured, color = sex)) +
  geom_point()
```

![](k-means_files/figure-html/plot-colored-1.png)<!-- --> 

# K-means

---
title: "k-means.R"
author: "f_pribahsnikf"
date: "2019-08-16"
---
