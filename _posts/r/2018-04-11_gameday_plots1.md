---
layout: post
title: "Visualizing Data with R and mlbgameday"
categories: r
tags: [r, baseball, mlbgameday]
---



## Install from CRAN

`install.packages("mlbgameday")`

## Gathering Data

The package is primarily a data package, and has no native plotting tools. However, there are several plotting options available by leveraging one of the R language's excellent plotting libraries.

For all of the following examples, we will use The pitch data for Jake Arrieta's no-hitter, which occurred on April 21, 2016.



{% highlight r %}
library(mlbgameday)
library(dplyr)

# Grap some Gameday data. We're specifically looking for Jake Arrieta's no-hitter.
gamedat <- get_payload(start = "2016-04-21", end = "2016-04-21")

# Subset that atbat table to only Arrieta's pitches and join it with the pitch table.
pitches <- inner_join(gamedat$pitch, gamedat$atbat, by = c("num", "url")) %>%
    subset(pitcher_name == "Jake Arrieta")
{% endhighlight %}

## Visualizing With ggplot


{% highlight r %}
library(ggplot2)

# basic example
ggplot() +
    geom_point(data=pitches, aes(x=px, y=pz, shape=type, col=pitch_type)) +
    coord_equal() + geom_path(aes(x, y), data = mlbgameday::kzone)
{% endhighlight %}

![plot of chunk unnamed-chunk-3](/assets/Rfig/unnamed-chunk-3-1.svg)

## Including Batter Stance


{% highlight r %}
library(ggplot2)

# basic example with stand.
ggplot() +
    geom_point(data=pitches, aes(x=px, y=pz, shape=type, col=pitch_type)) +
    facet_grid(. ~ stand) + coord_equal() +
    geom_path(aes(x, y), data = mlbgameday::kzone)
{% endhighlight %}

![plot of chunk unnamed-chunk-4](/assets/Rfig/unnamed-chunk-4-1.svg)

## Visualizing With Other Tools

The R language has no shortage of visualization tools. Other examples, including Plotly, can be found in the [package vignettes](https://github.com/keberwein/mlbgameday/blob/master/vignettes/pitch_plotting.Rmd).

![](https://raw.githubusercontent.com/keberwein/mlbgameday/master/man/figures/mlbgameday_hex.png)

