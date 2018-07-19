---
title: "SurvivalDataGeneration"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
The research question: Is the intervention able to significantly reduce the time until first sucicide attempt.

First we simluate data.  We use the simple.surv.sim function which produces survival data for a one time event, with baseline covariates.  

We can use the weibull distribution, since this is a common distribution for survival data.

The first two arguments are the number of people and the max number of days for the study.

beta0.ev = median value for events.  

Then I am creating two covariates one that is a normal distribution with a mean of zero and a standard deviation of one the other that is binary with a mean of .5, which will serve as the intervention variable.  

beta0.cens = assuming this is the same as the group who is not censored, because we are assuming the data are missing at random.

status = censored
stop = dependent variable time until event

In order to use the power analysis package for survival data, I need to change the column names where stop equals time, the intervention equals x, and the intervention has E for experiment and C for control for the 1's and 0's.  
```{r}
library(survsim)
set.seed(123)
survData = simple.surv.sim(100, 90, dist.ev = "weibull", anc.ev = 1, beta0.ev = 5, dist.cens = "weibull", anc.cens = 1, beta0.cens = 5, x = list(c("normal", 0,1), c("bern", .5)), beta = list(-.4, -.25))
colnames(survData)[4] = c("time")
colnames(survData)[6:7] = c("var1", "x")
survData$x = ifelse(survData$x == 1, "E", "C")
```
Now let us try the power analysis.  Something is wrong.  Everything has a power of 1.

The first argument should be the survival function, which is the dependent and the censoring variable across the intervention variable x.  Then we tell it what the data is, the number of people in the experiment and control, the hazzard ratio, the alpha level.  
```{r}
library(powerSurvEpi)
survPower = powerCT(Surv(survData$time, survData$status) ~ survData$x, dat = survData, nE = 10, nC = 10, RR = 1, alpha = .05)
summary(survPower)
```

