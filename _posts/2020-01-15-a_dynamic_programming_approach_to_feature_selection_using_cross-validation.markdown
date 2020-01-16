---
layout: post
title:      "A Dynamic Programming approach to Feature Selection using Cross-Validation"
date:       2020-01-16 04:29:19 +0000
permalink:  a_dynamic_programming_approach_to_feature_selection_using_cross-validation
---

## The Algorithm
The algorithm builds the entire <i>candidate</i>-solution space (size is <img src="https://quicklatex.com/cache3/0d/ql_77b3a68db664a7bd5bdad5352dd6010d_l3.png">) and uses dynamic programming to <i>efficiently</i> search for optimal solutions.

It turns out the candidate-solution space does indeed have the two qualities required in order to leverage Dynamic Programming:
    1. Optimal Substructure
    2. Overlapping Sub-problems.
    
<font style="font-size: x-small">Note: for an excellent guide on Dynamic Programming please see Gavis-Hughson, S. (2019) listed in the References section.</font>
    
The solution space is, therefore, built from the bottom up.  The idea is to utilize these properties in order to avoid traversing every combination of <img src="https://quicklatex.com/cache3/0d/ql_77b3a68db664a7bd5bdad5352dd6010d_l3.png"> features, where <img src="https://quicklatex.com/cache3/29/ql_831c2406b034c3ff4a4734ebb9a95129_l3.png"> is the total number of *continuous* features that we start with and <img src="https://quicklatex.com/cache3/b5/ql_f715c458bdf31ab130c365714436a3b5_l3.png"> varies from 1 to <img src="https://quicklatex.com/cache3/29/ql_831c2406b034c3ff4a4734ebb9a95129_l3.png">.  

Thus, feature-combinations occurring deeper in the tree will be consisered non-optimal (and thereby discarded) if they do not include optimal feature combinations occurring earlier in the tree.  Therefore, by using a Dynamic Programming approachy, <b>we avoid needlessly recomputing and re-testing optimal sub-problems that have already been encountered</b>.

