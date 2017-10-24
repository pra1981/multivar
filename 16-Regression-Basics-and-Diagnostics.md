# 16 - Regression Basics and Diagnostics
SIUE, F2017, Stat 589  
`r format(Sys.Date(), "%B %d, %Y")`  


***

### Load required package


```r
# install the necessary package if it doesn't exist
if (!require(MASS)) install.packages(`MASS`)
if (!require(dplyr)) install.packages(`dplyr`)
if (!require(ggplot2)) install.packages(`ggplot2`)
library(MASS)
library(dplyr)
library(ggplot2) 
```




## Introduction - relationships between two quantitative variables

Consider the bivariate data set `mammals` in the package `MASS` with two numeric variables - body and brain size of mammals.


```r
str(mammals)
```

```
# 'data.frame':	62 obs. of  2 variables:
#  $ body : num  3.38 0.48 1.35 465 36.33 ...
#  $ brain: num  44.5 15.5 8.1 423 119.5 ...
```

```r
head(mammals)
```

```
#                   body brain
# Arctic fox        3.38  44.5
# Owl monkey        0.48  15.5
# Mountain beaver   1.35   8.1
# Cow             465.00 423.0
# Grey wolf        36.33 119.5
# Goat             27.66 115.0
```

```r
summary(mammals)
```

```
#       body          brain     
#  Min.   :   0   Min.   :   0  
#  1st Qu.:   1   1st Qu.:   4  
#  Median :   3   Median :  17  
#  Mean   : 199   Mean   : 283  
#  3rd Qu.:  48   3rd Qu.: 166  
#  Max.   :6654   Max.   :5712
```

```r
mammals %>% 
  ggplot(aes(x = body, y = brain)) +
  geom_point()
```

<img src="figures/01-lec16-1.png" style="display: block; margin: auto;" />

This scatterplot is not very informative because of the scale. A log transformation may produce more informative result. `log()` computes the natural logarithm.


```r
mammals %>% 
  ggplot(aes(x = log(body), y = log(brain))) +
  geom_point()
```

<img src="figures/02-lec16-1.png" style="display: block; margin: auto;" />


## Correlation and simple regression line

After log transforming the variables, we can observe a linear trend; logarithms of body and brain size are positively correlated.


```r
cor(log(mammals))
```

```
#       body brain
# body  1.00  0.96
# brain 0.96  1.00
```

```r
cor(log(mammals$body), log(mammals$brain))
```

```
# [1] 0.96
```

Let us add a fitted straight line to the log-log plot (more on simple regression later).  The `lm()` function returns the coefficients of the line, and the line can be added to the log-log plot by the `abline()` function.


```r
mammals %>% 
  ggplot(aes(x = log(body), y = log(brain))) +
  geom_point() +
  stat_smooth(method = "lm", se = FALSE)
```

<img src="figures/04-lec16-1.png" style="display: block; margin: auto;" />

```r
# simple linear regression model on log(body) against log(brain)
lm(log(brain) ~ log(body), data = mammals)
```

```
# 
# Call:
# lm(formula = log(brain) ~ log(body), data = mammals)
# 
# Coefficients:
# (Intercept)    log(body)  
#       2.135        0.752
```

***

## Simple linear regression model

Regression is a general statistical method to fit a straight line or other model to data. The objective is to find a model for predicting the dependent variable (response) given one or more independent (predictor) variables.

The simplest example is a *simple linear regression model* of $Y$ on $X$, defined by
$$
Y = \beta_0 + \beta_1 X + \epsilon
$$
where $\epsilon$ is a random term. This linear model describes a straight line relation between the response variable $Y$ and predictor $X$.

In least squares regression the unknown parameters $\beta_0$ and $\beta_1$ are estimated by minimizing the sum of the squared deviations between the observed response $Y$ and the value $\hat{Y}$ predicted by the model. If these estimates are $b_0$ (intercept) and $b_1$ (slope), the estimated regression line is
$$
\hat{Y} = b_0 + b_1 X.
$$

