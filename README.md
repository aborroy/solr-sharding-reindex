# Alfresco Search Services Indexer for Sharding Environments

When re-indexing a living Alfresco Repository with SOLR Sharding and `solr.useDynamicShardRegistration` enabled, the new SOLR Shard Indexer services should be configured with Alfresco NodeState Tracker off. Using this approach, the SOLR Indexer services are not registered in the living Alfresco Repository as available SOLR Shards and the living system can operate normally.

This configuration provides two Docker Compose templates:

* [living](living) is an ACS server running 2 SOLR Shards configured with DB_ID method and Alfresco Search Services 1.4.3
* [indexer](indexer) is an Indexer service running 2 SOLR Shards configured with DB_ID method and Alfresco Serch Services 2.0.0.1

```
+----------------------------------------------+
|                                              |
|  ACS 6.2 (living)                            |
|                                              |
|   +----------------+  +-----------------+    |
|   |                |  |                 |    |
|   | solr6          |  | solr6secondary  |    |
|   | DB_ID          |  | DB_ID           |    |
|   | shard 1        |  | shard 2         |    |
|   |                |  |                 |    |
|   |                |  |                 |    |
|   | 1.4.3          |  | 1.4.3           |    |
|   |                |  |                 |    |
|   +----------------+  +-----------------+    |
|                                              |
+-----------+-------------------+--------------+
            ^                   ^
            |                   |
    +-------+--------+  +-------+---------+
    |                |  |                 |
    | solr6indexer   |  | solr62indexer   |
    | DB_ID          |  | DB_ID           |
    | shard 1        |  | shard 2         |
    |                |  |                 |    (indexer)
    |                |  |                 |
    | 2.0.0.1        |  | 2.0.0.1         |
    |                |  |                 |
    +----------------+  +-----------------+
```


You should review volumes, configuration, modules & tuning parameters before using this composition in **Production** environments.

# How to use this composition

Before running any Docker Compose, `shared-network` should be created, as it's used to communicate the living environment with the indexer environment.

```bash
$ docker network create shared-network
```

## Start Docker for Living Server

Start docker and check the services are running fine.

```bash
$ cd living
$ docker-compose up --build --force-recreate
```

## Services

Use the following username/password combination to login.

 - User: admin
 - Password: admin

Alfresco and related web applications can be accessed from the below URIs when the servers have started.

```
http://localhost:8080/alfresco   - Alfresco Repository
http://localhost:8080/share      - Alfresco Share
http://localhost:8083/solr       - Alfresco Search Services 1.4.3 (Shard 0)
http://localhost:8084/solr       - Alfresco Search Services 1.4.3 (Shard 1)
```

## Start Docker for Indexer Server

Once the Living Server is up and running, start docker and check the services are running fine.

```bash
$ cd indexer
$ docker-compose up --build --force-recreate
```

Some error messages like the following one will be shown in the log...

```
solr6indexer_1   | org.quartz.SchedulerException: Based on configured schedule, the given trigger
'Solr.ShardStatePublisher-alfresco' will never fire.
```

... but this is expected, as we want to index the living Alfresco Repository without being registered as available Shards in ACS.


## Services

There are only two Indexer SOLR Instances in this composition

```
http://localhost:8183/solr       - Alfresco Search Services 2.0.0.1 (Shard 0)
http://localhost:8184/solr       - Alfresco Search Services 2.0.0.1 (Shard 1)
```

Once these services are running, the Living Alfresco Repository will be indexed with 2.0 version with 2 Shards using DB_ID method.
