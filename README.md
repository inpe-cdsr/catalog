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


### Run the docker compose:

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
