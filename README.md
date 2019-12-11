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