*Cross-validation* over 5 *k-folds* is used with a scoring method to select the model (built on training data) that produces the least RMSE and the difference between that RMSE vs. the RMSE computed on the testing data, **with Condition Number <img src="https://quicklatex.com/cache3/04/ql_9675992d76ff1338f89a68e45eb74d04_l3.png">** (in order <b>to minimize colinearity</b>).  This basis is taken *directly* from statsmodels Github [source code](https://www.statsmodels.org/dev/_modules/statsmodels/regression/linear_model.html#RegressionResults.summary) for the OLS fit results `summary` method but I restrict it even further (statsmodels defines non-colinearity by virtue of this value being less than 1000). ("statsmodels.regression.linear_model — statsmodels v0.11.0rc1 (+56): RegressionResults.summary() method source code", 2019)

In this way, we minimize residuals and thereby select the most predictive model, based on the "best" (minimized $RMSE$) **non-colinear** feature-combination subset from the starting set of all features.

The procedure for this is summarized below in pseudo-code:<br><br>
<b>
&nbsp;&nbsp;&nbsp;set <img src="https://quicklatex.com/cache3/e2/ql_c4a3d1d91110b6817593708c295f19e2_l3.png"> #this is the table of optimal sub-problems<br><br>
&nbsp;&nbsp;&nbsp;for <img src="https://quicklatex.com/cache3/cd/ql_492369426810944b445aa6019e1fa6cd_l3.png"> to <img src="https://quicklatex.com/cache3/29/ql_831c2406b034c3ff4a4734ebb9a95129_l3.png"> (where <img src="https://quicklatex.com/cache3/c2/ql_67669894a1b5f90b828e772828fffec2_l3.png">) {<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;set <img src="https://quicklatex.com/cache3/d9/ql_853c2a75286925e908b9b24e68dd25d9_l3.png"> build each of <img src="https://quicklatex.com/cache3/6f/ql_a2d1e895cf2c46ec9066a6cccfcf536f_l3.png"> (from <img src="https://quicklatex.com/cache3/29/ql_831c2406b034c3ff4a4734ebb9a95129_l3.png"> starting features)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;set <img src="https://quicklatex.com/cache3/16/ql_8b8340cae2e5aed39e04221e24d5e816_l3.png"><br><br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;for each <img src="https://quicklatex.com/cache3/d4/ql_807b31c4c4fa5156cb77ceccba3ad7d4_l3.png"> in <img src="https://quicklatex.com/cache3/03/ql_42e48dbf5607ca98e15f6eff1ad74603_l3.png"> {<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;set <img src="https://quicklatex.com/cache3/2e/ql_07d3ceebdadcc921bce146d6383a3d2e_l3.png"><br><br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#qualify that current <img src="https://quicklatex.com/cache3/d4/ql_807b31c4c4fa5156cb77ceccba3ad7d4_l3.png"> is built from the last optimal sub-problem already computed - if not, then discard it<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if <img src="https://quicklatex.com/cache3/3c/ql_c043bc1d2d42b22f1eec5fea67514f3c_l3.png"> and <img src="https://quicklatex.com/cache3/df/ql_998f43c9d208a3b58add49f47979c5df_l3.png"> {<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;set <img src="https://quicklatex.com/cache3/ae/ql_28cec883d512946a7b2d0abd94e7deae_l3.png"><br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if <img src="https://quicklatex.com/cache3/9e/ql_178e9e178cb8c8575dc65bb28b4caa9e_l3.png"> not in <img src="https://quicklatex.com/cache3/d4/ql_807b31c4c4fa5156cb77ceccba3ad7d4_l3.png"><br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;continue #discard this <img src="https://quicklatex.com/cache3/d4/ql_807b31c4c4fa5156cb77ceccba3ad7d4_l3.png"> and loop to the next<br> 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<br><br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#otherwise this <img src="https://quicklatex.com/cache3/d4/ql_807b31c4c4fa5156cb77ceccba3ad7d4_l3.png"> contains <img src="https://quicklatex.com/cache3/9e/ql_178e9e178cb8c8575dc65bb28b4caa9e_l3.png"> (or <img src="https://quicklatex.com/cache3/f3/ql_ab231d742c2d1447ccf37b46793738f3_l3.png"> and this <img src="https://quicklatex.com/cache3/d4/ql_807b31c4c4fa5156cb77ceccba3ad7d4_l3.png"> is embryonic)<br> 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;set <img src="https://quicklatex.com/cache3/bc/ql_2121db5e49b90eca9211a7bffd9859bc_l3.png"> build 5-kfolds based on <img src="https://quicklatex.com/cache3/d4/ql_807b31c4c4fa5156cb77ceccba3ad7d4_l3.png"><br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;for each <img src="https://quicklatex.com/cache3/66/ql_caa57949b0e88ff4bcb60b1f069df166_l3.png"> in <img src="https://quicklatex.com/cache3/bc/ql_2121db5e49b90eca9211a7bffd9859bc_l3.png"> {<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;split data set into <img src="https://quicklatex.com/cache3/cd/ql_219c1ba60601b0242c78e68d401920cd_l3.png">and <img src="https://quicklatex.com/cache3/7f/ql_863e479341153515e2174045d4dd7c7f_l3.png"><br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;set <img src="https://quicklatex.com/cache3/65/ql_2fc88f15fd17183c7e8f7a46028f5765_l3.png"> build linear regression from <img src="https://quicklatex.com/cache3/7f/ql_863e479341153515e2174045d4dd7c7f_l3.png"><br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;set <img src="https://quicklatex.com/cache3/1f/ql_442e09a99a95a829a194caf446e81a1f_l3.png"> compute predictions with <img src="https://quicklatex.com/cache3/1a/ql_fecd64ba1a1bd808de662daa6d4bd11a_l3.png"> from <img src="https://quicklatex.com/cache3/7f/ql_863e479341153515e2174045d4dd7c7f_l3.png"><br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;set <img src="https://quicklatex.com/cache3/80/ql_df87ea20bcd15a5ca6574e5266a92880_l3.png"> compute predictions with <img src="https://quicklatex.com/cache3/1a/ql_fecd64ba1a1bd808de662daa6d4bd11a_l3.png"> from <img src="https://quicklatex.com/cache3/cd/ql_219c1ba60601b0242c78e68d401920cd_l3.png"><br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;set <img src="https://quicklatex.com/cache3/a3/ql_5a6c55e870d6a77241af15c49d033fa3_l3.png"> compute Root Mean Squared Error between <img src="https://quicklatex.com/cache3/af/ql_15e902e486ef42ddc13991b3de1bf5af_l3.png"> and <img src="https://quicklatex.com/cache3/c0/ql_fd67e9db04ded435c45400d1ad7fd1c0_l3.png">&nbsp;&nbsp;&nbsp;(i.e. - RMSE of <i>residuals</i> of <img src="https://quicklatex.com/cache3/7f/ql_863e479341153515e2174045d4dd7c7f_l3.png">)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;set <img src="https://quicklatex.com/cache3/a3/ql_5ef7840d7064de9e72233a3d2bebe5a3_l3.png"> compute Root Mean Squared Error between <img src="https://quicklatex.com/cache3/6c/ql_f115c72fc9d2218eec60ec2a1109a46c_l3.png"> and <img src="https://quicklatex.com/cache3/92/ql_b1d9b29e6749f21d3e91c6b8bfcd8192_l3.png">&nbsp;&nbsp;&nbsp;(i.e. - RMSE of <i>residuals</i> of <img src="https://quicklatex.com/cache3/cd/ql_219c1ba60601b0242c78e68d401920cd_l3.png">)<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;append <img src="https://quicklatex.com/cache3/4b/ql_50b283d401b62a15520551cc6164634b_l3.png"> to <img src="https://quicklatex.com/cache3/b0/ql_6a333daf3d99119f5163686569bf85b0_l3.png"><br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<br><br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;set <img src="https://quicklatex.com/cache3/d0/ql_61aa2f9f01c1e9d6509ef9364ebcd8d0_l3.png"> extract all <img src="https://quicklatex.com/cache3/83/ql_c41a8dec35945fabbdf35a2ec99fcb83_l3.png"> from <img src="https://quicklatex.com/cache3/b0/ql_6a333daf3d99119f5163686569bf85b0_l3.png"><br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;set <img src="https://quicklatex.com/cache3/1c/ql_e498bc31dd354bcfc5641f101fba461c_l3.png"><br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;set <img src="https://quicklatex.com/cache3/c8/ql_f968de185b504ccf1983bd8c6e4ddec8_l3.png"> extract all <img src="https://quicklatex.com/cache3/24/ql_044f3cc9d9411469e339bb7917e6a924_l3.png"> from <img src="https://quicklatex.com/cache3/b0/ql_6a333daf3d99119f5163686569bf85b0_l3.png"><br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;set <img src="https://quicklatex.com/cache3/4f/ql_ce27d5f9a5b089b0300a02d2ad71824f_l3.png"><br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if <img src="https://quicklatex.com/cache3/7e/ql_a19c8aeabb0c5e63878a15962e6f787e_l3.png"> is null then {<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;set <img src="https://quicklatex.com/cache3/b6/ql_5e3e703ceb77d59c44364b4f94dae2b6_l3.png"><br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;set <img src="https://quicklatex.com/cache3/50/ql_3d8431cdb1e071f4384239a36ded2050_l3.png"><br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;} else {<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if <img src="https://quicklatex.com/cache3/b0/ql_49878048a712d940aaf5fcf8f3ca0eb0_l3.png"> AND <img src="https://quicklatex.com/cache3/e3/ql_0eb730c3c90f85f74e74884869991ae3_l3.png"> then {<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;set <img src="https://quicklatex.com/cache3/b6/ql_5e3e703ceb77d59c44364b4f94dae2b6_l3.png"><br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;set <img src="https://quicklatex.com/cache3/50/ql_3d8431cdb1e071f4384239a36ded2050_l3.png"><br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;set <img src="https://quicklatex.com/cache3/60/ql_8e99bbd6bff37b5c35eab1c1b4b81e60_l3.png"><br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<br>
&nbsp;&nbsp;&nbsp;}<br>
</b>

<br><br>
**This results in cross-validation selecting the best *non-colinear* feature-combination subset for each <img src="https://quicklatex.com/cache3/b5/ql_f715c458bdf31ab130c365714436a3b5_l3.png">, from <img src="https://quicklatex.com/cache3/29/ql_831c2406b034c3ff4a4734ebb9a95129_l3.png"> starting features, that predicts the outcome, *price*, with the greatest accuracy (lowest <img src="https://quicklatex.com/cache3/47/ql_7953429af875db020f6595f75d80f447_l3.png">)**.

The total number of all possible combinations the algorithm will select from is<img src="https://quicklatex.com/cache3/db/ql_010b12517cc0e4477ee87e74a83905db_l3.png">, but it avoids traversing that entire space by leveraging dynamic programming.

That number can grow quite large rather quickly.

For instance, starting with $n=18$ features, we have <img src="https://quicklatex.com/cache3/da/ql_35f133817d0d6cc968387afbdc9cddda_l3.png"> possible combinations!

But, because it uses the dynamic programming approach (vs. brute force), this algorithm is nevertheless fast, all things considered.

<p><br><br>
## Source Code
The full source code is written in Python, specifically for Jupyter Notebooks, and can was uploaded to [this github repository](https://github.com/sacontreras/dsc-dp-cross-validation).  But it can be adapted for use outside of Juptyer Notebooks as you wish.

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

## Example
I have written a [sample Jupyter Notebook](https://github.com/sacontreras/dsc-dp-cross-validation/blob/master/CrossValFeatureSelection.ipynb) that utilizes the API and uploaded it to the [same repository](https://github.com/sacontreras/dsc-dp-cross-validation) in which the source code is found.

Here are a few snippets from that notebook.

### Snippet 1:
![snippet-1 could not be loaded](https://raw.githubusercontent.com/sacontreras/dsc-dp-cross-validation/master/output-1.png)

### Snippet 2:
![snippet-2 could not be loaded](https://raw.githubusercontent.com/sacontreras/dsc-dp-cross-validation/master/output-2.png)

### Snippet 3:
![snippet-3 could not be loaded](https://raw.githubusercontent.com/sacontreras/dsc-dp-cross-validation/master/output-3.png)

Please refer to [the notebook](https://github.com/sacontreras/dsc-dp-cross-validation/blob/master/CrossValFeatureSelection.ipynb) for the full example.



# References
Gavis-Hughson, S. (2019). The Ultimate Guide to Dynamic Programming - Simple Programmer. Retrieved from https://simpleprogrammer.com/guide-dynamic-programming/

statsmodels.regression.linear_model — statsmodels v0.11.0rc1 (+56): RegressionResults.summary() method source code. (2019). Retrieved from https://www.statsmodels.org/dev/_modules/statsmodels/regression/linear_model.html#RegressionResults.summary

Rabbit, B. (2018). Revisions to Is there a math nCr function in python? [duplicate]. Retrieved from https://stackoverflow.com/posts/4941932/revisions
