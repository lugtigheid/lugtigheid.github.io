---
title       : Data wrangling in R
subtitle    : May 11 2016
author      : Arthur Lugtigheid PhD, Data scientist
job         : Data science and visualisation team (Analysis directorate)
framework   : io2012        # {io2012, html5slides, shower, dzslides, ...}
highlighter : highlight.js  # {highlight.js, prettify, highlight}
hitheme     : tomorrow      # 
widgets     : []            # {mathjax, quiz, bootstrap}
mode        : selfcontained # {standalone, draft}
knit        : slidify::knit2slides
--- 

## Before we start...

* Slides are on <a href="http://cbas/lugtigha">http://cbas/lugtigha</a>

* I am working on some hand outs you can print, but the slides will always be available on above URL in CBAS. 

* Slides advance with arrow keys or by clicking on one of the sides of the presentation. 

* You can get an overview of slides by pressing 'o', go full screen with 'r'. 

* These slides were made in R using Markdown and Slidify. 

---

## Before we start...

Data frames (data.frames) are used for storing data tables; it is a list of vectors of equal length (very similar to an Excel sheet). Data frames are the single most used data type in R.  


```r
tax.mp <- data.frame(
    "sex" = c('M', 'M', 'M', 'M', 'F', 'F'),
    "Party" = c('Con', 'Con', 'Lab', 'Con', 'SNP', 'Lab'), 
    "Name" = c('Cameron', 'Osborne', 'Corbyn', 'Johnson', 'Sturgeon', 'Flint'),
    "Income" = c(200307, 198738, 70795 , 612583, 104000, 58724),
    "Tax" = c(75898, 72210, 18912, 276505, 31000, 12965)
    )
```


```
##   sex Party     Name Income    Tax
## 1   M   Con  Cameron 200307  75898
## 2   M   Con  Osborne 198738  72210
## 3   M   Lab   Corbyn  70795  18912
## 4   M   Con  Johnson 612583 276505
## 5   F   SNP Sturgeon 104000  31000
## 6   F   Lab    Flint  58724  12965
```

--- .quiz

## Quiz

Take 2 minutes to figure out what do these commands do: 


```r
tax.mp[1,2]

tax.con <- tax.mp$Party == 'Con'

mean(tax.mp$Income[tax.con])

with(tax.mp, sum(Income - Tax))

subset(tax.mp, Party == 'Lab', select = c('Name', 'Income'))
```

<small style="font-size: 50%">Answers on the next slide...</small>


--- .quiz



```r
tax.mp[2,4] # it selects the fourth column in the second row of tax.mp
```

```
## [1] 198738
```

```r
tax.con <- tax.mp$Party == 'Con'  # this creates a boolean vector where all conservatives are TRUE

mean(tax.mp$Income[tax.con])  # this calculates the mean income of the conservatives
```

```
## [1] 337209.3
```

```r
with(tax.mp, sum(Income - Tax))   # this calculates the difference between income and taxes for everyone
```

```
## [1] 757657
```

```r
subset(tax.mp, Party == 'Lab', select = c('Name', 'Income'))  # this selects the name and income of labour members
```

```
##     Name Income
## 3 Corbyn  70795
## 6  Flint  58724
```

--- .hadley

## Introducing... "The HadleyVerse"

_*TidyR*_: for tidying data frames

* gather()
* spread()
* separate()

*dplyR*: for manipulating data frames

* filter()
* arrange()
* select()
* mutate()

*ReadR*: for reading in data

* read_csv()

---

## The %>% (pipe) operator

In a pipe, the result of the left hand statement is handed over to the statement on the right (much like the UNIX pipe command, e.g. ls -l | grep key | less):


```r
data %>% function %>% function
```


```r
head(iris,2)
```

```
##   Sepal.Length Sepal.Width Petal.Length Petal.Width Species
## 1          5.1         3.5          1.4         0.2  setosa
## 2          4.9         3.0          1.4         0.2  setosa
```

```r
iris %>% head(2)
```

```
##   Sepal.Length Sepal.Width Petal.Length Petal.Width Species
## 1          5.1         3.5          1.4         0.2  setosa
## 2          4.9         3.0          1.4         0.2  setosa
```

--- .intermezzo

## Tidying data

In this section, we are going to learn how to create tidy data frames:
* gather()
* separate()
* spread()


--- .tidy

## What are tidy data?

