# Linear Regression

Source: [Linear Regression, Clearly Explained!!!](https://www.youtube.com/watch?v=nk2CQITm_eo)

***

#### Terminology

* Residuals: Distance from a line to a data point
* Fitting a line to a data: find the line with the least sum of squared residuals

#### Notes

Equation of the fit line:&#x20;

$$
y = y_{intercept} + slope * x
$$

If the $$slope$$ is not 0, that means knowing $$x$$ will help us make a guess about the value of $$y$$. $$R^2$$ (R-squared) helps knowing how good that guess is



**Computing** $$R^2$$

* Calculate the average value of $$y$$ = shift all the data points to the y-axis to emphasize that at this point we are only interested in $$y$$.
* Calculate $$SS(mean)$$ = sum of squares around the mean = $$(data-mean)^2$$ and $$Var(mean)$$ = variation around the mean = $$\frac {SS(mean)} n$$
* Calculate $$SS(fit) = (data-line)^2$$ and $$Var(fit) = \frac {SS(fit)} n$$
* Less variation around the fit than the mean = some of the variation in $$y$$ is explained by taking the value $$x$$ into account.

$$R^2 = \frac {Var(mean) - Var(fit)} {Var(mean)} = \frac {SS(mean) - SS(fit)} {SS(mean)}$$

$$R^2 = \frac {variation \ in \ y \ explained \ by \ x} {variation \ in \ y \ wihtout \ taking \ x \ into \ account}$$



* The value of $$R^2$$ indicates the reduction in variance when we take the value of $$x$$ into account. That is, the value of $$x$$ explains the $$R^2$$ variation in the value $$y$$.



**How to determine if** $$R^2$$ **value is statistically significant?**

Calculate its $$p$$-value

First, calculate $$F$$-value:

$$F = \frac {variation \ in \ y \ explained \ by \ x} {variation \ in \ y \ not \ explained \ by \ x}$$

$$F = \frac {SS(mean) - SS(fit)/(p_{fit} - p_{mean})} {SS(fit) / (n-p_{fit})}$$

where

* $$p_{fit}$$ = number of parameters in the fit line = 2 for $$y = y_{intercept} + slope * x$$, that is, $$y_{intercept}$$ and $$x$$
* $$p_{mean}$$ = number of parameters in the mean line = 1 for $$y=y_{intercept}$$, that is, $$y_{intercept}$$
* $$n$$ = sample size

How to calculate $$p$$-value from $$F$$-value?

* Generate random datasets, calculate $$F$$-values and plot on a histogram.
* $$p$$-value is the number of more extreme values on the histogram divided by all the values.
* Practically, approximate the histogram with a line instead of generating many random datasets. This is $$F$$-distribution.
  * Degrees of freedom determine the shape. $$p_{fit} - p_{mean}$$ and $$n-p_{fit}$$ are degrees of freedom.

#### Takeaways

Linear regression:

1. Quantifies the relationship in the data: this is $$R^2$$.
   1. This needs to be large.
2. Determines how reliable that relationship is: this the $$p$$-value we calculate with $$F$$.
   1. This needs to be small.
