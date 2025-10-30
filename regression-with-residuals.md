# Regression with residuals

## Purpose

```
This guide explains how to apply the *regression-with-residuals* approach to control for a covariate (such as body weight) that may be **collinear** with other predictors in a mixed-effects model.  
This method is useful when you want to assess the effect of an experimental factor (e.g., ovariectomy) on an outcome (e.g., brain structure) while accounting for confounding variables that are strongly correlated with both the predictor and outcome.

Reference: [Zhou & Wodtke, 2019](https://www.cambridge.org/core/journals/political-analysis/article/regressionwithresiduals-method-for-estimating-controlled-direct-effects/EA081ACCE12BD0C5BA9A59C414F6D411)

---

Including certain covariates directly in a model (like body weight) can introduce **collinearity**, especially when those covariates are affected by the experimental condition.  
The regression-with-residuals approach removes the shared variance between the covariate and the predictor, allowing you to include only the **independent portion** of the covariate in your model.

---

## Example: Controlling for body weight in a brain structure model

In this example, we are intrested in understanding the effect of an intervention (ovariectomizing mice) on brain structure. Ovariectomy leads to body weight increase, and given that weight directly affects brain volume, the effect of weight gain following ovariectomy can lead to confounding effects on the outcome measure. Here we are trying to estimate the effect of ovariectomy on brain structure, while controlling for the weight gain triggered by ovariectomy. 

### Step 1: Basic model

In the original model that we would run (model1), we are trying to estimate the longitudinal structural brain changes following ovariectomy. 

```r
model1 <- mincLmer(Jacobians (brain structure measure) ~ group * poly(age, degree = 2) + (1|ID))
```

model1 is a univariate voxel-wise nonlinear mixed-effects models which examines the interaction between group and age as predictors of neuranatomical change derived from absolute jacobian determinants. Group is categorical vairable (ovariectomised or control). Age is a continuous variable modelled quadratically. 

### Step 2: Attempt to add weight directly as a covariate

Normally, you would run the following model to account for weight gain:    

```r
model2 <- mincLmer(Jacobians ~ group * poly(age, degree = 2) + weight + (1|ID))
```

However, what you notice here is that including the variable of weight as a covariate in the model introduces collinearity (given that weight dramatically increases with age following ovariectomy). 
The resulting multicollinearity makes it difficult to interpret the unique contribution of each predictor.


To tackle this, we can use the residuals of weight that are extracted from the weight model to isolate the variance in weight not explained by age-related growth, intervention group or repeated measures.  
```r
weight_model <- Lmer(Weight ~ group * age + (1|ID))
```

This model explains how much of the variance in weight can be attributed to group, age, and repeated measures.
The remaining unexplained variance (residuals) represents individual differences in weight independent of these predictors.

Now we can extract the residuals of the weight model and append them back to the main dataset (assuming it is called data). 
data <- data %>%
  mutate(res_weight = residuals(weight_model))

### Final model
Use the residualized weight variable as a covariate in your brain model:
The final model will include the residuals as a covariate in the same univariate voxel-wise mixed-effect model, which otherwise retained the original strucutre:

```r
brain_res_model <- mincLmer(Jacobians ~ group * poly(age, degree = 2) + res_weight + (1|ID))
```
This final model maintains the same structure as the original but replaces raw weight with its residuals, reducing collinearity and improving interpretability.


```