Tidy datasets are easy to manipulate, model and visualise, and have a specific structure:

* each variable is a column, 
* each observation is a row, and 
* each type of observational unit is a table

<img src="assets/img/tidy_data.jpg" />

Tidy data provides a standard to move data around between tools. 



---

## What are tidy data?

Let's look at it the other way around - what are examples of messy data?

* Column headers are values, not variable names
* Multiple variables are stored in one column
* Variables are stored in both rows and columns
* Multiple types of observations stored in the same table
* A single observational unit is stored across multiple tables

How can we deal with each of these scenarios? (focus on first 3)

---

## Dealing with messy data

*Column headers are values, not variable names*


```
##   Department <22k 22-32k 32-42k >42k
## 1        BIS  998    370    264  139
## 2        DWP  474    470    443  806
## 3       DCLG  246    142    856  454
## 4        DFE  239     95    754  617
## 5        MOJ  438    136    702  414
```

Three variables - Department, income and frequency - but 5 columns. To tidy it, we need to **gather** the non-variable columns into a two-column key-value pair. 

You may know this as converting a wide dataset into a long dataset.

---

## Dealing with messy data

*Column headers are values, not variable names*


```r
gather(data=messy, key=income, value=frequency, -Department)
```

```r
messy %>% gather(income, frequency, -Department) %>% head(7)
```

```
##   Department income frequency
## 1        BIS   <22k       998
## 2        DWP   <22k       474
## 3       DCLG   <22k       246
## 4        DFE   <22k       239
## 5        MOJ   <22k       438
## 6        BIS 22-32k       370
## 7        DWP 22-32k       470
```

This form is tidy because each column represents a variable and each row represents an observation, in this case a demographic unit corresponding to a combination of department and income.

---

## Dealing with messy data

*Multiple variables stored in one column*


```
##    University M18-25 M25-35 F18-25 F25-35
## 1  Birmingham    920    115    815    116
## 2   Cambridge    662    184    679    269
## 3      Oxford    777    100    900    272
## 4 Southampton    528    102    840    285
## 5     Bristol    639    100    872    109
```

Four variables: University, Sex, Age range and frequency.

To tidy it, we need to first **gather** the non-variable columns into key - value pairs, then **seperate** the key (e.g. M18-25) into two variables.

---

## Dealing with messy data
*Multiple variables stored in one column (cont.)*


```r
messy %>% gather(key, value, -University) %>% head(10)
```

```
##     University    key value
## 1   Birmingham M18-25   920
## 2    Cambridge M18-25   662
## 3       Oxford M18-25   777
## 4  Southampton M18-25   528
## 5      Bristol M18-25   639
## 6   Birmingham M25-35   115
## 7    Cambridge M25-35   184
## 8       Oxford M25-35   100
## 9  Southampton M25-35   102
## 10     Bristol M25-35   100
```

---

## Dealing with messy data
*Multiple variables stored in one column (cont.)*


```r
messy %>% gather(key, value, -University) %>% separate(key, c('Sex','Age'), 1) %>% head(12)
```

```
##     University Sex   Age value
## 1   Birmingham   M 18-25   920
## 2    Cambridge   M 18-25   662
## 3       Oxford   M 18-25   777
## 4  Southampton   M 18-25   528
## 5      Bristol   M 18-25   639
## 6   Birmingham   M 25-35   115
## 7    Cambridge   M 25-35   184
## 8       Oxford   M 25-35   100
## 9  Southampton   M 25-35   102
## 10     Bristol   M 25-35   100
## 11  Birmingham   F 18-25   815
## 12   Cambridge   F 18-25   679
```

---

## Dealing with messy data
*Variables are stored in both rows and columns*


```
##   year month range  d1  d2  d3  d4  d5  d6  d7
## 1 2016     1   Min 996 646 665 677 801 750 509
## 2 2016     2   Min 608 556 922 823 847 641 851
## 3 2016     3   Min 968 740 870 686 993 891 995
## 4 2016     1   Max 724 871 889 979 584 560 859
## 5 2016     2   Max 582 704 786 641 771 628 692
## 6 2016     3   Max 883 713 833 500 989 794 988
```

This data set has variables in individual columns (id, year, month) and across columns (weekday) but also across rows (range). 

---

## Dealing with messy data
*Variables are stored in both rows and columns (cont.)*

To tidy this, we first gather the weekday columns: 


