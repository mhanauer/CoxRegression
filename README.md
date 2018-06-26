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

Want to see if the model is a good fit by comparing the residuals to the distribution of the covariates.  So if there is some relationship between the residuals with the covariate we may not have the correct relationship between the two (may need).  Residual is the difference between observed outcome and the expected based on model fit.  The expected number for this model should be similar if the model is a good fit.

Then we want to check the proportional assumption, which that harrzard functions are proportional over time.  So the differences should be the same over time.  They can be different, but must be the same difference.


Prop graph below is the difference in the beta for stage over time.  It should be the same over time not changing at different time points.
```{r}
library(survival)
resultNull = coxph(Surv(ttr, relapse)~ grp + gender)
rr.0 = residuals(resultNull, type = "martingale")
plot(rr.0 ~ grp) 


resultProp= coxph(Surv(pfs) ~ stage)
resultPropRes = cox.zph(resultProp, transform = "km")
resultPropRes
plot(resultPropRes)
```
You can deal with time varying covariates and thus time varying betas with time transfer function.  Not sure why you need the orginal varible without the tt.  
```{r}
resultPancTT = coxph(Surv(pfs)~ tt(stage))
resultPancTT
```
Now need to create your own data set and try to see if you can understand everything that is going on.  Still not sure that I understand how the censoring works.

https://www.datacamp.com/community/tutorials/survival-analysis-R

---
title: "Survival"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
Loading packages

Censoring takes place when someone drops and the censoring takes place at that time point. Put how do we know which time point the censoring takes place?  If long format, then the 1 would be placed at the time point in which the censoring takes place.

Futime is the time (let's just pretend days) until they either died or dropped out of the study.  So at that time point they are excluded from the study??  

So you need to define the event.  If death is the event, then they are not censored, because that is the event and we have the time until the event?  So if death is the event and they don't die, then those people will receive a one at the last time point.  If they are lost to follow up then they will receive one, because they will be censored at that time, because we do not know what happened them?  

So when the event happens is the data censored? Doesn't seem like it is?

The question is how do you construct a censoring variable.

Another question is including continous covariates in cox regression
You get a zero if you are censored.
```{r}
library(survminer)
library(survival)
library(dplyr)
ovarian <- ovarian %>% mutate(age_group = ifelse(age >=50, "old", "young"))
ovarian$age_group <- factor(ovarian$age_group)
```
Vertical lines indicate censored data.
```{r}
surv_object <- Surv(time = ovarian$futime, event = ovarian$fustat)
surv_object
fit1 <- survfit(surv_object ~ rx, data = ovarian)
fit1
summary(fit1)

ggsurvplot(fit1, data = ovarian, pval = TRUE)


ovarian$rx <- factor(ovarian$rx, 
                     levels = c("1", "2"), 
                     labels = c("A", "B"))
ovarian$resid.ds <- factor(ovarian$resid.ds, 
                           levels = c("1", "2"), 
                           labels = c("no", "yes"))
ovarian$ecog.ps <- factor(ovarian$ecog.ps, 
                          levels = c("1", "2"), 
                          labels = c("good", "bad"))

ovarian <- ovarian %>% mutate(age_group = ifelse(age >=50, "old", "young"))
ovarian$age_group <- factor(ovarian$age_group)


# Data seems to be bimodal
```
Now cox regression
```{r}
fit.coxph <- coxph(surv_object ~ rx + resid.ds + age_group + ecog.ps, 
                   data = ovarian)
ggforest(fit.coxph, data = ovarian)
```


```{r}
library(survminer)
library(survival)
ovarian
```

