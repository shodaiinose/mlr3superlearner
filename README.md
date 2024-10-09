
<!-- README.md is generated from README.Rmd. Please edit that file -->

# mlr3superlearner

<!-- badges: start -->

[![Lifecycle:
experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://lifecycle.r-lib.org/articles/stages.html#experimental)
[![R-CMD-check](https://github.com/nt-williams/mlr3superlearner/actions/workflows/R-CMD-check.yaml/badge.svg)](https://github.com/nt-williams/mlr3superlearner/actions/workflows/R-CMD-check.yaml)
<!-- badges: end -->

An modern implementation of the [Super
Learner](https://biostats.bepress.com/ucbbiostat/paper266/) prediction
algorithm using the [mlr3](https://mlr3.mlr-org.com/) framework, and an
adherence to the recommendations of [Phillips, van der Laan, Lee, and
Gruber (2023)](https://doi.org/10.1093/ije/dyad023)

## Installation

You can install the development version of mlr3superlearner from
[GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("nt-williams/mlr3superlearner")
```

## Example

``` r
library(mlr3superlearner)
#> Loading required package: mlr3learners
#> Loading required package: mlr3
library(mlr3extralearners)
library(mlr3filters)
library(mlr3pipelines)
library(mlr3torch)
#> Loading required package: torch

# No hyperparameters
mlr3superlearner(mtcars, "mpg", c("mean", "glm", "svm", "ranger"), "continuous")
#> ══ `mlr3superlearner()` ════════════════════════════════════════════════════════
#>                       Risk Coefficients
#> regr.featureless 37.926212            0
#> regr.lm          11.399403            0
#> regr.ranger       6.094831            1
#> regr.svm         11.399788            0

# With hyperparameters

# Add layer of feature selection
filter <- po(
  "filter",
  filter = flt("selected_features", learner = lrn("regr.cv_glmnet")),
  filter.cutoff = 1
)

fit <- mlr3superlearner(mtcars, "mpg", 
                        list("mean", "xgboost", "svm", "earth",
                             list("glm", filter = filter),
                             list("nnet", trace = FALSE),
                             list("ranger", num.trees = 500, id = "ranger1"),
                             list("ranger", num.trees = 1000, id = "ranger2"), 
                             list("mlp", epochs = 50, validate = 0.3, batch_size = 8, 
                                  t_opt("adam", lr = 0.1), id = "mlp")), 
                        "continuous", 
                        discrete = FALSE)
#> Warning in warn_deprecated("Learner$initialize argument 'data_formats'"):
#> Learner$initialize argument 'data_formats' is deprecated and will be removed in
#> the future.

fit
#> ══ `mlr3superlearner()` ════════════════════════════════════════════════════════
#>                                  Risk Coefficients
#> regr.earth                   7.534602   0.04454516
#> regr.mean                   37.672850   0.00000000
#> regr.mlp                     9.452834   0.02973931
#> regr.nnet_and_trace_FALSE   38.512142   0.00000000
#> regr.ranger1                 5.625693   0.73099357
#> regr.ranger2                 5.765940   0.00000000
#> regr.svm                    11.264799   0.00000000
#> regr.xgboost               227.277041   0.00000000
#> selected_features.regr.glm   7.250849   0.19472197

head(data.frame(pred = predict(fit, mtcars), truth = mtcars$mpg))
#>       pred truth
#> 1 21.10296  21.0
#> 2 20.98607  21.0
#> 3 24.68845  22.8
#> 4 20.30662  21.4
#> 5 17.35348  18.7
#> 6 19.47186  18.1
```

## Available learners

``` r
knitr::kable(available_learners("binomial"))
```

| learner         | mlr3_learner         | mlr3_package      | learner_package |
|:----------------|:---------------------|:------------------|:----------------|
| mean            | classif.featureless  | mlr3              | stats           |
| glm             | classif.log_reg      | mlr3learners      | stats           |
| glmnet          | classif.glmnet       | mlr3learners      | glmnet          |
| cv_glmnet       | classif.cv_glmnet    | mlr3learners      | glmnet          |
| knn             | classif.kknn         | mlr3learners      | kknn            |
| nnet            | classif.nnet         | mlr3learners      | nnet            |
| lda             | classif.lda          | mlr3learners      | MASS            |
| naivebayes      | classif.naive_bayes  | mlr3learners      | e1071           |
| qda             | classif.qda          | mlr3learners      | MASS            |
| ranger          | classif.ranger       | mlr3learners      | ranger          |
| svm             | classif.svm          | mlr3learners      | e1071           |
| xgboost         | classif.xgboost      | mlr3learners      | xgboost         |
| earth           | classif.earth        | mlr3extralearners | earth           |
| lightgbm        | classif.lightgbm     | mlr3extralearners | lightgbm        |
| randomforest    | classif.randomForest | mlr3extralearners | randomForest    |
| bart            | classif.bart         | mlr3extralearners | dbarts          |
| c50             | classif.C50          | mlr3extralearners | C50             |
| gam             | classif.gam          | mlr3extralearners | mgcv            |
| gaussianprocess | classif.gausspr      | mlr3extralearners | kernlab         |
| glmboost        | classif.glmboost     | mlr3extralearners | mboost          |
| nloptr          | classif.avg          | mlr3pipelines     | nloptr          |
| rpart           | classif.rpart        | mlr3              | rpart           |
| mlp             | classif.mlp          | mlr3torch         | torch           |

``` r
knitr::kable(available_learners("continuous"))
```

| learner         | mlr3_learner      | mlr3_package      | learner_package |
|:----------------|:------------------|:------------------|:----------------|
| mean            | regr.featureless  | mlr3              | stats           |
| glm             | regr.lm           | mlr3learners      | stats           |
| glmnet          | regr.glmnet       | mlr3learners      | glmnet          |
| cv_glmnet       | regr.cv_glmnet    | mlr3learners      | glmnet          |
| knn             | regr.kknn         | mlr3learners      | kknn            |
| nnet            | regr.nnet         | mlr3learners      | nnet            |
| ranger          | regr.ranger       | mlr3learners      | ranger          |
| svm             | regr.svm          | mlr3learners      | e1071           |
| xgboost         | regr.xgboost      | mlr3learners      | xgboost         |
| earth           | regr.earth        | mlr3extralearners | earth           |
| lightgbm        | regr.lightgbm     | mlr3extralearners | lightgbm        |
| randomforest    | regr.randomForest | mlr3extralearners | randomForest    |
| bart            | regr.bart         | mlr3extralearners | dbarts          |
| gam             | regr.gam          | mlr3extralearners | mgcv            |
| gaussianprocess | regr.gausspr      | mlr3extralearners | kernlab         |
| glmboost        | regr.glmboost     | mlr3extralearners | mboost          |
| rpart           | regr.rpart        | mlr3              | rpart           |
| mlp             | regr.mlp          | mlr3torch         | torch           |
