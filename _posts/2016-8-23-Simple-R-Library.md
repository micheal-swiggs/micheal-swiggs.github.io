---
layout: post
title: Creating a Simple R Library
---

Over time I created a lot of scripts that I use throughout a project. The method I use at the moment is to use `source("../somescript.R")` throughout a project. Obviously this is a bit backwards. So here is some notes for creating a simple R lib.



So start up R in the directory where you want to store you libraries. For me this is something like `/home/user/rlibs`.

Then make sure that you have the `devtools` package installed:

    install.packages("devtools")
    library(devtools)

Assuming that package is called `kangaroo` then create it with:

    create("kangaroo")

This will autogenerate the necessary folders/files in `/home/user/rlibs/kangaroo`:

    kangaroo/
        DESCRIPTION
        man/
        R/
            kangaroo-package.r

The next step is to move your functions to `kangaroo/R`. I had one function that I wanted to move from a script into the library:

    # In kangaroo/R
    touch hop.R

Within `hop.R` I have the following:

    hop <- function(){
        print("hop")
    }

*Installing* is simple, back in `/home/user/rlibs` I run the following:

    install("kangaroo")

and this installs the package locally!


Links

*https://hilaryparker.com/2014/04/29/writing-an-r-package-from-scratch/*
