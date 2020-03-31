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


---

# Notes to self

CTU-13 website is [here](https://www.stratosphereips.org/datasets-ctu13). Labeled
botnet traffic. That website is essentially the online appendix for "An empirical comparison of botnet detection methods" Sebastian Garcia, Martin Grill, Jan Stiborek and Alejandro Zunino. Computers and Security Journal, Elsevier. 2014. Vol 45, pp 100-123. http://dx.doi.org/10.1016/j.cose.2014.05.011

And that paper says that the datasets published on the website were generated using
`argus` and `ra` (ra == "read argus"). SecurityOnion _used_ to include argus, but
has recently moved away because they say that zeek (formerly known as bro) handles
everything that argus does, and more.

The projects _include argus and ra configurations._ In the READMEs for each dataset.
[Here's an example](https://mcfp.felk.cvut.cz/publicDatasets/CTU-Malware-Capture-Botnet-1/2013-10-01_README):


```
Netflows
========
The netflows are generated using the 2013-08-12_argus.conf file, the 2013-08-12_ra.conf file and the 2013-08-12_ralabel.conf conf file. We are using bidirectional argus records.
The command used is this:
1- argus -F argus.conf -r file.pcap -w file.argus
2- ralabel -f ralabel.conf -r file.argus -w file.argus.labeled
3- mv file.argus.labeled file.argus (this is to add labels to the argus file)
4- ra -F ra.conf -Z b -nr file.argus > file.argus.netflow.labeled

If you need the netflows without the labels, just regenerate them without the ralabel command.
```

[And another](https://mcfp.felk.cvut.cz/publicDatasets/CTU-Malware-Capture-Botnet-42/):

```
*.binetflow.2format
This format was generated specifically for the UM.es university. The command is: ra -n -c, -r file.biargus.labeled -s saddr daddr proto sport dport state stos dtos swin dwin shops dhops stime ltime sttl dttl tcprtt synack ackdat spkts dpkts sbytes dbytes sappbytes dappbytes dur pkts bytes appbytes rate srate drate label > file.binetflow.2format
```

... where the latter [has a subdirectory](https://mcfp.felk.cvut.cz/publicDatasets/CTU-Malware-Capture-Botnet-42/detailed-bidirectional-flow-labels/) which contains a `ra.conf` and an `argus.conf`.

Email from Sebastian Garcia (first author on above paper):

```
Hi Dave

You can find in each of the files in the folders of the detailed-bidirectional-flow-labels

For example https://mcfp.felk.cvut.cz/publicDatasets/CTU-Malware-Capture-Botnet-42/detailed-bidirectional-flow-labels/

One for argus and the other for ra tools

The command is
argus -F Argus.conf -r file.pcap -w - | ra -r - -n -F ra.conf -Z b > file.binetflow

sebas
```

# configuring a system

* get a pcap, such as [this one](https://mcfp.felk.cvut.cz/publicDatasets/CTU-Malware-Capture-Botnet-42/botnet-capture-20110810-neris.pcap)


## argus in a docker container on heroku, maybe useful as a worker dyno

```
apt install argus-client

#https://devcenter.heroku.com/articles/container-registry-and-runtime
heroku container:login
IMAGE=opennsm/argus
docker tag <image> registry.heroku.com/<app>/<process-type>
docker pull opennsm/argus:latest
docker push registry.heroku.com/<app>/<process-type>
```

## flask docker container from scratch using ubuntu base image

[https://runnable.com/docker/python/dockerize-your-flask-application](https://runnable.com/docker/python/dockerize-your-flask-application)

* What are the heroku limits on postgresql disk size free tier?

  [this](https://stackoverflow.com/a/58488268/5917194) says no size limit, only
  row limit, but idk what would happen if mongo-style and huge data values in
  single row

* mongodb free tier on heroku gives 500MB limit, that's pretty good. [https://elements.heroku.com/addons/mongolab](https://elements.heroku.com/addons/mongolab)

* to store files larger than 16MB in mongodb, use GridFS. Easy integration with pymongo. [https://api.mongodb.com/python/current/examples/gridfs.html](https://api.mongodb.com/python/current/examples/gridfs.html)


---

So I'm thinking, have a flask app which can:

1. upload a file (a pcap file). The upload route would receive the file via
   javascript, and the javascript would show status messages like "uploading...".
   When the upload completes, flask will return a message saying as much, prompting
   the website to make a call to update the list of unprocessed pcaps that have
   been uploaded.

   Each uploaded file will have a button which would trigger a "process" call.
   Calling this would send a javascript call to flask which would ask it to
   `argus | ra` the file into netflows. There would be too many netflows to add
   to a database one line at a time, but in theory they could all be added
   to their own mongodb.

   The netflows would be fed through a prediction model. If any prediction is
   above some user-specified threshold, then save it as an alert to a postgresql
   database. When the processing is done, le javascript would receive a resolved
   Promise which would ask for a list of alerts. The alert would just read out
   the netflow, in an html table. The table would include a button to mark the
   netflow alert as "resolved." This would just delete (or flag?) the alert from
   the alerts database. This _could_ be a pymongo db also, but meh might as well
   make it relational just to show them postgresql as well as pymongo.

That would be it, really. That's a lot of web development, combined with heroku
deployment, making predictions with pickled models, etc.

Students would need to train a model on (one?) of the CTU-13 dataset scenarios.
(Maybe more than one?) I would ask them to present a report on their chosen
model. Like a CRISP-DM-style report, model evaluation metrics and feature importances.

Oh and students would need to make a docker container which would have flask and
argus installed alongside one another.


# GCP docker-ready container
* [docker-compose](https://cloud.google.com/community/tutorials/docker-compose-on-container-optimized-os) on gcp (harder than normal, but minimally-so with this how-to)

Hmm, [flask has its own .flaskenv](https://flask.palletsprojects.com/en/1.1.x/cli/#environment-variables-from-dotenv) for the `flask run` CLI.

I got a docker-compose working which I think will respect $PORT when the web app
is built on heroku, but which allows docker-compose to override and run `flask run`
otherwise (heroku doesn't use docker-compose). Repo here: [https://github.com/deargle/security-analytics-docker](https://github.com/deargle/security-analytics-docker)
