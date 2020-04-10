Lab 6 -- Security Analytics Deploy using Databases via Docker
-------------------------------------------------------------

This lab integrates databases with the flask app. Specifically, `mongodb`
and `psql`, because why not. Heroku has add-ons that run both of these for you,
but we want to be able to test locally without having to rely on heroku services,
so hey let's use docker to run the databases locally.

Except haha you have Windows 10 Home, and at the time of writing this, it's not
easy to get docker running on Windows 10 Home unless you are using a developer
preview of W10H which includes WSL2. But hey you are you and you know GCP, and
GCP is good at docker, so let's just use that. Quit whining and buy a better
laptop eh?

Do this:
* GCP > probably create a new project > Compute Engine > VM instances >
  probably default resources, but change the boot disk OS to be a "Container
  Optimized OS." I think 10GB disk will be fine. **check the boxes to "allow HTTP
  traffic through the firewall.** Then, launch the instance. Connect to the instance
  using your favorite ssh method.

* This OS comes ready-to-go with Docker. It doesn't have `docker-compose` preinstalled
  though, and we'll need that, so follow [these instructions] to run docker-compose
  as -- heh -- a docker image, and to make an alias that will run that docker-compose
  docker image when you type `docker-compose`.

  [these instructions]: https://cloud.google.com/community/tutorials/docker-compose-on-container-optimized-os

