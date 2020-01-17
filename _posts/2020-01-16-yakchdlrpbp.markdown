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

But seriously, though, using the King County housing data set to build a Linear Regression project is so common these days that one can almost say, "if you've seen one then you've seen them all".

Well, my hope is that my project stands out so that one can take away something new so that the above euphemism doesn't apply.

You be the judge.
<p><br>
<h2>PROJECT OVERVIEW</h2>
<h3>Goals</h3>
Of course A goal in my project was to build a linear regression model that is statistically "reliable".

But, for me, an even more important goal was to put that model to use to solve a hypothetical "real world" problem one might encounter when selling a home in King County.

To that end, those goals were:
<ol>
<li>to <b><i>provide an answer to the question, "which features can be FEASIBLY addressed by a seller to increase the sale price of his/her home?"</i></b></li>
<li>to <b><i>provide a CONCRETE strategy using an optimized linear regression model to accomplish this</i></b>.</li>
</ol>
<p><br>
<h3>Approach</h3>
<span markdown="1">
My high-level approach was to build the most "robust", most predictive model - that is, with the highest *Coefficient of Determination*, \\(R^2\\), that *reliably* predicts our target, **price**, with minimized Root Moon Squared Error in the residuals - on the most optimal set of statistically significant features as possible.
</span>
<p><br>

<h3 markdown="1">High \\(R^2\\) is not enough!</h3>
<o><br>
<span markdown="1">
We are not only interested simply in a high value in the the model's *Coefficient of Determination*, \\(R^2\\), but we also want a good feel for the confidence of that measure.
</span>
<p><br>

<h3>"Overfitting" must be minimized</h3>

As part of the procedure when building the final linear regression model, I minimize overfitting with the use of *cross-validation* and combinatorics.
<p><br>

<h4>Data Bias must be minimized as much as possible</h4>
<span markdown="1">
I employ the standard *hold-out* set technique to separate data when building models into the usual 0.70/0.30 split ratio of *training* data to *testing* data sets.  Models are built using training data.  Part of stastical "reliability" is achieved targeting models with the minimal \\(\Delta RMSE\\) between the *predicted* target (**price**) values from the *training* data set vs. the actual target values from the hold-out *testing* data set.
</span>
<p><br>

<h4>Model validation, Multicollinearity, and Feature Selection</h4>
<span markdown="1">
Confidence in the computed *Coefficient of Determination*, \\(R^2\\), itself must be "measured" since not all \\(R^2\\)'s that are equal are created "equally" *given the possibility for collinearity* as well as "over-fitness"!  **These facets, if not addressed, will render a linear regression model stastically *unreliable***.
<br><br>
In order to produce a model in which we can be confident in \\(R^2\\), I validate it deterministically.
<br><br>
As will be shown, multicollinearity is a problem.  When multicollinearity is present in a model, we cannot be confident in the statistical significance (*p-value*) of a collinear predictor.  So, collinear predictors must be identified and dealt with in order to provide confidence in the measure of \\(R^2\\) and stastical signficance, in general.
</span>
<br><br>
There are two means of handling collinearity of predictors:
<ol>
    <li>Introduce an <i>interaction term</i>, which will effectively combine two collinear predictors in the model, or</li>
    <li>Drop a term (feature) from a given set of collinear predictors.</li>
</ol>
<br><br>
Either approach taken must be backed by mathematical rationale.  That is, some mathematically deterministic method must be employed to first *select* (identify) which features are collinear.

<p><br>
But, I take the latter approach.
<p><br>

<h4>Variance Inflation Factor</h4>
According to James, Witten, Hastie & Tibshirani,
<span class="indent">
a ... way to assess multi-collinearity is to compute the variance inﬂation factor (VIF). The VIF is the ratio of the variance of [the coefficient of a predictor] when ﬁtting the full model divided by the variance of [the coefficient of a predictor] if fit on its own. The smallest possible value for VIF is 1, which indicates the complete absence of collinearity. Typically in practice there is a small amount of collinearity among the predictors. As a rule of thumb, a VIF value that exceeds 5 or 10 indicates a problematic amount of collinearity.
<br><br>
(James, Witten, Hastie &amp; Tibshirani, 2012)
</span>
<p><br>

