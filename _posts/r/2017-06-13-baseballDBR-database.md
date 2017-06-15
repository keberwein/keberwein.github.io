---
layout: post
title: "Creating a Baseball Database with baseballDBR"
categories: r
tags: [r, sabermetrics]
---



My original motivation to write the `baseballDBR` package for R was to provide a quick and easy way to have access to Sean Lahman's Baseball Database. The `Lahman` package has been around for several years, and is a great resource, however it lacks consistant updates. Also, the CRAN repository has limits on how large data packages can be, and the `Lahman` package is currently pushing that limit.

The answer was an "open-data" format that is maintained by the Chadwick Bureau's [Baseball Databank](https://github.com/chadwickbureau/baseballdatabank), which is based on Sean Lahman's database, version 2015-01-24, but has additinal tables aggregated from [Retrosheet](https://github.com/chadwickbureau/baseballdatabank) data.

For further details, see the [GitHub page](https://github.com/keberwein/baseballDBR) for the `baseballDBR` package. In the meantime, we'll spin through a few lines of code that will quickly get us up and running.


{% highlight r %}
# Install the package from CRAN
install.packages(baseballDBR)
{% endhighlight %}


The following is based on the assumption we have an empty Postgres database called "lahman." If you prefer another database, the following method should also work with MySQL and the `RMySQL` package.


{% highlight r %}
library(baseballDBR)
library(RPostgreSQL)

# Load all tables into the Global Environment.
get_bbdb(AllTables = TRUE)

# Make a list of all data frames.
dbTables <- names(Filter(isTRUE, eapply(.GlobalEnv, is.data.frame)))

# Load data base drivers and load all data frames in a loop.
drv <- dbDriver("PostgreSQL")
con <- dbConnect(drv, host= "localhost", dbname= "lahman", user= "YOUR_USERNAME", password = "YOUR_PASSWORD")

for (i in 1:length(dbTables)) { 
    dbWriteTable(con, name =  dbTables[i], value = get0(dbTables[i]), overwrite = TRUE) 
}

# Disconnect from database.
dbDisconnect(con)
rm(con, drv)
{% endhighlight %}


![](https://github.com/keberwein/keberwein.github.io/blob/master/images/baseballDBR_hex.png?raw=true)
