# DSMR Reader - Docker - ARM (Raspberry Pi support)

[![Build Status](https://geertvdc.visualstudio.com/home-automation/_apis/build/status/dsmr-reader-docker-arm-CI)](https://geertvdc.visualstudio.com/home-automation/_build/latest?definitionId=26)
[![Download docker image](https://img.shields.io/badge/Docker-Hub-blue.svg?logo=docker)](https://hub.docker.com/r/geertvdc/dsmr-reader-docker-arm/)

A docker-compose file in order to start the following application in Docker:  
dsmr-reader (https://github.com/dennissiemensma/dsmr-reader)

This is a fork from https://github.com/xirixiz/dsmr-reader-docker that only supports x86/x64 to also be able to run this on the Raspberry Pi which has an ARM architecture.
Thanks to Xirixiz for setting up most of the work!

You should first add the user you run Docker with on your host file system to the dialout group:
```
sudo usermod -aG dialout $(whoami)
```

# Docker-compose

An example docker-compose.yaml file can be found here: https://raw.githubusercontent.com/geertvdc/dsmr-reader-docker/master/docker-compose.yaml

You should modify the docker-compose file with parameters that suit your environment, then run docker-compose afterwards:
```
docker-compose up -d 
```

After starting the containers with docker-compose, the dashboard is reachable at  
HTTP: http://\<hostname>:7777  

After starting the containers, don't forget to modify the default DSMR version (default is DSMR v4):  
http://\<hostname>:7777/admin/dsmr_datalogger/dataloggersettings/

# Docker run

Keep in mind the example below only runs dsmr, you need to run a postgres docker container or traditional postgres environment as well, since a database is needed.

```
docker run -d \
  --name dsmr \
  --restart always \
  -p 7777:80 \
  -e DB_HOST=x.x.x.x \
  -e DSMR_USER=dsmrreader \
  -e DSMR_PASSWORD=dsmrreader \
  -e DSMR_EMAIL=root@localhost \
  --device /dev/ttyUSB0:/dev/ttyUSB0 \
  xirixiz/dsmr-reader-docker
```

# Backup and restore meganism 1
dsmrdb in docker-compose is configured to use a docker volume. So when the application and docker containter have been removed, the postgres data still persists.

Also you could easily create a backup. Values depend on docker/docker-compose user and database variables:  
```
docker-compose stop dsmr
docker exec -t dsmrdb pg_dumpall -c -U dsmrreader > dsmrreader.sql
docker-compose start dsmr
```

Or drop the database and restore a backup. Values depend on docker/docker-compose user and database variables:
```
docker-compose stop dsmr
docker exec -t dsmrdb dropdb dsmrreader -U dsmrreader
docker exec -t dsmrdb createdb -O dsmrreader dsmrreader -U dsmrreader
cat dsmrreader.sql | docker exec -i dsmrdb psql -U dsmrreader
docker-compose start dsmr
```

# Backup and restore meganism 2
Ofcourse it's also possible to use Docker's own volume backup and restore megansim.

Backup:
```
docker run -it --rm -v dsmrdb:/volume -v /tmp:/backup alpine \
    tar -cjf /backup/dsmrdb.tar.bz2 -C /volume ./
```

Restore:
```
docker run -it --rm -v dsmrdb:/volume -v /tmp:/backup alpine \
    sh -c "rm -rf /volume/* /volume/..?* /volume/.[!.]* ; tar -C /volume/ -xjf /backup/dsmrdb.tar.bz2"
 ```

# Important notes
The current configuration has been tested on Raspberry Pi 2 / 3b+