* `git clone` [this repository](https://github.com/deargle/security-analytics-docker) and `cd` into it.
  That repository includes a `docker-compose` file in it. Look at that file:
  it declares three services, each of which is its own docker container. The first
  one, `web` I think, will be built from the directory of the git directory, as specified
  by the `Dockerfile` in the same. The other two -- mongodb and psql -- are pulled
  from the docker image repository.

  The docker-compose.yml file mounts the current directory as a "volume" within the
  docker container that will be built for `web` when you run `docker-compose build`.
  The docker-compose.yml file also specifies a port mapping for the web container.
  It will map container port `5000` to host port `80`. So, when this container is
  running, you should be able to visit your GCP instance's public IP address over
  http (not https!) and see your flask dev server running.

  Also contained in the directory is a `.flaskenv` file. This is a cool trick
  that doctors hate him for that flask will auto-read if you have `python-dotenv`
  available (peek in the Dockerfile and notice that it runs `pip install -r requirements.txt`,
  which requirements file contains, among other things, `python-dotenv`). Also,
  look in the docker-compose.yml file for web and notice that it runs `flask run`
  as its execution command. `flask run` will read from the `.flaskenv` to determine
  which environment variables to run, without you having to manually load the `.env`
  file each time you do this.

  But ah! We still need a `.env` file, because the `psql` image specified in the
  docker-compose.yml file needs to have `POSTGRES_PASSWORD` available on the
  environment. When the `docker-compose` command is run, it will itself read
  from a `.env` file to build out the docker-compose.yml configuration. These
  environment variables are not on their own passed into the _container_ running
  environment, but we _can_ pass env vars to running containers by specifying them
  in the docker-compose service `environment` yml key. Notice that that config
  file passes `POSTGRES_PASSWORD`. That's shorthand, docker-compose will expand
  that to `POSTGRES_PASSWORD=$POSTGRES_PASSWORD` when it runs `docker run` under
  the hood. Specifically, sonmething like `docker run POSTGRES_PASSWORD=$POSTGRES_PASSWORD`.

  Head hurt yet? Yay docker, it's so easy!

  Heroku doesn't let us use a docker-compose file, but we use that to our advantage here.
  Docker lets you register images on its own docker container registry, and then
  deploy those images as heroku apps. When we will register it, it will run from the
  `Dockerfile` in our current directory. Peek at the Dockerfile. Notice that down
  at the bottom it has a CMD statement that uses `gunicorn`. Recall that `flask run`
  is a development server and that it should _not_ be run in production. Whereas,
  gunicorn _is_ a production server. So what gives, why did we have a `command`
  specified in our docker-compose.yml that did `flask run`, but here's a `CMD` in
  the Dockerfile (the two are equivalent) which runs `gunicorn`? And now for the
  heart of the trick. The docker-compose `command` _overrides_ the Dockerfile
  `CMD`, so when we run `docker-compose up`, we get a dev environment. But!
  When we will deploy our docker container on heroku, only the Dockerfile will
  be used -- not docker-compose.yml -- so `gunicorn` will be run instead.

  Notice that the `gunicorn` command binds to port `$PORT`. That's not an env
  var that we provide. But, heroku _does_ provide that. It will be a random port.
  We must bind our web process to that port in order for heroku to serve our app
  when someone visits our url (which, recall, http uses port 80). Must do it. Must.

  If you don't understand all that, read it through 6 more times, and do a Docker
  quickstart and a docker-compose quickstart. It will take you probably, oh, 7 days
  to chew on.

* So! Now that you understand everything, let's get a dev environment.

  Run `docker-compose build`, then `docker-compose up` to run the images in your
  dev environment. The process will be run in the foreground, and you'll see colored
  log output for the three launched services. Check for any errors. In another shell instance,
  run `docker-compose ps` to list running processes. That gives a docker-compose-aware
  output of what you would/could otherwise see by running `docker ps`. (Go ahead, run
  both and compare outputs. They're querying the same daemon.) Notice that there
  is a host->guest port binding happening for the web container. Visit your public
  ip and check that the `/` route is working. (Look in `app.py` to see which
  routes go where).

* Databases!

* We'll use postgresql and also pymongo. There exist pip libraries for
  both that help them tie in to a flask app, so let's use them.

  We'll access our psql database url from the envvar `DATABASE_URL`, because that's
  where it will be found when we're using the psql heroku addon. Let's make this
  available to our app by adding it as an env var in the docker-compose.yml file.
  With docker-compose, a virtual private network is created which allows each
  service specified in a single docker-compose.yml file to access the others via
  simply its service name as the hostname. [the docs](https://hub.docker.com/_/postgres)
  say that the default user is `postgres`, and that the default schema will be the
  same name as the default user. The password is set in our env vars.

  Let's initialize the database using a script outside of our web server app.py.
  This way, we can insert some users just one time. I made a file in the repo called
  `init_psql_db.py` -- run it within the running container using docker-compose.

  Check out the docs when you run `docker-compose run`, then do the following.

      docker-compose run web python init_psql_db.py

  Remember that the psql container will need to be running when you run this, so
  that the web app can connect to it. (Confirm with `docker-compose ps`).

  ! You can also just use one shell and daemonize the docker-compose services
  by using the `-d` flag and running `docker-compose up -d`.

  If you run it more than once, you'll get an integrity violation error -- you should
  be familiar with what that is from your db classes. Don't worry everything is fine
  keep going.

  Look at flask route `/db_test` in `app.py`. Guess what it's going to by looking
  at the template it loads. Then, visit the route in a web browser.

  :tada:

* mongodb -- again, a flask helper. The heroku addon that we'll use [sets a config
  var named `MONGODB_URI`][1], so let's read from that in our app.py file
  (`os.getenv` and `docker-compose.yml`...)

  ~And oops the [mongodb docker image docs](https://hub.docker.com/_/mongo) say that
  we need to set some env vars for the root username and password. So let's add
  those to our `.env` file, and reference them in the `mongo` section of our
  docker-compose.yml, and also use them to build the web's `MONGODB_URI` env var.~

  [1]: https://devcenter.heroku.com/articles/mongolab#getting-your-connection-uri

  Just kidding, setting the above env vars makes auth a headache, and it's
  unnecessary for a dev environment anyway, so actually don't put any env vars
  for mongo in `.env` and modify the mongo url in `docker-compose.yml` to not
  use those either. Restart your docker containers to load the new config for mongo
  (`docker-compose down`, `docker-compose up`).

  We'll use the mongodb to store files. This is done under the mongo hood using
  GridFS, which library comes along with a `pymongo` python package install. We
  instantiate a `fs` var in `app.py` which we will use to access the mongo filestore.

  I wrote a simple get-only html route and template called `/files` and `files.html`
  respectively which let us (1) view a list of uploaded files, and (2) upload
  new files. Check out the javascript on `files.html`. Before I wrote this lab,
  I had never used javascript to work with files, but look at what I made. I
  just copied a bunch of pieces from the MDN page on working with files and
  it's mine now. I made two other routes -- `/uploads/<path:filename>` and
  `/files.json`, plus a convenience function `_get_files()`, in `app.py`. The
  `/uploads` route uses a function called `save_file` that the Flask-PyMongo library
  attaches to the flask-aware `mongo` object to, well, save a file. `/files.json` returns a,
  well, json representation of our list of files. We'll do something with that later
  with javascript to dynamically render our `files.html` page. The `/files` route
  returns HTML which is mostly just a few buttons and lists and le' javascript.


## Datastores on heroku

Dynos on heroku use ephemeral filesystems. Grok this. In order to be able to scale
dynos, such as what our `web` docker container will be, heroku will spin up multiple
copies of the web dyno and round-robin assign them to handle traffic. Each will be
a clone container, with its own filesystem, which filesystem will be independent of
the filesystems of containers of which it is a clone. Grok that. Heroku container
filesystems are not guaranteed to stick around. They are _ephemeral_. **ephemeral**.
Like the money in your bank account. _**ephemeral**_. So where can you store data,
if not on a container filesystem? If you were managing your own virtual machine,
or cloud instance, you could spin up a database running on the same machine and
call out to it from your web app. That's kind of what we're doing with our
docker-compose containers -- spinning up a web dyno and two database containers
that are all on the same machine, technically, but actually which run separately
and are networked together via docker. On heroku, you do not spin up datastore
services. Instead, you attach heroku _addons_ to your dynos to act as datastores.
Elsewhere, heroku or whomever is running a bank of postresql servers, or mongodb
servers, and they give you a slice of their services which can be used by your dyno.
Typically, when you attach a heroku datastore addon, a config var (env var) is set
on your dyno which is a connection string to the given heroku datastore addon.
The documentation for the addon will say which config vars get set where. This is
really nice, because if you use an env var by the same name in your code to tell
your apps how to talk to the datastores, then your app will be able to access
the _heroku specific_ datastore url when the app is running on heroku. That's why
we did what we did above, with the `os.getenv` business matching `DATABASE_URL`
and `MONGO_URI` or whatever it was and no I will not scroll up to check.

So all ("all") we have to do now is deploy our web docker container to the heroku
container repository, create a new heroku app, attach a psql and a mongodb addon
to that app, and then deploy our web container as a web dyno to that app. In some
order related to the above.

It should work seamlessly, but it probably won't.

## Actually doing the heroku deploy

Heroku won't use our docker-compose.yml, so let's make sure we can run our
docker image without docker-compose. Run `docker images` to see the list of
images that docker-compose has created and registered with the docker service.
Deploy the one that ends with `_web` -- it will be named based on the folder
name of your project. Try running it. For example, with my folder structure,
I run:

    docker run security-analytics-docker_web

Oops, gunicron not found. Oops, I never added it to the requirements.txt file,
because it wasn't needed when running `docker-compose`. I check/google `pip
gunicorn` to get the latest gunicorn version, and I add it to the requirements.txt
file.

    gunicorn~=20.0.4

I run `docker-compose build web` again. Docker recognizes that the line in the Dockerfile that copies `requirements.txt` needs to be run anew, so it does so, and runs every line after that instruction as well. It doesn't recreate the earlier layers of the container.

Then I run the `docker run` command above again. Oops, some obscure error that I have seen before which I know means that gunicorn > v20 is python3 only, but
apparently we're running python 2. I look to the top of the Dockerfile to see
what base image we're building from -- it's `ubuntu:16`-ish. I google and confirm
my inkling that the `python` command on this distro is python2. I know that
ubuntu _can_ run python3, but meh, everything was working before on python2
before gunicorn, so I just step back down to gunicorn v19 which I know from past
experience still supports python2. I update my requirements.txt:

    gunicorn~=19.10.0

`docker-compose build web` and then `docker run` again. Oops, now it complains
that `'' is not a valid port number.`. With my lightning-fast
discernment I remember that the dockerfile is trying to run a `gunicorn`
command which uses a `$PORT` env var. We haven't set it. It needs to be
passed to the docker container using the `-e` flag on the run command. So,
using arbitrary port 5000,

    docker run -e PORT=5000 security-analytics-docker_web

It now complains about the `MONGO_URI` not being set. I'm okay with that,
because I know that heroku will set that when I add the mongo addon.

So I follow the [stupid-easy instructions on heroku docs][1] for deploying a docker
container as a dyno. I run `heroku open` and see that the app has crashed. I
run `heroku logs` and see that it's still missing the mongo_uri. I now
add the postgresql and mongodb addons for heroku. I google them both, scroll
towards the bottom of the addons' landing pages, and find these commands:

    heroku addons:create (the one for psql)
    heroku addons:create (the one for mongodb)

Heroku automatically redeploys my app. I refresh the url, and boom it's working.
I then open the `lab-6.md` file in atom and update the instructions for how to
do what I just did. I now stop writing the lab.

[1]: https://devcenter.heroku.com/articles/container-registry-and-runtime

## Getting `heroku` commands inside the GCP container.

TODO: explain this better

You could use something like this https://hub.docker.com/r/wingrunr21/alpine-heroku-cli/,
creating an alias for `heroku` command pointing to the full `docker run` command
for the container, as above for `docker-compose`.

## What do I do with my code, I want to shut down the GCP instance

Commit your code to a git / github repo, silly! Do the `git init` thing from your
GCP shell, create a new github repo to push to, add your remote, and fire the
laser!

Then, you can destroy your GCP instance. later, if you want to work on it again,
you can spin up a new instance and do `git pull`, :tada:


## Using a more lightweight Docker basefile, based on Alpine instead of Ubuntu.

I saw references to an `alpine` base docker image which is much more lightweight
than the full `ubuntu16` image -- less running processes, much smaller, etc. I
google for one that provides python, which leads me to the [dockerhub page for all
official docker python images](https://hub.docker.com/_/python). I poke around
the readme some, and see in the examples, that it uses alpine as its base image.
(I could confirm by viewing these images' own Dockerfiles, but I'll just trust.)
See, docker images have deep dependencies. Most all start with a `FROM <something>`
statement at the top, upon which they build. We'll do the same. I edit my Dockerfile
top line to be `FROM python:2`, and I build and run again. It still works, but
now I'm trendy, so I keep it. Python3 is trendier, but meh for now.

## Running arbitrary commands from within the heroku environment

Remember above when we ran a python script which initialized our database?
We'll need to do that again, but pointing at the database that heroku will be using.
One way that we can do that is by launching a one-off dyno on heroku, which will
have our files in it, and running the initialization script from within that
one-off dyno. Another way that we could do it is by fetching our heroku DATABASE_URL
by running `heroku config` from a heroku cli, and then manually setting the `DATABASE_URL`
in our environment where our docker container could use it.
Let's do the first one, though.

    heroku run python init_psql_db.py

Then, visit your heroku app url and confirm that the `/db_test` lists your dummy
users. :tada:

# Deliverables

I don't know, just do the above. You'll need to piece together all of the lab...
pieces... soon in order to do the security project, but we're getting there.
Docker is enough for one lab.
