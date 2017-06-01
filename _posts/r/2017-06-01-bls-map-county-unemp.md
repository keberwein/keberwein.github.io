---
layout: post
title: "Mapping County Unemployment with blscrapeR"
categories: r
tags: [r, bls]
---


{% highlight r %}
library(knitr)
knitr::opts_chunk$set(cache=T, warning=F, message=F, cache.lazy=F, dpi = 180)
options(width=120, dplyr.width = 150)
{% endhighlight %}

The [blscrapeR package](https://github.com/keberwein/blscrapeR) makes it easy to produce choropleth maps of various employment and unemployment rates from the Bureau of Labor Statistics (BLS.) It’s easy enough to pull a metric for a certain county. The code below pulls the unemployment rates for Orange County, FL from the BLS API.


{% highlight r %}
library(blscrapeR)
df <- bls_api("LAUCN120950000000003",
              startyear = 2016, endyear = 2016)

head(df)
{% endhighlight %}


The only problem is, there are over 3,000 counties in the United States and an API query of that size would push any user well over the daily query limits of the BLS API.

To resolve the issue, the `blscrapeR` package includes a function that allows us to pull county statistics in the form of a text file from the BLS servers, which don’t count against a user’s daily query limit.

**NOTE:** You can use arguments to get data for a specific month, but if there is no date argument, the function will pull the most recent month in the data set.


{% highlight r %}
library(blscrapeR)
df <- get_bls_county()

head(df)
{% endhighlight %}

**Limitations:** The `get_bls_county()` function is only able to pull labor data for the past 12 months at time of query.

## Choropleth Mapping


Now that we’ve got the data, it’s time for the mapping. There are a few options here, but the simplest option would be to use the package’s  `bls_map_county()` function.


{% highlight r %}
library(blscrapeR)
bls_map_county(map_data = df, fill_rate = "unemployed_rate", 
               labtitle = "Unemployment Rate by County")
{% endhighlight %}

![](https://github.com/keberwein/keberwein.github.io/blob/master/images/blscrape_county_unemp.png?raw=true)

Maybe you just want one state? That's alright too.


{% highlight r %}
library(blscrapeR)
# Map the unemployment rate for the Southeastern United States.
df <- get_bls_county(stateName = "Florida")

bls_map_county(map_data=df, fill_rate = "unemployed_rate", 
               stateName = "Florida")
{% endhighlight %}

![](https://github.com/keberwein/keberwein.github.io/blob/master/images/bus_fl_map.png?raw=true)

## Custom Mapping

The `bls_map_county()` function produces a map that may not be your cup of tea. The function is only provided as a “quick function” to see if your data fit. The `blscrapeR` package provides the fortified map data, which includes longitud, latitude and FIPS codes. This data set is suitable for any kind of ggplot2 map you can think of.

First, call the internal map data set and have a look:


{% highlight r %}
library(blscrapeR)
us_map <- county_map_data

head(us_map)
{% endhighlight %}



{% highlight text %}
##      long      lat order  hole piece   group    id
## 1 1225889 -1275020     1 FALSE     1 01001.1 01001
## 2 1244873 -1272331     2 FALSE     1 01001.1 01001
## 3 1244129 -1267515     3 FALSE     1 01001.1 01001
## 4 1272010 -1262889     4 FALSE     1 01001.1 01001
## 5 1276797 -1295514     5 FALSE     1 01001.1 01001
## 6 1272367 -1296730     6 FALSE     1 01001.1 01001
{% endhighlight %}

Notice the id column looks a lot like one of the FIPS codes returned by the get_bls_county() function? This is actually a concatenation of the state + county FIPS codes. The first two numbers are the state FIPS and the last four are the county FIPS. These boundaries currently represent 20015/2016 and will be updated accordingly so they always represent the current year.

Next, produce your custom map.


{% highlight r %}
library(blscrapeR)
library(ggplot2)
# Get the most recent unemployment rate for each county on a national level.
df <- get_bls_county()
# Get map data
us_map <- county_map_data

# Insert larger breaks in unemployment rates
df$rate_d <- cut(df$unemployed_rate, breaks = c(seq(0, 10, by = 2), 35))
# Plot
ggplot() +
    geom_map(data=us_map, map=us_map,
             aes(x=long, y=lat, map_id=id, group=group),
             fill="#ffffff", color="#0e0e0e", size=0.15) +
    geom_map(data=df, map=us_map, aes_string(map_id="fips", fill=df$rate_d),
             color="#0e0e0e", size=0.15) +
    scale_fill_brewer()+
    coord_equal() +
    theme(axis.line=element_blank(),
          axis.text.x=element_blank(),
          axis.text.y=element_blank(),
          axis.ticks=element_blank(),
          axis.title.x=element_blank(),
          axis.title.y=element_blank(),
          panel.grid.major = element_blank(),
          panel.grid.minor = element_blank(),
          panel.border = element_blank(),
          panel.background = element_blank(),
          legend.title=element_blank())
{% endhighlight %}

![](https://github.com/keberwein/keberwein.github.io/blob/master/images/bls_custom_map.png?raw=true)

If you want more mapping options, there is more information in the `blscrapeR` [package vignettes](https://github.com/keberwein/blscrapeR/tree/master/vignettes).
