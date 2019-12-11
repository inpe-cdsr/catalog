# docker-compose

Docker compose related to DGI Catalog project.

`docker-compose.dev.yml` file contains development services using Docker images with volumes. The Docker images do not contain all code inside them, the code is added into the Docker containers through volumes.

`docker-compose.prod.yml` file contains production services using Docker images without volumes. The Docker images contain all code inside them and they do not use volumes to keep their code.


## Settings

Rename the files inside `env` folder:

```
cd env/

mv portal.env.EXAMPLE portal.env && \
mv api.env.EXAMPLE api.env && \
mv inpe_stac.env.EXAMPLE inpe_stac.env && \
mv stac_compose.env.EXAMPLE stac_compose.env && \
mv geoserver.env.EXAMPLE geoserver.env && \
mv nginx.env.EXAMPLE nginx.env
```

Update the files above with proper settings.


### Description

The files below are environment variables files that contain settings related to the services inside `docker-compose.*.yml` files.

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


### Run the docker compose:

- Development:

```
docker-compose -f docker-compose.dev.yml up
```

- Production:

```
docker-compose -f docker-compose.prod.yml up -d
```
