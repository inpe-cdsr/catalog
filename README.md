# docker-compose

Docker compose related to DGI Catalog project.

`docker-compose.dev.yml` file contains development services using Docker images with volumes. The Docker images do not contain all code inside them, the code is added into the Docker containers through volumes.

`docker-compose.prod.yml` file contains production services using Docker images without volumes. The Docker images contain all code inside them and they do not use volumes to keep their code.


## Settings

Create the environment files related to each application based on the example ones inside `env` folder:

```
cd env/

cp portal.env.EXAMPLE portal.env && \
cp api.env.EXAMPLE api.env && \
cp inpe_stac.env.EXAMPLE inpe_stac.env && \
cp stac_compose.env.EXAMPLE stac_compose.env && \
cp geoserver.env.EXAMPLE geoserver.env && \
cp nginx.env.EXAMPLE nginx.env
```

Update the files above with proper settings. The files are environment variables files that contain settings related to the services inside `docker-compose.*.yml` files.

Brief description of each file:

- `portal.env`: [dgi-catalog-frontend
](https://github.com/dgi-catalog/dgi-catalog-frontend) website settings;
- `dgi_catalog_backend.env`: [dgi-catalog-backend
](https://github.com/dgi-catalog/dgi-catalog-backend) service settings;
- `inpe_stac.env`: [inpe-stac
](https://github.com/gqueiroz/inpe-stac) service settings;
- `stac_compose.env`: [stac-compose
](https://github.com/dgi-catalog/stac-compose) service settings;
- `geoserver.env`: Geoserver settings;
- `nginx.env`: Nginx settings.


### Docker compose settings

This section describes the services inside `docker-compose.*.yml` files.


#### dgi_catalog_nginx

[Nginx](https://hub.docker.com/_/nginx) is a reverse proxy server.

Existing volumes:

- `/etc/nginx/conf.d/mysite.template`: Nginx settings file, such as upstreams and locations;

- `/var/www/html/datastore`: this folder should contain all static files that Nginx will serve, such as quicklooks and TIFFs.

Environment variables file is named `./env/nginx.env` and its variables are:

- `NGINX_HOST`: host name where the Nginx will run, such as `my.server.com`;

- `NGINX_PORT`: which port Nginx will run, such as `80` or `8080`.


#### dgi_catalog_portal

[dgi_catalog_portal](https://github.com/dgi-catalog/dgi-catalog-frontend) is a service that contains all front-end application. You can build its Docker image [here](https://github.com/dgi-catalog/dgi-catalog-frontend#run-the-application-inside-a-docker-image).

Environment variables file is named `./env/portal.env` and its variables are:

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

[dgi_catalog_api](https://github.com/dgi-catalog/dgi-catalog-backend) is a back-end service to [dgi_catalog_portal](https://github.com/dgi-catalog/dgi-catalog-frontend) web application. This service provides user authentication and an endpoint to download images.

Environment variables file is named `./env/api.env` and its variables are:

- `ENVIRONMENT`: which environment the service will run, the only acceptable options are: `DevelopmentConfig` or `ProductionConfig`;

- `PORT`: which port the service will run;

- `MYSQL_DB_USER`: MySQL database user;

- `MYSQL_DB_PASSWORD`: MySQL database user password;

- `MYSQL_DB_HOST`: MySQL host name;

- `MYSQL_DB_DATABASE`: MySQL database name;

- `JWT_SECRET`: a JWT secret randomically generate;

- `JWT_ALGORITHM`: JWT algorithm.


### Run the docker compose:

In order to download all Docker images from DGI registry, you need to log into it.

```
docker login registry.dpi.inpe.br
```

- Development:

```
docker-compose -f docker-compose.dev.yml up
```

- Production:

```
docker-compose -f docker-compose.prod.yml up -d
```


### Geoserver settings

On this section you will import your Shapefile files inside Geoserver using styles.

First, run `docker-compose.*.yml` file and open the Geoserver on `http://<your host>:8089/geoserver`.

Make sure you have all grids saved on Shapefile files. Put all these files inside `geoserver/grids` folder separate by subfolders, for example: the folder "`grid_cbers4_mux`" will have all files from CBERS4 grid Shapefile. There are default Shapefile files inside `geoserver/grids` folder.

Click on `Workspaces` option on left side menu and click on `Add new workspace`. Create a new workspace with the following information:

```
Name: vector_data

Namespace URI: <your host>/vector_data
```


#### Add the Shapefile files

Follow the steps below to add your Shapefile files to your Geoserver:

For the steps below, consider each `<data source name>` one element inside the following list: ["`grid_cbers4_mux`", "`grid_ibge_states`", "`grid_landsat_tm_amsul`", "`grid_sentinel_mgrs`"].

- Click on `Stores` option on left side menu and click on `Add new store`.

- Click on `New data source/Vector Data Sources/Shapefile`.

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

- Upload a style file by clicking on `Choose File` and select `geoserver/styles/grids.xml` file. Click on `Upload ...` button.

- Click on `Apply`.

- Click on `Publishing` tab and associate all layers with the `grids` style by clicking on `Associated` checkbox.

- Click on `Submit`.

Follow the same algorithm above to add a new style named `states` with the workspace called `vector_data`. Then, choose the `geoserver/styles/states.xml` file. On `Publishing` tab, associate just `grid_ibge_states` Shapefile file with `states` style.