<h4>Exploratory Data Analysis and Regression Diagnostics</h4>
<span markdown="1">
Utilizing Regression Diagnostics is a key part of the Exploratory Data Analysis phase.  A primary output of this phase is to produce a cleaned data set in which outlier and null values have been "dealt with".  Additionally, Regression Diagnostics, via visualization, provide some insight into the distributions and kurtosis of predictors, as they relate to the target response variable.  **Regression Diagnostics are, therefore, utilized to provide insight into whether or not predictors must be *transformed***.
</span>
<p><br>

<h4>Forward Selection of Features</h4>
<span markdown="1">
Rather than making "educated guesses" in the feature selection process, after cleaning the data set, I use *cross-validation*, *k-folds*, and *combinatorics* to select the "best" model (from the best feature-set combination) built on training data when compared to testing data, based a simple, ***greedy* forward selection** using dynamic programming. 

I authored <a href="https://sacontreras.github.io/a_dynamic_programming_approach_to_feature_selection_using_cross-validation">another blog post</a> focusing specifically on the algorithm I wrote to accomplish this task.

But, some effort is made up front to "intelligently" reduce the set of starting features by building a preliminary model and then removing features which obviously do not inluence **price** or are highly correlated, based on Regression Diagnostics as well as the *Variance Inflation Factor* (VIF) of a given feature.
</span>
<p><br>

<h4>Conditions for success - i.e. whether a linear regression model is "good" or "bad"</h4>
<span markdown="1">
Given the following conditions, we have a "GOOD" model when:
<br>
1. \\(R^2 > .60\\)
2. \\(|RMSE(test) - RMSE(train)| \approx 0\\)
3. low <i>Condition Number</i> (measure of collinearity)... much less than 1000; but **I target Condtion Number threshold of 100 or less**
<br><br>
The first condition says that we want models that determine the target with greater than 60% "confidence".
<br><br>
The second condition says that the bias toward the training data is minimal when compared to how the model performs on the "hold-out" test data.
<br><br>
The third condition requires that collinearity be mitigated/minimized.
</span>
<p><br>

<h4>Toward Regression: Most important aspect is understanding the data!</h4>
To that end, I follow the standard OSEMN model (Lau, 2019) and proceed according to the following steps:
<ol>
    <li>Import all necessary libraries and the the data set</li>
    <li>Cleaning the data set:
        <ol>
            <li>Clean null values, if any</li>
            <li>Clean outlier values, if any</li>
            <li>Convert feature data types as necessary</li>
        </ol>
    </li>
    <li>EDA: Gain familiarity with the data set by building a preliminary linear regression model
        <ol>
            <li>Utilize Regression Diagnostics: <b>using Regression Diagnostics is of fundamental importance since it will provide some guidance on whether or not a feature should be transformed</b></li>
            <li>Explore distriubutions</li>
            <li>Explore collinearity</li>
        </ol>
    </li>
    <li>EDA: Scale, Normalize, Transform features in the data set as necessary for the working model</li>
</ol>
<p><br>

<h4>Build the "Optimal" Model</h4>
After the data is cleaned and all of the required transformations exexuted on the predictors, and after having initially "weeded out" predictors which do not manifest a linear relaltionship to the target response variable, I execute the above-mentioned algorithm to select the set of features which will result in the optimal model according to the criteria mentioned in the "Conditions for Success" section.  At this point, I will have the optimal model for this data set that is statistically "reliable".
<p><br>

<h4>Run linear regression experiments to answer real questions</h4>
Now this is arguably the "funnest" phase of the project.

With the optimal model in hand, I attempt to apply it toward a hypothetical "real world" problem:
<ol>
    <li>Which features can be feasibly addressed by a seller to increase the sale price of his/her home?</li>
    <li>Provide a concrete strategy using the optimized linear regression model to accomplish this.</li>
</ol>
<p><br>

