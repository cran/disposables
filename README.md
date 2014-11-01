


# Disposable R packages, for testing purposes

[![Linux Build Status](https://travis-ci.org/gaborcsardi/disposables.png?branch=master)](https://travis-ci.org/gaborcsardi/disposables)
[![Windows Build status](https://ci.appveyor.com/api/projects/status/github/gaborcsardi/disposables)](https://ci.appveyor.com/project/gaborcsardi/disposables)

## Features

The disposable packages are installed in R's temporary directory,
so they are cleaned up at the end of the R session.

`disposables` cleans up after itself, if an error happens during the
installation or loading of the disposable packages. If `make_packages()`
fails because of an error, it leaves to temporary garbage behind. In
particular,
* it cleans up the library path and restores `.libPaths()`,
* removes the temporary source package directories,
* removes the installes packages from `lib_dir`, and
* unloads the packages that it loaded before the error.

## Installation

You can install this R package from Github:


```r
devtools::install_github("gaborcsardi/disposables")
```

## Usage

`make_packages()` creates, installs and loads R packages, it takes named
expressions, the names will be used as package names.


```r
library(disposables)
```

```
#> Loading required package: methods
```

```r
pkgs <- make_packages(
  foo1 = { f <- function() print("hello!") ; d <- 1:10 },
  foo2 = { f <- function() print("hello again!") ; d <- 11:20 }
)
```

The `foo1` and `foo2` packages are now loaded.


```r
"package:foo1" %in% search()
```

```
#> [1] TRUE
```

```r
"package:foo2" %in% search()
```

```
#> [1] TRUE
```

You can dispose them with `dispose_packages()`. This unloads the packages
and deletes them from the library directory.


```r
dispose_packages(pkgs)
"package:foo1" %in% search()
```

```
#> [1] FALSE
```

```r
"package:foo2" %in% search()
```

```
#> [1] FALSE
```

```r
file.exists(pkgs$lib_dir)
```

```
#> [1] FALSE
```

Here is a real example that tests cross-package inheritence of
[R6 classes](https://github.com/wch/R6).


```r
library(disposables)
library(testthat)
test_that("inheritance works across packages", {

  pkgs <- make_packages(
    imports = "R6",

    ## Code to put in package 'R6testA'
    R6testA = {
      AC <- R6Class(
        public = list(
          x = 1
        )
      )
    },

    ## Code to put in package 'R6testB'
    R6testB = {
      BC <- R6Class(
        inherit = R6testA::AC,
        public = list(
          y = 2
        )
      )
    }

  )

  ## In case of an error below
  on.exit(try(dispose_packages(pkgs), silent = TRUE), add = TRUE)

  ## Now ready for the tests
  B <- BC$new()
  expect_equal(B$x, 1)
  expect_equal(B$y, 2)

})
```

## License

MIT
