---
layout: post
title: "Calculate Inflation with the blscrapeR Package"
categories: r
tags: [r, bls]
---



The Consumer Price Index (CPI) is the main standard for tracking the inflation of the U.S. dollar. The various CPI measures are published monthly by the Bureau of Labor Statistics.

For this walk-through, we will be using the [blcsrapeR package](https://github.com/keberwein/blscrapeR) to download our data from the BLS and perform the calculation. The blscrapeR package can be installed via CRAN.


{% highlight r %}
install.packages(“blscrapeR”)
{% endhighlight %}

## CPI: Tracking Inflation

Although there are many measures of inflation, the CPI’s “Consumer Price Index for All Urban Consumers: All Items” is normally the headline inflation rate one would hear about on the news, [see FRED](https://fred.stlouisfed.org/series/CPIAUCSL).

Getting these data from the blscrapeR package is easy enough:


{% highlight r %}
library(blscrapeR)
df <- bls_api("CUSR0000SA0")

head(df)
{% endhighlight %}



{% highlight text %}
##   year period periodName   value footnotes    seriesID
## 1 2017    M04      April 244.158           CUSR0000SA0
## 2 2017    M03      March 243.752           CUSR0000SA0
## 3 2017    M02   February 244.456           CUSR0000SA0
## 4 2017    M01    January 244.158           CUSR0000SA0
## 5 2016    M12   December 242.821           CUSR0000SA0
## 6 2016    M11   November 242.199           CUSR0000SA0
{% endhighlight %}

Due to the limitations of the API, we are only able to gather twenty years of data per request. However the formula for calculating inflation is based on the 1980 dollar, so the data from the API aren't sufficient.

The package includes a function that collects information form the CPI beginning at 1947 and calculates inflation. Since these data are not pulled directly from the API, the inflation_adjust() function does not count against your daily API call limit.

To find out the value of a 1995 dollar in 2015, we just make a simple function call. Note that in the results, 2016 represents an incomplete year at the time this document was created.

Our result returns the inflation of a 1995 dollar over the past several years.


{% highlight r %}
library(blscrapeR)
df <- inflation_adjust(1995)

tail(df)
{% endhighlight %}



{% highlight text %}
## [1] 1.51 1.53 1.55 1.56 1.58 1.60
{% endhighlight %}

If we want to check our results, we can head over to the [CPI Inflation Calculator](https://data.bls.gov/cgi-bin/cpicalc.pl) on the BLS website.

## CPI: Tracking Escalation

Another typical use of the CPI is to determine price escalation. This is especially common in escalation contracts. While there are many different ways one could calculate escalation below is a simple example. **Note:** the BLS recommends using non-seasonally adjusted data for escalation calculations.

Suppose we want the price escalation of $100 investment we made in January 2014 to February 2015:


{% highlight r %}
library(blscrapeR)
df <- bls_api("CUSR0000SA0",
              startyear = 2014, endyear = 2015)
# Set base value.
base_value <- 100
# Get CPI from base period (January 2014).
base_cpi <- subset(df, year==2014 & periodName=="January", select = "value")
# Get the CPI for the new period (February 2015).
new_cpi <- subset(df, year==2015 & periodName=="February", select = "value")
# Calculate the updated value of our $100 investment.
(base_value / base_cpi) * new_cpi
{% endhighlight %}



{% highlight text %}
##       value
## 24 100.0442
{% endhighlight %}



{% highlight r %}
# Woops, looks like we lost a penny!
{% endhighlight %}

**Disclaimer:** Escalation is normally formulated by lawyers and bankers, the author(s) of this package or this post are neither, so the above should only be considered a code example.






