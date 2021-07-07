# Lab 4 -- Deploy Something

'Tis good to actually _use_ machine learning models. Goal of this lab, deploy a model.

I made an example repo that does the deed: [here](https://github.com/deargle/security-analytics-deploy-model)

To make the above repo, I did the following:

1.  Pickled an sklearn `pipeline` (not just the model!) from [this workflow of mine](https://github.com/deargle/deargle.github.io/blob/master/notebooks/ml_model_evaluation.ipynb)

    To get this, see [python docs here](https://docs.python.org/3/library/pickle.html#examples). Remember,
    I dumped the _entire pipeline_, after it had been fit. I did something like this within my pipeline-fitting
    loop:
    
        with open('{}.pkl'.format(classifier_name), 'wb') as f:
            pkl.dump(clf, f)
            
2.  Created a Flask application which loads the model on startup
3.  Copied the pickled classifier from step 1 into the directory of the new flask application
3.  Created functionality to read from a `.env` file a list of the
    features which the model actually retains after column transformation
4.  Paste the entire feature list from the original dataset in as a variable 
    for the flask application to see.
    
    To get this, I ran `list(pandasdataframe.columns.values)` on the untransformed dataset
5.  Stole a bootstrap starter template and used Flask's Jinja templating to create a simple
    webform which can respond to POST and GET methods. Use Jinja looping to output fields etc.
6.  Threw all of it into a heroku app. I had to `heroku config:set` the `MODEL_NAME` env var and the `MODEL_FEATURES` env var.
    
    * Go through a quick heroku tutorial
    
Flask is a widely-accessible web server which should be your friend. Jinja templating should also be your friend.
And heroku should be your friend. Heroku runs on AWS, but you will _not_ be bit by unexpected charges. Their free tier is very nice. You can leave an
app on there forever without getting charged unless you explicitly enable billing.

Some tips and tricks:
*   Go through a heroku python tutorial
*   Go through a flask quickstart
*   In the flask quickstart, you will see references to `flask run` commands, sometimes prefaced by environmental vars.
    I am using a trick where I put my local env vars in a `.env` file, and then I "source" that `.env` file to get its
    contents written to my... environment. This is the contents of my `.env` file:
  
        FLASK_APP=app.py
        FLASK_ENV=development
        MODEL_NAME=LogisticRegression.pkl
        MODEL_FEATURES="src_bytes,dst_bytes,protocol_type"
        
    You should adjust your accordingly. But note the two FLASK ones. After I have run `. .env`, (the `.` is an alias for `source`),
    having those vars in my env let me just run `flask run` without needing to run `FLASK_APP=app.py FLASK_ENV=development flask run` every time.
    Because I'm lazy.
*   Don't forget to set your heroku env vars.
*   To push to heroku, you have to turn your flask app into a git repository (`git init`). Heroku should (can) set a git remote in that repo called `heroku`,
    which points to the correct heroku url, which you can use by runnning `git push <remote> <branch>` e.g. `git push heroku master`. You need to have made
    at least one commit to your repo before you can push it! I usually do the following:
    
        git init
        git add .
        git commit -m 'initial commit'
        
    After that, as with any git repo, you can add other remotes -- like a link to a github one. I created a blank (totally blank!) github repo, grabbed its 
    "clone" url, and then from my git terminal I did `git remote add origin <github clone url>`. Then, I could do `git push origin master` to get a copy of 
    my git repo up on github.
   
   
# Deliverable

Using a dataset of your choice, go through the steps of deploying a model. Keep the features simple. You can fork and use my malware network traffic dataset.
Open a github issue pointing me to your heroku public url and also to a github upload of your heroku app.
   