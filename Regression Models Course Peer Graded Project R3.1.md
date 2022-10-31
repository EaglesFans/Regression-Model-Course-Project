---
title: "Regression Models Course Project"
author: "Choong-Hoon Hyun"
date: '2022-10-28'
output:
  word_document: default
  pdf_document: default
  html_document: default
---

### 1. Executive Summary
*Motor Trend* is interested in exploring the relationship between a set of variables and MPG. We will deep dive into the data set to answer the given two questions. Through the exploratory data analysis, I find out the meaningful two variables (type of transmission and number of cylinders) for the analysis. I use a linear regression model with two categorical variables. Even though the sample size in the data set is small, I can conclude that the vehicles with manual transmission are better for mpg than the ones with automatic transmission.

### 2. Exploratory Data Analysis (EDA)
```{r load the dataset, echo = TRUE}
data("mtcars")
```

Since we have various variables, it is worth while to find out the relationship between each variable by using correlation function in R. The correlation table can be found on the Appendix 1. MPG is correlated with drat (Rear axle ratio), vs (Engine (0 = V-shaped, 1 = straight)), am (Transmission (0 = automatic, 1 = manual)) as their values are above 0.5. (The values are 0.68117191, 0.6640389, and 0.59983243, respectively.) 
qsec (1/4 mile time, 0.41868403) and gear (Number of forward gears, 0.4802848) are not significantly correlated with the MPG. However, cyl (Number of cylinders, -0.8521620), disp (Displacement (cu.in.), -0.8475514), hp (Gross horsepower, -0.7761684), wt (Weight (1000 lbs), -0.8676594), and carb (Number of carburetors, -0.5509251) are negatively correlated with the MPG.

### 3. Automatic vs. manual transmission for MPG

According to the MPG table per type of transmission and the barplot on Appendix 2, it is obvious that manual transmission is better MPG than automatic transmission. The mean MPG for manual transmission is 24.4 while one for automatic transmission is 17.1.

### 4. Regression Models and coefficient interpretation
```{r regression models, echo = TRUE}
fit1 <- lm(mpg ~ factor(am) - 1, mtcars)
fit2 <- lm(mpg ~ factor(am) + factor(cyl) - 1, mtcars)
fit3 <- lm(mpg ~ factor(am) + factor(cyl) + disp - 1, mtcars)
fit4 <- lm(mpg ~ factor(am) + factor(cyl) + disp + wt - 1, mtcars)
```

```{r, ANOVA, echo = TRUE}
anova(fit1, fit2, fit3, fit4)
```

I make four regression models. As we can see ANOVA, the P-value of fit2 is almost 0 while fit3 and fit4 has much higher P-values. ANOVA explains that the cyl is the only variable to add for the model selection. We can disregard "disp" and "wt" variables for further analysis.  

### 5. Parallel line model of three levels of factor variable
```{r summary fit2, echo = TRUE}
summary(fit2)$coefficients
```

* Factor(am)0, 24.802, is the mean MPG of automatic transmission for the 4-cylinder engine. Factor(am)1, 27.362, is the estimated MPG of manual transmission for the 4-cylinder engine. The difference between factor(am)1 and factor(am)0, 2.56, is the slope of the MPG change between two types of transmissions. Cylinders are the dummy variables. It shows the MPG comparison between engine cylinders. It is notable that the estimated values are all statistically significant because it is less than the typical benchmarks (0.05).
* On the Appendix 3, the orange line shows the MPG comparison per type of transmission for 4-cylinder engine. The black line is the comparison for 6-cylinder engine. The blue one is for 8-cylinder engine. Since the linear regression lines are parallel with the same slope of 2.56, the manual transmission for all three cylinder engines has better MPG than automatic transmission.

### 6. Residual Plots and Diagnostics
The relevant plots can be found on Appendix 4.

* From the Residuals vs. Fitted plot, the linearity seems to hold reasonably well because the red line is close to the dashed line.
* The Q-Q plot indicates that the two distributions follow the normal distribution. However, we have some outliers. 
* The Scale-Location plot shows the similar pattern as Residuals vs. Fitted plot because the plot has the fitted values on the x-axis, and the square root of standardized residuals is on the y-axis.
* Residuals vs. Leverage plot tells us any observations that have substantial influence on the regression model. No observations fall outside of the border of Cook’s distance. There are not any influential observations in the model.

### 7. Conclusion
Based on our analysis, we can conclude that the vehicles with manual transmission have better MPG than ones with automatic. Considering only type of transmission, manual vehicles can go 7.3 miles further in average. However, if we take into account the number of engine cylinder, the linear regression model indicates that the vehicles with manual transmission can go 2.56 miles further than ones with automatic transmission per gallon.

### Appendix 

#### Appendix 1. Correlation Table: mpg vs. other variables
```{r correlation, echo = TRUE}
cor(mtcars)[1, ]
```

#### Appendix 2. Automatic transmission vs. manual transmission for MPG

```{r insalling Dplyr, echo = TRUE, message = TRUE}
library(dplyr)
```

```{r type of transmission vs. mpg, echo = TRUE}
mtcars %>% group_by(am) %>% summarize(mpg = mean(mpg))
```

```{r barplot, echo = TRUE, fig.width = 5,fig.height = 3}
library(ggplot2)
bar_plot <- ggplot(mtcars, aes(x = factor(am), y = mpg, fill = factor(am)))
bar_plot <- bar_plot + geom_boxplot() + labs(x = "Type of Transmission", y = "MPG", title = "Type of Transmission vs. MPG") + scale_x_discrete(labels=c("0" = "Automatic", "1" = "Manual"))
bar_plot
```

#### Appendix 3. Parallel line model plot

```{r parallel plot, echo = FALSE, fig.width = 5,fig.height = 3}
ggplot(mtcars, aes(x = am, y = mpg, col = factor(cyl))) + geom_point(size = 4, alpha = 0.5) + 
  geom_abline(intercept = summary(fit2)$coef[1], slope = summary(fit2)$coef[2]-summary(fit2)$coef[1], lwd = 1, col = "orange") + 
  geom_abline(intercept = summary(fit2)$coef[1] + summary(fit2)$coef[3], slope = summary(fit2)$coef[2]-summary(fit2)$coef[1], lwd = 1, col = "black") + 
  geom_abline(intercept = summary(fit2)$coef[1] + summary(fit2)$coef[4], slope = summary(fit2)$coef[2]-summary(fit2)$coef[1], lwd = 1, col = "blue") + 
  labs(x = "Type of Transmission", y = "MPG", title = "Type of Transmission vs. MPG")
```

#### Appendix 4. Residual Plots and Diagnostics

```{r diagnostic plots for fit 2, echo = TRUE}
par(mfrow = c(2, 2))
plot(fit2)
```
