---
layout: post
title: "Parallel Processing Baseball Data with R and mlbgameday"
categories: r
tags: [r, baseball, mlbgameday]
---




## Just In Time For Baseball

The `mlbgameday` package has just reached the milestone of version 0.1.0.

Designed to facilitate extract, transform and load for MLBAM “Gameday” data. The package is optimized for parallel processing of data that may be larger than memory. There are other packages in the R universe that were built to perform statistics and visualizations on these data, but [mlbgameday](https://github.com/keberwein/mlbgameday) is concerned primarily with data collection. More uses of these data can be found in the [pitchRx](https://github.com/cpsievert/pitchRx), [openWAR](https://github.com/beanumber/openWAR), and [baseballr](https://github.com/BillPetti/baseballr) packages.

## Install from CRAN

`install.packages("mlbgameday")`

## Parallel Processing

The package's internal functions are optimized to work with the `doParallel` package. By default, the R language will use one core of our CPU. The `doParallel` package enables us to use several cores, which will execute tasks simultaneously. In a standard regular season for all teams, the function has to process more than 2,400 individual files, which depending on your system, can take quite some time. Parallel processing speeds this process up by several times, depending on how many processor cores we choose to use.


{% highlight r %}
library(mlbgameday)
library(doParallel)

# First we need to register our parallel cluster.
# Set the number of cores to use as the machine's maximum number of cores minus 1 for background processes.
no_cores <- detectCores() - 1
cl <- makeCluster(no_cores)  
registerDoParallel(cl)

# Then run the get_payload function as normal.
innings_df <- get_payload(start = "2017-04-03", end = "2017-04-10")

# Don't forget to stop the cluster when finished.
stopImplicitCluster()
rm(cl)
{% endhighlight %}

## Non Parallel

Although the package is optimized for parallel processing, it will also work without registering a parallel backend. When only querying a single day's data, a parallel backend may not provide much additional performance. However, parallel backends are suggested for larger data sets, as the process will be faster by several orders of magnitude.

We can download and subset a small amount of data. In the example below, we'll look for Jake Arrienta's no-hitter in 2016.


{% highlight r %}
library(mlbgameday)
library(dplyr)

# Grap some Gameday data. We're specifically looking for Jake Arrieta's no-hitter.
gamedat <- get_payload(start = "2016-04-21", end = "2016-04-21")

# Subset that atbat table to only Arrieta's pitches and join it with the pitch table.
pitches <- inner_join(gamedat$pitch, gamedat$atbat, by = c("num", "url")) %>%
    subset(pitcher_name == "Jake Arrieta")
{% endhighlight %}

![](https://raw.githubusercontent.com/keberwein/mlbgameday/master/man/figures/mlbgameday_hex.png)
