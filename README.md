* First

Rename the files inside 'docker-env' folder:

```
cd docker-env

mv db-backups.env.EXAMPLE db-backups.env
mv db.env.EXAMPLE db.env
mv geoserver.env.EXAMPLE geoserver.env
```

Change the configurations of these files, if necessary.


* Run docker compose:

docker-compose -f docker-compose.yml up

docker-compose -f docker-compose.yml up -d


* Down docker compose:

docker-compose -f docker-compose.yml down


* Remove volumes and containers

docker-compose down --volumes


* Import grids on the PostGIS database:

ogr2ogr -append -f "PostgreSQL" PG:"host=localhost port=9002 dbname=vector_data user=postgres password=postgres" ../database/grid/grid_cbers4_mux_south_america/grid_cbers4_mux_south_america.shp -nln grid_cbers4_mux_south_america -a_srs EPSG:4326 -skipfailures -lco FID=ID -lco GEOMETRY_NAME=geom -nlt PROMOTE_TO_MULTI

>>> ogr2ogr -append -f "PostgreSQL" PG:"host=localhost port=9002 dbname=vector_data user=postgres password=postgres" grid_cbers4_mux_south_america.shp -a_srs EPSG:4326 -skipfailures -lco FID=ID -lco GEOMETRY_NAME=geom

ogr2ogr -append -f "PostgreSQL" PG:"host=localhost port=9002 dbname=vector_data user=postgres password=postgres" ../database/grid/grid_landsat_south_america/grid_landsat_south_america.shp -nln grid_landsat_south_america -a_srs EPSG:4326 -skipfailures -lco FID=ID -lco GEOMETRY_NAME=geom -nlt PROMOTE_TO_MULTI

ogr2ogr -append -f "PostgreSQL" PG:"host=localhost port=9002 dbname=vector_data user=postgres password=postgres" ../database/grid/vector_ibge_states_of_brazil/vector_ibge_states_of_brazil.shp -nln vector_ibge_states_of_brazil -a_srs EPSG:4326 -skipfailures -lco FID=ID -lco GEOMETRY_NAME=geom -nlt PROMOTE_TO_MULTI
