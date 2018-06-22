
![VeniceGeo](https://avatars2.githubusercontent.com/u/15457149?s=200&v=4) # Pelias in Docker

This repository contains a framework for downloading/preparing and building the the [Pelias Geocoder](https://github.com/pelias/pelias) using Docker and [Docker Compose](https://github.com/docker/compose#docker-compose).

## Projects

Example projects are included in the [projects](https://github.com/pelias/docker/tree/master/projects) directory.

We recommend you start with the `portland-metro` example as a first-time user; once you have successfully completed a build you can use this as a base to create your own projects.

## Not suitable for large geographies

We do not recommend running large extracts (anything larger than a US State) inside Docker, the scripts are **not suitable** for full planet builds. If you require global coverage, please see our [install documentation](https://pelias.io/install.html) or consider using the [geocode.earth](https://geocode.earth/) services hosted by members of our core team.

## Prerequisites

You will need to have `docker` and `docker-compose` installed before continuing. If you are not using the latest version, please mention that in any bugs reports.

If you are running OSX, you should also install `brew install coreutils` and max-out your Docker limits in `Docker > Preferences > Advanced`.

Scripts can easily download tens of GB of geographic data, so ensure you have enough free disk space!

## Installing the Pelias command

If you haven't done so already, you will need to ensure the `pelias` command is available on your path.

You can find the `pelias` file in the root of this repository.

Advanced users may have a preferance how this is done on their system, but a basic example would be to do something like:

```bash
git clone https://github.com/pelias/docker.git ~/pelias
ln -s ~/pelias/pelias /usr/local/bin/pelias
```

Once the command is correctly installed you should be able to run the following command to confim the pelias command is available on your path:

```bash
which pelias
```

### Resolving PATH issues

If you are having trouble getting this to work then quickly check that the target of the symlink is listed on your $PATH:

```bash
tr ':' '\n' <<< "$PATH"
```

If you used the `ln -s` command above then the directory `/usr/local/bin` should be listed in the output.

If the symlink target path is *not* listed, then you will either need to add its location to your $PATH or create a new symlink which points to a location which is already on your $PATH.

## Configure Environment

The `pelias` command looks for an `.env` file in your **current working directory**, this file contains information specific to your local environment.

If this is your first time, you should change directories to an example project before continuing:

```bash
cd projects/portland-metro
```

Ensure that your current working directory contains the files: `.env`, `docker-compose.yml` and `pelias.json` before continuing.

### Variable: DATA_DIR

The only mandatory variable in `.env` is `DATA_DIR`.

This path reflects the directory Pelias will use to store downloaded data and use to build it's other microservices.

You **must** create a new directory which you will use for this project, for example:

```bash
mkdir /tmp/pelias
```

Then use your text editor to modify the `.env` file to reflect your new path, it should look like this:

```bash
COMPOSE_PROJECT_NAME=pelias
DATA_DIR=/tmp/pelias
```

You can then list the environment variables to ensure they have been correctly set:

```bash
pelias system env
```

### Variables: COMPOSE_*

The compose variables are optional and are documented here: https://docs.docker.com/compose/env-file/

Note: changing the `COMPOSE_PROJECT_NAME` variable is not advisable unless you know what you are doing. If you are migrating from the deprecated `pelias/dockerfiles` repository then you can set `COMPOSE_PROJECT_NAME=dockerfiles` to enable backwards compatibility with containers created using that repository.

## CLI commands

```bash
$ pelias 

Usage: pelias [command] [action] [options]

  compose   pull                     update all docker images
  compose   logs                     display container logs
  compose   ps                       list containers
  compose   top                      display the running processes of a container
  compose   exec                     execute an arbitrary docker-compose command
  compose   run                      execute a docker-compose run command
  compose   up                       start one or more docker-compose service(s)
  compose   kill                     kill one or more docker-compose service(s)
  compose   down                     stop all docker-compose service(s)
  download  wof                      (re)download whosonfirst data
  download  gndb                     (re)download gndb data
  download  oa                       (re)download openaddresses data
  download  osm                      (re)download openstreetmap data
  download  tiger                    (re)download TIGER data
  download  transit                  (re)download transit data
  download  all                      (re)download all data
  elastic   drop                     delete elasticsearch index & all data
  elastic   create                   create elasticsearch index with pelias mapping
  elastic   start                    start elasticsearch server
  elastic   stop                     stop elasticsearch server
  elastic   status                   HTTP status code of the elasticsearch service
  elastic   wait                     wait for elasticsearch to start up
  import    wof                      (re)import whosonfirst data
  import    gndb                     (re)import gndb data
  import    oa                       (re)import openaddresses data
  import    osm                      (re)import openstreetmap data
  import    polylines                (re)import polylines data
  import    transit                  (re)import transit data
  import    all                      (re)import all data
  prepare   polylines                export road network from openstreetmap into polylines format
  prepare   interpolation            build interpolation sqlite databases
  prepare   placeholder              build placeholder sqlite databases
  prepare   all                      build all services which have a prepare step
  system    check                    ensure the system is correctly configured
  system    env                      display environment variables
  system    update                   update the pelias command by pulling the latest version
  ```

### Compose commands

The compose commands are available as a shortcut to running `docker-compose` directly, they will also ensure that your environment is correctly configured.

See the docker-compose documentation for more info: https://docs.docker.com/compose/overview/

```bash
pelias compose pull                     update all docker images
pelias compose logs                     display container logs
pelias compose ps                       list containers
pelias compose top                      display the running processes of a container
pelias compose exec                     execute an arbitrary docker-compose command
pelias compose run                      execute a docker-compose run command
pelias compose up                       start one or more docker-compose service(s)
pelias compose kill                     kill one or more docker-compose service(s)
pelias compose down                     stop all docker-compose service(s)
```

### Download commands

The download commands will fetch and update geographic data from source.

For example: `pelias download tiger` will fetch street data from the US Census Bureau and store it in the directory referenced by the `DATA_DIR` environment variable.

```bash
pelias download wof                      (re)download whosonfirst data
pelias download oa                       (re)download openaddresses data
pelias download osm                      (re)download openstreetmap data
pelias download tiger                    (re)download TIGER data
pelias download transit                  (re)download transit data
pelias download gndb                     (re)download gndb data
pelias download all                      (re)download all data
```

### Prepare commands

The prepare commands are used to run any commands which are required to setup/configure or build microservices.

For example: `pelias prepare interpolation` will build a street address interpolation index.

Note: the order of execution is important, the prepare commands require data, so they must be run *after* the download commands have fetched the data.

```bash
pelias prepare polylines                export road network from openstreetmap into polylines format
pelias prepare interpolation            build interpolation sqlite databases
pelias prepare placeholder              build placeholder sqlite databases
pelias prepare all                      build all services which have a prepare step
```

### Elastic commands

The elastic commands control starting/stopping/configuring elasticsearch.

The special `pelias elastic wait` command can be used in scripts to block the script execution until elasticsearch is ready to accept connections.

```bash
pelias elastic drop                     delete elasticsearch index & all data
pelias elastic create                   create elasticsearch index with pelias mapping
pelias elastic start                    start elasticsearch server
pelias elastic stop                     stop elasticsearch server
pelias elastic status                   HTTP status code of the elasticsearch service
pelias elastic wait                     wait for elasticsearch to start up
```

### Import commands

The import commands import source data in to elasticsearch.

```bash
pelias import wof                      (re)import whosonfirst data
pelias import oa                       (re)import openaddresses data
pelias import osm                      (re)import openstreetmap data
pelias import polylines                (re)import polylines data
pelias import transit                  (re)import transit data
pelias import gndb                     (re)import gndb data
pelias import all                      (re)import all data
```

### System commands

The system commands help debug issues with incorrectly set environment variables.

The `pelias system update` command can be used to ensure that the `pelias` command itself is up-to-date by pulling the latest source code from Github.

```bash
pelias system check                    ensure the system is correctly configured
pelias system env                      display environment variables
pelias system update                   update the pelias command by pulling the latest version
```

## Generic build workflow

The following shell script can be used to automate a build:

```bash
#!/bin/bash
set -x

# create directories
mkdir /code /data

# clone repo
cd /code
git clone https://github.com/pelias/docker.git
cd docker

# install pelias script
ln -s "$(pwd)/pelias" /usr/local/bin/pelias

# cwd
cd projects/portland-metro

# configure environment
sed -i '/DATA_DIR/d' .env
echo 'DATA_DIR=/data' >> .env

# run build
pelias compose pull
pelias elastic start
pelias elastic wait
pelias elastic create
pelias download all
pelias prepare all
pelias import all
pelias compose up
```

## View status of running containers

Once the build is complete, you can view the current status and port mappings of the Pelias docker containers:

```bash
pelias compose ps
```

## View logs and debug errors

You can inspect the container logs for errors by running:

```bash
pelias compose logs
```

## Make an example query

You can now make queries against your new Pelias build:

### API

- http://localhost:4000/v1/search?text=portland
- [http://localhost:4000/v1/search?text=1901 Main St](http://localhost:4000/v1/search?text=1901%20Main%20St)
- http://localhost:4000/v1/reverse?point.lon=-122.650095&point.lat=45.533467
- [http://localhost:4000/v1/convert?from=mgrs&to=decdeg&q=38S MB 43166 86546](http://localhost:4000/v1/convert?from=mgrs&to=decdeg&q=38S%20MB%2043166%2086546)

### Placeholder

- http://localhost:4100/demo/#eng

### PIP (point in polygon)

- http://localhost:4200/-122.650095/45.533467

### Interpolation

- http://localhost:4300/demo/#13/45.5465/-122.6351
