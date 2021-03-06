Motor Trends : Automatic or Manual transmission for better mileage ?
========================================================
    
**by P. Paquay**
    
## Executive summary
    
In this report we try to answer the question : "Is automatic or manual transmission better for mpg ?". To answer this question we used a dataset from the 1974 Motor Trend US magazine, and ran some statistical tests and a regression analysis. On one hand the statistical tests show (without controlling for other car design features) a difference in mean of about 7 miles more for the manual transmitted cars. On the other hand, the regression analysis indicate that by taking into account other variables like weight and 1/4 mile time, manual transmitted cars are only 2.9 miles better than automatic transmitted cars and also that this result is less significant than to consider weight and 1/4 mile time together.

## Cleaning data

The first step of our analysis is simply to load and take a look at the data.


```r
data(mtcars)
str(mtcars)
```

```
## 'data.frame':	32 obs. of  11 variables:
##  $ mpg : num  21 21 22.8 21.4 18.7 18.1 14.3 24.4 22.8 19.2 ...
##  $ cyl : num  6 6 4 6 8 6 8 4 4 6 ...
##  $ disp: num  160 160 108 258 360 ...
##  $ hp  : num  110 110 93 110 175 105 245 62 95 123 ...
##  $ drat: num  3.9 3.9 3.85 3.08 3.15 2.76 3.21 3.69 3.92 3.92 ...
##  $ wt  : num  2.62 2.88 2.32 3.21 3.44 ...
##  $ qsec: num  16.5 17 18.6 19.4 17 ...
##  $ vs  : num  0 0 1 1 0 1 0 1 1 1 ...
##  $ am  : num  1 1 1 0 0 0 0 0 0 0 ...
##  $ gear: num  4 4 4 3 3 3 3 4 4 4 ...
##  $ carb: num  4 4 1 1 2 1 4 2 2 4 ...
```

Now we coerce the "cyl", "vs", "gear", "carb" and "am" variables into factor variables.


```r
mtcars$cyl <- factor(mtcars$cyl)
mtcars$vs <- factor(mtcars$vs)
mtcars$gear <- factor(mtcars$gear)
mtcars$carb <- factor(mtcars$carb)
mtcars$am <- factor(mtcars$am)
```

For a better readability, we rename the levels of the "am" variable into "Auto" and "Manual".


```r
levels(mtcars$am) <- c("Auto", "Manual")
```

## Graphics

We begin by plotting boxplots of the variable "mpg" when "am" is "Auto" or "Manual" (see Figure 1 in the appendix). This plot hints at an increase in mpg when gearing was manual but this data may have other variables which may play a bigger role in determination of mpg.

We then plot the relationships between all the variables of the dataset (see Figure 2 in the appendix). We may note that variables like "wt", "cyl", "disp" and "hp" seem highly correlated together.

## Inference

We may also run some tests to compare the mpg means between automatic and manual transmissions.

### T-test

We begin by using a t-test assuming that the mileage data has a normal distribution.


```r
t.test(mpg ~ am, data = mtcars)
```

The p-value of 0.0014 clearly shows that the manual and automatic transmissions are significatively different.

### Wilcoxon test

Next we use a nonparametric Wilcoxon test to determine if there's a difference in the population means.


```r
wilcox.test(mpg ~ am, data = mtcars)
```

```
## Warning: cannot compute exact p-value with ties
```

Here again the p-value of 0.0019 allow us to reject the null hypothesis that the mileage data of the manual and automatic transmissions are from the same population (indicating a difference).

## Regression analysis

First we need to select a model, we proceed by using the Bayesian Information Criteria (BIC) in a stepwise algorithm. This algorithm does not evaluate the BIC for all possible models but uses a search method that compares models sequentially. Thus it bears some comparison to the classical stepwise method but with the advantage that no dubious p-values are used.


```r
model.all <- lm(mpg ~ ., data = mtcars)
n <- nrow(mtcars)
model <- step(model.all, direction = "backward", k = log(n))
```


```r
summary(model)
```

```
## 
## Call:
## lm(formula = mpg ~ wt + qsec + am, data = mtcars)
## 
## Residuals:
##    Min     1Q Median     3Q    Max 
## -3.481 -1.556 -0.726  1.411  4.661 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)    9.618      6.960    1.38  0.17792    
## wt            -3.917      0.711   -5.51    7e-06 ***
## qsec           1.226      0.289    4.25  0.00022 ***
## amManual       2.936      1.411    2.08  0.04672 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 2.46 on 28 degrees of freedom
## Multiple R-squared:  0.85,	Adjusted R-squared:  0.834 
## F-statistic: 52.7 on 3 and 28 DF,  p-value: 1.21e-11
```

The BIC algorithm tells us to consider "wt" and "qsec" as confounding variables. The individual p-values allows us to reject the hypothesis that the coefficients of "wt", "qsec" and "am" are null. The adjusted r-squared is 0.8336, so we may conclude that more than 83% of the variation is explained by the model.


