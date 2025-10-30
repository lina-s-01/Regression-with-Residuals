# Regression with residuals

This guide explains how to apply the regression-with-residuals approach to control for a covariate (such as body weight) that may be collinear with other predictors in a mixed-effects model. This method is useful when you want to assess the effect of an experimental factor (e.g., surgery, disease state) on an outcome (e.g., brain structure) while accounting for confounding variables that are strongly correlated with both the predictor and outcome.

Including certain covariates directly in a model (like body weight) can introduce collinearity, especially when those covariates are affected by the experimental condition.  
The regression-with-residuals approach removes the shared variance between the covariate and the predictor, allowing you to include only the independent portion of the covariate in your model.

Reference: [Zhou & Wodtke, 2019](https://www.cambridge.org/core/journals/political-analysis/article/regressionwithresiduals-method-for-estimating-controlled-direct-effects/EA081ACCE12BD0C5BA9A59C414F6D411)


## Example: Controlling for body weight in a brain structure model

In this example, we aim to understand the effect of an intervention (ovariectomy in mice) on brain structure. Ovariectomy causes an increase in body weight, which in turn directly influences brain volume. As a result, post-ovariectomy weight gain can confound the interpretation of structural brain changes. Our goal is therefore to estimate the effect of ovariectomy on brain structure while accounting for the weight gain it induces.

### Step 1: Basic model

In the original model (model1), we first estimate longitudinal changes in brain structure following ovariectomy: 

```r
model1 <- mincLmer(Jacobians ~ group * poly(age, degree = 2) + (1|ID))
```

This model is a univariate voxel-wise nonlinear mixed-effects model which captures neuroanatomical change as a function of ```group``` and ```age```. 
```Jacobians```: Jacobian determinants; voxel-wise measure of brain structural change 
```group``` is a categorical variable (ovariectomized or control)
```age``` is a continuous variable modelled quadratically
```(1|ID)```: random effect; subject-level intercept

### Step 2: Controlling for body weight

Normally, we would run the following model to account for weight gain:    

```r
model2 <- mincLmer(Jacobians ~ group * poly(age, degree = 2) + weight + (1|ID))
```

However, including weight directly as a covariate introduces collinearity because weight is strongly associated with both group and age. The resulting multicollinearity makes it difficult to interpret the unique contribution of each predictor.

#### Regression with residuals approach

To tackle this, we can use the residuals of weight that are extracted from the weight model to isolate the variance in weight not explained by age-related growth, intervention group or repeated measures.

```r
weight_model <- Lmer(Weight ~ group * age + (1|ID))
```

This model explains how much of the variance in weight can be attributed to group, age, and repeated measures. The remaining unexplained variance (residuals) represents individual differences in weight independent of these predictors.

Now we can extract the residuals of the weight model and append them back to the main dataset (assuming it is called data). 

```r
data <- data %>%
  mutate(res_weight = residuals(weight_model))
```

#### Final model

Use the residualized weight variable as a covariate in your brain model:

```r
brain_res_model <- mincLmer(Jacobians ~ group * poly(age, degree = 2) + res_weight + (1|ID))
```

This final model maintains the same structure as the original but replaces raw weight with its residuals, reducing collinearity and improving interpretability.



