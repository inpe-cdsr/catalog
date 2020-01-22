# catalog

This is the main repository related to CDSR Catalog project.

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

Update the files above with proper settings. The files are environment variables files that contain settings related to the services inside `docker-compose.*.yml` files.

Brief description of each file:

- `nginx.env`: [Nginx](https://hub.docker.com/_/nginx) settings;

- `frontend.env`: [catalog-frontend
](https://github.com/inpe-cdsr/catalog-frontend) website settings;

- `backend.env`: [catalog-backend
](https://github.com/inpe-cdsr/catalog-backend) service settings;

- `inpe_stac.env`: [inpe-stac
](https://github.com/inpe-cdsr/inpe-stac) service settings;

- `stac_compose.env`: [stac-compose
](https://github.com/inpe-cdsr/stac-compose) service settings;

- `geoserver.env`: [GeoServer](https://hub.docker.com/r/kartoza/geoserver/) settings;

- `db.env`: [MariaDB](https://hub.docker.com/_/mariadb) database settings;

- `phpmyadmin.env`: [phpMyAdmin](https://hub.docker.com/r/phpmyadmin/phpmyadmin/) settings.

The content of each file will be described afterwards.


### Docker compose services

This section describes the services inside `docker-compose.*.yml` files.


#### inpe_cdsr_nginx

`inpe_cdsr_nginx` is a service that runs a [Nginx](https://hub.docker.com/_/nginx) Docker image, that is a reverse proxy server. This service serves all other applications inside `docker-compose.*.yml` files.

Existing volumes:

- `/etc/nginx/conf.d/mysite.template`: Nginx settings file, such as upstreams and locations;

- `/var/www/html/datastore/TIFF`: this folder should contain all static files that Nginx will serve, such as quick looks and TIFFs.

Environment variables file is named as `./env_files/nginx.env` and its variables are:

- `NGINX_HOST`: which host name the Nginx service will run inside the Docker container, such as `my.server.com`;

- `NGINX_PORT`: which port the Nginx service will run inside the Docker container, such as `80` or `8080`.


#### inpe_cdsr_frontend

[inpe_cdsr_frontend](https://github.com/inpe-cdsr/catalog-frontend) is a service that contains all front-end application. You can build its Docker image [here](https://github.com/inpe-cdsr/catalog-frontend).

Existing volumes in development mode:

- `/app`: this folder needs to point to the [catalog-frontend](https://github.com/inpe-cdsr/catalog-frontend) source code, that is the [portal](https://github.com/inpe-cdsr/catalog-frontend/tree/master/portal) folder;

- `/app/src/assets/env.js`: this file contains the environment variables to run the application in development mode. In development mode, the Angular does not read `./env_files/frontend.env` file. `env.js` contains the same variables than `frontend.env` file.

Environment variables file is named as `./env_files/frontend.env` and its variables are:

- `URL_GEOSERVER`: [Geoserver](https://hub.docker.com/r/kartoza/geoserver/) URL that Nginx serves;

- `URL_VIA_CEP`: CEP webservice, such as `http://viacep.com.br/ws`;

- `URL_STAC_COMPOSE`: [stac-compose](https://github.com/inpe-cdsr/stac-compose) URL that Nginx serves;

- `URL_API`: [catalog-backend](https://github.com/inpe-cdsr/catalog-backend) URL that Nginx serves;

- `GRIDS`: list of grids saved on Geoserver separate by semicolon (`;`). Each grid has the following standard: `<title>:<layer>:<style>`, where:
    - `<title>`: a title that will be shown on the web portal;
    - `<layer>`: layer name on Geoserver;
    - `<style>`: layer style on Geoserver.

The file `./env_files/frontend.env` is just read in production mode.


#### inpe_cdsr_backend

[inpe_cdsr_backend](https://github.com/inpe-cdsr/catalog-backend) is a back-end service to [inpe_cdsr_frontend](https://github.com/inpe-cdsr/catalog-frontend) web application. This service provides user authentication and an endpoint to download satellite images.

Existing volumes:

- `/TIFF`: this folder should contain all static files, such as quick looks and TIFFs.

Existing volumes in development mode:

  - `/app`: this folder needs to point to the [catalog-backend](https://github.com/inpe-cdsr/catalog-backend) source code, that is the [root](https://github.com/inpe-cdsr/catalog-backend) folder.

Environment variables file is named as `./env_files/backend.env` and its variables are:

- `ENVIRONMENT`: which environment the service will run, the only acceptable options are: `DevelopmentConfig` or `ProductionConfig`;

- `PORT`: which port the service will run inside the Docker container;

- `MYSQL_DB_USER`: MySQL database user;

- `MYSQL_DB_PASSWORD`: MySQL database user password;

- `MYSQL_DB_HOST`: MySQL host name;

- `MYSQL_DB_DATABASE`: MySQL database name;

- `JWT_SECRET`: a JWT secret randomically generated;

- `JWT_ALGORITHM`: JWT algorithm.

`MYSQL_DB_USER_*` environment variables are related to MySQL database connection and `JWT_*` ones are related to [JWT](https://pyjwt.readthedocs.io/en/latest/) encryption.


#### inpe_cdsr_inpe_stac

[inpe_cdsr_inpe_stac](https://github.com/inpe-cdsr/inpe-stac) is a [SpatioTemporal Asset Catalog (STAC)](https://github.com/radiantearth/stac-spec) service. This service is a STAC implementation for INPE Catalog.

Existing volumes in development mode:

  - `/inpe_stac`: this folder needs to point to the [inpe-stac](https://github.com/inpe-cdsr/inpe-stac) source code, that is the [inpe_stac](https://github.com/inpe-cdsr/inpe-stac/tree/master/inpe_stac) folder.

Environment variables file is named as `./env_files/inpe_stac.env` and its variables are:

- `BASE_URI`: base URI that Nginx points to `inpe_cdsr_inpe_stac` service.

- `FLASK_APP`: file that will be executed when the service starts, default can be used. This can be changed if you build a new image with a new `app.py` location;

- `FLASK_ENV`: which environment the service will run, the only acceptable options are: `development` or `production`;

- `FLASK_RUN_HOST`: which host name the service will run inside the Docker container;

- `FLASK_RUN_PORT`: which port the service will run inside the Docker container;

- `DB_HOST`: MySQL host name and port (e.g `localhost:3306`);

- `DB_USER`: MySQL database user;

- `DB_PASS`: MySQL database user password;

- `DB_NAME`: MySQL database name;

- `FILE_ROOT`: URI that points to a folder where satellite images are;

- `API_VERSION`: STAC application version.

`DB_USER_*` environment variables are related to MySQL database connection.


#### inpe_cdsr_stac_compose

[inpe_cdsr_stac_compose](https://github.com/inpe-cdsr/stac-compose) is a [SpatioTemporal Asset Catalog (STAC)](https://github.com/radiantearth/stac-spec) compose service. This service serves various STAC applications, called providers.

Existing volumes:

- `/bdc-stac-compose/bdc_search_stac/providers/static/providers.json`: STAC compose providers on a JSON object format, where the key is the STAC application name and its internal keys are:

    - `url`: string. STAC application URI;

    - `type`: string. Application type, in other words, `stac`;

    - `version`: string. STAC application version;

    - `method`: string. HTTP method used to access the STAC application;

    - `filter_mult_collection`: boolean.

        - 1: if this STAC application accepts multiple collection search;

        - 0: if this STAC application does not accept multiple collection search.

    - `require_credentials`: boolean.

        - 1: if to download the file requires user credentials;

        - 0: if to download the file does not require user credentials.

    - `downloadable`: boolean.

        - 1: if the files, this STAC provides, are downloadable on the portal;

        - 0: if the files, this STAC provides, are not downloadable on the portal.

    - `ignore_provider_validation`: boolean.

        - 1: if the service is not allowed to validate the provider before using it;

        - 0: if the service is allowed to validate the provider before using it.

An example file can be found in [`./volumes/stac-compose/providers/providers.json`](./volumes/stac-compose/providers/providers.json). This file will be used by [stac-compose](https://github.com/inpe-cdsr/stac-compose) that runs inside `docker-compose.*.yml` files. In order to add a new provider, just insert a new key inside the JSON object in the file and fill it with the correct settings.

Existing volumes in development mode:

  - `/bdc-stac-compose`: this folder needs to point to the [stac-compose](https://github.com/inpe-cdsr/stac-compose) source code, that is the [root](https://github.com/inpe-cdsr/stac-compose) folder.

Environment variables file is named as `./env_files/stac_compose.env` and its variables are:

- `FLASK_ENV`: which environment the service will run, the only acceptable options are: `development` or `production`;

- `SERVER_HOST`: which host name the service will run inside the Docker container;

- `SERVER_PORT`: which port the service will run inside the Docker container.


#### inpe_cdsr_portainer

`inpe_cdsr_portainer` is a service that runs a [Portainer](https://hub.docker.com/r/portainer/portainer/) Docker image. Its settings can be found on its [repository on Docker hub](https://hub.docker.com/r/portainer/portainer/) or [documentation](https://portainer.readthedocs.io/en/1.23.0/index.html).

Existing volumes:

- `/data`: a folder where Portainer will persist its data;

- `/var/run/docker.sock`: path where Portainer will attempt to connect to the local Docker engine.

#### inpe_cdsr_geoserver

`inpe_cdsr_geoserver` is a service that runs a [GeoServer](https://hub.docker.com/r/kartoza/geoserver/) Docker image. Its settings can be found on its [repository on Docker hub](https://hub.docker.com/r/kartoza/geoserver/).

Existing volumes:

- `/opt/geoserver/data_dir`: a folder where GeoServer will persist its data;

- `/usr/local/tomcat/webapps/geoserver/WEB-INF/web.xml`: GeoServer settings file.

Extra existing volumes:

- `/home/data/grids`: a folder where the grids in Shapefile format are saved to be served by GeoServer.

Environment variables file is named as `./env_files/geoserver.env` and its variables are described on Geoserver Docker hub on [Run (manual docker commands)](https://hub.docker.com/r/kartoza/geoserver/) section.


#### inpe_cdsr_db

`inpe_cdsr_db` is a service that runs a [MariaDB](https://hub.docker.com/_/mariadb) Docker image, that is a database server.

Existing volumes:

- `/var/lib/mysql`: folder where MySQL will persist its data.

Environment variables file is named as `./env_files/db.env` and its variables are:

- `MYSQL_ROOT_PASSWORD`: default password to MySQL database.


#### inpe_cdsr_admin

`inpe_cdsr_admin` is a service that runs a [phpMyAdmin](https://hub.docker.com/r/phpmyadmin/phpmyadmin/) Docker image.

Environment variables file is named as `./env_files/phpmyadmin.env` and its variables are described on phpMyAdmin Docker hub on [Environment variables summary](https://hub.docker.com/r/phpmyadmin/phpmyadmin/) section.


### Run the docker compose

This section describes how to run the `docker-compose.*.yml` files, in development and production mode, and the available endpoints.


#### Development mode

`docker-compose.dev.yml` file uses volumes that point to the source code of some services, hence you need download the other repositories first, in order to execute this file.

Assuming that you are inside `catalog` folder, go back one folder and clone the other repositories:

```
$ cd .. && \
git clone https://github.com/inpe-cdsr/catalog-frontend.git && \
git clone https://github.com/inpe-cdsr/catalog-backend.git && \
git clone https://github.com/inpe-cdsr/inpe-stac.git && \
git clone https://github.com/inpe-cdsr/stac-compose.git
```

For each repository you cloned before, you need to build its development Docker image by following the instructions inside each repository: [catalog-frontend](https://github.com/inpe-cdsr/catalog-frontend), [catalog-backend](https://github.com/inpe-cdsr/catalog-backend), [inpe-stac](https://github.com/inpe-cdsr/inpe-stac) and [stac-compose](https://github.com/inpe-cdsr/stac-compose).

Angular does not read environment variables because it is executed on the interface, then you need to create a JavaScript file with the necessary variables in development mode. In order to do that, get into the `catalog-frontend` volume folder, copy the environment file and edit it if it is necessary:

```
$ cd catalog/volumes/catalog-frontend/ && \
  cp env.js.EXAMPLE env.js
```

After the steps above, you are able to run the development file:

```
$ docker-compose -f docker-compose.dev.yml up
```


#### Production mode

Before running in production mode, you must log in the following registry in order to download all necessary Docker images:

```
$ docker login registry.dpi.inpe.br
```

When you run in production mode for the first time, all Docker images will be downloaded. If they are not downloaded, for some reason, you can pull all Docker images using the following command:

```
$ docker pull <Docker image>
```

Where `<Docker image>` is the image used inside each service on the `docker-compose.prod.yml` file.

Use the following command to run in production mode:

```
$ docker-compose -f docker-compose.prod.yml up -d
```

#### Endpoints

After running the docker compose, Nginx will serve all applications with the host and port you defined inside [env_files/nginx.env](./env_files/nginx.env) file (e.g. `http://localhost:8089`).

The following endpoints are now available:

- `/catalogo`: [catalog-frontend](https://github.com/inpe-cdsr/catalog-frontend) application;

- `/api`: [catalog-backend](https://github.com/inpe-cdsr/catalog-backend) application;

- `/inpe-stac`: [inpe-stac](https://github.com/inpe-cdsr/inpe-stac) application;

- `/stac-compose`: [stac-compose](https://github.com/inpe-cdsr/stac-compose) application;

- `/geoserver`: [Geoserver](https://hub.docker.com/r/kartoza/geoserver/) application;

- `/portainer`: [portainer](https://hub.docker.com/r/portainer/portainer/) application;


### Database settings

Follow the instructions of the [README.md](https://github.com/inpe-cdsr/database/blob/master/README.md) file inside the [database](https://github.com/inpe-cdsr/database) repository to initialize correctly your [MariaDB](https://mariadb.com/) database.

In order to open [MariaDB](https://mariadb.com/), access [phpMyAdmin](https://www.phpmyadmin.net/) on `http://<your host>:8099`.


### GeoServer settings

On this section you will import your Shapefile files inside [GeoServer](http://geoserver.org/).

First, run `docker-compose.*.yml` file and open the [GeoServer](http://geoserver.org/) on `http://<your host>:8089/geoserver`.

Make sure you have all grids saved on Shapefile files. Put all these files inside `volumes/geoserver/grids` folder separate by subfolders, for example: the folder "`grid_cbers4_mux`" will have all files from CBERS4 grid Shapefile. There are default Shapefile files inside `volumes/geoserver/grids` folder.

Click on `Workspaces` option on left side menu and click on `Add new workspace`. Create a new workspace with the following information:

```
Name: vector_data

Namespace URI: <your host>/vector_data
```


#### Add the Shapefile files

Follow the steps below to add your Shapefile files to your [GeoServer](http://geoserver.org/):

For the steps below, consider each `<data source name>` one element inside the following list: ["`grid_cbers4_mux`", "`grid_ibge_states`", "`grid_landsat_tm_amsul`", "`grid_sentinel_mgrs`"].

- Click on `Stores` option on left side menu and click on `Add new store`.

- Click on `New data source/Vector Data Sources/Shapefile`.

- Choose the workspace called `vector_data`.

- Fill `Data Source Name` and `Description` with `<data source name>`.

- Select your Shapefile related to `<data source name>` inside `/home/data/grids/` folder.

- Click on `Save` button.

- `New layer` page will be opened, then click on `Publish` button.

- Inside `Edit Layer` page, make sure all information is correct.

    - Check if `Coordinate Reference Systems` are corrects.

    - Inside `Bounding Boxes` section, compute the native bounding boxes by clicking on `Compute from data` and `Compute from native bounds`.

    - Click on `Save` button.


#### Create the styles

Follow the steps below to create styles related to your Shapefile files:

- Click on `Styles` option on left side menu and click on `Add a new style`. Create a new style with the following information:

```
Name: grids

Workspace: vector_data
```

- Upload a style file by clicking on `Choose File` and select `volumes/geoserver/styles/grids.xml` file. Click on `Upload ...` button.

- Click on `Apply`.

- Click on `Publishing` tab and associate all layers with the `grids` style by clicking on `Associated` checkbox.

- Click on `Submit`.

Follow the same algorithm above to add a new style named `states` with the workspace called `vector_data`. Then, choose the `volumes/geoserver/styles/states.xml` file. On `Publishing` tab, associate just `grid_ibge_states` Shapefile file with `states` style.
