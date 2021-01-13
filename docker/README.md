ATAV Database Docker Percona
========================

The instruction of Docker setup for ATAV database.

## Requirement

* docker-compose >= 3.0
* percona-server 5.6.45 (docker image)
* mysql client (optional)
* disable THP:
    * access Docker VM
    ```
    docker run -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh
    ```
    * Then run:
    ```
    echo never > /sys/kernel/mm/transparent_hugepage/enabled
    echo never > /sys/kernel/mm/transparent_hugepage/defrag
    ```

## Run

```
cd atav-database
```

#### Delete and Create data volumn
```
docker volume rm atavdb_mysqldata
docker volume create atavdb_mysqldata
```

#### Initialize Docker Percona Server (TokuDB enabled) 
```
docker run -d --name atavdb \
-v atavdb_mysqldata:/var/lib/mysql:rw \
-v $(pwd)/data:/var/lib/mysql-files:rw \
-p 3333:3306 \
-e MYSQL_ROOT_PASSWORD=root \
-e INIT_TOKUDB=1 \
percona/percona-server:5.6.45
```

#### Restore database schema
```
docker exec -i atavdb mysql -uroot -proot < atav-database/data/atavdb_schema.sql
docker exec -i atavdb mysql -uroot -proot < atav-database/data/externaldb_schema.sql
```

#### Load testing data
```
gunzip ./data/atavdb_load_data/*
for file in ./data/atavdb_load_data/*; do docker exec -i atavdb mysql -uroot -proot atavdb -e "load data infile '/var/lib/mysql-files/atavdb_load_data/${file##*/}' into table ${file##*/}" ; done
```

## Check

#### Check database records
```
docker exec -it atavdb mysql -uroot -proot atavdb -e "show table status"

# use this only when mysql client installed
mysql -h127.0.0.1 -uroot -proot atavdb -P 3333 -e "show table status"
```

#### Start a bash session
```
docker exec -it atavdb bash
```




