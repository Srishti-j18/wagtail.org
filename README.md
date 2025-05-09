# Wagtail.org

This is the source code to [Wagtail's website](https://wagtail.org)

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/wagtail/wagtail.org)

## Requirements

-   Docker or Vagrant (see below)
-   [Fabric](https://www.fabfile.org/), [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) and [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli) (only for downloading production / staging data)

## Installation (Docker Compose)

You first need to install [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/install/). Once they are installed, run the following commands to get up and running:

```
git clone https://github.com/wagtail/wagtail.org.git
cd wagtail.org
make setup
```

This will create a set of Docker containers on your local machine and also create a blank database.

### Starting the development environment

Run the following command to start the Docker containers:

```
make start
```

Then, to start the development server, open a new terminal window and run:

```
make runserver
```

This will launch `django-admin runserver` in the web container, which serves the site on http://localhost:8000/, with the admin at http://localhost:8000/admin/.

To run migrations within Docker you'll need to run:

```
make migrate
```

### Creating a superuser

To create a new superuser locally, run:

```
make superuser
```

### Setting up the cache table

This should only need to be done the first time you set up a new database:

```
make sh
python manage.py createcachetable
```

### Pulling production data / media

If you'd like to work with production data and have access, run the following commands:

```
fab pull-production-data
fab pull-production-media
```

### Pulling staging data

If you'd like to work with staging data and have been granted access, run the following commands if you're on Mac/Linux (POSIX):

```
fab pull-staging-data
```

Access will only be given when absolutely necessary.

If you're on Windows, we recommend you follow the [specific steps for Windows.](#pulling-staging-data-on-windows)

## Installation (Vagrant)

You first need to install [Vagrant](https://www.vagrantup.com/) and [Virtualbox](https://www.virtualbox.org/). Once they are installed, run the following commands to get up and running:

```
git clone https://github.com/wagtail/wagtail.org.git
cd wagtail.org
vagrant up
```

This will download the base image and provision a local VM that will run the site locally.

You will need to apply migrations, create a super user, and create a cache table once the vagrant environment is setup.

```
vagrant ssh
./manage.py migrate
./manage.py createsuperuser
./manage.py createcachetable
```

## Usage (Vagrant)

Common Vagrant commands:

-   `vagrant up` starts the VM
-   `vagrant halt` stops the VM
-   `vagrant ssh` opens a shell in the VM
-   `vagrant destroy` deletes the VM

Shortcut commands:

Within the VM shell, you can run `./manage.py` to run any Django management command. But we have added a couple of shortcuts to save on typing:

-   `dj <command> [args]` - Runs a management command (eg, `dj shell`)
-   `djrun` - Starts the webserver on port 8000

## Frontend tooling (Docker and Vagrant)

During the initial setup, you will need to build the frontend assets:

-   `nvm use` to use the suggested node version (requires [nvm](https://github.com/nvm-sh/nvm), [fnm](https://github.com/Schniz/fnm) or similar. You'll also need to run `nvm install` to install and activate the version of node required for the project)
-   `npm i` to install dependencies
-   `npm run build` to compile CSS & JS

If you are editing frontend assets, you will also need to rebuild assets while
you edit:

-   `npm run start` start the Webpack build in watch mode, without live-reload
-   `npm run start:reload` start the Webpack server build on port 3000 with live-reload

Other common commands:

-   `npm run lint` lint JS & CSS files
-   `npm run format` format files

For more info see [Frontend general info](docs/frontend/general-info.md)

## Pulling staging data on Windows

The `fab pull-staging-data` command does not currently work on Windows due to the way quotes are handled.

The recommended workaround is to run the necessary commands manually. It's important to note that there are 3 categories of commands to run:

1. The ones to be run on the `web` container.
2. The ones to be run on the `db` container.
3. The ones to be run directly on a local terminal.

You will need two separate terminal instances for the `web` and local. We suggest using the VS code editor, and the terminal in it. The `db` container instructions should be done from within the Docker Desktop terminal because that is one of the easiest ways to [activate a shell within Docker](https://docs.docker.com/desktop/use-desktop/container/#open-the-integrated-terminal).

The commands, the location, and the order in which to run them are as follows:

1. Run the following on your local terminal instance: `heroku pg:backups:download --output=here.dump --app wagtail-org-staging`.
2. In the `db` container terminal, run: `dropdb --if-exists --host db --username=wagtailorg wagtailorg`.
3. Afterwards, still in the `db` container, run `createdb --host db --username=wagtailorg wagtailorg`.
4. To connect the local postgres DB, run the following in the `db` container: `psql -d wagtailorg -U wagtailorg -c "CREATE SCHEMA heroku_ext;"`
5. Still in the `db` container, run: `pg_restore --clean --no-acl --if-exists --no-owner --host db --username=wagtailorg -d wagtailorg /app/here.dump `.
6. In your `web` container, run: `psql -c "UPDATE wagtailcore_site SET hostname = 'localhost', port = 8000 WHERE is_default_site IS TRUE;"`.
7. Finally, in the local terminal, run: `rm here.dump`.

With `npm` installed, you can run `npm install` directly, since nvm isn't supported for Windows (non-POSIX). Optionally, `fnm` can be used to first switch to the appropriate version of node.

Afterwards, the recommended next step would be setting up your [frontend tooling.](#frontend-tooling-docker-and-vagrant)

## Deployment

The site is hosted on Heroku, and is deployed automatically. `main` deploys to production, and `staging` deploys to staging.

## docs.wagtail.org

Wagtail documentation is hosted at [readthedocs](https://readthedocs.org/). A Cloudflare worker is in place to rewrite canonical URLs on old versions of the documentation - see `conf/wagtaildocs-cloudflare-worker.js`.
