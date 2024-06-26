
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
library(remotes, quietly = T)
remotes::install_github("kreutz-lab/simFrB")
#> Using GitHub PAT from the git credential store.
#> Downloading GitHub repo kreutz-lab/simFrB@HEAD
#> Warning in untar2(tarfile, files, list, exdir, restore_times): skipping pax
#> global extended headers
#> Warning in untar2(tarfile, files, list, exdir, restore_times): skipping pax
#> global extended headers
#> 
#> ── R CMD build ─────────────────────────────────────────────────────────────────
#>       ✔  checking for file 'C:\Users\mengerj\AppData\Local\Temp\RtmpicraMd\remotes9ec842a83188\kreutz-lab-simFrB-0f221f9/DESCRIPTION'
#>       ─  preparing 'simFrB':
#>    checking DESCRIPTION meta-information ...     checking DESCRIPTION meta-information ...   ✔  checking DESCRIPTION meta-information
#>       ─  checking for LF line-endings in source and make files and shell scripts
#>   ─  checking for empty or unneeded directories
#>       ─  building 'simFrB_0.0.0.9000.tar.gz'
#>      
#> 
#> Installiere Paket nach 'C:/Users/mengerj/AppData/Local/R/cache/R/renv/library/simFrB-61fc1892/R-4.3/x86_64-w64-mingw32'
#> (da 'lib' nicht spezifiziert)
```

Generally there or two modes of operation:

1.  Use Available Data to estimate coefficients from. This will be most
    accurate for Benchmark data with known ground truth.

``` r
library(simFrB)
#load Experimental Data
exp.df.all <-
  simFrB::loadDataFromGit(
    "https://raw.githubusercontent.com/kreutz-lab/dia-benchmarking/main/data/diaWorkflowResults_allDilutions.rds")

exp.df <- simFrB::subsetDIAWorkflowData(
  data = exp.df.all,
  DIAWorkflow = "DIANN_DIANN_AI_GPF",
  experimentalComparisonGroups = c("1-12","1-25"),
  rowSubset = seq(1,500),
  colSubset = c(seq(1,6),seq(24,29))
)

exp.df <- exp.df[rowSums(exp.df, na.rm = TRUE) > 0, ]
exp.mtx <- as.matrix(exp.df)
DE_idx <- grep("ECOLI",rownames(exp.df)) #This needs to be adapted depending on how you know which feature is differentially expressed

sim <- simFrB::msb.applyFunctionWithSeed(simFrB::msb.simulateDataFromBenchmark,
                                         seed = 123,
                                         mtx = exp.mtx,
                                      DE_idx = DE_idx,
                                      int.mean = mean(exp.mtx,na.rm = T),
                                      nFeatures = 1000,
                                      nSamples = 10)
#> No groupDesign for mtx provided, assuming first half of samples
#>                 are in group 1 and second half in group 2. If this is not the case
#>                 please provide groupDesign_mtx.
#> no nDE given, using default value of 15% of nFeatures
#> fitting from feature 1 to 454
#> using input data to estimate feature standard deviations.
#> By default DIMAR is assuming that the first half of the columns belong to group 1 and the second half to group 2.
#>           If this is not the case, please provide the group vector to the dimarConstructDesignMatrix function.
#> [1] "Pattern of MVs is learned by logistic regression."
#> applying dimar coefficients from 1 to 334
#> applying dimar coefficients from 335 to 667
#> applying dimar coefficients from 668 to 1000
```

1.  Use Pre calculated Coefficients stored in data/jointCoefs.rds

``` r
load("data/jointCoefs.rda")

sim <- simFrB::msb.simulateDataFromBenchmark(jointCoefs = jointCoefs,
                                      nFeatures = 6000,
                                      nSamples = 30)
