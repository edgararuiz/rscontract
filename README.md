
<!-- README.md is generated from README.Rmd. Please edit that file -->

# rscontract

<!-- badges: start -->

[![Lifecycle:
maturing](https://img.shields.io/badge/lifecycle-maturing-blue.svg)](https://www.tidyverse.org/lifecycle/#maturing)
[![Travis build
status](https://travis-ci.org/rstudio/rscontract.svg?branch=master)](https://travis-ci.org/rstudio/rscontract)
[![Codecov test
coverage](https://codecov.io/gh/rstudio/rscontract/branch/master/graph/badge.svg)](https://codecov.io/gh/rstudio/rscontract?branch=master)
[![CRAN
status](https://www.r-pkg.org/badges/version/rscontract)](https://CRAN.R-project.org/package=rscontract)
<!-- badges: end -->

  - [Intro](#intro)
  - [Functions](#functions)
  - [Installation](#installation)
  - [Examples](#examples)
      - [Basic example](#basic-example)
      - [Modified spec](#modified-spec)
      - [Action buttons](#action-buttons)
      - [From a file](#from-a-file)
      - [R code as a value](#r-code-as-a-value)
      - [More than one schema](#more-than-one-schema)

## Intro

Makes it easier for other R packages or R projects to integrate with the
RStudio Connections Contract. It provides several options to describe
the structure of your connection. One of the options provided by
`rscontract` is to use a YAML file that can contain the structure of the
connection, and easily convert that into a proper RStudio Connections
contract with a couple of lines of
code:

<!--html_preserve-->

<img src='man/figures/yamltocontract.png' width ='1000px'/><br/><!--/html_preserve-->

## Functions

It provides two levels of integration abstraction with the Connections
pane:

1.  `rscontract_spec()` (higher) - Enables the user to pass a
    hierarchical list to describe the structure of the connection
    (catalog/schema/table). The defaults are setup so you can open a
    very simple connection without changing any arguments. The idea is
    to allow you to easily iterate through small argument changes as you
    are learning how the Connection pane works.
2.  `rscontract_ide()` (lower) - The arguments of this function matches
    one-to-one with the expected entries needed to open a Connection
    pane.

The `as_rscontract()` function converts a variable into the same format
that `rscontract_ide()` returns. This makes it possible for objects such
as `list`s returned by `yaml::read_yaml()` to work.

There are three functions that actually interact with the RStudio IDE:

1.  `rscontract_open()` - Opens the Connection pane. It requires a
    properly formatted Connections pane provided by the
    `rscontract_spec()`, `rscontract_ide()`, or `as_rscontract()`
    functions.
2.  `rscontract_update()` - Refreshes the already opened Connections
    pane
3.  `rscontract_close()` - Closes the Connections pane.

## Installation

You can install the development version from
[GitHub](https://github.com/) with:

``` r
# install.packages("remotes")
remotes::install_github("rstudio/rscontract")
```

## Examples

### Basic example

The stock output of `rscontract_spec()` is loaded into a variable
`spec`. This way it is possible to display its contents before using it
to open a new connection.

``` r
library(rscontract)
```

``` r
library(rscontract)

spec <- rscontract_spec()
str(spec)
#> List of 13
#>  $ connection_object: NULL
#>  $ type             : chr "spec_type"
#>  $ host             : chr "spec_host"
#>  $ icon             : NULL
#>  $ name             : chr ""
#>  $ connect_script   : chr "library(connections)\n[Place your code here]"
#>  $ disconnect_code  : chr "function() rscontract_close('spec_host', 'spec_type')"
#>  $ preview_code     : chr "function(){}"
#>  $ catalog_list     : chr "sample_catalog()"
#>  $ object_types     : chr "default_types()"
#>  $ object_list      : NULL
#>  $ object_columns   : NULL
#>  $ actions          : NULL
#>  - attr(*, "class")= chr "rscontract_spec"
```

The connection can now be opened with `spec`.

``` r
rscontract_open(spec)
```

Notice above the values of the `type` and `host` entries inside `spec`.
Those are the two pieces of information needed by RStudio to identify
the connection that needs to be updated, or
closed.

<!--html_preserve-->

<img src='man/figures/open-1.png' width ='400px'/><br/><!--/html_preserve-->

`rscontract` comes with a basic example accessible via the
`sample_catalog()` function. By default, `rscontract_spec()` uses
`sample_catalog()` in the `object_types` entry to automatically give you
working sample Connections pane.

``` r
rscontract_update("spec_host", "spec_type")
```

After closing the connection, the content from the `connect_script`
variable can be
seen.

``` r
rscontract_close("spec_host", "spec_type")
```

<!--html_preserve-->

<img src='man/figures/closed-1.png' width ='400px'/><br/><!--/html_preserve-->

### Modified spec

To start creating your own connection setup, simply modify the arguments
in `rscontract_spec()` that you wish to test. Here is an example of a
few modifications that are possible to make:

``` r
spec <- rscontract_spec(
  type = "my_type",
  host = "my_host", 
  icon = system.file("images", "rstudio-icon.png", package = "rscontract"),
  name = "Modified Name",
  connect_script = "[This is my connection code]",
  disconnect_code = "function() rscontract_close('my_host', 'my_type')",
  preview_code = "function(catalog, schema, table, limit) data.frame(catalog, schema, table, limit)"
)

rscontract_open(spec)
```

<!--html_preserve-->

<img src='man/figures/modified.png' width ='800px'/><br/><!--/html_preserve-->

### Action buttons

The Connections pane also give you the ability to add custom buttons at
the top of the pane. These can be setup to run a specific R instruction
once clicked. To add them simply modify the `action` entry in the spec.
In the example below, “hello” is sent to the R Console when ‘Button 1’
is clicked.

``` r
spec$actions <- list(
  "Button 1" = list(
    icon = system.file("images", "rstudio-icon.png", package = "rscontract"),
    callback = function() print("hello")
  )
)

rscontract_open(spec)
```

<!--html_preserve-->

<img src='man/figures/button-1.png' width ='400px'/><br/><!--/html_preserve-->

``` r
#> rscontract_open(spec)
#> [1] "hello"
```

To add flexibility, wrap the list preparation inside a function:

``` r
spec_function <- function(x, message) {
  x$actions <- list(
  "Button 1" = list(
    icon = system.file("images", "rstudio-icon.png", package = "rscontract"),
    callback = function() print(message)
  ))
  x
}

rscontract_open(spec_function(spec, "test"))
```

``` r
#> rscontract_open(spec_function(spec, "test"))
#> [1] "test"
```

``` r
rscontract_close("my_host", "my_type")
```

### From a file

A YAML file can be used to create the connection. The structure and name
of each field has to match to what is expected by `rscontract`. The
example below shows a basic example of the names and the expected type
of input. By default, the content in the following fields will be
evaluated is if it was R code:

  - `disconnect_code`
  - `preview_code`

Here is an sample file included as part of the `rscontract` package:

``` yaml
name: My Connection
type: my_connection
host: hosted_here
connect_script: my_function_connect(path = "myfile.txt")
disconnect_code:  function() rscontract_close("hosted_here", "my_connection")
preview_code: function(table, view, ...) c(table, view)
catalog_list:
  catalogs:
    name: my_catalog
    type: catalog
    schemas:
      - name: my_schema
        type: schema
        tables:
          - name: my_table
            type: table
            fields:
              - name: field1
                type: nbr
              - name: field2
                type: chr
```

The key of using a YAML file, is to coerce it into a contract format
using `as_rscontract()`. Then use `rscontract_open()` to start the
connection.

``` r
# Obtains the path to the sample YAML file
contract_file <- system.file("specs", "simple.yml", package = "rscontract")
# Reads the YAML file using the `yaml` package
contract <- yaml::read_yaml(contract_file)
# Coerces list into a contract spec
spec <- as_rscontract(contract)
# Opens the connection
rscontract_open(spec)
```

<!--html_preserve-->

<img src='man/figures/yaml-1.png' width ='400px'/><br/><!--/html_preserve-->

### R code as a value

In order to pass R code instead of the value, then use a sub-entry
called `code` for the entry you wish to modify.

``` yaml
name:
  code: toupper("my_title")
type:
  code: tolower("TYPE")
host: host
connect_script: Place connection code here
disconnect_code:  function() rscontract_close("host", "type")
preview_code: function(table, view, ...) c(table, view)
catalog_list:
  catalogs:
    name: my_catalog
    type: catalog
    schemas:
      - name: my_schema1
        type: schema
        tables:
          - name: my_view1
            type: view
            fields:
              code: list(list(name = "ext_function", type = "int"))
```

``` r
contract_file <- system.file("specs", "full.yml", package = "rscontract")
contract <- yaml::read_yaml(contract_file)
spec <- as_rscontract(contract)
rscontract_open(spec)
```

<!--html_preserve-->

<img src='man/figures/yaml-2.png' width ='400px'/><br/><!--/html_preserve-->

### More than one schema

Here is an example of a Contract with multiple Schemata:

``` yaml
name: displayName
type: type
host: host
connect_script: Place connection code here
disconnect_code:  function() rscontract_close("host", "type")
preview_code: function(table, view, ...) c(table, view)
catalog_list:
  catalogs:
    name: my_catalog
    type: catalog
    schemas:
      - name: my_schema1
        type: schema
        tables:
          - name: my_table1
            type: table
            fields:
              - name: field1
                type: nbr
              - name: field2
                type: chr
          - name: my_view1
            type: view
            fields:
              - name: field3
                type: nbr
              - name: field4
                type: chr
      - name: my_schema2
        type: schema
        tables:
          - name: my_table4
            type: table
            fields:
              - name: field5
                type: nbr
              - name: field6
                type: chr
          - name: my_view2
            type: view
            fields:
              - name: field7
                type: nbr
              - name: field8
                type: chr
```

``` r
contract_file <- system.file("specs", "two-schemas.yml", package = "rscontract")
contract <- yaml::read_yaml(contract_file)
spec <- as_rscontract(contract)
rscontract_open(spec)
```

<!--html_preserve-->

<img src='man/figures/yaml-3.png' width ='400px'/><br/><!--/html_preserve-->
