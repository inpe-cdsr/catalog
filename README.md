# docker-compose

Docker compose related to DGI Catalog project.

`docker-compose.dev.yml` file contains development services using Docker images with volumes. The Docker images do not contain all code inside them, the code is added into the Docker containers through volumes.

`docker-compose.prod.yml` file contains production services using Docker images without volumes. The Docker images contain all code inside them and they do not use volumes to keep their code.


## Install

Create a new folder where you will let all your repositories, for example `dgi-catalog`:

```
$ mkdir dgi-catalog
```

Clone the [docker-compose](https://github.com/dgi-catalog/docker-compose) repository inside the previous folder:

```
$ cd dgi-catalog/ && \
git clone https://github.com/dgi-catalog/docker-compose && \
cd docker-compose/
```


### Settings

Create the environment files related to each application based on the example ones inside [env_files](./env_files) folder:

```
$ cd env_files/

$ cp nginx.env.EXAMPLE nginx.env && \
  cp portal.env.EXAMPLE portal.env && \
  cp api.env.EXAMPLE api.env && \
  cp inpe_stac.env.EXAMPLE inpe_stac.env && \
  cp stac_compose.env.EXAMPLE stac_compose.env && \
  cp geoserver.env.EXAMPLE geoserver.env && \
  cp db.env.EXAMPLE db.env && \
  cp phpmyadmin.env.EXAMPLE phpmyadmin.env && \
  cp publisher.env.EXAMPLE publisher.env
```

Update the files above with proper settings. The files are environment variables files that contain settings related to the services inside `docker-compose.*.yml` files.

Brief description of each file:

- `portal.env`: [dgi-catalog-frontend
](https://github.com/dgi-catalog/dgi-catalog-frontend) website settings;

- `api.env`: [dgi-catalog-backend
](https://github.com/dgi-catalog/dgi-catalog-backend) service settings;

- `inpe_stac.env`: [inpe-stac
](https://github.com/gqueiroz/inpe-stac) service settings;
- `stac_compose.env`: [stac-compose
](https://github.com/dgi-catalog/stac-compose) service settings;

- `geoserver.env`: Geoserver settings;

- `nginx.env`: Nginx settings.

The content of each file will be described afterwards.


### Docker compose services

This section describes the services inside `docker-compose.*.yml` files.


#### dgi_catalog_nginx

`dgi_catalog_nginx` is a service that runs a [Nginx](https://hub.docker.com/_/nginx) Docker image, that is a reverse proxy server. This service serves all other applications inside `docker-compose.*.yml` files.

Existing volumes:

- `/etc/nginx/conf.d/mysite.template`: Nginx settings file, such as upstreams and locations;

- `/var/www/html/datastore`: this folder should contain all static files that Nginx will serve, such as quicklooks and TIFFs.

Environment variables file is named as `./env_file/nginx.env` and its variables are:

- `NGINX_HOST`: host name where the Nginx will run, such as `my.server.com`;

- `NGINX_PORT`: which port Nginx will run inside the container, such as `80` or `8080`.


#### dgi_catalog_portal

[dgi_catalog_portal](https://github.com/dgi-catalog/dgi-catalog-frontend) is a service that contains all front-end application. You can build its Docker image [here](https://github.com/dgi-catalog/dgi-catalog-frontend).

Existing volumes in development mode:

  - `/app`: this folder needs to point to the [dgi-catalog-frontend](https://github.com/dgi-catalog/dgi-catalog-frontend) source code, that is the [portal/](https://github.com/dgi-catalog/dgi-catalog-frontend/tree/master/portal) folder.

Environment variables file is named as `./env_files/portal.env` and its variables are:

- `URL_GEOSERVER`: [Geoserver](https://hub.docker.com/r/kartoza/geoserver/) URL that Nginx serves;

- `URL_VIA_CEP`: CEP webservice, such as `http://viacep.com.br/ws`;

- `URL_STAC_COMPOSE`: [stac-compose](https://github.com/dgi-catalog/stac-compose) URL that Nginx serves;

- `URL_API`: [dgi-catalog-backend](https://github.com/dgi-catalog/dgi-catalog-backend) URL that Nginx serves;

- `PROVIDERS_TOKEN`: list of providers that user credentials will be added before providers URL in order to download their images with authentication;

- `GRIDS`: list of grids saved on Geoserver separate by semicolon (`;`). Each grid has the following standard: `<title>:<layer>:<style>`, where:
    - `<title>`: a title that will be shown on the web portal;
    - `<layer>`: layer name on Geoserver;
    - `<style>`: layer style on Geoserver.


#### dgi_catalog_api

[dgi_catalog_api](https://github.com/dgi-catalog/dgi-catalog-backend) is a back-end service to [dgi_catalog_portal](https://github.com/dgi-catalog/dgi-catalog-frontend) web application. This service provides user authentication and an endpoint to download satellite images.

Existing volumes in development mode:

  - `/app`: this folder needs to point to the [dgi-catalog-backend](https://github.com/dgi-catalog/dgi-catalog-backend) source code, that is the [root](https://github.com/dgi-catalog/dgi-catalog-backend) folder.

Environment variables file is named as `./env_files/api.env` and its variables are:

- `ENVIRONMENT`: which environment the service will run, the only acceptable options are: `DevelopmentConfig` or `ProductionConfig`;

- `PORT`: which port the service will run inside the Docker container;

- `MYSQL_DB_USER`: MySQL database user;

- `MYSQL_DB_PASSWORD`: MySQL database user password;

- `MYSQL_DB_HOST`: MySQL host name;

- `MYSQL_DB_DATABASE`: MySQL database name;

- `JWT_SECRET`: a JWT secret randomically generated;

- `JWT_ALGORITHM`: JWT algorithm.

`MYSQL_DB_USER_*` environment variables are related to MySQL database connection and `JWT_*` ones are related to [JWT](https://pyjwt.readthedocs.io/en/latest/) encryption.


#### dgi_catalog_inpe_stac

[dgi_catalog_inpe_stac](https://github.com/gqueiroz/inpe-stac) is a [SpatioTemporal Asset Catalog (STAC)](https://github.com/radiantearth/stac-spec) service. This service is a STAC implementation for INPE Catalog.

Existing volumes in development mode:

  - `/inpe_stac`: this folder needs to point to the [inpe-stac](https://github.com/gqueiroz/inpe-stac) source code, that is the [inpe_stac](https://github.com/gqueiroz/inpe-stac/tree/master/inpe_stac) folder.

Environment variables file is named as `./env_files/inpe_stac.env` and its variables are:

- `BASE_URI`: base URI that Nginx points to `dgi_catalog_inpe_stac` service.

- `FLASK_APP`: file that will be executed when the service starts, default can be used. This can be changed if you build a new image with a new `app.py` location;

- `FLASK_ENV`: which environment the service will run, the only acceptable options are: `development` or `production`;

- `DB_HOST`: MySQL host name and port (e.g `localhost:3306`);

- `DB_USER`: MySQL database user;

- `DB_PASS`: MySQL database user password;

- `DB_NAME`: MySQL database name;

- `FILE_ROOT`: URI that points to a folder where satellite images are;

- `API_VERSION`: STAC application version;

`DB_USER_*` environment variables are related to MySQL database connection.


#### dgi_catalog_stac_compose

[dgi_catalog_stac_compose](https://github.com/dgi-catalog/stac-compose) is a [SpatioTemporal Asset Catalog (STAC)](https://github.com/radiantearth/stac-spec) compose service. This service serves various STAC applications, called providers.

Existing volumes:

- `/bdc-stac-compose/bdc_search_stac/providers/static/providers.json`: STAC compose providers on JSON format, following the pattern:

```
{
  "<STAC application name>": {
    "url": "<STAC application URI>",
    "type":"stac",
    "version": "<STAC application version>",
    "method": "<method used by STAC application>"
  },
  ...
}
```

Existing volumes in development mode:

  - `/bdc-stac-compose`: this folder needs to point to the [stac-compose](https://github.com/dgi-catalog/stac-compose) source code, that is the [root](https://github.com/dgi-catalog/stac-compose) folder.

Environment variables file is named as `./env_files/stac_compose.env` and its variables are:

- `PORT`: which port the server will run inside the Docker container.


#### dgi_catalog_portainer

`dgi_catalog_portainer` is a service that runs a [Portainer](https://hub.docker.com/r/portainer/portainer/) Docker image. Its settings can be found on its [repository on Docker hub](https://hub.docker.com/r/portainer/portainer/).

#### dgi_catalog_geoserver

`dgi_catalog_geoserver` is a service that runs a [Geoserver](https://hub.docker.com/r/kartoza/geoserver/) Docker image. Its settings can be found on its [repository on Docker hub](https://hub.docker.com/r/kartoza/geoserver/).

Extra existing volumes:

- `/home/data/grids`: a folder where the grids in Shapefile format are saved to be served by Geoserver.

Environment variables file is named as `./env_files/geoserver.env` and its variables are described on Geoserver Docker hub on [Run (manual docker commands)](https://hub.docker.com/r/kartoza/geoserver/) section.

### Run the docker compose:

In order to download all Docker images from DGI registry, you need to log in it.

```
$ docker login registry.dpi.inpe.br
```


#### Development

In order to run in development mode, you need to build all development images with the instructions inside each repository linked in [this section](#docker-compose-services).

```
$ docker-compose -f docker-compose.dev.yml up
```


#### Production

When you run in production mode, all Docker images will be downloaded. If they are not you can pull all Docker images using the following command:

```
$ docker pull <Docker image>
```

Where `<Docker image>` is the image inside each service on the `docker-compose.prod.yml` file.

With all Docker images downloaded, you can run the docker compose:

```
$ docker-compose -f docker-compose.prod.yml up -d
```

#### Endpoints

After running the docker compose, Nginx will serve all applications with the host and port you defined inside [env_files/nginx.env](./env_files/nginx.env) file (e.g `http://localhost:8089`).

The following endpoints are now available:

- `/catalogo`: [dgi-catalog-frontend](https://github.com/dgi-catalog/dgi-catalog-frontend) application;

- `/api`: [dgi-catalog-backend](https://github.com/dgi-catalog/dgi-catalog-backend) application;

- `/inpe-stac`: [inpe-stac](https://github.com/gqueiroz/inpe-stac) application;

- `/stac-compose`: [stac-compose](https://github.com/dgi-catalog/stac-compose) application;

- `/geoserver`: [Geoserver](https://hub.docker.com/r/kartoza/geoserver/) application;

- `/portainer`: [portainer](https://hub.docker.com/r/portainer/portainer/) application;


### Database settings

On this section you will do some configurations to your [MariaDB](https://mariadb.com/) database.

In order to open [MariaDB](https://mariadb.com/), access [phpMyAdmin](https://www.phpmyadmin.net/) on `http://<your host>:8099`.


#### Import databases

>>> TODO

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
