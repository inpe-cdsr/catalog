# Quick Start

This is a quick start related to CDSR Catalog project.

The repository contains two docker composes to run the project applications:

- `docker-compose.dev.yml` file contains development services using Docker images with volumes. The Docker images do not contain all code inside them, the code is added into the Docker containers through volumes.

- `docker-compose.prod.yml` file contains production services using Docker images without volumes. The Docker images contain all code inside them and they do not use volumes to keep their code.


## Install

Create a new folder where you will let all your repositories called `inpe-cdsr`:

```
$ mkdir inpe-cdsr
```

Clone the [catalog](https://github.com/inpe-cdsr/catalog) repository inside the previous folder and get into it:

```
$ cd inpe-cdsr/ && \
git clone https://github.com/inpe-cdsr/catalog.git && \
cd catalog/
```


### Settings

Create the environment files related to each application based on the example ones inside [env_files](./env_files) folder:

```
$ cd env_files/

$ cp nginx.env.EXAMPLE nginx.env && \
  cp frontend.env.EXAMPLE frontend.env && \
  cp backend.env.EXAMPLE backend.env && \
  cp inpe_stac.env.EXAMPLE inpe_stac.env && \
  cp stac_compose.env.EXAMPLE stac_compose.env && \
  cp geoserver.env.EXAMPLE geoserver.env && \
  cp db.env.EXAMPLE db.env && \
  cp phpmyadmin.env.EXAMPLE phpmyadmin.env
```


#### Environment variables settings

Open `nginx.env` file and set a host name and port that Nginx will use to serve all the applications publicly. These settings need to be edited on another files as well, such as `frontend.env` and `inpe_stac.env`, by updating the URLs according to domain and port you chose, in other words, you must replace `localhost:8089` by your configuration or let these ones if you are running locally.

`docker-compose.*.yml` files expect your TIFF files are mounted in `/data/TIFF` folder, hence you must mount them in this folder. Satellites folders are expected to be inside `/data/TIFF` folder. For example: you have TIFF files related to CBERS-4 and Landsat 5 satellites, then their folders must be mounted in `/data/TIFF/CBERS4` and `/data/TIFF/LANDSAT5` respectively. All subfolders inside `/data/TIFF` folder are considered satellites folders.  


### Run the docker-compose in production mode

Before running in production mode, you must log in the following registry in order to download all necessary Docker images:

```
$ docker login registry.dpi.inpe.br
```

When you run in production mode for the first time, all Docker images will be downloaded. If they are not downloaded, for some reason, you can pull all Docker images using the following command:

```
$ docker pull <Docker image>
```

Where `<Docker image>` is the image used inside each service on the `docker-compose.prod.yml` file.

Use the following commands to run in production mode:

- Run containers in the foreground:

```
$ docker-compose -f docker-compose.prod.yml up
```

- Run containers in the background:

```
$ docker-compose -f docker-compose.prod.yml up -d
```


### Database settings

This section assumes you have run a `docker-compose.*.yml` file before continuing.

On another console, clone the [database](https://github.com/inpe-cdsr/database) repository inside `inpe-cdsr` folder and get into it:

```
$ cd ../ && \
git clone https://github.com/inpe-cdsr/database.git && \
cd database/
```

Create the database schemas with the following commands:

```
$ docker exec -i inpe_cdsr_db sh -c 'exec mysql -uroot -p"$MYSQL_ROOT_PASSWORD"' < db_structure/catalogo.sql

$ docker exec -i inpe_cdsr_db sh -c 'exec mysql -uroot -p"$MYSQL_ROOT_PASSWORD"' < db_structure/cadastro.sql
```


### Endpoints

After running the docker compose, Nginx will serve all applications with the host and port you defined inside [env_files/nginx.env](./env_files/nginx.env) file (e.g. `http://localhost:8089`).

The following endpoints are now available:

- [/catalogo](http://localhost:8089/catalogo): [catalog-frontend](https://github.com/inpe-cdsr/catalog-frontend) application;

- [/api](http://localhost:8089/api): [catalog-backend](https://github.com/inpe-cdsr/catalog-backend) application;

- [/inpe-stac](http://localhost:8089/inpe-stac): [inpe-stac](https://github.com/inpe-cdsr/inpe-stac) application;

- [/stac-compose](http://localhost:8089/stac-compose): [stac-compose](https://github.com/inpe-cdsr/stac-compose) application;

- [/geoserver](http://localhost:8089/geoserver): [GeoServer](https://hub.docker.com/r/kartoza/geoserver/) application;

- [/portainer](http://localhost:8089/portainer): [portainer](https://hub.docker.com/r/portainer/portainer/) application;
