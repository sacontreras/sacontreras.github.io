---
layout: post
title:      "A Dynamic Programming approach to Feature Selection using Cross-Validation"
date:       2020-01-16 04:29:19 +0000
permalink:  a_dynamic_programming_approach_to_feature_selection_using_cross-validation
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

<h2>The Algorithm</h2>
The algorithm builds the entire <i>candidate</i>-solution space (size is <span>\\({n \choose k}\\)</span>) and uses dynamic programming to <i>efficiently</i> search for optimal solutions.

It turns out the candidate-solution space does indeed have the two qualities required in order to leverage Dynamic Programming:
<ol>
    <li>Optimal Substructure</li>
    <li>Overlapping Sub-problems</li>
</ol>
    
The solution space is, therefore, built from the bottom up.  The idea is to utilize these properties in order to avoid traversing every combination of \\({n \choose k}\\) features, where \\(n\\) is the total number of *continuous* features that we start with and \\(k\\) varies from \\(1\\) to \\(n\\).  

Thus, feature-combinations occurring deeper in the tree will be consisered non-optimal (and thereby discarded) if they do not include optimal feature combinations occurring earlier in the tree.  Therefore, by using a Dynamic Programming approach, <b>we avoid needlessly recomputing and re-testing optimal sub-problems that have already been encountered</b>.

