# Elastic Search

**Node** - is an instance of Elastic Search. Each node will store a part of our data. Each node belongs to cluster

**Cluster** - Cluster can have multiple dependant nodes. We can run multiple clusters of Elastic Search.  For Eg: We can have Ecommerce Product cluster and Performance monitoring clusters etc.

![Clusters](cluster.png)

**Documents** Each unit of data is called documents. Documents are JSON objects. Every data we try to index will not just store your source data but also some metadata

![Documents](documents.png)

**Index** Index can contain more than one related documents.

![Index](index.png)
 

 To check the Elastic search by running Kibana
 Kibana -> Dev Tools -> Console

 ```
 GET /_cluster/health
 ```

 To check the basic information like IP and other Elastic search node details
 ```
 GET /_cat/nodes?v
 ```
 ![Node Details](node_details.png)

 To get list of indices
 ```
 GET /_cat/indices?v
 ```

![Indices Details](indices_data.png)


All the above Query DSL were executed in Kibana Console. We can exactly do the same thing using CURL as well.
For the Query DSL
```
GET _search
{
  "query": {
    "match_all": {}
  }
}
```
CURL command is
```bash
curl -XGET "http://localhost:9200/_search" -H 'Content-Type: application/json' -d'{  "query": {    "match_all": {}  }}'
```
We can use any HTTP client to do this work.


###Sharding and Scalability
If we want to store 1TB of data and we have only one node with 500GB storage then its very clear that Elastic search cannot index that. If we add additional node with sufficient capactiy then ES can index that data in both node with the help of **Sharding**. Sharding is the concept of diving the index in to pieces. Each piece is called *Shard*. Sharding is done in index level not on the document level. The main purpose of Shardinig is to scale the data horizontally.
- A Elastic search index may contain more than one Apache Lucene Indices
- A shard has no predefined sizes, it grows as document added to it
- A Shard may store upto two billion documents

The purpose of Sharding
- Mainly to be able to store more documents
- To easier fit large indices on to nodes
- Improved performances
    - Parallelization of queries increases the throiughput of an index

Configuring the number of Shards
- An index contain a single shards by default
- Indices in Elastic search < 7.0.0 were created with 5 Shards
    - This often led to over sharding 
- To increase number of  Shards then there is something called ***SplitAPI***
- To reduce the number of Shards then ***ReduceAPI*** must be used

![Indices Details](indices_data.png)
In the above reference *pri column* mentions **Primary Sharding**. So in this case Kibana and other services are utilising 1 Shards per index. Thats by default.


How many Shards is optimal?
- Many factos are involved
    - Number of nodes and their capacity
    - Number of indices and their size
    - Number of queries
    - If you anticipate millions of documents? Consider adding a couple of shards
    - If only thousands of records then default should be sufficient


### Replication
Replication is a concept of having a copy of node in case if node fails. Elastic search supports replication by native and enabled by default.
- Replication is configured at index level
- Replication works by creating copies of shards, referred to as replica shards
- A shard that has replicated is called *Primary shard*
- A primary shard and its replica shards are referred to as *Replication group*
- Replica Shards are a complete copy of shard
- A replica shard can serve search requests, exactly like a primary shards
- A number of replicas can be configured during the index creation

![ReplicaShards](replica1.png)

Multiple copies of Replica in different node is the ideal way to store REplioca because in case one node is down other will immediately take up work.

**Snapshots** are the backup copy of the index. So we can even restore the snapshots when it is required.

Difference between Replica and Snapshot is Replioca works in live environment, whereas Snapshots are backups

Replica shards is just like any other shards. So in case if we want to increase throughput of the index then the ideal way is to have one primary shards and multiple Replica Shards.


CPU Parallelization increase performance if multiple replica shards are in same node

To create replica shards then following command need to be executed

```
PUT /pages
```
Here *pages* represent the name of the replica shard.

After creating the page status goes to yellow. the reason is because once this node goes down then there is no use of that Replica Shard which we created. To represent that this yellow warning is appearing.

To see below result
```
GET /_cat/indices?v
```

![Indices](indices_data1.png)

In the above command 'v' indicates verbose. If you would have observed Kibana shards the replica Shards are 0 it means that as of now they have ver limited data so it is set to 0. But when we have new node then Kibana Replica node will get auto set to 1. Because of property ***AutoExpandReplicas***. This dynamically changing the values based on number of nodes getting added.

To see all *Shards*
```
GET /_cat/shards?v
```
![Indices](shard_data.png)

In the above image unassigned means there is no node assigned to that Replica Shard. There is no use of having Replica Shard if no nodes are configured.










