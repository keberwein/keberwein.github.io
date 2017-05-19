---
layout: post
title: "Using PL/R and PL/Python in Postgres"
categories: r
tags: [r, python, PL/R, Postgres]
---



I’ve recently been exploring options to calculate median and quartiles in my Postgres database. If you’re familiar with quartiles you know how handy they can be. There’s a few different options in the Postgres universe to accomplish this, so I figured I would give them all a whirl and see which was the friendliest (and fastest) on my CPU.

## The Data

I’m using the “batting” table for Sean Lahman’s [baseball database](http://www.seanlahman.com/baseball-archive/statistics/) as my proof of concept. The table has just under 100,000 rows. Not too big, but a good test case. For my example here, I’m using the “r” column, which indicates total runs scored for a season.

## The R Method

[PL/R](http://www.joeconway.com/plr/) is a popular Postgres extension. If you haven’t checked it out, I would highly recommend it.

* Pros: Very fast and simple code. Only one line of actual R code used to build this function.

* Cons: R isn’t pre-installed on most systems and PLR isn’t shipped with Postgres. Takes a bit of system config., but not too much.


{% highlight sql %}
CREATE OR REPLACE FUNCTION r_quartile(ANYARRAY) RETURNS ANYARRAY AS $$
quantile(arg1, probs = seq(0, 1, 0.25), names = FALSE)
$$ LANGUAGE 'plr';

CREATE AGGREGATE quartile (ANYELEMENT) (
sfunc = array_append,
stype = ANYARRAY,
finalfunc = r_quartile,
initcond = '{}');
{% endhighlight %}

## The Python Method

I’m a big fan of Python in general. It’s currently one of my favorite ETL languages. Here I’m using yet another great Postgres extension called [PL/Python](http://www.postgresonline.com/journal/archives/99-Quick-Intro-to-PLPython.html).

* Pros: Python is pre-installed on most Linux and Mac systems making set up a breeze.

* Cons: Surprisingly slow! Python ties to pipe all the data into the interpreter and then back out again as a function result. Too much system cost for me!


{% highlight sql %}
CREATE TYPE boxplot_values AS (
  min       numeric,
  q1        numeric,
  median    numeric,
  q3        numeric,
  max       numeric
);

CREATE OR REPLACE FUNCTION public._final_boxplot(strarr numeric[])
  RETURNS boxplot_values AS
$BODY$
    x = strarr
    a.sort()
    i = len(a)
    return ( a[0], a[i//4], a[i//2], a[i*3//4], a[-1] )
$BODY$
  LANGUAGE plpythonu IMMUTABLE
  COST 100;
 
CREATE AGGREGATE boxplot(numeric) (
  SFUNC=array_append,
  STYPE=numeric[],
  FINALFUNC=_final_boxplot,
  INITCOND='{}'
);
{% endhighlight %}

## The C Method

Everyone remember C from your CS-101 class in college? Yeah, that’s why no one likes to write it. Fortunately, this is a pre-packaged Postgres extension written in C called [Quantile](https://github.com/tvondra/quantile). I’m not going to post the mile-long C code here, but you can see it on the GitHub repo.

* Pros: BLAZING FAST! Returned an array faster than native SQL could calculate a median! I ended up putting the R solution into production because PL/R has room for further application, but if I were looking for speed and nothing else, the Quantile extension is the clear winner.

* Cons: A third-party extension, so you’re at the mercy of the developers to keep things updated. This particular repo is about four years old and looks to be updated on a regular basis.

![](https://github.com/keberwein/keberwein.github.io/blob/master/images/rpython_functiontimes.png?raw=true)
