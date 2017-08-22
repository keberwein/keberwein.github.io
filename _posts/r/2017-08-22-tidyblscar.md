---
layout: post
title: "Tidy-er BLS data with the blscarpeR package"
categories: r
tags: [r, bls]
---



The recent release of the [blscrapeR](https://github.com/keberwein/blscrapeR) package brings the "tidyverse" into the fold. Inspired by my recent collaboration with Kyle Walker on his excellent [tidycensus](https://github.com/walkerke/tidycensus) package, blscrapeR has been optimized for use within the tidyverse as of the current version 3.0.0.

## New things you'll notice right away include:

* All data now returned as tibbles.

* dplyr and purrr are now imported packages, along with magrittr and ggplot, which were imported from the start.

* No need to call any packages other than tidyverse and blscrapeR.


## Major internal changes

* Switched from base R to dplyr in instances where performance could be increased.

* Standard apply functions replaced with purrr `map()` functions where performance could be increased.


{% highlight r %}
install.packages("blscrapeR")
{% endhighlight %}

## The BLS: More than Unemployment

The American Time Use Survey is one of the BLS' more interesting data sets. Below is an API query that compares the time Americans spend watching TV on a daily basis compared to the time spent socializing and communicating.

It should be noted, some familiarity with BLS series id numbers is required here. The [BLS Data Finder](https://beta.bls.gov/dataQuery/search) is a nice tool to find series id numbers.


{% highlight r %}
library(blscrapeR)
library(tidyverse)
tbl <- bls_api(c("TUU10101AA01014236", "TUU10101AA01013951")) %>%
    spread(seriesID, value) %>%
    dateCast() %>%
    rename(watching_tv = TUU10101AA01014236, socializing_communicating = TUU10101AA01013951)
tbl
{% endhighlight %}



{% highlight text %}
## # A tibble: 3 x 7
##    year    period periodName footnotes socializing_communicating watching_tv       date
## * <dbl>    <list>     <list>    <list>                     <dbl>       <dbl>     <date>
## 1  2014 <chr [1]>  <chr [1]> <chr [1]>                      0.71        2.82 2014-01-01
## 2  2015 <chr [1]>  <chr [1]> <chr [1]>                      0.68        2.78 2015-01-01
## 3  2016 <chr [1]>  <chr [1]> <chr [1]>                      0.65        2.73 2016-01-01
{% endhighlight %}

## Unemployment Rates

The main attraction of the BLS are the monthly employment and unemployment data. Below is an API query and plot of three of the major BLS unemployment rates.

* U-3: The "official unemployment rate." Total unemployed, as a percent of the civilian labor force.

* U-5: Total unemployed, plus discouraged workers, plus all other marginally attached workers, as a percent of the civilian labor force plus all marginally attached workers.

* U-6: Total unemployed, plus all marginally attached workers, plus total employed part time for economic reasons, as a percent of the civilian labor force plus all marginally attached workers.


{% highlight r %}
library(blscrapeR)
library(tidyverse)
tbl <- bls_api(c("LNS14000000", "LNS13327708", "LNS13327709"), registrationKey = "BLS_KEY") %>%
    spread(seriesID, value) %>%
    dateCast() %>%
    rename(u3_unemployment = LNS14000000, u5_unemployment = LNS13327708, u6_unemployment = LNS13327709)


ggplot(data = tbl, aes(x = date)) + 
    geom_line(aes(y = u3_unemployment, color = "U-3 Unemployment")) +
    geom_line(aes(y = u5_unemployment, color = "U-5 Unemployment")) + 
    geom_line(aes(y = u6_unemployment, color = "U-6 Unemployment")) + 
    labs(title = "Monthly Unemployment Rates") + ylab("value") +
    theme(legend.position="top", plot.title = element_text(hjust = 0.5)) 
{% endhighlight %}

![plot of chunk unnamed-chunk-4](/assets/Rfig/unnamed-chunk-4-1.png)

For more information and examples, please see the [package vignettes](https://github.com/keberwein/blscrapeR/tree/master/vignettes).

![](https://github.com/keberwein/blscrapeR/blob/master/man/figures/blscrapeR_hex.png?raw=true)
