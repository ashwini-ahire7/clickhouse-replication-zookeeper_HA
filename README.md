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
Creating clickhouse03 ... done
Creating clickhouse01 ... done
Attaching to zookeeper3, zookeeper2, zookeeper1, clickhouse01, clickhouse02
```

When containers are running, connect to clickhouse-client and test




To take it down - docker-compose down


![Screenshot 2023-04-24 at 12 59 53 PM](https://user-images.githubusercontent.com/124853365/233915986-0ab036b4-eb59-437d-b52f-c6d09020f7f6.png)


![Screenshot 2023-04-24 at 1 13 35 PM](https://user-images.githubusercontent.com/124853365/233916039-84bf311f-b757-4ee5-b22d-10df4575e673.png)
