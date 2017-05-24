---
layout: post
title: "How to Install R Ubuntu 16.04 Xenial"
categories: r
tags: [r]
---



The long-awaited new Ubuntu LTS Xenial Xerus was released last week. I wrote a tutorial on installing R and R-Studio on the old 14.04 LTS, so I figured I’d update that document. Not much has changed for the new 16.04 version but there are new repositories.

### Install R-Base

You can find R-Base in the Software Center; this would be the easy way to do it. However, the Software Center versions are often out of date, which can be a pain moving foward when your packages are based on the most current version of R Base. The easy fix is to download and install R Base directly from the Cran servers.

### Add R repository

First, we’ve got to add a line to our /etc/apt/sources.list file. This can be accomplished with the following. Note the “xenial” in the line, indicating Ubuntu 16.04. If you have a different version, just change that.


{% highlight bash %}
sudo echo "deb http://cran.rstudio.com/bin/linux/ubuntu xenial/" | sudo tee -a /etc/apt/sources.list

{% endhighlight %}

### Add R to Ubuntu Keyring

First


{% highlight bash %}
 gpg --keyserver keyserver.ubuntu.com --recv-key E084DAB9

{% endhighlight %}

Then


{% highlight bash %}
 gpg -a --export E084DAB9 | sudo apt-key add -

{% endhighlight %}

### Install R Base


{% highlight bash %}
sudo apt-get update
sudo apt-get install r-base r-base-dev
{% endhighlight %}

### Install RStudio

RStudio is not currently in the Software Center, but that may contain an outdated version. It can easily be installed manually.


{% highlight bash %}
sudo apt-get install gdebi-core
wget https://download1.rstudio.org/rstudio-1.0.143-amd64.deb
sudo gdebi -n rstudio-1.0.143-amd64.deb
rm rstudio-1.0.143-amd64.deb
{% endhighlight %}


![](https://github.com/keberwein/keberwein.github.io/blob/master/images/ubuntuRtem.png?raw=true)
