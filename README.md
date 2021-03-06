## rmote

![gif](https://cloud.githubusercontent.com/assets/1275592/9618810/99cf5b2c-50be-11e5-885d-a4de16919271.gif)

### Running R on a remote server

R users often find themselves needing to log in to a remote machine to do analysis.  Sometimes this is due to data restrictions, computing power on the remote machine, etc.  Users can ssh in and run R in a terminal, but it is not possible to look at graphics, etc.

There are currently three approaches that I am aware of to deal with this:

1. Install [RStudio Server](https://www.rstudio.com/products/rstudio-server-pro/) on the remote server and use that from a web browser on your local machine.  Graphics output is shown in the IDE.
2. Use X11 forwarding (`ssh -X|Y`).  Graphics output is sent back to your machine.
3. Use VNC with a linux desktop environment like KDE or GNOME.

Whenever possible, #1 is by far the best way to go and is one of the beautiful things about RStudio Server.  #2 is not a good choice - plots are very slow to render, the quality is terrible, and it doesn't support recent advances in plot outputs like [htmlwidgets](http://htmlwidgets.org) (unless you launch firefox through X11, which will mean you might get to look at one plot per hour).  #3 is okay if it is available and you are comfortable working in one of these desktop environments.

There could be other obvious ways to deal with this that I am oblivious to.

### A problem

Often we do not have the choice of installing RStudio Server or a desktop environment on the remote machine.  Also, some users prefer to work in a terminal sending commands from a favorite text editor on a local machine, but still want to see graphics.  We would like to have something better than X11 forwarding to view graphics and other output when running R in a terminal on a remote machine.

### A solution

The rmote package is an attempt to make working in R over ssh on a server a bit more pleasant in terms of viewing output.  It uses [servr](https://github.com/yihui/servr) on the remote machine to serve R graphics as they are created.  These can be viewed on the local machine in a web browser. The user's local browser will automatically refresh each time a new output is available.

Currently there is support for lattice, ggplot2, htmlwidgets, and help output.

### Usage

1. Choose a port to run your remote server on (default is 4321)
2. ssh into the remote machine, mapping the port on the remote back to your local machine:

    ```
    ssh -L 4321:localhost:4321 -L 8100:localhost:8100 user@remote
    ```

    I also add port 8100 so I can forward shiny apps back to my local machine on a dedicated port.

3. On the remote machine launch R and install the latest version of servr and rmote (one time only)

    ```r
    install.packages("rmote", repos = c(CRAN = "http://cran.rstudio.com",
      tessera = "http://packages.tessera.io"))
    ```

    or alternatively

    ```r
    devtools::install_github(c("yihui/servr", "hafen/rmote"))
    ```

    Note that this package will probably never be on CRAN since it overwrites some standard R S3 methods.

4. Run the following in R on the remote:

    ```r
    rmote::start_rmote()
    ```

    To view some of the options for this, see `?start_rmote`.  One option is the port, which needs to match the one your forwarded in step 2 (4321 is the default.)

5. On your local machine, open up your web browser to `localhost:4321`

    Now as you create compatible plots on your remote machine, your browser on your local fmachine will automatically update to show the results.  For example, try running each of the following in succession on the server:

    ```r
    ?plot

    qplot(mpg, wt, data=mtcars, colour=cyl)

    dotplot(variety ~ yield | year * site, data=barley)

    library(rbokeh)
    figure() %>% ly_hexbin(rnorm(10000), rnorm(10000))
    ```

This process is slightly more tedious than `ssh -X` for initial setup, but much faster and more functional.

### Other useful utilities

Another interesting package to check out that allows you to control R on a remote machine from your local R session is - [remoter](https://cran.r-project.org/web/packages/remoter/vignettes/remoter.html).

If you must work on a remote terminal, here are some additional utilities that help make things nice:

- [colorout](https://github.com/jalvesaq/colorout)
- [Sublime](https://www.sublimetext.com) + [SFTP](http://wbond.net/sublime_packages/sftp)
- [Vim](http://www.vim.org) + [Vim-R-plugin](https://github.com/vim-scripts/Vim-R-plugin)


## Installation ##

[![CRAN](http://www.r-pkg.org/badges/version/rmote)](http://cran.r-project.org/package=rmote)
[![Build Status](https://travis-ci.org/cloudyr/rmote.png?branch=master)](https://travis-ci.org/cloudyr/rmote)
[![codecov.io](http://codecov.io/github/cloudyr/rmote/coverage.svg?branch=master)](http://codecov.io/github/cloudyr/rmote?branch=master)

This package is not yet on CRAN. To install the latest development version you can install from the cloudyr drat repository:

```R
# latest stable version
install.packages("rmote", repos = c(getOption("repos"), "http://cloudyr.github.io/drat"))
```

Or, to pull a potentially unstable version directly from GitHub:

```R
if(!require("ghit")){
    install.packages("ghit")
}
ghit::install_github("cloudyr/rmote")
```


---
[![cloudyr project logo](http://i.imgur.com/JHS98Y7.png)](https://github.com/cloudyr)