```r
messy %>% 
  gather(day, value, d1:d7) %>% 
  head(8)
```

```
##   year month minmax day value
## 1 2016     1    Min  d1   996
## 2 2016     2    Min  d1   608
## 3 2016     3    Min  d1   968
## 4 2016     1    Max  d1   724
## 5 2016     2    Max  d1   582
## 6 2016     3    Max  d1   883
## 7 2016     1    Min  d2   646
## 8 2016     2    Min  d2   556
```

Mostly tidy, but range is still not a variable (it stores names of variables).

---

## Dealing with messy data
*Variables are stored in both rows and columns (cont.)*

Fixing this requires the spread function: 


```r
messy %>% 
  gather(day, value, d1:d7) %>% 
  spread(range, value) %>% 
  head(8)
```

```
##   year month day Max Min
## 1 2016     1  d1 724 996
## 2 2016     1  d2 871 646
## 3 2016     1  d3 889 665
## 4 2016     1  d4 979 677
## 5 2016     1  d5 584 801
## 6 2016     1  d6 560 750
## 7 2016     1  d7 859 509
## 8 2016     2  d1 582 608
```

--- .quiz

## Quiz

Consider these two data sets: which is the tidy data set?


```
##   id        x         y
## 1  1 6.244945 0.9122943
## 2  2 6.377398 3.2430906
## 3  3 5.097869 1.7203113
## 4  4 7.901603 5.3240741
```

```
##   id variable     value
## 1  1        x 6.2449450
## 2  2        x 6.3773985
## 3  3        x 5.0978685
## 4  4        x 7.9016026
## 5  1        y 0.9122943
## 6  2        y 3.2430906
## 7  3        y 1.7203113
## 8  4        y 5.3240741
```

--- .quiz

## Quiz

Consider these data:


```r
stocks %>% head(5)
```

```
##         time          X          Y          Z
## 1 2009-01-01  1.4997698  3.2066752 -3.8921020
## 2 2009-01-02  1.2464219 -0.9456069  0.4057550
## 3 2009-01-03 -0.2634953 -1.6370437  3.5708525
## 4 2009-01-04 -0.2069673  0.8187246  0.4574388
## 5 2009-01-05  2.0559941 -0.7150004 -3.9079440
```

What does this command do?


```r
stocks %>% gather(key, value, -time) %>% spread(key, value) %>% head(5)
```



--- .intermezzo

## Data manipulation

In this section, we are going to learn how to manipulate data frames using "Split - Apply - Combine" types of analyses, e.g.:

* filter() and select()
* arrange()
* mutate() and transmute()
* summarise()

---

## Let's load some data


```r
# install the package - all flights departing from NYC in 2013
install.packages('nycflights13')

# load the packages
library(nycflights13)
library(dplyr)

# we can now use the 'flights' data.frame as normal:
summary(flights)

# convert to a 'tibble' -  A tbl is just a special kind of data.frame. They make your data easier to look at, but 
# also easier to work with; a tbl is straightforwardly derived from a data.frame structure using tbl_df().

# The tbl format changes how R displays your data, but it does not change the data's underlying data structure. A
# tbl inherits the original class of its input, in this case, a data.frame. This means that you can still 
# manipulate the tbl as if it were a data.frame; you can do anything with the flights tbl that you could do with 
# the flights data.frame.
flights <- tbl_df(flights)

# let's have a better look at it
glimpse(flights)
```

---


```r
# let's have a better look at it
glimpse(flights)
```

