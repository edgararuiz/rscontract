
<!-- README.md is generated from README.Rmd. Please edit that file -->

# rscontract

<!-- badges: start -->

[![Travis build
status](https://travis-ci.org/edgararuiz/rscontract.svg?branch=master)](https://travis-ci.org/edgararuiz/rscontract)
[![Codecov test
coverage](https://codecov.io/gh/edgararuiz/rscontract/branch/master/graph/badge.svg)](https://codecov.io/gh/edgararuiz/rscontract?branch=master)
<!-- badges: end -->

Provides a generic implementation of the RStudio Connection Contract to
make it easier for database connections, and other type of connections,
opened via R packages integrate with the Connections Pane inside the
RStudio IDE.

## Installation

You can install the development version from
[GitHub](https://github.com/) with:

``` r
# install.packages("remotes")
remotes::install_github("edgararuiz/rscontract")
```

## Example

This is a basic example which shows you how to solve a common problem:

``` r
library(rscontract)

con <- rscontract_open(rscontract_spec())

rscontract_update(con)

rscontract_close(con)
```
