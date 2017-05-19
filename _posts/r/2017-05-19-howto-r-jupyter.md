---
layout: post
title: "How to Install R Kernel for Jupyter on Mac OS X"
categories: r
tags: [r, python, jupyter]
---



IPython is a great tool for developers, particularly for R programmers who are accustomed to the luxury of running blocks of code during development. The ability to add an R kernel to the IPython environment gives one the ability to run Python and R side-by-side in the same programming environment.

![](https://github.com/keberwein/keberwein.github.io/blob/master/images/jupyter_small.png?raw=true)

### Update: This install method is less involved

Get zmq dependencies. Note: Make sure you’ve got Xcode installed.

*If you use Homebrew:*


{% highlight bash %}
xcode-select --install 
brew install zmq
{% endhighlight %}

*Or, if you use MacPorts*


{% highlight bash %}
sudo port install zmq
export CPATH=/opt/local/include 
export LIBRARY_PATH=/opt/local/lib
{% endhighlight %}

Next, fire up R, install from source and start your kernel.


{% highlight r %}
install.packages(c('rzmq','repr','IRkernel','IRdisplay'),
                 repos = c('http://irkernel.github.io/', getOption('repos')), type = 'source')
IRkernel::installspec(user = FALSE)
{% endhighlight %}

That should work. If not, the instructions below show you how to clone the IRkernel GitHub repo and install from source on your local machine.

### My original method: If the above method doesn't work, you may have more luck here.

*If you use Homebrew:*


{% highlight bash %}
brew install libzmq3
brew install czmq zmq
{% endhighlight %}

Assuming that those libraries brewed without any errors, start R in your terminal by typing “R” or fire up R-Studio. Install these three packages. Note, it may be a good idea to install them one at a time. Note, since the rzmq package includes dependencies, we’ll be cloning the GitHub repo and installing it locally.


{% highlight bash %}
git clone https://github.com/armstrtw/rzmq.git --recursive
{% endhighlight %}

Make sure to place the file in your R working directory. Then in R:


{% highlight r %}
library(RCurl)
library(devtools)

install_local('./rzmq')  

install_github('IRkernel/repr')

install_github("IRkernel/IRdisplay")

install_github("IRkernel/IRkernel")
{% endhighlight %}

At this point the R kernel should work (in theory) by executing the installspec() function from your new IRkernel package but…

In my case, installspec() wouldn’t fire up, so I did a little detective work. Run the following command in R to find the path IRkernel is hitting.


{% highlight r %}
print(system.file("kernelspec", package = "IRkernel"))
{% endhighlight %}

Chances are the package is sending the R kernel to somewhere like *“/Library/Frameworks/R.framework/Versions/3.1/Resources/library/IRkernel/kernelspec”*.

If that is the case, then you’ve quickly found the problem that took me hours of detective work to track down.

In that case, there is a simple work-around. In your terminal type:


{% highlight bash %}
ipython kernelspec install --replace --name ir --user /Library/Frameworks/R.framework/Versions/3.1/Resources/library/IRkernel/kernelspec

{% endhighlight %}

After you run that in terminal, go back into R and run:


{% highlight r %}
library(IRdisplay)
library(IRkernel) 

installspec()
{% endhighlight %}

At this point you should be set to go. Fire up your terminal one more time, throw the IPython command and keep your fingers crossed!


{% highlight bash %}
ipython notebook
{% endhighlight %}

My environment = OS X 10.10, R 3.1, Python 3, your results may vary!

![](https://github.com/keberwein/keberwein.github.io/blob/master/images/jupyter_large.png?raw=true)