*Cross-validation* over 5 *k-folds* is used with a scoring method to select the model (built on training data) that produces the least RMSE and the difference between that RMSE vs. the RMSE computed on the testing data, **with Condition Number \\(\le 100\\)** (in order <b>to minimize colinearity</b>).  This basis is taken *directly* from statsmodels Github [source code](https://www.statsmodels.org/dev/_modules/statsmodels/regression/linear_model.html#RegressionResults.summary) for the OLS fit results `summary` method but I restrict it even further (statsmodels defines non-colinearity by virtue of this value being less than 1000). ("statsmodels.regression.linear_model — statsmodels v0.11.0rc1 (+56): RegressionResults.summary() method source code", 2019)

In this way, we minimize residuals and thereby select the most predictive model, based on the "best" (minimized \\(RMSE\\)) **non-colinear** feature-combination subset from the starting set of all features.

The procedure for this is summarized below in pseudo-code:<br><br>
<span class="pseudocode">
//this is the table of optimal sub-problems<br>
set \\( optimal\\_feature\\_subsets := \\) new list<br><br>
for \\( k := 1 \\) to \\( n \\) (where \\( n := |\{starting\\ features\}| \\))<br>
<span class="indent">
set \\( feature\\_subsets := \\) build each of \\( k\\_features := {n \choose k}=\frac{n!}{k! \cdot (n-k)!} \\) (from \\( n \\) starting features)<br>
set \\(depth := k - 1\\)<br><br>
for each \\( feature\\_subset \\) in \\( \\{feature\\_subset: feature\\_subset \in feature\\_subsets\\} \\)<br>
<span class="indent">
set \\( closest\\_prior\\_depth \\) \\( := min(len(optimal\\_feature\\_subsets)-1, depth-1) \\)<br><br>
//qualify that current \\( feature\\_subset \\) is built from the last optimal sub-problem already computed - if not, then discard it<br>
if \\( depth > 0 \\) and \\( closest\\_prior\\_depth \ge 0 \\) then<br>
<span class="indent">
set \\( last\\_optimal\\_feat\\_combo := optimal\\_feature\\_subset[closest\\_prior\\_depth] \\)<br>
if \\( last\\_optimal\\_feat\\_combo \\) not in \\( feature\\_subset \\) then<br>
<span class="indent">
continue #discard this \\( feature\\_subset \\) and loop to the next<br>
</span>
</span>
<br><br>
//otherwise this \\( feature\\_subset \\) contains \\( last\\_optimal\\_feat\\_combo \\) (or \\( depth==0 \\) and this \\( feature\\_subset \\) is embryonic)<br>
set \\( kf := \\) build 5-kfolds based on \\( feature\\_subset \\)<br>
<br>
for each \\(fold\\) in \\(kf\\) {<br>
<span class="indent">
split data set into \\( partition_{test} \\) and \\( partition_{train} \\)<br>
set \\( lin\\_reg\\_model := \\) build linear regression from \\( partition_{train} \\)<br>
set \\( target_{train\\_predicted} := \\) compute predictions with \\( lin\\_reg\\_model \\) from \\( partition_{train} \\)<br>
set \\( target_{test\\_predicted} := \\) compute predictions with \\( lin\\_reg\\_model \\) from \\( partition_{test} \\)<br>
set \\( RMSE_{train} := \\) compute Root Mean Squared Error between \\( target_{train\\_actual} \\) and \\( target_{train\\_predicted} \\) (i.e. - \\( RMSE \\) of <i>residuals</i> of \\( partition_{train} \\))<br>
set \\( RMSE_{test} := \\) compute Root Mean Squared Error between \\( target_{test\\_actual} \\) and \\( target_{test\\_predicted} \\) (i.e. - \\( RMSE \\) of <i>residuals</i> of \\( partition_{test} \\))<br>
append \\( (RMSE_{train}, RMSE_{test}) \\) to \\( scores\\_list_{fold} \\)<br>
</span>
<br><br>
set \\( scores\\_list_{fold, RMSE_{train}} := \\) extract all \\( RMSE_{train} \\) from \\( scores\\_list_{fold} \\)<br>
set \\( RMSE := \frac{\sum RMSE_{train}}{size(scores\\_list_{fold, RMSE_{train}})} \\)<br>
set \\( scores\\_list_{fold, RMSE_{test}} := \\) extract all \\( RMSE_{test} \\) from \\( scores\\_list_{fold} \\)<br>
set \\( \Delta RMSE := \frac{\sum |RMSE_{train} - RMSE_{train}|}{size(scores\\_list_{fold, RMSE_{train}})} \\)<br><br>
if \\( RMSE_{best} \\) is null then<br>
<span class="indent">
set \\( RMSE_{best} := RMSE \\)<br>
set \\( \Delta RMSE_{best} := \Delta RMSE \\)<br>
</span>
else<br>
<span class="indent">
if \\( RMSE < RMSE_{best} \\) AND \\( lin\\_reg\\_model.condition\\_number \le 100 \\) then<br>
<span class="indent">
set \\(RMSE_{best} := RMSE\\)<br>
set \\( \Delta RMSE_{best} := \Delta RMSE \\)<br>
set \\( optimal\\_feature\\_subsets[depth] := feature\\_subset \\)<br><br>
</span>
</span>
</span>
</span>
</span>
<br>

**This results in cross-validation selecting the best *non-colinear* feature-combination subset for each \\( k \\), from \\( n \\) starting features, that predicts the outcome, *price*, with the greatest accuracy (lowest \\( \Delta RMSE \\))**.

The total number of all possible combinations the algorithm will select from is \\( \sum_{r=1}^n {n \choose r} = {n \choose 1} + {n \choose 2} + \cdot \cdot \cdot + {n \choose n}= 2^n-1 \\), but it avoids traversing that entire space by leveraging dynamic programming.

That number can grow quite large rather quickly.

For instance, starting with \\( n=18 \\) features, we have \\( \sum_{r=1}^{18} {18 \choose r} = 2^{18}-1 = 262143 \\) possible combinations!

But, because it uses the dynamic programming approach (vs. brute force), this algorithm is nevertheless fast, all things considered.
<p><br><br>

<h2>Source Code</h2>

The full source code is written in Python, specifically for Jupyter Notebooks, and can was uploaded to <a href="https://github.com/sacontreras/dsc-dp-cross-validation" target="_blank">this github repository</a>.  But it can be adapted for use outside of Juptyer Notebooks as you wish.
<p><br>
The essence is captured in the following function, a snippet from full source code:
<pre>
def cv_selection_dp(
    X
    , y
    , folds=5
    , reverse=False
    , smargs=None):

    cols = ['n_features', 'features', condition_no, rsquared, adjusted_rsquared, pvals, rmse, delta_rmse]
    scores_df = pd.DataFrame(columns=cols)

    target_cond_no = None
    if smargs is not None:
        target_cond_no = smargs['cond_no']
    if target_cond_no is None:
        target_cond_no = 1000 #default definition of "non-colinearity" used by statsmodels - see https://www.statsmodels.org/dev/_modules/statsmodels/regression/linear_model.html#RegressionResults.summary

    cv_feat_combo_map, _ = cv_build_feature_combinations(X, reverse=reverse)

    if cv_feat_combo_map is None:
        return
    
    base_feature_set = list(X.columns)
    n = len(base_feature_set)
    
    best_feat_combo = []
    best_score = None

    best_feat_combos = []
    
    for _, list_of_feat_combos in cv_feat_combo_map.items():
        n_choose_k = len(list_of_feat_combos)
        k = len(list_of_feat_combos[0])
        depth = k-1
        s_n_choose_k = "{} \\choose {}"
        display(
            HTML(
                "&lt;p>&lt;br>&lt;br>Cross-validating ${}={}$ combinations of {} features (out of {}) over {} folds using score &lt;b>{}&lt;/b> and target cond. no = {}...".format(
                    "{" + s_n_choose_k.format(n, k) + "}"
                    , n_choose_k
                    , k
                    , n
                    , folds
                    , condition_no_and_pvals_and_rsq_and_adjrsq_and_rmse_and_delta_rmse
                    , target_cond_no
                )
            )
        )

        n_discarded = 0
        n_met_constraints = 0

        for feat_combo in list_of_feat_combos:           
            feat_combo = list(feat_combo)

            closest_prior_depth = min(len(best_feat_combos)-1, depth-1)
            if depth > 0 and closest_prior_depth >= 0:
                last_best_feat_combo = best_feat_combos[closest_prior_depth]
                last_best_feat_combo_in_current_feat_combo = set(last_best_feat_combo).issubset(set(feat_combo))
                if last_best_feat_combo_in_current_feat_combo:
                    #print("depth is {}, best_feat_combos[last_saved_depth]: {}".format(depth, best_feat_combos[last_saved_depth]))
                    #print("feat_combo: {}".format(feat_combo))
                    #print("best_feat_combos[last_saved_depth] in feat_combo: {}".format(last_best_feat_combo_in_current_feat_combo))
                    pass
                else:
                    n_discarded += 1
                    #display(HTML("DISCARDED feature-combo {} since it is not based on last best feature-combo {}; discarded so far: {}".format(feat_combo, last_best_feat_combo, n_discarded)))
                    continue

            _, _, _, _, score = cv_score(
                X
                , y
                , feat_combo
                , folds
                , condition_no_and_pvals_and_rsq_and_adjrsq_and_rmse_and_delta_rmse
            )

            #now determine if this score is best
            is_in_conf_interval = False not in [True if pval >= 0.0 and pval <= 0.05 else False for pval in score[3]]
            is_non_colinear = score[0] <= target_cond_no
            if is_non_colinear and is_in_conf_interval and (best_score is None or (score[1] > best_score[1] and score[2] > best_score[2])):
                n_met_constraints += 1
                best_score = score
                best_feat_combo = feat_combo
                if len(best_feat_combos) < k:
                    best_feat_combos.append(feat_combo)
                else:
                    best_feat_combos[depth] = feat_combo
                print(
                    "new best {} score: {}, from feature-set combo: {}".format(
                        condition_no_and_pvals_and_rsq_and_adjrsq_and_rmse_and_delta_rmse
                        , best_score
                        , best_feat_combo
                    )
                )
                data = [
                    {
                        'n_features': len(feat_combo)
                        , 'features': feat_combo
                        , condition_no: score[0]
                        , rsquared: score[1]
                        , adjusted_rsquared: score[2]
                        , pvals: score[3]
                        , rmse: score[4]
                        , delta_rmse: score[5]
                    }
                ]
                mask = scores_df['n_features']==k
                if len(scores_df.loc[mask]) == 0:
                    scores_df = scores_df.append(data, ignore_index=True, sort=False)
                else:
                    keys = list(data[0].keys())
                    replacement_vals = list(data[0].values())
                    scores_df.loc[mask, keys] = [replacement_vals]
        
        if n_discarded > 0:
            display(
                HTML(
                    "&lt;p>DISCARDED {} {}-feature combinations that were not based on prior optimal feature-combo {}".format(
                        n_discarded
                        , k
                        , last_best_feat_combo
                    )
                )
            )
        if n_met_constraints > 0:
            display(
                HTML(
                    "&lt;p>cv_selection chose the best of {} {}-feature combinations that met the constraints (out of {} considered)".format(
                        n_met_constraints
                        , k
                        , n_choose_k - n_discarded
                    )
                )
            )
        if n_choose_k - n_discarded - n_met_constraints > 0:
            display(
                HTML(
                    "&lt;p>{} {}-feature combinations (out of {} considered) failed to meet the constraints&lt;p>&lt;br>&lt;br>".format(
                        n_choose_k - n_discarded - n_met_constraints
                        , k
                        , n_choose_k - n_discarded
                    )
                )
            )
    
    display(HTML("&lt;h2>Table of cv_selected Optimized Feature Combinations&lt;/h2>"))
    print_df(scores_df)

    display(HTML("&lt;h4>cv_selected best {} = {}&lt;/h4>".format(condition_no_and_pvals_and_rsq_and_adjrsq_and_rmse_and_delta_rmse, best_score)))
    display(
        HTML(
            "&lt;h4>cv_selected best feature-set combo ({} of {} features) {} based on {} scoring method with target cond. no. {}&lt;/h4>".format(
                len(best_feat_combo)
                , len(base_feature_set)
                , best_feat_combo
                , condition_no_and_pvals_and_rsq_and_adjrsq_and_rmse_and_delta_rmse
                , target_cond_no
            )
        )
    )
    display(HTML("&lt;h4>starting feature-set:{}&lt;/h4>".format(base_feature_set)))
    to_drop = list(set(base_feature_set).difference(set(best_feat_combo)))
    display(HTML("&lt;h4>cv_selection suggests dropping {}.&lt;/h4>".format(to_drop if len(to_drop)>0 else "&lt;i>no features&lt;/i> from {}".format(base_feature_set))))

    return (scores_df, best_feat_combo, best_score, to_drop)
</pre>
<p><br><br>
<h2>Example</h2>

I have written a <a href="https://github.com/sacontreras/dsc-dp-cross-validation/blob/master/CrossValFeatureSelection.ipynb" target="_blank">sample Jupyter Notebook</a> that utilizes the API and uploaded it to the <a href="https://github.com/sacontreras/dsc-dp-cross-validation" target="_blank">same repository</a> in which the source code is found.
<p><br>
Here are a few snippets from that notebook.
<p><br>
<h3>Snippet 1:</h3>
<img src="https://raw.githubusercontent.com/sacontreras/dsc-dp-cross-validation/master/output-1.png">
<br>
<h3>Snippet 2:</h3>
<img src="https://raw.githubusercontent.com/sacontreras/dsc-dp-cross-validation/master/output-2.png">
<br>
<h3>Snippet 3:</h3>
<img src="https://raw.githubusercontent.com/sacontreras/dsc-dp-cross-validation/master/output-3.png">
<p><br><br>
Please refer to <a href="https://github.com/sacontreras/dsc-dp-cross-validation/blob/master/CrossValFeatureSelection.ipynb" target="_blank">the notebook</a> for the full example.
<p><br><br>
<h1>References</h1>
Gavis-Hughson, S. (2019). The Ultimate Guide to Dynamic Programming - Simple Programmer. Retrieved from https://simpleprogrammer.com/guide-dynamic-programming/
<p><br>
statsmodels.regression.linear_model — statsmodels v0.11.0rc1 (+56): RegressionResults.summary() method source code. (2019). Retrieved from https://www.statsmodels.org/dev/_modules/statsmodels/regression/linear_model.html#RegressionResults.summary
<p><br>
Rabbit, B. (2018). Revisions to Is there a math nCr function in python? [duplicate]. Retrieved from https://stackoverflow.com/posts/4941932/revisions
