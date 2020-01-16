---
layout: post
title:      "YAKCHDLRPBP?!"
date:       2020-01-16 20:21:34 +0000
permalink:  yakchdlrpbp
---
<style type="text/css" rel="stylesheet">
    .pseudocode {
        font-family: 'Lucida Console';
        font-weight: bold;
        display: inline-block;
        background-color: LightGray;
    }
    .indent {
        display: inline-block;
        margin-left: 40px;
    }
</style>

YAKCHDLRPBP stands for "Yet Another King County Housing Data Linear Regression Project Blog Post".

Yep.  That's right.  I just coined a new acronym.

But seriously, though, using the King County housing data set to build a Linear Regression project is so common these days that one can almost say, "if you've seen one then you'vre seen them all".

Well, my hope is that my project stands out so that one can take away something new, so that the above euphamism doesn't apply.

You be the judge.

## Project Goals
Of course A goal in my project was to build a linear regression that is statistically "reliable".

But, for me, an even more important goal was to put that model to use to solve a hypothetical "real world" problem one might encounter when selling a home in King County.

To that end, those goals were:
<ol>
<li>to <b><i>provide an answer to the question, "which features can be FEASIBLY addressed by a seller to increase the sale price of his/her home?"</i></b></li>
<li>to <b><i>provide a CONCRETE strategy using an optimized linear regression model to accomplish this</i></b>.</li>
</ol>

So let us dive right in!

## Linear Regression
### My Approach

My high-level approach was to build the most "robust", most predictive model - that is, with the highest *Coefficient of Determination*, \\(R^2\\), that *reliably* predicts our target, **price**, with minimized Root Moon Squared Error in the residuals - on the most optimal set of statistically significant features as possible.

#### High \\(R^2\\) is not enough!
We are not only interested simply a high value in the the model's *Coefficient of Determination*, \\(R^2\\), but we also want a good feel for the confidence of that measure.

#### "Overfitting" must be minimized
As part of the procedure when building the final linear regression model, I minimize overfitting with the use of *cross-validation* and combinatorics.

#### Data Bias must be minimized as much as possible
I employ the standard *hold-out* set technique to separate data when building models into the usual $0.70$/$0.30$ split of *training* data and *testing* data sets.  Models are built using training data.  Part of stastical "reliability" is achieved targeting models with the minimal \\(\Delta RMSE\\) between the *predicted* target (**price**) values from the *training* data set vs. the actual target values from the hold-out *testing* data set.

### Model validation, Multicollinearity, and Feature Selection
Confidence in the computed *Coefficient of Determination*, \\(R^2\\), itself must be "measured" since not all \\(R^2\\)'s that are equal are created "equally" *given the possibility for collinearity* as well as "over-fitness"!  **These facets, if not addressed, will render a linear regression model stastically *unreliable***.

In order to produce a model in which we can be confident in \\(R^2\\), I validate it deterministically.

As will be shown, multicollinearity is a problem.  When multicolinearity is present in a model, we cannot be confident in the statistical significance (*p-value*) of a colinear predictor.  So, collinear predictors must be identified and dealt with in order to provide confidence in the measure of $R^2$ and stastical signficance, in general.

There are two means of handling colinearity of predictors:
<ol>
    <li>Introduce an *interaction term*, which will effectively combine two colinear predictors in the model, or</li>
    <li>Drop a term (feature) from a given set of collinear predictors.</li>
</ol>

Either approach taken must be backed by mathematical rationale.  That is, some mathematically deterministic method must be employed to first *select* (identify) which features are colinear.

But, **I take the latter approach**.

#### Variance Inflation Factor
According to James, Witten, Hastie & Tibshirani,

    a ... way to assess multi-collinearity is to compute the variance inﬂation factor (VIF). The VIF is the ratio of the variance of [the coefficient of a predictor] when ﬁtting the full model divided by the variance of [the coefficient of a predictor] if fit on its own. The smallest possible value for VIF is 1, which indicates the complete absence of collinearity. Typically in practice there is a small amount of collinearity among the predictors. As a rule of thumb, a VIF value that exceeds 5 or 10 indicates a problematic amount of collinearity.  (James, Witten, Hastie & Tibshirani, 2012)

#### Exploratory Data Analysis and Regression Diagnostics
Utilizing Regression Diagnostics is a key part of the Exploratory Data Analysis phase.  A primary output of this phase is to produce a cleaned data set in which outlier and null values have been "dealt with".  Additionally, Regression Diagnostics, via visualization, provide some insight into the distributions and kurtosis of predictors, as they relate to the target response variable.

#### Forward Selection of Features
Rather than making "educated guesses" in the feature selection process, after cleaning the data set, I use *cross-validation*, *k-folds*, and *combinatorics* to select the "best" model (from the best feature-set combination) built on training data when compared to testing data, based a simple, ***greedy* forward selection** using dynamic programming. 

The details and pseudocode for the algorithm are listed in the [Cross-Validation Forward Selection of Features](Appendix.ipynb) in the appendix.  

So, some effort is made up front to "intelligently" reduce the set of starting features by building a preliminary model and then removing features which obviously do not inluence **price** or are highly correlated, based on Regression Diagnostics as well as the *Variance Inflation Factor* (VIF) of a given feature.

# References
James, G., Witten, D., Hastie, T., & Tibshirani, R. (2012). An Introduction to Statistical Learning with Applications in R [Ebook] (7th ed.). New York, New York: Springer Science+Business Media. Retrieved from https://faculty.marshall.usc.edu/gareth-james/ISL/ISLR%20Seventh%20Printing.pdf
