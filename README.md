# compose

## First

Rename the files inside 'docker-env' folder:

```
cd docker-env

mv db-backups.env.EXAMPLE db-backups.env
mv db.env.EXAMPLE db.env
mv geoserver.env.EXAMPLE geoserver.env
```

Change the configurations of these files, if necessary.


### Run and down a docker compose:

docker-compose -f docker-compose.yml up

docker-compose -f docker-compose.yml up -d

docker-compose -f docker-compose.yml down