<h2>THE PROCESS</h2>
<h3>Cleaning the Data Set and EDA</h3>
<img src="img/3/EDA-1.png">
<p>
<img src="img/3/EDA-2.png">
<p>
<img src="img/3/EDA-3.png">
<p>
<img src="img/3/EDA-4.png">
<p>
<img src="img/3/EDA-5.png">
<p>
<img src="img/3/EDA-6.png">
<p>
<img src="img/3/EDA-7.png">
<p>
<img src="img/3/EDA-8.png">
<p>
<img src="img/3/EDA-9.png">
<p>
<img src="img/3/EDA-10.png">
<p>
<img src="img/3/EDA-11.png">
<p><br>
<font style="font-size:smaller">Note: output has been truncated in order to save space, therefore the list above is incomplete; for the full list, please refer to <a href="https://github.com/sacontreras/dsc-project-module-01/blob/master/Step2.ipynb" target="_blank">the Step 2 Jupyter notebook</a>.</font>
<p><br>
<img src="img/3/EDA-12.png">
<p>
<img src="img/3/EDA-13.png">
<p>
<img src="img/3/EDA-14.png">
<p>
<img src="img/3/EDA-15.png">
<p>
<img src="img/3/EDA-16.png">
<p>
<img src="img/3/EDA-17.png">
<p>
<img src="img/3/EDA-18.png">
<p>
<img src="img/3/EDA-19.png">
<p><br>

<h3>EDA: Getting to know the data via a Preliminary Linear Regression Model (bassed on the cleaned data set)</h3>
<img src="img/3/EDA-PLRM-1.png">
<p>
<img src="img/3/EDA-PLRM-2.png">
<p>
<img src="img/3/EDA-PLRM-3.png">
<p>
<img src="img/3/EDA-PLRM-4.png">
<p>
<img src="img/3/EDA-PLRM-5.png">
<p>
<img src="img/3/EDA-PLRM-6.png">
<p>
<img src="img/3/EDA-PLRM-7.png">
<p>
<img src="img/3/EDA-PLRM-8.png">
<p>
<img src="img/3/EDA-PLRM-9.png">
<p>
<img src="img/3/EDA-PLRM-10.png">
<p><br>

<h3>EDA: Iterative Model Refinement</h3>
<img src="img/3/EDA-PLRM-11.png">
<p><br>
<font style="font-size:smaller">Note: output has been truncated in order to save space, therefore, I present only the resulting model comparison; for the full model output, please refer to <a href="https://github.com/sacontreras/dsc-project-module-01/blob/master/KingCountyHouseSales.ipynb" target="_blank">the main notebook</a>.</font>
<p><br>
<img src="img/3/EDA-PLRM-12.png">
<p>
<img src="img/3/EDA-PLRM-13.png">
<p>
<img src="img/3/EDA-PLRM-14.png">
<p><br>

<h3>Regression Diagnositcs: A Deeper Feature Understanding and a Pathway to Feature Tuning</h3>
<img src="img/3/FT-1.png">
<p>
<img src="img/3/FT-2-1.png">
<p>
<img src="img/3/FT-2-2.png">
<p><br>
<font style="font-size:smaller">Note: although it is not stated in "plain English" in the notebook, <i>the fact that the distrubtion of residuals is heteroskedastic (as well as the distribution of the predictor itself) suggests that <b>this predictor ought to be transformed (log-transformed)</b></i>.</font>
<p><br>
<img src="img/3/FT-2-3.png">
<p>
<img src="img/3/FT-3-1.png">
<p>
<img src="img/3/FT-3-2.png">
<p>
<img src="img/3/FT-3-3.png">
<p>
<img src="img/3/FT-4-1.png">
<p>
<img src="img/3/FT-4-2.png">
<p><br>
<font style="font-size:smaller">Note: although it is not stated in "plain English" in the notebook, <i>the fact that the distrubtion of residuals is homoskedastic (as well as a relatively normal distribution of the predictor itself) suggests that <b>this predictor should NOT be transformed (log-transformed)</b></i>; thus, it is simply tracked as a "normal" continuous variable (not to be transformed later).</font>
<p><br>
<img src="img/3/FT-4-3.png">
<p><br>
<font style="font-size:smaller">Note: output has been truncated in order to save space but from here I follow the same pattern to complete this study of all of the predictors in the last model to get a clear idea of which ones should be transformed and which shouldn't; for the full output of Regression Diagnostics, please refer to <a href="https://github.com/sacontreras/dsc-project-module-01/blob/master/KingCountyHouseSales.ipynb" target="_blank">the main notebook</a>.</font>
<p><br>

