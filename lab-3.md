Lab 3: Script-kiddie machine learning model fitting and evaluation
==================================================================

* task uses one dataset, credit card fraud. load using fetch_openml
* use sklearn pipeline from that one example for test-train-split, preprocessing and fitting
* fit 3 different classifiers models (e.g., logistic regression, SVM, decision tree) with their defaults
* plot the three models on a ROC curve and on a precision-recall curve
* for each model, report accuracy, AUC of ROC, and AUC of precision-recall
* for the model with the highest precision-recall AUC, report the cutoff threshold that maximizes the F1 score. Report the confusion matrix for the same threshold.


What?
-----

See [this example notebook](https://github.com/deargle/deargle.github.io/blob/master/notebooks/ml_model_evaluation.ipynb). It has a lot of links as comments.

Also see [this example notebook](https://github.com/deargle/deargle.github.io/blob/master/notebooks/ml_datasets_examples.ipynb) for how to load datasets using `fetch_openml`

Also, read these recent posts on my blog.

* [Notes on ML Evaluation](https://daveeargle.com/2020/02/11/notes-on-ml-evaluation/)
* [OpenML Rabbithole](https://daveeargle.com/2020/02/06/openml-rabbithole/)



This lab is confusing
---------------------

Your mom.

I'm exhausted after making the example notebook. Submit pull requests that modify this lab writeup. If you can do everything I did in the lab write-up,
then mission accomplished.



Deliverables
------------

Open a github "Issue" _for each lab deliverable bullet listed below_. cc me. I will close the issue when you have completed the deliverable to my satisfaction.

* A jupyter notebook that does all of the things at the top of this lab document. Your notebook should be well-commented. You can either use code
  comments, or you can use markdown jupyter cells. Your final product should be something you're comfortable sharing as a portfolio piece.
* Do at least one pull request to improve this lab write-up.


