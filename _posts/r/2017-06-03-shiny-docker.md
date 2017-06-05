---
layout: post
title: "Shiny Server on Docker: CentOS 7 Edition"
categories: r
tags: [r, shiny]
---


Docker is generally used for application development and deployment. While it is possible to develop and deploy Shiny applications in Docker containers, I have found it is much more useful to keep a Shiny Docker container that is a twin of my production server. This allows us to test new versions and new applications before putting them into production.

## Why Not Use a VM?

There are articles written all over the internet about this, so I don't want to rehash too much. From my perspective, the advantages of Docker as a test enviornment are:

* Faster start time (way faster!)

* Better performance. The ability to use system hardware instead of hardware abstraction.

* Disposable. If we make a mistake, it's much easier to delete the container and spin up a new one.

## Why CentOS?

Most enterprise production enviornments, that I'm aware of, use either RHEL or CentOS. Many Docker containers for Shiny and R use a Debian OS. The differences are minimal, however they are different. It doesn't seem logical to test in Debian and deploy in RHEL.

## The Rocker Project

Carl Boettiger and Dirk Eddelbuettel maintain the [rocker-org](https://github.com/rocker-org) project, which hosts several Docker containers relating to the world of R. Many of these containers can be fired up in only a couple of minutes. I highly recommend checking it out.

## Shiny on CentOS 

The following walk-through uses a docker image from my [GitHub repository](https://github.com/keberwein/docker_shiny-server_centos7). You can either do a git clone, or follow the link above and see the `README` section.


{% highlight bash %}
git clone https://github.com/keberwein/docker_shiny-server_centos7
{% endhighlight %}


### This configuration includes:

* R

* RStudio Server

* Shiny-Server

### Additional R Packages include:

* tidyverse

* plotly

* DT

## Setup

1. Install [Docker](https://docs.docker.com/engine/installation/) on your system.

2. Download or clone this repository.

### Build the Dockerfile


{% highlight bash %}
docker build /YOUR_PATH_TO/docker_shiny-server_centos7 --tag="shiny-server"
{% endhighlight %}

### View Your Docker Images


{% highlight bash %}
docker images
{% endhighlight %}

### Run your Shiny-Server Docker image.


{% highlight bash %}
docker run -p 3838:3838 -p 8787:8787 shiny-server
{% endhighlight %}

* Shiny-Server is running at localhost:3838

* RStudio Server is running at localhost:8787

* The username and password for RStudio Server is `rstudio`.

# Modify the Docker Container

This is a bare-bones container, so there is a good chance you will want to do some additional configuration. The command below will start your Docker instance and dump you into the root shell.


{% highlight bash %}
docker run -p 3838:3838 -p 8787:8787 -it shiny-server /bin/bash
{% endhighlight %}

* Arg -i tells docker to attach stdin to the container.

* Arg -t tells docker to give us a pseudo-terminal.

* Arg /bin/bash will run a terminal process in your container.

### Install Additional Stuff

Maybe you need a PostgreSQL instance?


{% highlight bash %}
yum install postgresql-devel -y
{% endhighlight %}

### Exit the Container


{% highlight bash %}
exit
{% endhighlight %}

### Find the Container ID


{% highlight bash %}
docker ps -a
{% endhighlight %}

### Use Docker Commit to Save

The syntax is:


{% highlight bash %}
docker commit [CONTAINER ID] [REPOSITORY:TAG]
{% endhighlight %}

It should look something like this:


{% highlight bash %}
docker commit b59185b5ba4b docker-shiny:shiny-server-v2
{% endhighlight %}

### See New Container


{% highlight bash %}
docker images
{% endhighlight %}