<h3>Feature Scaling/Normalization and Transformation: Put the insight garnered from the Analysis of Regression Diagnostics to use!</h3>
<img src="img/3/DE-1.png">
<p>
<img src="img/3/DE-2.png">
<p>
<img src="img/3/DE-3.png">
<p>
<img src="img/3/DE-4.png">
<p>
<img src="img/3/DE-5.png">
<p>
<img src="img/3/DE-6.png">
<p>
<img src="img/3/DE-7.png">
<p><br>

<h3>EDA, Round 2: Distributions and Linearity Study after scaling/transforming continuous predictors, as well as the response variable</h3>
<img src="img/3/EDA2-1.png">
<p>
<img src="img/3/EDA2-2.png">
<p>
<img src="img/3/EDA2-3.png">
<p>
<img src="img/3/EDA2-4.png">
<p>
<img src="img/3/EDA2-5.png">
<p>
<img src="img/3/EDA2-6.png">
<p><br>

<h3>Iterative Refinement Continues</h3>
<img src="img/3/PLRM2-1.png">
<p><br>
<font style="font-size:smaller">Note: output has been truncated in order to save space, therefore, I present only the resulting model comparison; for the full model output, please refer to <a href="https://github.com/sacontreras/dsc-project-module-01/blob/master/KingCountyHouseSales.ipynb" target="_blank">the main notebook</a>.</font>
<p><br>
<img src="img/3/PLRM2-2.png">
<p>
<img src="img/3/PLRM2-3.png">
<p><br>
<font style="font-size:smaller">Note: from there, I follow this same pattern through a few more iterations in an attempt to vet out the features that are the most collinear; it involved examing correlation matrices and so forth but I will skip over that output since, as it turns out, that effort is largely unnecesary when we can simply rely on mathematics to tell us the answer; so, I will just skip forward to the juicy part but for a full treatment of iterative refinement, please have a look at <a href="https://github.com/sacontreras/dsc-project-module-01/blob/master/KingCountyHouseSales.ipynb" target="_blank">the main notebook</a>.</font>
<p><br>

<h3>Submit Candidate-Feature Basis for Cross-Validation Forward Selection of Optimal Features</h3>
<img src="img/3/CVFFS-1.png">
<p><br>
<font style="font-size:smaller">Note: for details on how this is done, please refer to <a href="https://sacontreras.github.io/a_dynamic_programming_approach_to_feature_selection_using_cross-validation" target="_blank">the blog post</a> I wrote about this topic.</font>
<p><br>
<img src="img/3/CVFFS-2.png">
<p>
<img src="img/3/CVFFS-3.png">
<p>
<img src="img/3/CVFFS-4.png">
<p><br>
<font style="font-size:smaller" markdown="1">
Note: output has been truncated in order to save space but from here but pattern is followed until it exhausts all available combinations that qualify; the end results is a list of feature subsets (of length \\(k\\)) that are optimal based on: max \\(R^2\\), minimized RMSE, and Condition No, \\(\le 100\\) (and therefore are considered minimally collinear); to see the full output, please see the separate <a href="https://github.com/sacontreras/dsc-project-module-01/blob/master/CrossValFeatureSelection.ipynb" target="_blank">Cross-Validation Forward Selection of Features notebook</a> wrote specifically for this purpose..</font>
<p><br>
<img src="img/3/CVFFS-5.png">
<p>
<img src="img/3/CVFFS-6.png">
<p><br>
Skipping over some details that can be viewed in the main note book, we have the FINAL MODEL...
<p><br>
<img src="img/3/CVFFS-7.png">
<p>
<img src="img/3/CVFFS-8.png">
<p>
<img src="img/3/CVFFS-9.png">
<p><br>

