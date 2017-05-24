---
layout: post
title: "How to Pimp Your Rprofile"
categories: r
tags: [r]
---



After you’ve been using R for a little bit, you start to notice people talking about their .Rprofile as if it’s some mythical being. Nothing magical about it, but it can be a big time-saver if you find yourself typing things like, `summary()` or, the ever-hated, `stringasfactors=FALSE`, ad nauseam.

### Where is my .Rprofile?

The simple answer is, if you don’t know, then you probably don’t have one. R-Studio doesn’t include one unless you tell it to. In Mac and Linux the .Rprofile is usually a hidden file in your user’s home directory. In Windows the most common place is `C:\Program Files\R\Rx.x\etc`.

### Check to see if I have an .Rprofile

Before creating a new profile, fire up R and check to see if you have an existing .Rprofile lying around. Like I said, it’s usually a hidden file.


{% highlight r %}
c(Sys.getenv("R_PROFILE_USER"), file.path(getwd(),".Rprofile"))
{% endhighlight %}

### How to create an .Rprofile

Assuming you don’t already have one, these files are easy to create. Open a text editor and name your blank file .Rprofile with no trailing extension and place it in the appropriate directory. After populating the file, you’ll have to restart R for the settings to take affect.

### Sample .Rprofile

Below is a snapshot of mine. Of coarse, you can make this as simple or as complex as you like.



{% highlight r %}
## Print this on start so I know it's loaded.
cat("Loading custom .Rprofile")

## A little gem from Hadley Wickam that will set your CRAN mirror and automatically load devtools in interactive sessions.
.First <- function() {
  options(
    repos = c(CRAN = "https://cran.rstudio.com/"),
    setwd("~/Documents/R"),
    deparse.max.lines = 2)
}

if (interactive()) {
  suppressMessages(require(devtools))
}

## Nice option for local work. I keep it commented out so my code can remain portable.
## options(stringsAsFactors=FALSE)

## Increase the size of my Rhistory file, becasue I like to use the up arrow!
Sys.setenv(R_HISTSIZE='100000')

## Create invisible environment ot hold all your custom functions
.env <- new.env()

## Single character shortcuts for summary() and head().
.env$s <- base::summary
.env$h <- utils::head

#ht==headtail, i.e., show the first and last 10 items of an object.
.env$ht <- function(d) rbind(head(d,10),tail(d,10))

## Read data on clipboard.
.env$read.cb <- function(...) {
  ismac <- Sys.info()[1]=="Darwin"
  if (!ismac) read.table(file="clipboard", ...)
  else read.table(pipe("pbpaste"), ...)
}

## List objects and classes.
.env$lsa <- function() {
{
    obj_type <- function(x) class(get(x, envir = .GlobalEnv)) # define environment
    foo = data.frame(sapply(ls(envir = .GlobalEnv), obj_type))
    foo$object_name = rownames(foo)
    names(foo)[1] = "class"
    names(foo)[2] = "object"
    return(unrowname(foo))
}

## List all functions in a package.
.env$lsp <-function(package, all.names = FALSE, pattern) {
    package <- deparse(substitute(package))
    ls(
        pos = paste("package", package, sep = ":"),
        all.names = all.names,
        pattern = pattern
    )
}

## Open Finder to the current directory. Mac Only!
.env$macopen <- function(...) if(Sys.info()[1]=="Darwin") system("open .")
.env$o       <- function(...) if(Sys.info()[1]=="Darwin") system("open .")


## Attach all the variables above
attach(.env)

## Finally, a function to print out all the functions you have defined in the .Rprofile.
print.functions <- function(){
	cat("s() - shortcut for summary\n",sep="")
	cat("h() - shortcut for head\n",sep="")
	cat("read.cb() - read from clipboard\n",sep="")
	cat("lsa() - list objects and classes\n",sep="")
	cat("lsp() - list all functions in a package\n",sep="")
	cat("macopen() - open finder to current working directory\n",sep="")
}
{% endhighlight %}

### Limitations and gotchas

The major disadvantage to all this is code portability. For example, if you set your .Rprofile to load `dplyr` on every session, when someone else tries to run your code, it won’t work. For this reason, I’m a little picky about my settings, opting for functions that will only be used in interactive sessions.