```r
anova(lm(mpg ~ am, data = mtcars), lm(mpg ~ am + qsec, data = mtcars), lm(mpg ~ am + wt + qsec, data = mtcars))
```

```
## Analysis of Variance Table
## 
## Model 1: mpg ~ am
## Model 2: mpg ~ am + qsec
## Model 3: mpg ~ am + wt + qsec
##   Res.Df RSS Df Sum of Sq    F  Pr(>F)    
## 1     30 721                              
## 2     29 353  1       368 60.9 1.7e-08 ***
## 3     28 169  1       183 30.3 7.0e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

We may notice that when we compare the model with only "am" as independant variable and our chosen model, we reject the null hypothesis that the variables "wt" and "qsec" don't contribute to the accuracy of the model.

The regression suggests that, "wt" and "qsec" variables remaining constant, manual transmitted cars can drive 2.9358 more miles per gallon on average than automatic transmitted cars, and the results are statistically significant.


```r
confint(model)
```

```
##                2.5 % 97.5 %
## (Intercept) -4.63830 23.874
## wt          -5.37333 -2.460
## qsec         0.63457  1.817
## amManual     0.04573  5.826
```

More accurately, we are 95% confident that the difference in miles per gallon between manual and automatic transmitted cars lies somewhere in the interval [0.0457, 5.8259].

## Residuals and diagnostics

### Residual analysis

We begin by studying the residual plots (see Figure 3 in the appendix). These plots allow us to verify some assumptions made before. We have to point that due to the small sample size 

1. The Residuals vs Fitted plot seem to verify the independance assumption as the points are randomly scattered on the plot (a Durbin-Watson test further confirms this assumption at the 0.05 level).
2. The Normal Q-Q plot seem to indicate that the residuals are normally distributed as the points hug the line closely (a Shapiro-Wilk test further confirms this assumption at the 0.05 level).
3. The Scale-Location plot seem to verify the constant variance assumption as the points fall in a constant band (a Breusch-Pagan test further confirms this assumption at the 0.05 level).

### Leverages

We begin by computing the leverages for the "mtcars" dataset.


```r
leverage <- hatvalues(model)
```

Are any of the observations in the dataset outliers ? We find the outliers by selecting the observations with a hatvalue > 0.5.


```r
leverage[which(leverage > 0.5)]
```

```
## named numeric(0)
```

### Dfbetas

Next we look at the Dfbetas of the observations.


```r
influential <- dfbetas(model)
```

Are any of the observations in the dataset influential ? We find the influential observations by selecting the ones with a dfbeta > 1 in magnitude.


```r
influential[which(abs(influential) > 1)]
```

```
## [1] 1.094
```

This influential observation corresponds to the Chrysler Imperial.

## Appendix

### Figure 1 : Boxplots of "mpg" vs. "am"


```r
plot(mpg ~ am, data = mtcars, main = "Mpg by transmission type", xlab = "Transmission type", ylab = "Miles per gallon")
```

![plot of chunk unnamed-chunk-14](figure/unnamed-chunk-14.png) 

### Figure 2 : Pairs graph


```r
pairs(mtcars, panel = panel.smooth, main = "Pairs graph for MTCars")
```

![plot of chunk unnamed-chunk-15](figure/unnamed-chunk-15.png) 

### Figure 3 : Residual plots


```r
par(mfrow = c(2, 2))
plot(fitted(model), residuals(model), xlab = "Fitted values", ylab = "Residuals", main = "Residuals vs Fitted")
abline(h = 0, col = "red")
qqnorm(residuals(model))
qqline(residuals(model), col = "red")
plot(fitted(model), sqrt(abs(rstandard(model))), xlab = "Fitted values", ylab = "Square Root of Standardized Residuals", main = "Scale-Location")
```

![plot of chunk unnamed-chunk-16](figure/unnamed-chunk-16.png) 


```r
library(lmtest)
```

```
## Loading required package: zoo
## 
## Attaching package: 'zoo'
## 
## Les objets suivants sont masqués from 'package:base':
## 
##     as.Date, as.Date.numeric
```

```r
bptest(model)
```

```
## 
## 	studentized Breusch-Pagan test
## 
## data:  model
## BP = 6.187, df = 3, p-value = 0.1029
```

We do not reject the null hypothesis that the variance is the same for all observations at the 0.05 level. There is relatively weak evidence against the assumption of constant variance.


```r
dwtest(model, alternative = "two.sided")
```

```
## 
## 	Durbin-Watson test
## 
## data:  model
## DW = 1.714, p-value = 0.2527
## alternative hypothesis: true autocorrelation is not 0
```

In this case we do not reject the null hypothesis at the 0.05 level; there is very little evidence of nonzero autocorrelation in the residuals.


```r
shapiro.test(residuals(model))
```

```
## 
## 	Shapiro-Wilk normality test
## 
## data:  residuals(model)
## W = 0.9411, p-value = 0.08043
```

We do not reject the null hypothesis that the residuals are normal at the 0.05 level.