#> No input matrix provided
#> no nDE given, using default value of 15% of nFeatures
#> applying dimar coefficients from 1 to 316
#> applying dimar coefficients from 317 to 632
#> applying dimar coefficients from 633 to 948
#> applying dimar coefficients from 949 to 1264
#> applying dimar coefficients from 1265 to 1579
#> applying dimar coefficients from 1580 to 1895
#> applying dimar coefficients from 1896 to 2211
#> applying dimar coefficients from 2212 to 2527
#> applying dimar coefficients from 2528 to 2843
#> applying dimar coefficients from 2844 to 3158
#> applying dimar coefficients from 3159 to 3474
#> applying dimar coefficients from 3475 to 3790
#> applying dimar coefficients from 3791 to 4106
#> applying dimar coefficients from 4107 to 4422
#> applying dimar coefficients from 4423 to 4737
#> applying dimar coefficients from 4738 to 5053
#> applying dimar coefficients from 5054 to 5369
#> applying dimar coefficients from 5370 to 5685
#> applying dimar coefficients from 5686 to 6000
```

``` r
head(sim[,,1])
#>              group1_1 group1_2 group1_3 group1_4 group1_5 group1_6 group1_7
#> feature_1    16.76577 17.36646 16.46576 17.67319 17.25391 17.30143       NA
#> feature_2_DE 21.82664 20.83489 21.55430 20.82666 20.97709 21.01400 21.40457
#> feature_3          NA 17.11768       NA 17.29566 15.11785 14.97760       NA
#> feature_4    20.91279 20.32987 20.22190 20.61534 20.06344 20.44953 20.76403
#> feature_5    15.67708 17.21497 16.74591 16.45736 16.61640 17.83990 17.89042
#> feature_6    23.51221 22.35637 22.47397 22.89102 22.87916 22.54169 23.20558
#>              group1_8 group1_9 group1_10 group1_11 group1_12 group1_13
#> feature_1          NA 17.34063        NA        NA  16.83116        NA
#> feature_2_DE 20.94151 20.86819  22.07049  21.31183  20.50197  20.66283
#> feature_3    17.86083 16.09678  16.45548  15.90363  14.91542  17.14560
#> feature_4    20.64517 20.43723  20.57914  20.72919  20.53584  20.68455
#> feature_5    17.08755 14.06102  16.65025  16.80553  16.64296        NA
#> feature_6    23.03450 23.16501  22.48665  22.28642  22.77655  22.72155
#>              group1_14 group1_15 group2_16 group2_17 group2_18 group2_19
#> feature_1           NA  15.83510  17.68981        NA  16.56912  17.68557
#> feature_2_DE  20.68945  21.61390  20.16146  19.12283  19.71076  19.78356
#> feature_3           NA        NA  14.97023  17.05489  14.34601  15.10124
#> feature_4     20.31077  19.70984  20.18567  20.41315  20.43196  20.95875
#> feature_5           NA        NA  16.42285  16.60812  15.26953  16.21896
#> feature_6     22.12506  22.46405  22.88906  21.98919  22.09698  23.06340
#>              group2_20 group2_21 group2_22 group2_23 group2_24 group2_25
#> feature_1     16.72052  17.14894  17.06193  15.58739  17.69400  17.35400
#> feature_2_DE  20.35814  20.98950  20.47576  19.49347  19.85025  18.87193
#> feature_3     16.38673  17.18397  14.29873  18.07476  17.78277        NA
#> feature_4     20.07099  20.33709  20.08741  20.56340  20.40731  20.68915
#> feature_5           NA  17.32024  16.16152  16.30670        NA  17.61262
#> feature_6     22.33384  22.54675  23.00266  22.37769  22.58796  23.24252
#>              group2_26 group2_27 group2_28 group2_29 group2_30
#> feature_1     17.56819  18.99654  17.46651  18.74488  17.63494
#> feature_2_DE  19.02931  19.34307  19.04407  19.93392  20.79556
#> feature_3     16.66164  15.96532        NA  15.78079  16.15334
#> feature_4     21.08401  20.41823  20.51154  20.53918  21.14499
#> feature_5     17.06050  16.17798  16.87317        NA  16.45592
#> feature_6     22.58044  23.15755  21.90542  22.36117  23.14612
```

The object returned by msb.simulateDataFromBenchmark contains the
original coefficients, as well as the drawn coefficients that were used
the simulate the given data as attributes.

``` r
str(attributes(sim)$expCoefs)
#> List of 3
#>  $ rowCoefs      :'data.frame':  8742 obs. of  7 variables:
#>   ..$ rowIDs           : chr [1:8742] "rowID1" "rowID2" "rowID3" "rowID4" ...
#>   ..$ lmCoefs          : num [1:8742] 4.7 3.87 2.73 3.31 2.97 ...
#>   ..$ lmCoefs_sd       : num [1:8742] 0.288 0.192 0.256 0.271 0.341 ...
#>   ..$ ifDE             : chr [1:8742] "notDE" "notDE" "notDE" "notDE" ...
#>   ..$ FCCoefs          : num [1:8742] 0 0 0 0 0 0 0 0 0 0 ...
#>   ..$ dimarCoefs_group1: num [1:8742] 4.63 3.63 3.67 3.88 4.09 ...
#>   ..$ dimarCoefs_group2: num [1:8742] 4.63 3.63 3.67 3.88 4.09 ...
#>   ..- attr(*, "dimarXtype")= num [1:1008] 0 1 2 2 2 2 2 2 2 2 ...
#>  $ colCoefs      :'data.frame':  46 obs. of  2 variables:
#>   ..$ lmCoefs   : num [1:46] 0 -0.0141 -0.0445 -0.0584 -0.0901 ...
#>   ..$ dimarCoefs: num [1:46] -7.53 -8.02 -7.2 -7.09 -7.92 ...
#>  $ dimarIntercept: Named num [1:2] 0 -3.86
#>   ..- attr(*, "names")= chr [1:2] "Intercept" "mean"
```

``` r
str(attributes(sim)$usedCoefs)
#> List of 3
#>  $ newLmCoefs   :List of 4
#>   ..$ featureCoefs: Named num [1:6000] -1.6 2.39 -2.63 1.57 -2.64 ...
#>   .. ..- attr(*, "names")= chr [1:6000] "rowID6126" "rowID2993_DE" "rowID727" "rowID134" ...
#>   ..$ sampleCoefs : Named num [1:30] -0.0542 -0.0846 -0.1324 -0.1713 -0.0602 ...
#>   .. ..- attr(*, "names")= chr [1:30] "colID45" "colID44" "colID23...3" "colID38...4" ...
#>   ..$ FCCoefs     : Named num [1:6000] 0 -1.4 0 0 0 ...
#>   .. ..- attr(*, "names")= chr [1:6000] "rowID6126" "rowID2993_DE" "rowID727" "rowID134" ...
#>   ..$ featureSd   : Named num [1:6000] 0.675 0.551 1.054 0.315 0.945 ...
#>   .. ..- attr(*, "names")= chr [1:6000] "rowID6126" "rowID2993_DE" "rowID727" "rowID134" ...
#>  $ newDimarCoefs: Named num [1:12032] 0 -3.86 -7.23 -7.55 -7.16 ...
#>   ..- attr(*, "names")= chr [1:12032] "Intercept" "mean" "colID45" "colID44" ...
#>   ..- attr(*, "xtype")= num [1:12032] 0 1 2 2 2 2 2 2 2 2 ...
#>  $ newDE_idx    : int [1:900] 2010 3137 4736 3844 4771 2912 2476 2063 1115 2094 ...
```
