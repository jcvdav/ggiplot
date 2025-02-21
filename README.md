
<!-- README.md is generated from README.Rmd. Please edit that file -->

# ggiplot

<!-- badges: start -->

[![R-universe status
badge](https://grantmcdermott.r-universe.dev/badges/ggiplot)](https://grantmcdermott.r-universe.dev)
[![Docs](https://img.shields.io/badge/docs-homepage-blue.svg)](https://grantmcdermott.com/ggiplot/index.html)
[![R-CMD-check](https://github.com/grantmcdermott/ggiplot/actions/workflows/R-CMD-check.yaml/badge.svg)](https://github.com/grantmcdermott/ggiplot/actions/workflows/R-CMD-check.yaml)
<!-- badges: end -->

This package provides a **ggplot2** equivalent of the base
[`fixest::iplot()`](https://lrberge.github.io/fixest/reference/coefplot.html)
function. The goal of **ggiplot** is to produce nice [event
study](https://theeffectbook.net/ch-EventStudies.html) plots with
minimal effort, but with lots of scope for further customization.

## Installation

The package is not yet on CRAN, but can be installed from R-universe:

``` r
install.packages("ggiplot", repos = "https://grantmcdermott.r-universe.dev")
```

## Quickstart

A detailed [introductory
vignette](http://grantmcdermott.com/ggiplot/articles/ggiplot.html) with
many examples is provided on the package homepage (or, by typing
`vignette("ggiplot")` in your R console). But here are a few quickstart
examples to whet your appetite. First, a basic event study plot.

``` r
library(ggiplot)
library(fixest)

est_did = feols(y ~ x1 + i(period, treat, 5) | id+period, base_did)

# iplot(est_did) ## base version
ggiplot(est_did) ## this package
```

<img src="man/figures/README-example1-1.png" width="100%" />

The above plot call and output should look very familiar to regular
**fixest** users. But note that `ggiplot()` supports several features
that are not available in the base `iplot()` version. For example,
plotting multiple confidence intervals and aggregate treatments effects.

``` r
ggiplot(
    est_did,
    ci_level = c(.8, .95),
    aggr_eff = "post", aggr_eff.par = list(col = "orange")
)
```

<img src="man/figures/README-example2-1.png" width="100%" />

And you can get quite fancy, combining lists of complex multiple
estimation objects with custom themes, and so on.

``` r
base_stagg_grp = base_stagg
base_stagg_grp$grp = ifelse(base_stagg_grp$id %% 2 == 0, 'Evens', 'Odds')

est_twfe_grp = feols(
    y ~ x1 + i(time_to_treatment, treated, ref = c(-1, -1000)) | id + year,
    data = base_stagg_grp, split = ~grp
)

est_sa20_grp = feols(
    y ~ x1 + sunab(year_treated, year) | id + year,
    data = base_stagg_grp, split = ~grp
)

ggiplot(
    list("TWFE" = est_twfe_grp, "Sun & Abraham (2020)" = est_sa20_grp),
    ref.line = -1,
    main = "Staggered treatment: Split mutli-sample",
    xlab = "Time to treatment",
    multi_style = "facet",
    geom_style = "ribbon",
    facet_args = list(labeller = labeller(id = \(x) gsub(".*: ", "", x))),
    theme = theme_minimal() +
        theme(
            text = element_text(family = "HersheySans"),
            plot.title = element_text(hjust = 0.5),
            legend.position = "none"
        )
)
```

<img src="man/figures/README-example3-1.png" width="100%" />
