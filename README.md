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


## Random Help
If you need to delete your docker containers and volumes use this commands:
- `docker rm -f $(docker ps -a -q)`
- `docker volume rm $(docker volume ls -q)`

### Useful Links
- [Example citus docker](https://github.com/citusdata/docker/blob/master/docker-compose.yml)
- [Old example citus with hasura](https://github.com/rongfengliang/citus-hasura-graphql)
- [Safe incremental rollups on Postgres and Citus](https://gist.github.com/marcocitus/1ac72e7533dbb01801973ee51f89fecf)
- [citus great tutorial realtime analytics](https://docs.citusdata.com/en/v11.0/get_started/tutorial_realtime_analytics.html)