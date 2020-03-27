Project proposal for security analytics class
=============================================

_Notes from a conversation with the school of engineering cyber analytics professor_

Okay -- The security analytics students also need a project for my class. We came up with an idea that I thought might also work for yours. It's bigger than the learning objectives for my class only, but I think it jives well with yours. It is this:

That they make a few interoperable tools similar to SecurityOnion network security monitoring, which I had them use a bit last semester. The tools would be:

1.  that they train a machine learning model using python sklearn to predict whether netflow traffic is malicious. I have had them use kdcup99 labeled network data in the past, but I know there are a few other similar datasets floating around

2.  that they serialize that machine learning model and make it available as a web api, using python Flask. That that api accept a netflow record or records, and that it make a multiclass prediction for each record for whether it is malicious (or what kind of malicious)

3.  that they write another small program which receives a pcap file and which can parse netflow records out of it, using Bro or the like. This program would call the api from (2), receive the predictions, and if the predictions are over some threshold, present the results to the user somehow.

    3.5   that the program from 3 may save alerts it detects to a database

4.  that there would be another web app which queries the database from 3.5 and displays records from it in HTML. this app would also have other database interactivity features, such as a way to mark alerts as "resolved" etc. It would be an intrusion detection and NSM app, similar to Squert from SO (if you're familiar with security onion)

5.  that they deploy each of these apps online, to e.g. heroku

An addendum, I think this would be a valuable portfolio piece for my students as they apply to predictive analytics jobs and the like.



Datasets
--------

Two datasets.

They can use CTU-13.  They are familiar with it already.  The data looks a little different than kdcup.  It is netflow data.

It is labeled, yes, but mostly at the level of malicious vs benign.

I would also want them to present model evaluation statistics for the model(s) that they make -- precision-recall curves and the like

they have sklearn pipelines which make the model training part pretty flexible


Deadlines
---------

April 30th 2020 last day of classes, submission due then. Presentations last week of classes.
