Coloring\_a\_Dendrogram
================
Chase Clark
June 17, 2018

### Introduction

Dendrograms are a popular way to summarize high-dimensional data in a way that is (somewhat) more interpretable by our puny-human brains. They are used in IDBac to show similarity of protein fingerprints between samples but also, more traditionally, to show phylogenetic relationships. It is also used in many other ways.

Because of their use in IDBac and 16S analysis, I think these will be the first of the higher-level uses of programming that you will encounter during your project. Also, coloring or otherwise distinguishing the labels of a dendrogram tends to be one of the most useful ways to interact and highlight portions of a tree.

Therefore, I wrote a quick intro to coloring dendrograms in R. While I say it is an "intro" and does it covers some of the very basics of R, it is not meant as a complete introduction... aka it is not expected that you will be coloring dendrograms after a first-pass/ or that you won't have to ask me for help or won't have to search google/stackoverflow/community.rstudio.com for help.

------------------------------------------------------------------------

Begin
=====

### Need to load in R libraries that we'll use for this experiment

``` r
# install.packages("dendextend")  # If you haven't already installed it, remove the '#" at the beginning of this line and run it. 
library(dendextend)
```

    ## 
    ## ---------------------
    ## Welcome to dendextend version 1.8.0
    ## Type citation('dendextend') for how to cite the package.
    ## 
    ## Type browseVignettes(package = 'dendextend') for the package vignette.
    ## The github page is: https://github.com/talgalili/dendextend/
    ## 
    ## Suggestions and bug-reports can be submitted at: https://github.com/talgalili/dendextend/issues
    ## Or contact: <tal.galili@gmail.com>
    ## 
    ##  To suppress this message use:  suppressPackageStartupMessages(library(dendextend))
    ## ---------------------

    ## 
    ## Attaching package: 'dendextend'

    ## The following object is masked from 'package:stats':
    ## 
    ##     cutree

### What is an `hclust()` (hierarchical clustering) object in R?

We're going to create a dendrogram from a dataset that comes with R. It has samples (type of car) as rows, and numeric properties of each car in columns. Let's take a peek at what the data matrix looks like:

``` r
head(mtcars)
```

    ##                    mpg cyl disp  hp drat    wt  qsec vs am gear carb
    ## Mazda RX4         21.0   6  160 110 3.90 2.620 16.46  0  1    4    4
    ## Mazda RX4 Wag     21.0   6  160 110 3.90 2.875 17.02  0  1    4    4
    ## Datsun 710        22.8   4  108  93 3.85 2.320 18.61  1  1    4    1
    ## Hornet 4 Drive    21.4   6  258 110 3.08 3.215 19.44  1  0    3    1
    ## Hornet Sportabout 18.7   8  360 175 3.15 3.440 17.02  0  0    3    2
    ## Valiant           18.1   6  225 105 2.76 3.460 20.22  1  0    3    1

We're going to do some voodoo here (aka we'll go over this step another time) to create a default distance matrix and then a hierarchcial clustering object. The hierarchical clustering object is then assigned to a variable named `hclustObject`

``` r
hclustObject <- hclust(dist(mtcars))
```

Let's take a peek at what this new `hclustObject` is:

``` r
hclustObject
```

    ## 
    ## Call:
    ## hclust(d = dist(mtcars))
    ## 
    ## Cluster method   : complete 
    ## Distance         : euclidean 
    ## Number of objects: 32

It's fine that this is some mumbo-jumbo for you at this point. It just says what the input to hclust was, the clustering and distance methods that were used, and the number of cars (samples). But let's take a closer at the type of object `hclustObject` is:

``` r
typeof(hclustObject)
```

    ## [1] "list"

``` r
length(hclustObject)
```

    ## [1] 7

So, it's a list with 7 elements. Now let's take a peek at what each of the elements are, in this list returned by `hclust()`

``` r
names(hclustObject)
```

    ## [1] "merge"       "height"      "order"       "labels"      "method"     
    ## [6] "call"        "dist.method"

I'm only going to talk about a couple of these. For more info on this and other results returned by a function you can always type a "?" followed by the function name: `?hclust` (In this case you would scroll down to the "Values" section).

### Subsetting a List in R by name

To access an element of a list *by name* you can use the dollar sign `$` to separate the variable and the name of the list element: `hclustObject$order`

------------------------------------------------------------------------

### What is the list element `order`, returned by `hclust()`?

Let's look at what the `order` list element is:

Your samples were in a particular order in the original matrix (order of rows), and in the same order in the distance matrix.

Now that you have a dendrogram... they are in a different order. `hclustObject$order` is a vector of integers that map the new order back to the original.

Let's look at only the first element in the vector- `[[1]]` gives you the first element

``` r
hclustObject$order[[1]]
```

    ## [1] 31

So, the first row (sample) in your original matrix is now in position 36 of the dendrogram.

------------------------------------------------------------------------

### What is the list element `labels`, returned by `hclust()`?

`labels` is the most useful list element for us:

Here we use single brackets `[]`, and the sequence of 1 through 5, to return only the labels in positions 1, 2, 3, 4, and 5

``` r
1:5
```

    ## [1] 1 2 3 4 5

``` r
hclustObject$labels[1:5]
```

    ## [1] "Mazda RX4"         "Mazda RX4 Wag"     "Datsun 710"       
    ## [4] "Hornet 4 Drive"    "Hornet Sportabout"

So, in the dendrogram, these are the first five labels.

------------------------------------------------------------------------

### Coloring dendrogram labels with dendextend

Let's convert `hclustObject` into something the package `dendextend` will be happy to work with.

