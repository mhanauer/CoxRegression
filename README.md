---
title: "Test"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
Teaching survival analysis
```{r}
library(survival)
library(survminer)
library(dplyr)
library(psych)
```
Need to think about right censoring.  Because people don't actually get the event (i.e. don't die or don't respond to treatment).  Therefore, for some people it unclear how much time it takes to reach some event.
Censoring can never happen, because of the event (i.e. because of death).  

So I think the survival function works by taking the proportion of people that survived at time one, mutiplied by prop at time and so on until all t's.

Something about censoring data at a certain point.  

Need cox model to include covariates

Cox model assuming the hazards are the same for a subgroup across time

What is the difference between hazzard and surivial?

Why were they censored?  Because they did not have the event happen to them?  So are they excluded from the analysis?

I think all variables in this model have to be binary?  So need to dichotmize the age variable.

Need to make sure all covariates are factors for later analysis (just do 1's and 0's for now).
```{r}
data(ovarian)
head(ovarian)
sum(ovarian$fustat)
age = ovarian %>% mutate(ageGroup = ifelse(age >=50, "0", "1"))
```
Need to create a survival object, which I think just identifies which data points to censored or excluded?

Then I think even if you want to run a cox model

First run a Kaplan-Meier, because that is what you do first?  I don't think you include any covariates in this model?

I think we are getting surival or harzzard rates for those in two different treatments.

Don't understand how the censoring works in the data analysis.  When is the data censored?  After the censoring occurs?
```{r}
survObject = Surv(time = ovarian$futime, ovarian$fustat)
survObject
fit1 = survfit(survObject ~ rx, data = ovarian)
summary(fit1)

ggsurvplot(fit1, data = ovarian, pval = TRUE)
```
Now try with cox model, which can include several covariates at the same time.
```{r}
fix.coxph = coxph(survObject ~ rx +resid.ds + age, + ecog.ps, data = ovarian)
summary(fix.coxph)

ggforest(fix.coxph, data = ovarian)
```