```
## Observations: 336,776
## Variables: 19
## $ year           (int) 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2013,...
## $ month          (int) 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,...
## $ day            (int) 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,...
## $ dep_time       (int) 517, 533, 542, 544, 554, 554, 555, 557, 557, 55...
## $ sched_dep_time (int) 515, 529, 540, 545, 600, 558, 600, 600, 600, 60...
## $ dep_delay      (dbl) 2, 4, 2, -1, -6, -4, -5, -3, -3, -2, -2, -2, -2...
## $ arr_time       (int) 830, 850, 923, 1004, 812, 740, 913, 709, 838, 7...
## $ sched_arr_time (int) 819, 830, 850, 1022, 837, 728, 854, 723, 846, 7...
## $ arr_delay      (dbl) 11, 20, 33, -18, -25, 12, 19, -14, -8, 8, -2, -...
## $ carrier        (chr) "UA", "UA", "AA", "B6", "DL", "UA", "B6", "EV",...
## $ flight         (int) 1545, 1714, 1141, 725, 461, 1696, 507, 5708, 79...
## $ tailnum        (chr) "N14228", "N24211", "N619AA", "N804JB", "N668DN...
## $ origin         (chr) "EWR", "LGA", "JFK", "JFK", "LGA", "EWR", "EWR"...
## $ dest           (chr) "IAH", "IAH", "MIA", "BQN", "ATL", "ORD", "FLL"...
## $ air_time       (dbl) 227, 227, 160, 183, 116, 150, 158, 53, 140, 138...
## $ distance       (dbl) 1400, 1416, 1089, 1576, 762, 719, 1065, 229, 94...
## $ hour           (dbl) 5, 5, 5, 5, 6, 5, 6, 6, 6, 6, 6, 6, 6, 6, 6, 5,...
## $ minute         (dbl) 15, 29, 40, 45, 0, 58, 0, 0, 0, 0, 0, 0, 0, 0, ...
## $ time_hour      (time) 2013-01-01 05:00:00, 2013-01-01 05:00:00, 2013...
```

---

## select()

The select() function is the same as the base subset() function, but also allows fancy use of helper functions like starts_with(), ends_with(), matches() and contains():

What might the following R commands do?


```r
select(flights, month)

flights %>% select(month)

flights %>% select(-day)

flights %>% select(1:3)

flights %>% select(flight:dest)
```

More examples on the next couple of slides...

---

## select()

Select all columns to do with delays:


```r
flights %>% select(ends_with('delay'))
```

```
## Source: local data frame [336,776 x 2]
## 
##    dep_delay arr_delay
##        (dbl)     (dbl)
## 1          2        11
## 2          4        20
## 3          2        33
## 4         -1       -18
## 5         -6       -25
## 6         -4        12
## 7         -5        19
## 8         -3       -14
## 9         -3        -8
## 10        -2         8
## ..       ...       ...
```

---

## select()

Select all columns to do with arrivals:


```r
flights %>% select(contains('arr'))
```

```
## Source: local data frame [336,776 x 5]
## 
##    arr_time sched_arr_time arr_delay carrier        CarrierName
##       (int)          (int)     (dbl)   (chr)              (chr)
## 1       830            819        11      UA             United
## 2       850            830        20      UA             United
## 3       923            850        33      AA           American
## 4      1004           1022       -18      B6            JetBlue
## 5       812            837       -25      DL              Delta
## 6       740            728        12      UA             United
## 7       913            854        19      B6            JetBlue
## 8       709            723       -14      EV Atlantic_Southeast
## 9       838            846        -8      B6            JetBlue
## 10      753            745         8      AA           American
## ..      ...            ...       ...     ...                ...
```

---

## filter()

Filter allows you to subset by rows. It will take any logical statement (something that will produce TRUE and FALSE values, e.g. a boolean vector):

What might the following R commands do?


```r
# single criterion
filter(flights, origin == 'JFK')
flights %>% filter(dep_delay < 10)
flights %>% filter(carrier %in% c('UA', 'DL', 'AA'))

# Multiple criteria
flights %>% filter(dep_delay < 10 & arr_delay > 10)
flights %>% filter(dep_delay < 10 | arr_delay > 10)

# Multiple arguments are equivalent to and
flights %>% filter(dep_delay < 10, arr_delay == 1)
```

---

## arrange()

The arrange() function lets you order the rows by one or more columns in ascending or descending order:

What might the following R commands do?


```r
arrange(flights, dep_time)

flights %>% arrange(dep_delay)

flights %>% arrange(dep_delay, arr_delay)

flights %>% arrange(desc(dep_delay))
```

---

## mutate()

In contrast to select(), which retains a subset of all variables, mutate() creates new columns which are added to a copy of the dataset.

What might the following R commands do?


```r
flights$total_delay <- flights$dep_delay + flights$arr_delay

mutate(flights, total_delay2 = dep_delay + arr_delay)

flights %>% mutate(total_delay = dep_delay + arr_delay)

flights %>% mutate(date = paste(year, month, day, sep='-'))
```

Which of these will change (i.e. add a column to) the object called flights?

---

## summarise() or summarize()

Summarises multiple values into a single value

What might the following R commands do?


