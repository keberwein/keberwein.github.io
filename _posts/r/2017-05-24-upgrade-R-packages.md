---
layout: post
title: "Upgrade R Without Losing Your Packages"
image:
  feature: https://github.com/keberwein/keberwein.github.io/blob/master/images/rterminal1.png?raw=true
  thumb: https://github.com/keberwein/keberwein.github.io/blob/master/images/rterminal1.png?raw=true #keep it square 200x200 px is good
categories: r
tags: [r]
---


Since the first publication of this post, a couple of packages have emerged to automate this process. The [installr package]() for Windows and the [updateR package](https://github.com/AndreaCirilloAC/updateR) for OS X are particularly good. However, this continues to be a popular post, so I have decided to keep it up. This work-flow is short, sweet, and cross-platform.

**1. Before you upgrade, build a temp file with all of your old packages.**


{% highlight r %}
tmp <- installed.packages()
installedpkgs <- as.vector(tmp[is.na(tmp[,"Priority"]), 1])
save(installedpkgs, file="installed_old.rda")
{% endhighlight %}

**2. Install the new version of R and let it do it’s thing.**

**3. Once you’ve got the new version up and running, reload the saved packages and re-install them from CRAN.**


{% highlight r %}
tmp <- installed.packages()
installedpkgs.new <- as.vector(tmp[is.na(tmp[,"Priority"]), 1])
missing <- setdiff(installedpkgs, installedpkgs.new)
install.packages(missing)
update.packages()
{% endhighlight %}

**If you had any packages from BioConductor, you will need to reload those as well.**


{% highlight r %}
chooseBioCmirror()
biocLite() 
load("installed_old.rda")
tmp <- installed.packages()
installedpkgs.new <- as.vector(tmp[is.na(tmp[,"Priority"]), 1])
missing <- setdiff(installedpkgs, installedpkgs.new)
for (i in 1:length(missing)) biocLite(missing[i])
{% endhighlight %}

All done, now you can get back to cracking out R code. This method helped me save a lot of time, hope someone else finds it useful!

