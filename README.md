
<!-- README.md is generated from README.Rmd. Please edit that file -->
vqtl
====

[![Travis-CI Build Status](https://travis-ci.org/rcorty/vqtl.svg?branch=master)](https://travis-ci.org/rcorty/vqtl) [![codecov](https://codecov.io/gh/rcorty/vqtl/branch/master/graph/badge.svg)](https://codecov.io/gh/rcorty/vqtl) [![CRAN\_Status\_Badge](http://www.r-pkg.org/badges/version/vqtl)](https://CRAN.R-project.org/package=vqtl) [![CRAN\_Status\_Badge](http://cranlogs.r-pkg.org/badges/vqtl)](https://CRAN.R-project.org/package=vqtl)

The 'vqtl' package conducts QTL mapping using an elaboration of the traditional Haley-Knott model. It uses the double generalized linear model (DGLM) to model systematic effects on mean and variance. These systematic effects can be both "nuisance effects" that are uninteresting *per se*, but are valuable to "correct for" as well as genetic effects which are of immediate interest.

Installation
------------

You can install the current stable version of `vqtl` from CRAN with:

``` r
install.packages(pkgs = 'vqtl')
```

You can install newer version from github with:

``` r
install.packages("devtools")
devtools::install_github("rcorty/vqtl")
```

Typical usage
-------------

First we'll simulate an `rross` object, using a utility from `qtl`. Note that we load library `qtl` before library `vqtl`. This is necessary so that `vqtl::scanonevar` overrides `qtl::scanonevar` and not the other way around.

``` r
library(qtl)
library(vqtl)
#> 
#> Attaching package: 'vqtl'
#> The following object is masked from 'package:qtl':
#> 
#>     scanonevar

set.seed(27599)

test.cross <- qtl::sim.cross(map = qtl::sim.map(len = rep(20, 5), eq.spacing = FALSE))
```

We'll create two additional columns in the phenotype dataframe and calculate genotype probabilities at each pseudolocus using the Hidden Markov Model provided by `qtl`.

``` r
test.cross[['pheno']][['sex']] <- sample(x = c(0, 1),
                                         size = qtl::nind(test.cross),
                                         replace = TRUE)
test.cross[['pheno']][['sire']] <- factor(x = sample(x = 1:5,
                                                     size = qtl::nind(test.cross),
                                                     replace = TRUE))
test.cross <- qtl::calc.genoprob(cross = test.cross, step = 2)
```

Now that we have a `cross` object that's ready for analysis, we can use the `scanonevar` function to conduct a genome scan using the DGLM. Note that we use two formulas -- one for the mean and one for the variancee.

The mean formula must have one variable to the left of the `~` and that variable must be in `cross$pheno`. The mean and variance formula can have any number of variables to the right of the `~`. Valid variables are (a) in `cross$pheno`, (b) the name of a marker, or (c) a special keyword.

The special keywords for mean.formula are `mean.QTL.add` and `mean.QTL.dom`. The special keywords for var.formula are `var.QTL.add` and `var.QTL.dom`.

``` r
sov <- scanonevar(cross = test.cross,
                  mean.formula = phenotype ~ sex + D1M2 + mean.QTL.add + mean.QTL.dom,
                  var.formula = ~ sire + D2M3 + var.QTL.add + var.QTL.dom)
```
