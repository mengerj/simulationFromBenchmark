
<!-- README.md is generated from README.Rmd. Please edit that file -->

# simFrB

### Why this simulation Tool?

Simulated omics data often serves as a basis to evaluate or create
analysis tools, plan experiments and get a deeper understanding of the
data. Simulations usually make many assumptions about the distribution
of the data and often times simplify complex relationships observed in
experimental data.

One major challenge in simulating omics data is simulating missing value
patterns. Simulations might take data dependencies into consideration by
including values missing non at random, but we found that such
approaches still fail to reproduce results observed in experimental
data.

While trying to validate results of statistical analysis tools used in
Fröhlich et. al (), we observed a strong influence of the missing value
pattern on statistical results if data was prepossessed with imputation,
as done in the SAM method. While for experimental data, imputation
strongly increased the pAUC of all statistical tests, it decreased the
pAUC for simulated data.

This divergence was found to be caused by **feature dependent
correlations of the missing value pattern**, that got lost in other
simulation approaches. The implemented simulation tool leverages allows
to leverage benchmark data in order to simulate new count matrices with
realistic missing value patterns, by estimating coefficients for both
intensity (linear model) and missingness (logit model) for each feature
and drawing jointly to simulate data that keeps correlations found in
the experimental data.

## Usage

For now this package can be installed from this github repository

``` r
library(remotes)
remotes::install_github("kreutz-lab/simFrB")
#> Using GitHub PAT from the git credential store.
#> Skipping install of 'simulationFromBenchmark' from a github remote, the SHA1 (d7b958c5) has not changed since last install.
#>   Use `force = TRUE` to force installation
```

Generally there or two modes of operation:

1.  Use Available Data to estimate coefficients from. This will be most
    accurate for Benchmark data with known ground truth.

``` r
#exp.data <- load("pathToExpData")
#DE_idx <- grep("DE",rownames(exp.data)) #This needs to be adapted depending on how you know which feature is differentially expressed
```

1.  Use Pre calculated Coefficients stored in data/jointCoefs.rds
