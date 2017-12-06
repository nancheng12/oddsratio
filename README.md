
#### General

[![Project Status: Active – The project has reached a stable, usable state and is being actively developed.](http://www.repostatus.org/badges/latest/active.svg)](http://www.repostatus.org/#active) [![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.807719.svg)](https://doi.org/10.5281/zenodo.807719)

| Resource:     | CRAN                                                                                                                                                                     | Travis CI                                                                                                                                               | Appveyor                                                                                                                                                                      |
|---------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| *Platforms:*  | *Multiple*                                                                                                                                                               | *Linux & macOS*                                                                                                                                         | *Windows*                                                                                                                                                                     |
| R CMD check   | <a href="https://cran.r-project.org/web/checks/check_results_oddsratio.html"><img border="0" src="http://www.r-pkg.org/badges/version/oddsratio" alt="CRAN version"></a> | <a href="https://travis-ci.org/pat-s/oddsratio"><img src="https://travis-ci.org/pat-s/oddsratio.svg?branch=dev" alt="Build status"></a>                 | <a href="https://ci.appveyor.com/project/pat-s/oddsratio"><img src="https://ci.appveyor.com/api/projects/status/p72njexjxqa9ko96/branch/dev?svg=true" alt="Build status"></a> |
| Test coverage |                                                                                                                                                                          | <a href="https://codecov.io/gh/pat-s/oddsratio"><img src="https://codecov.io/gh/pat-s/oddsratio/branch/dev/graph/badge.svg" alt="Coverage Status"/></a> |                                                                                                                                                                               |

#### CRAN

[![CRAN\_Status\_Badge](http://www.r-pkg.org/badges/version/oddsratio)](https://cran.r-project.org/package=oddsratio) [![Downloads](https://cranlogs.r-pkg.org/badges/oddsratio?color=brightgreen)](https://www.r-pkg.org/pkg/oddsratio) ![](https://cranlogs.r-pkg.org/badges/grand-total/oddsratio)

Functions for calculation and plotting of odds ratios of Generalized Additive (Mixed) Models and Generalized Linear (Mixed) Models with a binomial response variable (i.e. logistic regression models).

Installation
------------

Install from CRAN:

``` r
install.packages("oddsratio")
```

Get the development version from Github:

``` r
remotes::install_github("pat-s/oddsratio@dev")
```

Examples
--------

### GLM

Odds ratio calculation of predictors `gre` & `gpa` of a fitted model `fit_glm` with increment steps of 380 and 5, respectively.
For factor variables (here: `rank` with 4 levels), automatically all odds ratios corresponding to the base level (here: `rank1`) are returned including their respective confident intervals. The default level is 95%. However, other levels can be specified with the param `CI`. Data source: <http://www.ats.ucla.edu/stat/r/dae/logit.htm>

``` r
pacman::p_load(oddsratio, mgcv)
df <- data_glm
df$rank <- factor(df$rank)
fit_glm <- glm(admit ~ gre + gpa + rank, data = df, family = "binomial")

or_glm(data = df, model = fit_glm, 
       incr = list(gre = 380, gpa = 5, CI = 0.95))
```

### GAM

For GAMs, the calculation of odds ratio is different. Due to its non-linear definition, odds ratios do only apply to specific value changes and are not constant throughout the whole value range of the predictor as for GLMs. Hence, odds ratios of GAMs can only be computed for one predictor at a time by holding all other predictors at a fixed value while changing the value of the specific predictor. Confident intervals are currently fixed to the 95% level for GAMs. Data source: `?mgcv::predict.gam()`

Here, the usage of `or_gam()` is shown by calculating odds ratios of pred `x2` for a 20% steps across the whole value range of the predictor.

``` r
set.seed(1234)
n <- 200
sig <- 2
df <- gamSim(1, n = n,scale = sig, verbose = FALSE)
df$x4 <- as.factor(c(rep("A", 50), rep("B", 50), rep("C", 50), rep("D", 50)))
fit_gam <- mgcv::gam(y ~ s(x0) + s(I(x1^2)) + s(x2) + offset(x3) + x4, data = df)

or_gam(data = df, model = fit_gam, pred = "x2",
       percentage = 20, slice = TRUE)
```

If you want to compute a single odds ratio for specific values, simply set param `slice = FALSE`:

``` r
or_gam(data = df, model = fit_gam,
       pred = "x2", values = c(0.099, 0.198))
```

Plotting of GAM smooths is also supported:

``` r
plot_gam(fit_gam, pred = "x2", title = "Predictor 'x2'")
```

<p align="center">
<img src="https://raw.githubusercontent.com/pat-s/oddsratio/dev/man/figures/plot_gam.png" width="80%"/>
</p>
Insert the calculated odds ratios into the smoothing function:

``` r
plot_object <- plot_gam(fit_gam, pred = "x2", title = "Predictor 'x2'")
or_object <- or_gam(data = df, model = fit_gam, 
                    pred = "x2", values = c(0.099, 0.198))

plot <- insert_or(plot_object, or_object, or_yloc = 3,
                  values_xloc = 0.04, line_size = 0.5, 
                  line_type = "dotdash", values_yloc = 0.5,
                  arrow_col = "red")
plot
```

<p align="center">
<img src="https://raw.githubusercontent.com/pat-s/oddsratio/dev/man/figures/plot_gam2.png" width="80%"/>
</p>
Insert multiple odds ratios into one smooth:

``` r
or_object2 <- or_gam(data = df, model = fit_gam, pred = "x2", 
                     values = c(0.4, 0.6))

insert_or(plot, or_object2, or_yloc = 2.1, values_yloc = 2,
          line_col = "green4", text_col = "black",
          rect_col = "green4", rect_alpha = 0.2,
          line_alpha = 1, line_type = "dashed",
          arrow_xloc_r = 0.01, arrow_xloc_l = -0.01,
          arrow_length = 0.01, rect = T)   
```

<p align="center">
<img src="https://raw.githubusercontent.com/pat-s/oddsratio/dev/man/figures/plot_gam3.png" width="80%"/>
</p>
