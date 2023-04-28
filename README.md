# clickhouse-replication-zookeeper_HA

Data Replication in ClickHouse Cluster with 3 node (Docker Based Setup) and 3 node Zookeeper Cluster to have fault tolerant setup supporting replication.

## Requirement -- Download these files from Github repo
Download `complete-project.tar.gz` file, uncompressed it and go to directory to run docker-compose command to start setup.

```
docker-compose.yaml
config.xml
clusters.xml
listen_host.xml
zookeeper.xml
macros_1.xml
macros_2.xml
macros_3.xml
```

Execute below commands from docker Host :
-d to run process in background.

```
root@ip-10-0-6-24:~/ashwini_work/clickhouse_Ashwini/clickhouse_replication_zookeeper_HA# docker-compose up -d
Creating network "clickhouse_replication_zookeeper_ha_my-network" with driver "bridge"
Creating zookeeper3 ... done
Creating zookeeper1 ... done
Creating zookeeper2 ... done
Creating clickhouse02 ... done
Creating clickhouse01 ... done
Creating clickhouse03 ... done
Attaching to zookeeper3, zookeeper2, zookeeper1, clickhouse01, clickhouse02, clickhouse03
```

When containers are running, connect to clickhouse-client and test

```
clickhouse01 :) CREATE DATABASE IF NOT EXISTS Example_DB ON CLUSTER `{cluster}`

CREATE DATABASE IF NOT EXISTS Example_DB ON CLUSTER `{cluster}`

Query id: b294309d-950e-4d41-83c3-3e6398f07bf4

┌─host─────────┬─port─┬─status─┬─error─┬─num_hosts_remaining─┬─num_hosts_active─┐
│ clickhouse01 │ 9000 │      0 │       │                   2 │                0 │
│ clickhouse02 │ 9000 │      0 │       │                   1 │                0 │
│ clickhouse03 │ 9000 │      0 │       │                   0 │                0 │
└──────────────┴──────┴────────┴───────┴─────────────────────┴──────────────────┘

3 rows in set. Elapsed: 0.190 sec.

clickhouse01 :)

clickhouse01 :) SELECT hostName(), * FROM clusterAllReplicas('cluster_demo_ash', system,
                one);

SELECT
    hostName(),
    *
FROM clusterAllReplicas('cluster_demo_ash', system, one)

Query id: ce15bd4b-af92-41e5-99b7-3bbcb4d4bffe

┌─hostName()───┬─dummy─┐
│ clickhouse01 │     0 │
└──────────────┴───────┘
┌─hostName()───┬─dummy─┐
│ clickhouse02 │     0 │
└──────────────┴───────┘
┌─hostName()───┬─dummy─┐
│ clickhouse03 │     0 │
└──────────────┴───────┘

3 rows in set. Elapsed: 0.016 sec.

clickhouse01 :) CREATE TABLE Example_DB.product ON CLUSTER '{cluster}'
                               (
                                    created_at DateTime,
                                    product_id UInt32,
                                    category UInt32
                               ) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{cluster}/{shard}/Example_DB/product', '{replica}')
                               PARTITION BY toYYYYMM(created_at)
                               ORDER BY (product_id, toDate(created_at), category)
                               SAMPLE BY category;

CREATE TABLE Example_DB.product ON CLUSTER `{cluster}`
(
    `created_at` DateTime,
    `product_id` UInt32,
    `category` UInt32
)
ENGINE = ReplicatedMergeTree('/clickhouse/tables/{cluster}/{shard}/Example_DB/product', '{replica}')
PARTITION BY toYYYYMM(created_at)
ORDER BY (product_id, toDate(created_at), category)
SAMPLE BY category

Query id: 43da5375-4f3d-4ab7-aa55-064d43bb7bc0

┌─host─────────┬─port─┬─status─┬─error─┬─num_hosts_remaining─┬─num_hosts_active─┐
│ clickhouse01 │ 9000 │      0 │       │                   2 │                0 │
│ clickhouse02 │ 9000 │      0 │       │                   1 │                0 │
│ clickhouse03 │ 9000 │      0 │       │                   0 │                0 │
└──────────────┴──────┴────────┴───────┴─────────────────────┴──────────────────┘

3 rows in set. Elapsed: 0.552 sec.

clickhouse01 :) SELECT * FROM system.zookeeper WHERE path = '/clickhouse/tables/cluster_demo_ash/1/Example_DB/product' FORMAT Vertical ;

SELECT *
FROM system.zookeeper
WHERE path = '/clickhouse/tables/cluster_demo_ash/1/Example_DB/product'
FORMAT Vertical

Query id: 407dd4c7-e4bd-4ad3-986d-1d6c52f22eac

Row 1:
──────
name:  alter_partition_version
value:
path:  /clickhouse/tables/cluster_demo_ash/1/Example_DB/product

Row 2:
──────
name:  metadata
value: metadata format version: 1
date column:
sampling expression: category
index granularity: 8192
mode: 0
sign column:
primary key: product_id, toDate(created_at), category
data format version: 1
partition key: toYYYYMM(created_at)
granularity bytes: 10485760

path:  /clickhouse/tables/cluster_demo_ash/1/Example_DB/product

Row 3:
──────
name:  temp
value:
path:  /clickhouse/tables/cluster_demo_ash/1/Example_DB/product

Row 4:
──────
name:  table_shared_id
----- O/P Truncated -----
```

Insert data from multiple clickhouse nodes as all nodes are writable and then verify.
```

clickhouse01 :) INSERT INTO Example_DB.product VALUES (now(),9832,11);


INSERT INTO Example_DB.product FORMAT Values

Query id: 6cd6e9b8-fbce-4dea-a1b1-f3e8b8e5b542

Ok.

1 row in set. Elapsed: 0.074 sec.

clickhouse01 :) SELECT *
                FROM Example_DB.product;

SELECT *
FROM Example_DB.product

Query id: d2e4a42c-9edd-4338-b425-554c4ed1c8c3

┌──────────created_at─┬─product_id─┬─category─┐
│ 2023-04-24 04:58:22 │         12 │       14 │
│ 2023-04-24 04:57:50 │        987 │       13 │
│ 2023-04-24 04:57:46 │       1834 │       15 │
│ 2023-04-24 04:57:42 │       5672 │       16 │
│ 2023-04-24 04:57:09 │       9832 │       11 │
└─────────────────────┴────────────┴──────────┘
┌──────────created_at─┬─product_id─┬─category─┐
│ 2023-04-24 04:58:25 │       1834 │       15 │
└─────────────────────┴────────────┴──────────┘

6 rows in set. Elapsed: 0.003 sec.

clickhouse01 :)

```




To destoy cluster, run below command

```
docker-compose down
```

Attached screenshot for reference 

![Screenshot 2023-04-24 at 12 59 53 PM](https://user-images.githubusercontent.com/124853365/233915986-0ab036b4-eb59-437d-b52f-c6d09020f7f6.png)


![Screenshot 2023-04-24 at 1 13 35 PM](https://user-images.githubusercontent.com/124853365/233916039-84bf311f-b757-4ee5-b22d-10df4575e673.png)
