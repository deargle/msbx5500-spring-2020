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

  And oops the [mongodb docker image docs](https://hub.docker.com/_/mongo) say that
  we need to set some env vars for the root username and password. So let's add
  those to our `.env` file, and reference them in the `mongo` section of our
  docker-compose.yml, and also use them to build the web's `MONGODB_URI` env var.

  [1]: https://devcenter.heroku.com/articles/mongolab#getting-your-connection-uri
