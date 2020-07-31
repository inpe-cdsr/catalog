# Quick Start

This is a quick start file related to CDSR project. This file explains how to install the project and do the basic settings.


## Requirements

Install the following requirements before continuing this documentation.

- [Docker](https://docs.docker.com/engine/install/ubuntu)
- [Manage Docker as a non-root user](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user)
- [Docker Compose](https://docs.docker.com/compose/install)


## Install

Create a new folder called `inpe-cdsr`, where you are going to put [catalog](https://github.com/inpe-cdsr/catalog) repository:

```
$ mkdir inpe-cdsr
```

Clone the [catalog](https://github.com/inpe-cdsr/catalog) repository inside the previous folder and get in it:

```
$ cd inpe-cdsr/ && \
  git clone https://github.com/inpe-cdsr/catalog.git && \
  cd catalog/
```


### Settings

Get in [env_files](./env_files) folder and create the environment files related to each application based on the example ones inside it:

```
$ cd env_files/ && \
  for file in *.env.EXAMPLE; do cp "$file" "${file/.EXAMPLE}"; done
```

Update the files above with proper settings. The files are environment variables files that contain settings related to the services inside `docker-compose.[dev|prod].yml` files.


#### Environment variables basic settings

Open `nginx.env` file and set a host name that Nginx will use to serve all the applications publicly. Port you should leave the default value, but if you would like to change it, you must change the binding port on `inpe_cdsr_nginx` service inside `docker-compose.[dev|prod].yml` files.

The settings you chose to `nginx.env` file, you must add them to `frontend.env` and `inpe_stac.env` files as well, by updating the URLs according to domain and port you chose previously, in other words, you must replace `localhost:8089` by the settings inside `nginx.env` file. You can leave the default values to the `nginx.env`, `frontend.env` and `inpe_stac.env` files if you are running locally.

If you are inside [env_files](./env_files) folder, then go to the root folder and create the `.env` file:

```
$ cd .. && \
  cp .env.EXAMPLE .env
```

`docker-compose.[dev|prod].yml` files expect that your TIFF files are mounted in a folder that is defined by `DATA_TIFF` environment variable. This environment variable is inside the `.env` file. Change the `.env` file with the path to your TIFF files. For example: you have TIFF files related to CBERS-4 and Landsat 5 satellites, then their folders must be mounted in `/data/TIFF/CBERS4` and `/data/TIFF/LANDSAT5` folders respectively. All subfolders inside `/data/TIFF` folder are considered satellites folders.

Environment variables advanced settings can be found in [this section](./README.md#docker-compose-services).


## Run the docker compose files

This section describes how to run the `docker-compose.[dev|prod].yml` files, in development and production mode, and show the available endpoints.


### Development mode

`docker-compose.dev.yml` file uses volumes that point to the source code of some services, hence you need to download the other repositories first, in order to execute this file.

Assuming that you are inside `catalog` folder, then go back one folder and clone the other repositories:

```
$ cd .. && \
  git clone https://github.com/inpe-cdsr/catalog-frontend.git && \
  git clone https://github.com/inpe-cdsr/catalog-backend.git && \
  git clone https://github.com/inpe-cdsr/inpe-stac.git && \
  git clone https://github.com/inpe-cdsr/stac-compose.git
```

For each repository you cloned before, you need to build its development Docker image by following the instructions inside each repository: [catalog-frontend](https://github.com/inpe-cdsr/catalog-frontend), [catalog-backend](https://github.com/inpe-cdsr/catalog-backend), [inpe-stac](https://github.com/inpe-cdsr/inpe-stac) and [stac-compose](https://github.com/inpe-cdsr/stac-compose).

Angular does not read environment variables because it is executed in the browser, then you need to create a JavaScript file with the necessary variables in development mode. In order to do that, get in the `catalog-frontend` volume folder, copy the environment file and edit it if necessary:

```
$ cd catalog/volumes/catalog-frontend/ && \
  cp env.js.EXAMPLE env.js
```

Use the following command to run the docker compose file in development mode:

```
$ docker-compose -f docker-compose.dev.yml up
```


### Production mode

Before running in production mode, you must log in the following Docker registry in order to download all necessary Docker images:

```
$ docker login registry.dpi.inpe.br
```

If you get any error related to invalid certificate, you must insert `registry.dpi.inpe.br` registry in the list of insecure registries. In order to do that, create or edit the `/etc/docker/daemon.json` file and add te following line:

```
{
  "insecure-registries" : ["registry.dpi.inpe.br"],
  [...]
}
```

Restart Docker with the following command and try to log in the above registry again.

```
$ sudo service docker restart
```

Use the following commands to run in production mode:

```
$ docker-compose -f docker-compose.prod.yml up -d
```

When you run in production mode for the first time, all Docker images will be downloaded **automatically**. If they are not downloaded, for some reason, you can pull all Docker images using the following command:

```
$ docker pull <Docker image>
```

Where `<Docker image>` is the image inside each service in the [docker-compose.prod.yml](./docker-compose.prod.yml) file.


### Endpoints

After running the docker compose file, Nginx will serve all applications with the host and port you defined inside [env_files/nginx.env](./env_files/nginx.env) file (e.g. `http://localhost:8089`).

The following endpoints are now available:

- [/catalogo](http://localhost:8089/catalogo): [catalog-frontend](https://github.com/inpe-cdsr/catalog-frontend) application;

- [/api](http://localhost:8089/api): [catalog-backend](https://github.com/inpe-cdsr/catalog-backend) application;

- [/inpe-stac](http://localhost:8089/inpe-stac): [inpe-stac](https://github.com/inpe-cdsr/inpe-stac) application;

- [/stac-compose](http://localhost:8089/stac-compose): [stac-compose](https://github.com/inpe-cdsr/stac-compose) application;

- [/geoserver](http://localhost:8089/geoserver): [GeoServer](https://hub.docker.com/r/kartoza/geoserver/) application;

- [/portainer](http://localhost:8089/portainer): [portainer](https://hub.docker.com/r/portainer/portainer/) application;


## Database settings

This section assumes you have run a `docker-compose.[dev|prod].yml` file before continuing.

On another console, clone the [database](https://github.com/inpe-cdsr/database) repository inside `inpe-cdsr` folder and get in it:

```
$ cd inpe-cdsr/ && \
  git clone https://github.com/inpe-cdsr/database.git && \
  cd database/
```

Create the database schemas based on the scripts inside `db_structure` with the following commands:

```
$ docker exec -i inpe_cdsr_db sh -c 'exec mysql -uroot -p"$MYSQL_ROOT_PASSWORD"' < db_structure/catalogo.sql

$ docker exec -i inpe_cdsr_db sh -c 'exec mysql -uroot -p"$MYSQL_ROOT_PASSWORD"' < db_structure/cadastro.sql
```

In order to open [MariaDB](https://mariadb.com/), access [phpMyAdmin](https://www.phpmyadmin.net/) on `http://<your host>:8099` (e.g. [http://localhost:8099](http://localhost:8099)).