```r
# our data has NULL (NA) values, so we need to tell R to remove those
with(flights, mean(dep_delay, na.rm=T))

# in dplyr
summarise(flights, mean(dep_delay, na.rm=T))

flights %>% summarise(mean(dep_delay, na.rm=T))

summarise(group_by(flights, carrier), mean(dep_delay, na.rm=T))
```

---


```r
flights %>% 
  group_by(carrier) %>% 
  summarise(avg_delay = mean(dep_delay, na.rm = T)) %>% 
  arrange(avg_delay)
```

```
## Source: local data frame [16 x 2]
## 
##    carrier avg_delay
##      (chr)     (dbl)
## 1       US  3.782418
## 2       HA  4.900585
## 3       AS  5.804775
## 4       AA  8.586016
## 5       DL  9.264505
## 6       MQ 10.552041
## 7       UA 12.106073
## 8       OO 12.586207
## 9       VX 12.869421
## 10      B6 13.022522
## 11      9E 16.725769
## 12      WN 17.711744
## 13      FL 18.726075
## 14      YV 18.996330
## 15      EV 19.955390
## 16      F9 20.215543
```

---

## And finally... ReadR

Let's read in some election data. I will use this in a future session.


```r
# if you haven't already done so, install the package
install.packages('readr')
```


```r
# load the library
library(readr)

# load the file
election.results <- read_csv('./assets/election_results_2015.csv')

# how big?
dim(election.results)
```

```
## [1] 651 147
```
I challenge thee to <a href='assets/election_results_2015.csv'>download the data</a> and think about how you might tidy these data using tidyr and produce meaningful data using dplyr. 

---

## Some examples

*How many people voted?*


```r
election.results %>% 
  filter(Country=='England')%>% 
  rename(nvotes = `Total number of valid votes counted`) %>%
  summarise(total=sum(nvotes)/sum(Electorate))
```

```
## Source: local data frame [1 x 1]
## 
##       total
##       (dbl)
## 1 0.6601301
```

---

## Some examples

*In which constituency was the pirate party most popular?*


```r
election.results %>% 
  arrange(desc(Pirate)) %>%
  select(`Constituency Name`, Region, Pirate) %>%
  head(3)
```

```
## Source: local data frame [3 x 3]
## 
##    Constituency Name     Region Pirate
##                (chr)      (chr)  (int)
## 1 Manchester Central North West    346
## 2           Vauxhall     London    201
## 3   Salford & Eccles North West    183
```

---

## Some examples

*Other possible questions:*

* Who would have won if there was proportional representation?
* What was the share of votes of the BNP in England and Scotland?

What questions would you want to answer from these data?

---

## Further reading

* I highly recommend the <a href="https://www.rstudio.com/wp-content/uploads/2015/02/data-wrangling-cheatsheet.pdf">data wrangling cheatsheet</a> (RStudio)
* <a href="https://rpubs.com/bradleyboehmke/data_wrangling">Data processing with tidyr and dplyr</a>

dplyr:

* <a href="https://cran.rstudio.com/web/packages/dplyr/vignettes/introduction.html">An introduction to dplyr</a> (RStudio)
* <a href="http://www.r-bloggers.com/hands-on-dplyr-tutorial-for-faster-data-manipulation-in-r/">Hands-on dplyr tutorial for faster data manipulation in R</a> (R-Bloggers)
* <a href="https://www.dropbox.com/sh/i8qnluwmuieicxc/AAAgt9tIKoIm7WZKIyK25lh6a">Comprehensive Tutorial with examples</a> (Hadley Wickham, R! 2014)

tidyr:

* <a href="http://www.r-bloggers.com/introducing-tidyr/">Introducting tidyr</a> (R-Bloggers)
* <a href="http://www.cookbook-r.com/Manipulating_data/Converting_data_between_wide_and_long_format/">Converting data between wide and long></a> (R cookbook)

---

## The road ahead

*There will be more sessions*

* In the next session (Wed 25 May) we will look at visualising our data in R. We'll learn some basic plotting, but also how to use a plotting library called ggplot2. 

* In the session after that (likely Wed 8th of June) we'll look at a case study that combines all the skills we have picked up up to that point. 

*Wrapping up:*

* Don't forget to leave me feedback: http://goo.gl/forms/yvYPJ7OEP5

* Slides are on <a href="http://cbas/lugtigha">http://cbas/lugtigha</a>
