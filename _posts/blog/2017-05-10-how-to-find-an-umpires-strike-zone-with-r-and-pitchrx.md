---
layout: post
title: "Find Umpire Strike Zones with R and pitchRx"
categories: saber, r, blog
excerpt:
image:
  feature:
---

Ever wonder how “high” or “low” an umpire’s strikezone is compared to the rest of the leauge? Thanks to some public data and the PitchRx package, it’s easy to use a cluster analysis to figure it out!

## Tim Timmons vs. the League

For this analysis I’m going to pick on umpire Tim Timmons, for no other reason than I like his name. The first thing I’ll want to do is create a subset where Timmons is the home plate umpire. Note, in the SQL query at the end of the article I’ve already sorted my data to only include home plate umpires and named the data frame ‘umpZone.’

Before we get going, let’s load some required packages.

```
library(dplyr)
library(ggplot2)
library(pitchRx)
library(mgcv)
```
The data here are quite large and are collected from my PitchRx database. Refer to the SQL at the end of the article for the data collection method.

Now that we’ve got all the pitches where Timmons was the home plate ump., the next thing is to parse the data even further to include only pitches that were a called ball or strike.

```
TimNoswing

AllNoswing
```

## Visualizing the Strike Zone

Now we get to the fun part! There’s a couple ways to go about this. The simplest way is to go directly to the strikeFX function that’s included in the pitchRx package. To visualize strike zones for Timmons vs. the rest of the league you would do something like this.

```
strikeFX(TimNoswing, geom="tile", density1=list(des="Called Strike"), density2=list(des="Ball"), 
         layer=facet_grid(.~stand))

```

...and then

```
strikeFX(AllNoswing, geom="tile", density1=list(des="Called Strike"), density2=list(des="Ball"), 
         layer=facet_grid(.~stand))
```

### Timmons

<img src="/images/TimBallStrike.png" alt="image">

### Leauge

<img src="/images/Allumps.png" alt="image">
	

## What do the Heat Maps Say?

The “all umpires” map looks much more filled out compared to Timmons’ simply because there are more data points. However, Timmons appears to have a comparable strike zone to the rest of the umpires in the league.

There are a couple interesting points to both maps.

* The Strike zone appears to be a bit low in general.

* The zone appears to shift depending on the batter’s stance.

The reason for the zone appearing low could very well depend on the batter’s height. The true zone is between a batter’s waist and knees, so the zone is variable depending on height. In this case we’re using a “standardized” strike zone based on an average height. Also, it would be helpful to keep in mind that Pitch f/x technology is very good but may not be perfect.

The shift of the strike zone between left-handed and right-handed batters also has a simple explanation. When looking at the heat maps it’s helpful to imagine a batter standing there, most likely crowding the plate.

## Alternate Visualization Methods

It’s been proposed that the ‘mgcv’ package in R provides superior strike zone visualizations due to its ability to “smooth” the data accurately using cross-validation. The result is a better looking heat map that enables us to see the zone a little better. But there is one caveat, this method is a RAM killer! I’m working with 8 GB of RAM so I only applied this method to Timmons’ data frame. I would recommend more if you want to apply this to a larger data frame.

First, create a binomial model from the ‘TimNoswing’ data frame.

```
StrikeModel
```

Then, use strikeFX to plot the model.

```
strikeFX(TimNoswing, model=StrikeModel, layer=facet_grid(.~stand))

```

### Timmons with Model

<img src="/images/TimStrikeModel.png" alt="image">

If you’ve used pitchRx to create a database already, a SQL query via your R database package is a good solution because it allows you to pull ONLY the data you need for the analysis. The data I’m using here comes fro the follow query written directly into R via my database connection. I put this at the end of the article simply because SQL code isn’t the most exiting thing in the world but I thought it was important to include.

```
#Use SQL to isolate data frame
data = dbSendQuery(con,
                   "SELECT u.name AS umpName, a.batter_name, a.pitcher_name, a.stand, a.p_throws,
                   a.b AS b, a.s AS s, a.o AS o, a.b_height AS b_height,
                   p.x AS x, p.y AS y, p.start_speed AS start_speed, p.end_speed AS end_speed, p.sz_top AS sz_top, p.sz_bot AS sz_top, 
                   p.pfx_x AS pfx_x, p.pfx_z AS pfx_z, p.px AS px, p.pz AS pz, p.x0 AS x0, p.y0 AS y0, p.z0 AS z0, 
                   p.vx0 AS vx0, p.vy0 AS vy0, p.vz0 AS vz0, p.ax AS ax, p.ay AS ay, p.az AS az,
                   p.break_y AS break_y, p.break_angle AS break_angle, p.break_length AS break_length, 
                   p.pitch_type AS pitch_type, p.zone AS zone, p.nasty AS nasty, p.count AS count, p.des AS des,
                   u.id AS ump_id, a.pitcher AS pitcher_id, a.batter AS batter_id
                   
                   FROM atbat a
                   INNER JOIN umpire u
                   ON a.gameday_link = u.gameday_link
                   INNER JOIN pitch p
                   ON p.gameday_link = a.gameday_link
                   
                   WHERE u.position = 'home' AND a.date > '2014_07_01' AND a.date < '2014_09_01'")

#Fetch batting into data frame
umpZone= fetch(data, n = -1)
```