<h3>The FINAL Model</h3>
<img src="img/3/FM-1.png">
<p><br>

<h4>And how does it compare to the very first Preliminary Model?</h4>
<img src="img/3/FM-2.png">
<p>
<img src="img/3/FM-3.png">
<p><br>
Now we can finally move on to the most entertaining portion of the project: A "REAL WORLD" PROBLEM!
<p><br>

<h2>PUTTING THE FINAL MODEL TO USE: Solving a Real World Problem</h2>
<img src="img/3/RWP-1.png">
<p>
<img src="img/3/RWP-2.png">
<p>
<img src="img/3/RWP-3.png">
<p><br>
Let us see if the mathematics agrees.
<p><br>
<img src="img/3/RWP-4.png">
<p>
<img src="img/3/RWP-5.png">
<p>
<img src="img/3/RWP-6.png">
<p>
<img src="img/3/RWP-7.png">
<p><br>
In order to answer this question, we must write a funtion to "undo" transformation and scaling (since that is how the values are stored in the dataframe).
<p><br>
The code below accomplishes this:
<p><br>
<img src="img/3/RWP-8.png">
<p>
<img src="img/3/RWP-9.png">
<p><br>
We can now answer this question:
<p><br>
<img src="img/3/RWP-10.png">
<p>
<img src="img/3/RWP-11.png">
<p>
<img src="img/3/RWP-12.png">
<p>
<img src="img/3/RWP-13.png">
<p>
<img src="img/3/RWP-14.png">
<p><br>
Answering this question requires the following special-purpose function:
<p><br>
<img src="img/3/RWP-15-1.png">
<p>
<img src="img/3/RWP-15-2.png">
<p>
<img src="img/3/RWP-15-3.png">
<p>
<img src="img/3/RWP-15-4.png">
<p>
<img src="img/3/RWP-15-5.png">
<p>
<img src="img/3/RWP-15-6.png">
<p>
<img src="img/3/RWP-15-7.png">
<p>
<img src="img/3/RWP-15-8.png">
<p>
<img src="img/3/RWP-15-9.png">
<p>
<img src="img/3/RWP-15-10.png">
<p>
<img src="img/3/RWP-15-11.png">
<p>
<img src="img/3/RWP-15-12.png">
<p>
<img src="img/3/RWP-16-1.png">
<p>
<img src="img/3/RWP-16-2.png">
<p>
<img src="img/3/RWP-16-3.png">
<p>
<img src="img/3/RWP-17.png">
<p>
<img src="img/3/RWP-18.png">
<p>
<img src="img/3/RWP-19.png">
<p><br>

<h2>FIN</h2>
<p><br><br>
<h1>References</h1>
James, G., Witten, D., Hastie, T., & Tibshirani, R. (2012). An Introduction to Statistical Learning with Applications in R [Ebook] (7th ed.). New York, New York: Springer Science+Business Media. Retrieved from https://faculty.marshall.usc.edu/gareth-james/ISL/ISLR%20Seventh%20Printing.pdf
<p><br>
Lau, Dr. C. H. (2019). 5 Steps of a Data Science Project Lifecycle. Retrieved from https://towardsdatascience.com/5-steps-of-a-data-science-project-lifecycle-26c50372b492
<p><br>
Kurtosis. (2019). Retrieved from https://www.investopedia.com/terms/k/kurtosis.asp
<p><br>
Introduction to Mesokurtic. (2019). Retrieved from https://www.investopedia.com/terms/m/mesokurtic.asp
<p><br>
Understanding Leptokurtic Distributions. (2019). Retrieved from https://www.investopedia.com/terms/l/leptokurtic.asp
<p><br>
What Does Platykurtic Mean?. (2019). Retrieved from https://www.investopedia.com/terms/p/platykurtic.asp