Remembe that `hclustObject` is simply the results returned by the function `hclust()`.

``` r
hclustObject
```

    ## 
    ## Call:
    ## hclust(d = dist(mtcars))
    ## 
    ## Cluster method   : complete 
    ## Distance         : euclidean 
    ## Number of objects: 32

Convert to dendrogram

``` r
hclustObject <-  hclustObject %>%  as.dendrogram()
```

``` r
hclustObject
```

    ## 'dendrogram' with 2 branches and 32 members total, at height 425.3447

### Quick Detour: Pipes!

So, what the heck was the: `%>%` ??

It's called a pipe and was silently imported from the `magrittr` package when you loaded dendextend (`library(dendextend)`)

This pipe is an important concept to learn as it is used heavily by "tidyverse" packages like `dplyr` and `purrr` which is what you will be learning for data manipulation

*`%>%` takes the value from the function on the left and provides it as the input data to the function on the right.*

So, taking another look:

`hclustObject <-  hclustObject %>%  as.dendrogram()`

We are taking the value of `hclustObject` and providing it as the input data to the function `as.dendrogram()`

In "base R" this would have been written as:

`hclustObject <- as.dendrogram(hclustObject)`

This example doesn't make it obvious why `%>%` is helpful, but the power of it will become obvious when you start *chaining* functions together. Eg:

``` r
 1 + 1 %>% 
       sum(1) %>% 
       sum(1)
```

    ## [1] 4

Also important to point out is `%>%` inserts the values of the previous function into the *first* argument of the next function. If you need to insert the value as a second, third, etc argument, you can use a period `.` to denote where to insert the value.

``` r
sum(2, 2)
```

    ## [1] 4

Is the same as:

``` r
2 %>%  
  sum(2)
```

    ## [1] 4

Is the same as:

``` r
2 %>%  
  sum(., 2)
```

    ## [1] 4

### Back to the dendrogram

``` r
hclustObject
```

    ## 'dendrogram' with 2 branches and 32 members total, at height 425.3447

We can color the leaves (tips of the dendrogram) with `color_labels()`

``` r
hclustObject %>% color_labels("blue")
```

    ## Warning in stats::cutree(tree, k = k, h = h, ...): NAs introduced by
    ## coercion

    ## Error in if (min(k) < 1 || max(k) > n) stop(gettextf("elements of 'k' must be between 1 and %d", : missing value where TRUE/FALSE needed

But notice that you must provide a vector of colors the same length as leaves in `hclustObject`

One way to do this is to "repeat" a color as many times as the length (number) of leaves in `hclustObject`

*Note*: now `hclustObject` is not a list and you must use `labels()` to get the dendrgram's labels

``` r
labels(hclustObject) %>% 
  length
```

    ## [1] 32

So we need to repeat "blue" 51 times we could use \`rep("blue, times = 32)") *however* let's write it in a way that will work again if we were to provide it another dendrogram with a different number of leaves (samples).

``` r
newColors <- rep("blue", times = length(labels(hclustObject)))
```

Now `newColors` will always be the correct length no matter with dendrogram it is provided

Let's plot it without coloring:

``` r
hclustObject %>%
  plot
```

![](Coloring_a_Dendrogram_files/figure-markdown_github/unnamed-chunk-22-1.png)

And now plot with all labels colored blue:

``` r
hclustObject %>% 
  color_labels(col= newColors) %>% 
  plot
```

![](Coloring_a_Dendrogram_files/figure-markdown_github/unnamed-chunk-23-1.png)

### Exercise:

Color all car labels black except Fiats, which should be red.

Find and return the rownames of mtcars that contain "Fiat"

``` r
fiats <- rownames(mtcars)[grep("Fiat", rownames( mtcars))]
```

Now the variable `fiats` is a character vector:

``` r
fiats
```

    ## [1] "Fiat 128"  "Fiat X1-9"

Now we need to find where these occur in the dendrogram by matching against the dendrogram's labels.

``` r
# We actually have two options here that will give us the same results. We can use ==  or %in%
# which(labels(hclustObject) == fiats)

labelLocation <- which(labels(hclustObject) %in% fiats)

labelLocation # show value of labelLocation
```

    ## [1] 19 20

So, "Fiat 128" "Fiat X1-9" are labels 19 and 20 in the dendrogram. \***We don't know which is label 19 and which is 20 but for this exercise we don't need to**

``` r
colorsToUse <- rep("black", length(labels(hclustObject)))
colorsToUse # Show value of colorsToUse
```

    ##  [1] "black" "black" "black" "black" "black" "black" "black" "black"
    ##  [9] "black" "black" "black" "black" "black" "black" "black" "black"
    ## [17] "black" "black" "black" "black" "black" "black" "black" "black"
    ## [25] "black" "black" "black" "black" "black" "black" "black" "black"

Overwrite `colorsToUse` to a value of "red" at the indices in `labelLocation`

``` r
colorsToUse[labelLocation] <- "red"
colorsToUse
```

    ##  [1] "black" "black" "black" "black" "black" "black" "black" "black"
    ##  [9] "black" "black" "black" "black" "black" "black" "black" "black"
    ## [17] "black" "black" "red"   "red"   "black" "black" "black" "black"
    ## [25] "black" "black" "black" "black" "black" "black" "black" "black"

``` r
hclustObject %>% 
  color_labels(col= colorsToUse) %>% 
  plot
```

![](Coloring_a_Dendrogram_files/figure-markdown_github/unnamed-chunk-29-1.png)

### How would you color everything black, except Fiats red and Mercedes ("Merc") blue?
