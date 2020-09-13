# Dominance Analysis Extended to Multiple Equation Models
## Parameter Estimate Relative Importance (PERI)

Dominance analysis (DA) determines the relative importance of independent variables as well as parameter estimates in an estimation model based on contribution to an overall model fit statistic (see Grömping, 2007 as well as Luchman, Lei, and Kaplan, 2020 for discussion).  DA is an ensemble method in which importance determinations about independent variables or parameter estimates are made by aggregating results across multiple models, though the method usually requires the ensemble contain each possible combination of the independent variables in the full model.  

The all possible combinations ensemble with p independent variables in the full model results in 2^p-1 models estimated.  That is, each combiation of p variables alterating between included versus excluded (i.e., the 2 base to the exponent) where the constant[s]-only model is omitted (i.e., the -1 representing the distinct combination where no independent variables are included; see Budescu, 1993).

`domme` is implemented as a flexible wrapper command that can be used with most Stata estimation commands that use Stata's `ml` estimation and allow `constraints()`. `domme` can produce its own fit statistics for commands that return the `e(ll)` log-likelihood scalar.

`domme` is an extension of [domin](https://github.com/jluchman/domin/blob/master/README.md) and readers are referred to the `domin` page for basics on dominance analysis.

Some examples of the command as applied to Stata estimation commands are shown below.  

# Examples
## Seemingly Unrelated Regession

Seemingly unrelated regression (i.e., `sureg`) is an analysis that cannot be accommodated with the `domin` command but can easily be accommodated by `domme`.

An example `sureg` analysis is reported below.

```
.     sysuse auto
(1978 Automobile Data)

.     sureg (price = length foreign gear_ratio) (headroom = mpg)

Seemingly unrelated regression
--------------------------------------------------------------------------
Equation             Obs   Parms        RMSE    "R-sq"       chi2        P
--------------------------------------------------------------------------
price                 74       3    2319.086    0.3733      47.80   0.0000
headroom              74       1     .764981    0.1712      16.18   0.0001
--------------------------------------------------------------------------

------------------------------------------------------------------------------
             |      Coef.   Std. Err.      z    P>|z|     [95% Conf. Interval]
-------------+----------------------------------------------------------------
price        |
      length |   71.43614   16.93133     4.22   0.000     38.25134    104.6209
     foreign |   3903.345   826.7769     4.72   0.000     2282.892    5523.798
  gear_ratio |  -2512.173   954.5322    -2.63   0.008    -4383.022   -641.3239
       _cons |  -846.5025   5212.856    -0.16   0.871    -11063.51    9370.507
-------------+----------------------------------------------------------------
headroom     |
         mpg |  -.0618228   .0153694    -4.02   0.000    -.0919463   -.0316992
       _cons |   4.309902   .3391911    12.71   0.000     3.645099    4.974704
------------------------------------------------------------------------------
```

`sureg` does not produce its own fit statistic but does record the `e(ll)` scalar.  In the example below, `fitstat(e(), mcf)` is a built-in option that computes a fit statistic to dominance analyze.  The dominance results for the `sureg` command are reported below.  Note that the focus of `domme` is on parameter estimate importance/PERI and not the importance of independent variables.

```
.     domme (price = length foreign gear_ratio) (headroom = mpg), reg(sureg (price = length foreign gear_ratio) (headroom = mpg)) fitstat(e(), mcf)

Total of 15 models/regressions

General dominance statistics: sureg
Number of obs             =                      74
Overall Fit Statistic     =                  0.0322

            |      Dominance      Standardized      Ranking
            |      Stat.          Domin. Stat.
------------+------------------------------------------------------------------------
price       |
 length     |         0.0109      0.3375            1 
 foreign    |         0.0070      0.2169            3 
 gear_ratio |         0.0051      0.1576            4 
headroom    |
 mpg        |         0.0093      0.2881            2 
-------------------------------------------------------------------------------------
Conditional dominance statistics
-------------------------------------------------------------------------------------

                   #param_ests:  #param_ests:  #param_ests:  #param_ests:
                             1             2             3             4
    price:length        0.0100        0.0116        0.0118        0.0101
   price:foreign        0.0004        0.0056        0.0097        0.0124
price:gear_ratio        0.0042        0.0059        0.0060        0.0042
    headroom:mpg        0.0087        0.0093        0.0097        0.0094
-------------------------------------------------------------------------------------
Complete dominance designation
-------------------------------------------------------------------------------------

                        dominated?:  dominated?:  dominated?:  dominated?:
                            length      foreign   gear_ratio          mpg
    dominates?:length            0            0            1            0
   dominates?:foreign            0            0            0            0
dominates?:gear_ratio           -1            0            0            0
       dominates?:mpg            0            0            0            0
-------------------------------------------------------------------------------------

Strongest dominance designations

price:length completely dominates price:gear_ratio
headroom:mpg conditionally dominates price:gear_ratio
price:length conditionally dominates headroom:mpg
price:length generally dominates price:foreign
headroom:mpg generally dominates price:foreign
price:foreign generally dominates price:gear_ratio
```
Note that any parameters not included in the dominance analysis are considered to be a part of the constant model for built-in fit statistics like `e(mcf)`.

## Zero Inflated Poisson Regression

A zero inflated Poisson (i.e., `zip`) is another analysis that can be accommodated by `domme`

```
.     generate zi_pr = price*foreign

.     domme (zi_pr = headroom trunk) (inflate = gear_ratio turn), reg(zip zi_pr headroom trunk) f(e(), bic) ropt(inflate(gear_ratio turn)) reverse

Total of 15 models/regressions

General dominance statistics: Zero-inflated Poisson regression
Number of obs             =                      74
Overall Fit Statistic     =              18246.3488
Constant-only Fit Stat.   =              20760.9084

            |      Dominance      Standardized      Ranking
            |      Stat.          Domin. Stat.
------------+------------------------------------------------------------------------
zi_pr       |
 headroom   |      -124.0851      0.0493            2 
 trunk      |     -2344.9819      0.9326            1 
inflate     |
 gear_ratio |       -26.9970      0.0107            3 
 turn       |       -18.4955      0.0074            4 
-------------------------------------------------------------------------------------
Conditional dominance statistics
-------------------------------------------------------------------------------------

                     #param_ests:  #param_ests:  #param_ests:  #param_ests:
                               1             2             3             4
    zi_pr:headroom     -248.5252     -165.5652      -82.6051        0.3549
       zi_pr:trunk    -2469.4220    -2386.4619    -2303.5019    -2220.5418
inflate:gear_ratio      -41.6540      -31.8827      -22.1114      -12.3401
      inflate:turn      -33.1525      -23.3812      -13.6099       -3.8386
-------------------------------------------------------------------------------------
Complete dominance designation
-------------------------------------------------------------------------------------

                        dominated?:  dominated?:  dominated?:  dominated?:
                          headroom        trunk   gear_ratio         turn
  dominates?:headroom            0           -1            0            0
     dominates?:trunk            1            0            1            1
dominates?:gear_ratio            0           -1            0            1
      dominates?:turn            0           -1           -1            0
-------------------------------------------------------------------------------------

Strongest dominance designations

zi_pr:trunk completely dominates zi_pr:headroom
zi_pr:trunk completely dominates inflate:gear_ratio
zi_pr:trunk completely dominates inflate:turn
inflate:gear_ratio completely dominates inflate:turn
zi_pr:headroom generally dominates inflate:gear_ratio
zi_pr:headroom generally dominates inflate:turn
```
