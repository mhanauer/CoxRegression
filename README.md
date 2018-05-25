---
title: "Test1"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
Survival Analysis Chapter 3

When dealing with survival data it is important to get accurate estimates of survival time.  This is often problematic, because of right censoring. Right censoring occurs when the event never happens so it is not possible to get an accurate estimate of time to the event.  For example, if a researcher was evaluating the time in a treatment it took an individual to receive a score high enough to move them out of the classification of depression, not everyone in the treatment will improve enough and therefore still be classified as depressed.

The Nelson-Altschuler is one way to calculate the median survival time with right censored data.  First, you calculate the hazard function, which is the summed probability of the event happening.  Then the survival function is exp(-H) of the hazard function.  

Below is an example from the book Applied Survival Analysis Using R.

There are six-time points in the tt variable indicating the time point.  

n.risk = the number of people left in the study (i.e. people who are still alive or didn't drop out)
n.event = number of people during that time point that experienced that event.

resultsFH on its own gives the median survival time and the summary gives the probability of survival at a given time point.
```{r}
library(asaur)
library(survival)
tt = c(7,6,6,5,2,4)
cens = c(0,1,0,0,1,1)
Surv(tt, cens)

resultsFH = survfit(Surv(tt, cens) ~ 1, conf.type = "log-log", type = "fh")
summary(resultsFH)
```
Here we are trying to get the difference between survival rates among a treatment and control group.  We have the time that we  
```{r}
tt = c(6,7,10, 15, 19, 25)
delta = c(1,0,1,1,0,1)
trt = c(0,0,1,0,1,1)

survdiff(Surv(tt, delta)~trt)
```
Next step with different data set.  Attach means you can access the variable names directly.  This might be useful when trying to access variable when you make a bunch of transformation.  Just attach each time and then keep the name the same.  

So get the overall progression rate.

When rho is 1 then there is more emphasis placed at the beginning of the suvival curve 

We are looking at the difference between survual curves or probaiblitesi among two gruops.  First create a survival data set then do the difference function.
```{r}
head(pancreatic)
attach(pancreatic)
Progress.d = as.Date(as.character(progression), format = "%m/%d/%Y")
OnStudy.d = as.Date.character(as.character(onstudy), format = "%m/%d/%Y")
Death.d = as.Date(as.character(death), format = "%m/%d/%Y")


progressOnly = Progress.d-OnStudy.d
overallSurvival = Death.d - OnStudy.d
pfs = pmin(progressOnly, overallSurvival)
pfs
pancreatic$stage
attach(pharmacoSmoking)
survdiff(Surv(ttr, relapse) ~ grp, rho = 1)
```
Now trying multivariate with fixed factors.  TTR is time to something, and relapse if they dropped out and did not go into relapse they are taken out of the numerator at the end or when they drop out.
Compare nested models with ANOVA

Here is an example of a full cox model assessing survial rates over time.  TTR is how much time until relapse, relapse is the censoring indicator with 1 meaning they replased and 0 meaning they did not relapse.  Employment is a categorical variable describing their employment at baseline.  We develop two model one with employment and one with out.  To interpret the regression coefficent you need to expontiate them, which can usually be interpreated as the percentage increase for coefficent  above 1 and a percentage decrease for coefficent below one.  For example, grppathOnly there was a statistically significantly decrease 45% in time to death relative to the patch only.   

We then compared the models with and without additional variable employment using the anova function.  We found no statistically significant difference between the two models indicating that employment is not explaining any significant changes in survival time.
```{r}
attach(pharmacoSmoking)
grp= relevel(grp, ref= 2)
head(pharmacoSmoking)
modelA = coxph(Surv(ttr, relapse) ~ grp)
summary(modelA)
1-exp(modelA$coefficients)
modelC = coxph(Surv(ttr, relapse) ~ grp + employment)
summary(modelC)
anova(modelA, modelC)
```
Now doing spline regression with some outcomes, because at different levels of the outcome there maybe a nonlinear relationship (i.e. stronger relationshop between younger folks than older folks).  No evidence that a nonlinear relationship is statistically signifantly related so no nolinear form.
```{r}
attach(pharmacoSmoking)
modelSpline = coxph(Surv(ttr, relapse) ~ grp + employment + pspline(yearsSmoking, df = 0))
summary(modelSpline)

```
Now checking model assumptions
```{r}

```




