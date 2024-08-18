PCA & Clustering
================
Carissa Ing
2024-08-18

``` r
library(readr)
library(tidyverse)
```

    ## Warning: package 'ggplot2' was built under R version 4.2.3

    ## Warning: package 'tibble' was built under R version 4.2.3

    ## Warning: package 'dplyr' was built under R version 4.2.3

    ## Warning: package 'lubridate' was built under R version 4.2.3

``` r
library(ClusterR)
library(kernlab)
```

``` r
wine <- read_csv("wine.csv") %>% 
  mutate(color_num = ifelse(color == "white", 1, 2))
```

## Clustering by Red vs. White Wine After PCA

### PCA for Feature Reduction

``` r
pca_result = prcomp(wine[,1:11], center = TRUE, scale. = TRUE)
summary(pca_result)
```

    ## Importance of components:
    ##                           PC1    PC2    PC3     PC4     PC5     PC6     PC7
    ## Standard deviation     1.7407 1.5792 1.2475 0.98517 0.84845 0.77930 0.72330
    ## Proportion of Variance 0.2754 0.2267 0.1415 0.08823 0.06544 0.05521 0.04756
    ## Cumulative Proportion  0.2754 0.5021 0.6436 0.73187 0.79732 0.85253 0.90009
    ##                            PC8     PC9   PC10    PC11
    ## Standard deviation     0.70817 0.58054 0.4772 0.18119
    ## Proportion of Variance 0.04559 0.03064 0.0207 0.00298
    ## Cumulative Proportion  0.94568 0.97632 0.9970 1.00000

``` r
#Chose to use 7 PCs because proportion of variance explained begins to flatten
pca_data = pca_result$x[, 1:7]
```

### K-Means Clustering

``` r
set.seed(123)
kmeans_result = kmeans(pca_data, centers = 2, nstart = 25)

#FInd Accuracy:
wine$kmeans_cluster = kmeans_result$cluster
table(wine$color_num, wine$kmeans_cluster)
```

    ##    
    ##        1    2
    ##   1 4824   74
    ##   2   25 1574

``` r
accuracy = sum(wine$color_num == wine$kmeans_cluster) / nrow(wine)
print(paste("Clustering Accuracy:", round(accuracy * 100, 2), "%"))
```

    ## [1] "Clustering Accuracy: 98.48 %"

### Hierarchical Clustering

``` r
distance_matrix = dist(pca_data)
hc = hclust(distance_matrix, method = "average")
hc_clusters = cutree(hc, k = 2)

#Find Accuracy:
wine$hc_cluster = hc_clusters
table(wine$color_num, wine$hc_cluster)
```

    ##    
    ##        1    2
    ##   1 4897    1
    ##   2 1599    0

``` r
hc_accuracy = sum(wine$color_num == wine$hc_cluster) / nrow(wine)
print(paste("Hierarchical Clustering Accuracy:", round(hc_accuracy * 100, 2), "%"))
```

    ## [1] "Hierarchical Clustering Accuracy: 75.37 %"

Between K-Means clustering and hierarchical clustering methods, K-Means
is superior in terms of accuracy, outperforming hierarchical clustering
by over 20%.

## Clustering by Wine Quality After PCA

``` r
#find how many clusters
summary(wine$quality)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   3.000   5.000   6.000   5.818   6.000   9.000

``` r
#adjust quality vals to match cluster assignments
wine <- wine %>% 
  mutate(quality_edit = quality - 2)
```

### K-Means Clustering

``` r
set.seed(123)
kmeans_result = kmeans(pca_data, centers = 7, nstart = 25)

#FInd Accuracy:
wine$kmeans_cluster = kmeans_result$cluster
table(wine$quality_edit, wine$kmeans_cluster)
```

    ##    
    ##       1   2   3   4   5   6   7
    ##   1   4   7   1   2   5   4   7
    ##   2  15  28   2  21  60  26  64
    ##   3 199 665  22 102 442 244 464
    ##   4 266 642   9 609 476 487 347
    ##   5 140 126   1 447 110 212  43
    ##   6  14  20   0  96  26  35   2
    ##   7   0   0   0   4   1   0   0

``` r
accuracy = sum(wine$quality_edit == wine$kmeans_cluster) / nrow(wine)
print(paste("Clustering Accuracy:", round(accuracy * 100, 2), "%"))
```

    ## [1] "Clustering Accuracy: 12.44 %"

### Hierarchical Clustering

``` r
distance_matrix = dist(pca_data)
hc = hclust(distance_matrix, method = "average")
hc_clusters = cutree(hc, k = 7)

#Find Accuracy:
wine$hc_cluster = hc_clusters
table(wine$quality_edit, wine$hc_cluster)
```

    ##    
    ##        1    2    3    4    5    6    7
    ##   1   29    0    0    0    0    0    1
    ##   2  215    0    1    0    0    0    0
    ##   3 2111   22    1    4    0    0    0
    ##   4 2822   10    0    1    2    1    0
    ##   5 1077    1    0    1    0    0    0
    ##   6  193    0    0    0    0    0    0
    ##   7    5    0    0    0    0    0    0

``` r
hc_accuracy = sum(wine$quality_edit == wine$hc_cluster) / nrow(wine)
print(paste("Hierarchical Clustering Accuracy:", round(hc_accuracy * 100, 2), "%"))
```

    ## [1] "Hierarchical Clustering Accuracy: 0.48 %"

When trying to create clusters to distinguish between wine qualities,
K-Means performs over 100 times better than hierarchical clustering. In
fact, hierarchical clustering performs exceptionally poorly, failing to
cluster even 1% of the data accurately. Although K-Means does
significantly better than hierarchical clustering, it still
misclassifies over 85% of the wines. This suggests that further data
wrangling and/or additional information may be necessary to
differentiate between the qualities.
