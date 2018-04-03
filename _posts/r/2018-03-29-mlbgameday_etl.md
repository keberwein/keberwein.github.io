---
layout: post
title: "Automated Data Collection with R and mlbgameday"
categories: r
tags: [r, baseball, mlbgameday]
---



Opening day is on the way Time to set up a persistent database to collect every pitch thrown in this year's baseball season.

The `mlbgameday` package is designed to facilitate extract, transform and load for MLBAM “Gameday” data. The package is optimized for parallel processing of data that may be larger than memory. Learn more about the project [here](https://github.com/keberwein/mlbgameday).

## Install from CRAN

`install.packages("mlbgameday")`

## Creating a Database


{% highlight r %}
library(mlbgameday)
library(RSQLite)
library(doParallel)

# Create an empty database instance.
con <- dbConnect(RSQLite::SQLite(), dbname = "mlbgameday.sqlite3")

# Optional: Use parallel processing to populate the database with Spring Training Games.

# Set the number of cores to use as the machine's maximum number of cores minus 1 for background processes.
no_cores <- detectCores() - 2
cl <- makeCluster(no_cores)  
registerDoParallel(cl)

get_payload(start = "2018-01-01", end = "2018-03-28", db_con = con)

# Stop and remove cluster.
stopImplicitCluster()
rm(cl)
{% endhighlight %}

## Extract Transform Load of MLB Advanced Media Data

Once you have a database in-place, you can get started quickly. The `mlbgameday` package will work if your current database was gathered using the `pitchRx` package.


{% highlight r %}
library(mlbgameday)
library(RSQLite)

# Log into your database and retreive the most recent date.
con <- dbConnect(RSQLite::SQLite(), dbname = "mlbgameday.sqlite3")

db_end <- dbGetQuery(con, "SELECT MAX(date) FROM atbat")

# Use the max date +1 as the start date and today -1 for the end date for your new payload.
get_payload(start = as.Date(db_end[1,1]) + 1, end = Sys.Date() - 1, db_con = con)
{% endhighlight %}

## Task Scheduling

I prefer to pull the day's data early in the morning (for the day before.) What ever time you choose, you want to consider time zones and allow enough additional time to cover rain delays for late games, as not to miss any information. There are various task scheduling tools, depending on your operating system.

* Linux or OSx: Cron is pretty much the universal standard. Cron is command line driven, but GUI interfaces exist for both operating systems.

* Windows: Several options, but the built-in task scheduler is probably the best.

![](https://raw.githubusercontent.com/keberwein/mlbgameday/master/man/figures/mlbgameday_hex.png)
