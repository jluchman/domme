# Dominance Analysis Extended to Multiple Equation Models
## Parameter Estimate Relative Importance (PERI)

Dominance analysis (DA) determines the relative importance of independent variables as well as parameter estimates in an estimation model based on contribution to an overall model fit statistic (see Grömping, 2007 as well as Luchman, Lei, and Kaplan, 2020 for discussion).  DA is an ensemble method in which importance determinations about independent variables or parameter estimates are made by aggregating results across multiple models, though the method usually requires the ensemble contain each possible combination of the independent variables in the full model.  

The all possible combinations ensemble with p independent variables in the full model results in 2^p-1 models estimated.  That is, each combiation of p variables alterating between included versus excluded (i.e., the 2 base to the exponent) where the constant[s]-only model is omitted (i.e., the -1 representing the distinct combination where no independent variables are included; see Budescu, 1993).

`domme` is implemented as a flexible wrapper command that can be used with most Stata estimation commands that use Stata's `ml` estimation and allow `constraints()`.
