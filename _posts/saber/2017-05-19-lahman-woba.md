---
layout: post
title: "HOW to Calculate wOBA in the Lahman Database"
categories: saber
tags: [sabermetrics, sql]
---



Weighted On-base Average (wOBA) is one of the stats du jour currently thrown around by baseball fanatics and sabermatricians alike. The formula for wOBA isn't difficult. According to <a href="http://www.fangraphs.com/library/offense/woba/" target="_blank">Fangraphs</a>, the wOBA formula for 2013 was:

$$\frac{(0.690*uBB + 0.722*HBP + 0.888*1B + 1.271*2B + 1.616*3B + 2.101*HR)} {(AB+BB-IBB+SF+HBP)}$$

## The Problem

If you're running a version of the <a href="http://www.seanlahman.com/baseball-archive/statistics/" target="_blank">Lahman Database</a>, this little equation is no problem since all the counting stats come from Lahman's 'Batting' table.

The one massive catch is, the wOBA scale, i.e. the numbers multiplied by the counting stats, changes every year. This makes it difficult to compare wOBA over several different years without writing a massive SQL string.

## The Solution

Fortunately Fangraphs also provides [this little beauty](http://www.fangraphs.com/guts.aspx?type=cn) that lists the wOBA scales for every year from 1871 to present, which is luckily the same data range as the Lehman Database!

* Download the csv file from Fangraphs and import it into your database schema as a separate table. Different databases have different methods for this but it's normally quite simple.

* Name your new table Guts (or whatever you want).

Presto! You can now compare wOBA values over several years with a single query. Assuming you named your new table “Guts” your SQL query should look something like this:


{% highlight sql %}
ELECT CONCAT(m.nameFirst, ' ', m.nameLast) AS Name,
b.playerID, b.yearID, b.AB, b.stint,
		(g.wBB*(b.BB-b.IBB)+g.wHBP*b.HBP+g.w1B*(b.H-b.2B-b.3B-b.HR)
		+g.w2B*b.2B+g.w3B*b.3B+g.wHR*b.HR)/(b.AB+b.BB-b.IBB+b.SF+b.HBP) wOBA

FROM Batting b
JOIN Guts g
ON g.yearID = b.yearID
JOIN Master m
ON m.playerID = b.playerID

WHERE b.yearID >= 2000 AND AB > 300 #The AB>300 gets rid of pitchers
GROUP BY b.playerID
ORDER BY wOBA DESC
{% endhighlight %}


## The Results

The below chart shows the wOBA leader since 2000 was Manny Ramirez, set during his monster season with the Cleveland Indians that year (the Fangraphs stats agree with this). It’s interesting to see that among the wOBA leaders from 2000 to 2013, the top of the list were all in the 2000 season. Interesting since the 2000 season was considered the “steroid era” and was before the MLB officially began testing for the drug in 2003.

To check, if you take out the yearID constraint from the WHERE clause, the all-time wOBA leader was Shoeless Joe Jackson in 1911 (hey Cleveland again!) and if you know your baseball you’ll know that Jackson’s 1911 season frequently tops the list for several batting stats.

![](https://github.com/keberwein/keberwein.github.io/blob/master/images/wOBA-1024x432.png?raw=true)
