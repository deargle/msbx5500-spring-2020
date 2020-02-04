2-3-2020 Class Notes
====================

# Opening

* job search update
* share recent experiences
* opportunity to ask questions

# topics for today:

* analytics overview and random deep dive into pet subtopics
     * supervised vs unsupervised?
     * supervised
          * what is it
               * future predictor
               * guessing machine
               * shiny conceited self-important magic 8-ball orb
          * how does it decide what guesses to make
               * looks at answers from the past and looks for patterns (maybe)
                    * terminology
                         * independent variables, predictors, … synonyms
                         * dependent variable, target
               * types
                    * p-coin flip (coin could be weighted)
                    * mean regressor
                    * logistic regression
                    * SVM
                    * neural networks
                    * naïve bayes
                    * decision trees (supervised segmentation)
               * how to determine algorithm parameters?
          * how do you know whether it's any good
               * submit it to a test where you already know the answers
                    * what data do you test it against?
                         * what it was trained on?
                         * something it hasn't seen yet?
                              * if you don't have a lot of data to start with, this can hurt bad
                              * Cross-validation with folds?
               * count the number of "right" and "wrong" guesses it makes
               * metrics
                    * accuracy?
                         * sucks
                    * what else??? depends what you care about more
                         * TPR
                         * FPR
                         * PPV
                         * F1
                    * there's a million different synonyms for the above metrics
                    * all relate to a "confusion matrix"
                    * visualizations of performance?
                         * ROC Curve and AUC
                              * roc curve not impacted by imbalance in class priors
                                   * how could that be ?????
          * what do you do with a model once you've trained it?
               * make predictions!
          * do you ever update the model?
               * only if you get paid
               * how to update it?
                    * retrain entire model?
                    * naïve bayes, just increment counts?
          * for supervised, what if we're predicting more than just 2 outcomes?
               * pretend that you're not! if 9 outcomes, make 9 separate models, each of which asks "is it this particular outcome or not?" You will get 9 probabilities of class-membership for a particular datum.
          * Oh, what output format do you get for "predictions"?
               * get probabilities if you can!
                    * but we're trying to make a yes/no guess?
                         * use a cutoff threshold
                              * what threshold?
                                   * arbitrary
     * unsupervised?
          * but what can you do with data if you're not trying to make any predictions with it?
               * look for natural "groups"
                    * but how identify groups?
                         * measure distances between points
                              * Points???? huh?
                                   * each instance / row in your data is an n dimensional point, where n is the number of features
                                   * how to visualize a point in a, e.g., 100-dimensional space?
                                        * option 1: be a computer
                                        * option 2: "dimensionality reduction" e.g. Principal Component Analysis, tSNE
                                             * if 3d, how much "value" does that 3rd dimension add if all points could just as well lay on a 2d plain? This is the principle of dimensionality reduction. find a smaller number of "planes" / axes
                         * cluster based on distance jumps, compare points pairwise
                              * hierarchical clustering, dendograms