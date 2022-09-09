# Citus + Hasura Integration

Docker-compose file to run citus and hasura together, This repository assumes you have some level of knowledge in citus, hasura and docker-compose, if you know about those but is not that knowledgeble when it comes to citus, here is a quick explanation:

> Citus is a postgres extension that distributes data across multiples nodes in a cluster
> with a horizontal-scaling approach.

If you need more info on citus check the **links at the end of this readme**.

## Importing CSV in citus hasura docker

Check the citus (master) postgres container id, The one with this port:
`0.0.0.0:5432 => 5432/tcp, :::5432=>5432/tcp`
get its ID and use this command:
`docker cp path/to/csv <CONTAINER_ID>:/name_of_csv.csv`
Example:
`docker cp users.csv 06ae36661338:/users.csv`

After that you can enter in postgres (master) with:
`sudo docker exec -it citus_master /bin/bash`
and Now that you entered you can simply run:

```bash
#the -U flag stands for "user"
psql -U postgres
```

Now that we can execute some commands directly in postgres and we have the csv inside
we can copy it to a table we want (the table should be already created):

```bash
# here "companies" is an empty table I've created before hand
\copy users from 'users.csv' with csv
```

## Rollups

# Help

## Deleting docker stuff to reset

If you need to delete your docker containers and volumes use this commands:

- `docker rm -f $(docker ps -a -q)`
- `docker volume rm $(docker volume ls -q)`

## Insert dummy data easily

You can create some dummy data in postgres by using the `generate_series` function like this:

```sql
-- simple example
INSERT INTO users (hash_key, content, unix_timestamp, "timestamp")
SELECT
md5(random()::text),
'this is some content that will be the same for every new row', -- USE SINGLE QUOTES!!!
-- https://stackoverflow.com/questions/22964272/postgresql-get-a-random-datetime-timestamp-between-two-datetime-timestamp
date_part('epoch', NOW() + (random() * (NOW()+'90 days' - NOW())) + '30 days'),
NOW() + (random() * (NOW()+'90 days' - NOW())) + '30 days'
FROM generate_series(1, 10000) s(i)
```

Check here for an [stackoverflow example](https://stackoverflow.com/questions/24841142/how-can-i-generate-big-data-sample-for-postgresql-using-generate-series-and-rand), if you need more info, go to postgres docs!

## Info about query in postgres

You can get some info about some query like this:

```sql
EXPLAIN (ANALYZE, COSTS OFF)
SELECT * FROM users
```

REF: [stackoverflow](https://stackoverflow.com/questions/9063402/get-execution-time-of-postgresql-query)

## Unsafe triggers in citus

There is probably a better solution, but for now what I know is that you can run this befor any query that has a trigger to avoid the "unsafe trigger" error:

```
SET citus.enable_unsafe_triggers TO on;
```

### Useful Links

- [Example citus docker](https://github.com/citusdata/docker/blob/master/docker-compose.yml)
- [Old example citus with hasura](https://github.com/rongfengliang/citus-hasura-graphql)
- [Safe incremental rollups on Postgres and Citus](https://gist.github.com/marcocitus/1ac72e7533dbb01801973ee51f89fecf)
- [citus great tutorial realtime analytics](https://docs.citusdata.com/en/v11.0/get_started/tutorial_realtime_analytics.html)

## Useful Postgres stuff

Some explanations and links more specific to postgres

### Postgres Links:

- [pg hero for checking performance](https://github.com/ankane/pghero)
- [stackoverflow what columns generally make good indexes](https://stackoverflow.com/questions/107132/what-columns-generally-make-good-indexes)
- [tableplus for database management](https://tableplus.com/)

## Useful Docker explanations and links

If are a bit unsure about docker stuff this should give you a help, this is not exactly a begginner guide, just a couple infos about it.

### Data in docker

Docker works with containers, those containers represent our "services", for example you can have a docker container that has a postgres image, so when this container is up
you have a postgres db running, this container can have data inside but if you delete this container this data is gonna dissappear too. If you want to PERSIST data for a container you can create a VOLUME, if your container is deleted but your volume is not you will still have this data persisted when you recreate this container. This does mean that if you want to completely reset this container's data you need to delete both container and volume.