For a set $(x_i,y_i)$, $i=1,2,\ldots,n$, the errors in this estimate are $y_i - \hat{y}_i$. Least squares regression obtains the estimated intercept $b_0$ and slope $b_1$ that minimizes the sum of squared errors:
$$
\sum^n_{i=1} (y_i - \hat{y}_i)^2.
$$

A *multiple linear regression model* has more than one predictor variable (multiple predictors). Linear models can describe many relations other than straight lines or planes.

*Any model that is linear in the parameters is considered a linear model.* Thus, a quadratic relation $y = \beta_0 + \beta_1 x + \beta_2 x^2$ corresponds to a linear model with two predictors, $X_1 = X$ and $X_2 = X^2$. 

The exponential relation $y = \beta_0 e^{\beta_1 x}$ is not linear, but the relation can be expressed by taking the natural logarithm of both sides. The corresponding linear equation is $\ln y = \ln \beta_0 + \beta_1 x.$

***

## Fitting the simple linear regression on the `cars` data

Consider the data set `cars` containg the speed and stopping distance of a sample of cars (dataset `cars' in R). Is there a linear relation between stopping distance and speed of a car?


```r
str(cars)     # str() displays structure of data
```

```
# 'data.frame':	50 obs. of  2 variables:
#  $ speed: num  4 4 7 7 8 9 10 10 10 11 ...
#  $ dist : num  2 10 4 22 16 10 18 26 34 17 ...
```

```r
head(cars)
```

```
#   speed dist
# 1     4    2
# 2     4   10
# 3     7    4
# 4     7   22
# 5     8   16
# 6     9   10
```

```r
summary(cars)
```

```
#      speed           dist    
#  Min.   : 4.0   Min.   :  2  
#  1st Qu.:12.0   1st Qu.: 26  
#  Median :15.0   Median : 36  
#  Mean   :15.4   Mean   : 43  
#  3rd Qu.:19.0   3rd Qu.: 56  
#  Max.   :25.0   Max.   :120
```

```r
cars %>% 
  ggplot(aes(x = speed, y = dist)) +
  geom_point()
```

<img src="figures/05-lec16-1.png" style="display: block; margin: auto;" />

The scatterplot above reveals that there is a positive association between distance and speed of cars. The relation between distance and speed could be approximated by a line or perhaps by a parabola. 

We start with the simplest model, the straight line model. The response variable in this example is stopping distance dist and the predictor variable speed is speed. To fit a straight line model
$$
dist = \beta_0 + \beta_1 speed + \epsilon,
$$
we need estimates of the intercept $\beta_0$ and the slope $\beta_1$ of the line.

### The `lm()` function and the model formula

The linear model function is `lm`. This function estimates the parameters of a linear model by the least squares method. A linear model is specified in R by a model `formula`.

The R formula that specifies a simple linear regression model $dist = \beta_0 + \beta_1 speed + \epsilon$ is simply

> `dist` ~ `speed`

The model formula is the first argument to the `lm` (linear model) function.


```r
fit0 <- lm(dist ~ speed, data = cars)
print(fit0)
```

```
# 
# Call:
# lm(formula = dist ~ speed, data = cars)
# 
# Coefficients:
# (Intercept)        speed  
#      -17.58         3.93
```

The fitted regression line is:

> `dist` = -17.579 + 3.932 `speed`

According to this model, average stopping distance increases by 3.932 feet for each additional mile per hour of speed.


```r
cars %>% 
  ggplot(aes(x = speed, y = dist)) +
  geom_point() +
  stat_smooth(method = "lm", se = FALSE)
```

<img src="figures/07-lec16-1.png" style="display: block; margin: auto;" />


## Residuals and Diagnostics

The residuals are the vertical distances from the observed stopping distance `dist` (the plotting symbol) to the line. The paired observations are $(x_i, y_i) = ({\tt speed}_i, {\tt dist}_i)$, $i=1, \ldots, 50$. 

The residuals are
$$
e_i = y_i - \hat{y}_i
$$
where $\hat{y}_i$ denotes the value of stopping distance predicted by the model at speed $x_i$.

Note in the figure above that the model tends to fit better at slow speeds than at higher speeds. It is somewhat easier to analyze the distribution of the errors using a residual plot (residuals vs fitted values). One way to generate the residual plot is to use the plot method for the `lm` result.

The argument `which = 1` specifies the type of plot (residuals vs fitted values). The `add.smooth` argument controls whether a certain type of curve is fitted to the residuals.


```r
plot(fit0, which = 1, add.smooth = FALSE)
```

<img src="figures/08-lec16-1.png" style="display: block; margin: auto;" />

The residual plot above has three unusually large residuals labeled; observations 23, 35, and 49. One can also observe that the residuals are closer to zero at slow speeds; the variance of the residuals is not constant across all speeds, but increasing with speed. 

Inference (tests or confidence intervals) about the model is usually based on the assumption that the errors are normally distributed with mean zero and constant variance.

***

## Multiple Predictors: Volume of Black Cherry Trees

The data were collected from 31 black cherry trees in the Allegheny National Forest, Pennsylvania, in order to find an estimate for the volume of a tree (and therefore the timber yield), given its height and girth diameter. 


```r
str(trees)
```

```
# 'data.frame':	31 obs. of  3 variables:
#  $ Girth : num  8.3 8.6 8.8 10.5 10.7 10.8 11 11 11.1 11.2 ...
#  $ Height: num  70 65 63 72 81 83 66 75 80 75 ...
#  $ Volume: num  10.3 10.3 10.2 16.4 18.8 19.7 15.6 18.2 22.6 19.9 ...
```

```r
cor(trees)    # correlation matrix
```

```
#        Girth Height Volume
# Girth   1.00   0.52   0.97
# Height  0.52   1.00   0.60
# Volume  0.97   0.60   1.00
```

```r
pairs(trees)
```

<img src="figures/09-lec16-1.png" style="display: block; margin: auto;" />


The correlation between girth diameter and volume is 0.97, indicating a strong positive linear relation between `Girth` and `Volume`. The correlation between height and volume is 0.60, which indicates a moderately strong positive linear association between `Height` and `Volume`.

As a preliminary analysis, let us fit a simple linear regressions with `Volume` as response and `Girth` as predictor.

> Model 1: `Volume` ~ `Girth`


```r
fit1 <- lm(Volume ~ Girth, data = trees)
fit1
```

```
# 
# Call:
# lm(formula = Volume ~ Girth, data = trees)
# 
# Coefficients:
# (Intercept)        Girth  
#      -36.94         5.07
```

```r
trees %>% 
  ggplot(aes(x = Girth, y = Volume)) +
  geom_point() +
  stat_smooth(method = "lm", se = FALSE)
```

<img src="figures/10-lec16-1.png" style="display: block; margin: auto;" />

In Model 1, the estimated intercept is -36.943 and the estimated slope is 5.066. According to this model, the average volume increases by 5.066 cubic feet for each additional 1 inch in diameter.



```r
par(mfcol = c(1,2)) 
plot(fit1, which = 1:2, add.smooth=TRUE)
```

<img src="figures/11-lec16-1.png" style="display: block; margin: auto;" />
The argument `which = 1:2` requests two plots: a plot of residuals vs fits (1) and a QQ plot to check for normality of residuals (2). 

Note that a curve has been added in the residual plots. This curve is a fitted lowess (local polynomial regression) curve, called a smoother. The residuals are assumed iid (independent and identically distributed), but there is a pattern evident. 

The residuals in the first model have a "U" shape or bowl shape. This pattern could indicate that there is a variable missing from the model. 

In the QQ plot, normally distributed residuals should lie approximately along the reference line shown in the plot. The observation with the largest residual corresponds to the tree with the largest volume, observation 31. It also has the largest height and diameter.


## Multiple Linear Regression Model

A multiple linear regression model with response variable $Y$ and two predictor variables $X_1$ and $X_2$ is
$$
Y = \beta_0 + \beta_1 X_1 + \beta_2 X_2 +  \epsilon,
$$
where $\epsilon$ is a random term. For inference we assume that the errors are normally distributed and independent with mean zero and common variance $\sigma^2$.

***

Let us fit a multiple linear regression with `Volume` as response and `Girth` and `Height` as predictors.

> Model 2:  `Volume` ~ `Girth` + `Height` 


```r
fit2 <- lm(Volume ~ Girth + Height, data = trees)
fit2
```

```
# 
# Call:
# lm(formula = Volume ~ Girth + Height, data = trees)
# 
# Coefficients:
# (Intercept)        Girth       Height  
#     -57.988        4.708        0.339
```

The fitted model is
$$
{\tt Volume} = - 57.9877 + 4.7082 \times {\tt Girth} + 0.3393 \times {\tt Height}. 
$$

According to this model, when height is held constant, average volume of a tree increases by 4.7082 cubic feet for each additional inch in diameter. When diameter is held constant, average volume of a tree increases by 0.3393 cubic feet for each additional inch of height.


```r
par(mfcol = c(1,2))
plot(fit2, which = 1:2)
```

<img src="figures/13-lec16-1.png" style="display: block; margin: auto;" />

These residual plots for Model 2 look similar to the corresponding plots for Model 1. The "U" shaped pattern of residuals in the plot of residuals vs fits may indicate that a quadratic term is missing from the model.

Finally, let us fit a model that also includes a the square of diameter as a predictor. The model formula is

> Model 3: `Volume` ~ `Girth` + `I(Girth^2)` + `Height`

where `I(Girth^2)` means to interpret `Girth^2` "as is" (the square of `Girth`) rather than interpret the exponent as a formula operator.


```r
fit3 <- lm(Volume ~ Girth + I(Girth^2) + Height, data = trees)
fit3
```

```
# 
# Call:
# lm(formula = Volume ~ Girth + I(Girth^2) + Height, data = trees)
# 
# Coefficients:
# (Intercept)        Girth   I(Girth^2)       Height  
#      -9.920       -2.885        0.269        0.376
```

```r
par(mfcol = c(1,2))
plot(fit3, which = 1:2)
```

<img src="figures/14-lec16-1.png" style="display: block; margin: auto;" />

For Model 3, the plot of residuals vs fits does not have the "U" shape that was apparent for Models 1 and 2. The residuals are approximately centered at 0 with constant variance. In the normal QQ plot, the residuals are close to the reference line on the plot. These residual plots are consistent with the assumption that errors are iid with a ${\tt Normal}(0,\sigma^2)$ distribution.

***

## The `summary()` and `anova()` methods for `lm`

The summary of the fitted model contains additional information about the model. In the result of `summary` we find a table of the coefficients with standard errors, a five number summary of residuals, the coefficient of determination ($R^2$), and the residual standard error.


```r
summary(fit3)
```

```
# 
# Call:
# lm(formula = Volume ~ Girth + I(Girth^2) + Height, data = trees)
# 
# Residuals:
#    Min     1Q Median     3Q    Max 
# -4.293 -1.669 -0.102  1.785  4.349 
# 
# Coefficients:
#             Estimate Std. Error t value Pr(>|t|)    
# (Intercept)  -9.9204    10.0791   -0.98  0.33373    
# Girth        -2.8851     1.3099   -2.20  0.03634 *  
# I(Girth^2)    0.2686     0.0459    5.85  3.1e-06 ***
# Height        0.3764     0.0882    4.27  0.00022 ***
# ---
# Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
# 
# Residual standard error: 2.6 on 27 degrees of freedom
# Multiple R-squared:  0.977,	Adjusted R-squared:  0.975 
# F-statistic:  383 on 3 and 27 DF,  p-value: <2e-16
```

The adjusted $R^2$ value of 0.9745 indicates that more than $97\%$ of the total variation in `Volume` about its mean is explained by the linear association with the predictors `Girth`, `Girth^2`, and `Height`. The residual standard error is 2.625. This is the estimate of $\sigma$, the standard deviation of the error term $\epsilon$ in Model 3.

The table of coefficients includes standard errors and $t$ statistics for testing $H_0$ : $\beta_j = 0$ vs $H_1$ : $\beta_j \neq  0$. The p-values of the test statistics are given under `Pr(>|t|)`. We reject the null hypothesis $H_0$ : $\beta_j = 0$ if the corresponding p-value is less than the significance level. At significance level 0.05 we conclude that `Girth`, `Girth^2`, and `Height` are significant.

The analysis of variance (ANOVA) table for this model is obtained by the anova function.


```r
anova(fit3)
```

```
# Analysis of Variance Table
# 
# Response: Volume
#            Df Sum Sq Mean Sq F value  Pr(>F)    
# Girth       1   7582    7582  1100.5 < 2e-16 ***
# I(Girth^2)  1    213     213    30.9 6.8e-06 ***
# Height      1    125     125    18.2 0.00022 ***
# Residuals  27    186       7                    
# ---
# Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

From the ANOVA table, one can observe that `Girth` explains most of the total variability in the response, but the other predictors are also significant in Model 3.

A way to compare the models (Model 1, Model 2, and Model 3) is to list all of the corresponding `lm` objects as arguments to `anova`:


```r
anova(fit1, fit2, fit3)
```

```
# Analysis of Variance Table
# 
# Model 1: Volume ~ Girth
# Model 2: Volume ~ Girth + Height
# Model 3: Volume ~ Girth + I(Girth^2) + Height
#   Res.Df RSS Df Sum of Sq    F  Pr(>F)    
# 1     29 524                              
# 2     28 422  1       102 14.9 0.00065 ***
# 3     27 186  1       236 34.2 3.1e-06 ***
# ---
# Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

This table shows that the residual sum of squares decreased by 102.38 from 524.30 when `Height` was added to the model, and decreased another 235.91 from 421.92 when the square of girth was added to the model.

***

### Interval estimates for new observations

Regression models are models for predicting a response variable given one or more predictor variables. We can use the function `predict` for `lm` to obtain predictions of the response variable. The `predict` method for lm also provides two types of interval estimates for the response:

1. Prediction intervals for new observations, for given values of the predictor variables.
2. Confidence intervals for the expected value of the response for given values of the predictors.

To predict volume for new trees with given diameter and height, the `predict` method can be used. Store the diameter and height values for the new tree(s) in a data frame using the identical names as in the original model formula specified in the `lm` call. For example, to apply the Model 3 fit to obtain a point estimate for the volume of a new tree with diameter 16 in. and height 70 ft., we enter


```r
new <- data.frame(Girth=16, Height=70)
predict(fit3, newdata=new)  # predicted value under Model 3
```

```
#  1 
# 39
```

The predicted volume of the new tree is 39.0 cubic feet. This estimate is about 10% lower than the prediction we obtained from Model 1, which used only diameter as a predictor.


```r
new1 <- data.frame(Girth=16)
predict(fit1, newdata = new1) # predicted value under Model 1
```

```
#  1 
# 44
```

To obtain a prediction interval or a confidence interval for the volume of the new tree, the predict method is used with an argument called interval specified. The confidence level is specified by the level argument, which is set at 0.95 by default. One can abbreviate the argument values. For a prediction interval specify `interval="pred"` and for a confidence interval use `interval="conf"`.


```r
predict(fit3, newdata=new, interval="pred")
```

```
#   fit lwr upr
# 1  39  33  45
```

The prediction interval for volume of a randomly selected new tree of girth diameter 16 and height 70 is (33.2, 44.8) cubic feet.

The confidence interval for the expected volume of all trees of girth diameter 16 and height 70 is obtained by


```r
predict(fit3, newdata=new, interval="conf")
```

```
#   fit lwr upr
# 1  39  37  41
```
so the confidence interval estimate for expected volume is (36.8, 41.2) cubic feet. 

The prediction interval is wider than the confidence interval because the prediction for a single new tree must take into account the variation about the mean and also the variation among all trees of this girth diameter and height.

To obtain point estimates or interval estimates for several new trees, one would store the new values in a data frame like our data frame new. For example, if we require confidence intervals for girth diameter 16, at a sequence of values of height 65 to 70, we can do the following.


```r
girth <- 16
height <- seq(65, 70, 1)
new <- data.frame(Girth=girth, Height=height)
predict(fit3, newdata=new, interval="conf")
```

```
#   fit lwr upr
# 1  37  34  40
# 2  38  35  40
# 3  38  35  41
# 4  38  36  41
# 5  39  36  41
# 6  39  37  41
```

