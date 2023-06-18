# An example Rails + Docker app


You could use this example app as a base for your new project or as a guide to
Dockerize your existing Rails app.

The example app is minimal but it wires up a number of things you might use in
a real world Rails app, but at the same time it's not loaded up with a million
personal opinions.

For the Docker bits, everything included is an accumulation of [Docker best
practices](https://nickjanetakis.com/blog/best-practices-around-production-ready-web-apps-with-docker-compose)
based on building and deploying dozens of assorted Dockerized web apps since
late 2014.

**This app is using Rails 7.0.5 and Ruby 3.2.2**. 


## Table of contents

- [Tech stack](#tech-stack)
- [Main changes vs a newly generated Rails app](#main-changes-vs-a-newly-generated-rails-app)
- [Running this app](#running-this-app)
- [Files of interest](#files-of-interest)
  - [`.env`](#env)
  - [`run`](#run)
- [Running a script to automate renaming the project](#running-a-script-to-automate-renaming-the-project)
- [Updating dependencies](#updating-dependencies)
- [See a way to improve something?](#see-a-way-to-improve-something)
- [Additional resources](#additional-resources)
  - [Learn more about Docker and Ruby on Rails](#learn-more-about-docker-and-ruby-on-rails)
  - [Deploy to production](#deploy-to-production)

## Tech stack

If you don't like some of these choices that's no problem, you can swap them
out for something else on your own.

### Back-end

- [PostgreSQL](https://www.postgresql.org/)
- [Redis](https://redis.io/)
- [Sidekiq](https://github.com/mperham/sidekiq)
- [Action Cable](https://guides.rubyonrails.org/action_cable_overview.html)
- [ERB](https://guides.rubyonrails.org/layouts_and_rendering.html)

### Front-end

- [esbuild](https://esbuild.github.io/)
- [Hotwire Turbo](https://hotwired.dev/)
- [StimulusJS](https://stimulus.hotwired.dev/)
- [TailwindCSS](https://tailwindcss.com/)
- [Heroicons](https://heroicons.com/)

## Main changes vs a newly generated Rails app

Here's a run down on what's different. You can also use this as a guide to
Dockerize an existing Rails app.

- **Core**:
    - Use PostgreSQL (`-d postgresql)` as the primary SQL database
    - Use Redis as the cache back-end
    - Use Sidekiq as a background worker through Active Job
    - Use a standalone Action Cable process
- **App Features**:
    - Add `pages` controller with a home page
    - Add `up` controller with 2 health check related actions
- **Config**:
    - Log to STDOUT so that Docker can consume and deal with log output 
    - Credentials are removed (secrets are loaded in with an `.env` file)
    - Extract a bunch of configuration settings into environment variables
    - Rewrite `config/database.yml` to use environment variables
    - `.yarnc` sets a custom `node_modules/` directory
    - `config/initializers/rack_mini_profiler.rb` to enable profiling Hotwire Turbo Drive
    - `config/initializers/assets.rb` references a custom `node_modules/` directory
    - `config/routes.rb` has Sidekiq's dashboard ready to be used but commented out for safety
    - `Procfile.dev` has been removed since Docker Compose handles this for us
- **Assets**:
    - Use esbuild (`-j esbuild`) and TailwindCSS (`-c tailwind`)
    - Add `postcss-import` support for `tailwindcss` by using the `--postcss` flag
    - Add ActiveStorage JavaScript package
- **Public:**
    - Custom `502.html` and `maintenance.html` pages
    - Generate favicons using modern best practices

Besides the Rails app itself, a number of new Docker related files were added
to the project which would be any file having `*docker*` in its name. Also
GitHub Actions have been set up.

## Running this app

You'll need to have [Docker installed](https://docs.docker.com/get-docker/).
It's available on Windows, macOS and most distros of Linux. If you're new to
Docker and want to learn it in detail check out the [additional resources
links](#learn-more-about-docker-and-ruby-on-rails) near the bottom of this
README.

You'll also need to enable Docker Compose v2 support if you're using Docker
Desktop. On native Linux without Docker Desktop you can [install it as a plugin
to Docker](https://docs.docker.com/compose/install/linux/). It's been generally
available for a while now and very stable. This project uses a specific Docker
Compose profiles feature that only works with Docker Compose v2.

If you're using Windows, it will be expected that you're following along inside
of [WSL or WSL
2](https://nickjanetakis.com/blog/a-linux-dev-environment-on-windows-with-wsl-2-docker-desktop-and-more).
That's because we're going to be running shell commands. You can always modify
these commands for PowerShell if you want.

#### Clone this repo anywhere you want and move into the directory:

```sh
git clone 
cd rails_app

# Optionally checkout a specific tag, such as: git checkout 0.8.0
```

#### Copy an example .env file because the real one is git ignored:

```sh
cp .env.example .env
```

#### Build everything:

*The first time you run this it's going to take 5-10 minutes depending on your
internet connection speed and computer's hardware specs. That's because it's
going to download a few Docker images and build the Ruby + Yarn dependencies.*

```sh
docker compose up --build
```

Now that everything is built and running we can treat it like any other Rails
app.

#### Setup the initial database:

```sh
# You can run this from a 2nd terminal.
./run rails db:setup
```


#### Check it out in a browser:

Visit <http://localhost:8000> in your favorite browser.

#### Running the test suite:

```sh
# You can run this from the same terminal as before.
./run test
```

You can also run `./run test -b` with does the same thing but builds your JS
and CSS bundles. This could come in handy in fresh environments such as CI
where your assets haven't changed and you haven't visited the page in a
browser.

#### Stopping everything:

```sh
# Stop the containers and remove a few Docker related resources associated to this project.
docker compose down
```

You can start things up again with `docker compose up` and unlike the first
time it should only take seconds.

## Files of interest

I recommend checking out most files and searching the code base for `TODO:`,
but please review the `.env` and `run` files before diving into the rest of the
code and customizing it. Also, you should hold off on changing anything until
we cover how to customize this example app's name with an automated script
(coming up next in the docs).

### `.env`

This file is ignored from version control so it will never be commit. There's a
number of environment variables defined here that control certain options and
behavior of the application. Everything is documented there.

Feel free to add new variables as needed. This is where you should put all of
your secrets as well as configuration that might change depending on your
environment (specific dev boxes, CI, production, etc.).

### `run`

You can run `./run` to get a list of commands and each command has
documentation in the `run` file itself.

It's a shell script that has a number of functions defined to help you interact
with this project. It's basically a `Makefile`.

This comes in handy to run various Docker commands because sometimes these
commands can be a bit long to type. 

*If you get tired of typing `./run` you can always create a shell alias with
`alias run=./run` in your `~/.bash_aliases` or equivalent file. Then you'll be
able to run `run` instead of `./run`.*

#### Sanity check to make sure the tests still pass:

It's always a good idea to make sure things are in a working state before
adding custom changes.

```sh
# You can run this from the same terminal as before.
./run test
```
## Updating dependencies

Let's say you've customized your app and it's time to make a change to your
`Gemfile` or `package.json` file.

Without Docker you'd normally run `bundle install` or `yarn install`. With
Docker it's basically the same thing and since these commands are in our
`Dockerfile` we can get away with doing a `docker compose build` but don't run
that just yet.

#### In development:

You can run `./run bundle:outdated` or `./run yarn:outdated` to get a list of
outdated dependencies based on what you currently have installed. Once you've
figured out what you want to update, go make those updates in your `Gemfile`
and / or `package.json` file.

Then to update your dependencies you can run `./run bundle:install` or `./run
yarn:install`. That'll make sure any lock files get copied from Docker's image
(thanks to volumes) into your code repo and now you can commit those files to
version control like usual.

Alternatively for updating your gems based on specific version ranges defined
in your `Gemfile` you can run `./run bundle:update` which will install the
latest versions of your gems and then write out a new lock file.

You can check out the `run` file to see what these commands do in more detail.

#### In CI:

You'll want to run `docker compose build` since it will use any existing lock
files if they exist. You can also check out the complete CI test pipeline in
the `run` file under the `ci:test` function.

#### In production:

This is usually a non-issue since you'll be pulling down pre-built images from
a Docker registry but if you decide to build your Docker images directly on
your server you could run `docker compose build` as part of your deploy
pipeline.

## See a way to improve something?

If you see anything that could be improved please open an issue or start a PR.
Any help is much appreciated!

## Additional resources

Now that you have your app ready to go, it's time to build something cool! If
you want to learn more about Docker, Rails and deploying a Rails app here's a
couple of free and paid resources. There's Google too!

### Learn more about Docker and Ruby on Rails

#### Official documentation 

- <https://docs.docker.com/>
- <https://guides.rubyonrails.org/>


