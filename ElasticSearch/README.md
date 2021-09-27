# Elastic Search
[ScriptedUpdates](#ScriptedUpdates)
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


> No matter how many shards we create if there is no proper nodes then soon or later CPU will run out of its space

Replication will kick in only if there are more than one node.

**Configuring Nodes**
Ideal approach to configure nodes is to download elastic search for each adn every node. But if we are in development mode then we can do that in different way.


To configure nodes to the existing cluster, just node name need to be specifed in elasticsearch folder -> config -> elasticsearch.yml file. ***node.name*** need to be configured. For production best practice is we need to specify cluster as well. But if we dont specify by default whatever cluster is available it will get added there.

**Another approach to add node**
```
bin/elasticsearch.bat -Enode.name=node-2 -Epath.data=./node-2/data -Epath.logs=./node-2/logs
```

>Elastic search clusters consist of one or more nodes. Data is stored on shards and shards are stored on nodes.

### Node-Roles

**Master**
- A node may be elected as the cluster's master node
- A master node is responsible for creating and deleting indices, among other
- A node with this role will not automatically become the master node
  - Unless there are no other master-eligible nodes
- May be used for having dedicated master nodes
  - Useful for large clusters

  ```
  node.master: true|false
  ```

**Data**
- Enables a node to store data
- Storing data includes performing queries related to that data such as search queries
- For relatively small clusters this role is almost always enabled
- Useful for having dedicated master nodes
- Used as part of configuring dedicated master node

```
node.data: true|false
```

**Ingest**
- Enables a node to run a pipeline
- Ingest a pipelines are a series of steps (processors) that are performed when indexing documents.
  - processors may manipulate documents e.g. resolving an IP to Iat/Ion
- A simplified version of logstash, directly within Elasticsearch
- Mainly for simple data transformation
- If there are more document to ingest then it is relatively more complicated
- Filebeat is using ingest 
- This role is useful for running dedicated ingest nodes

```
node.ingest: true|false
```

**Machine Learning**
- node.ml identifies  a node as a machine learning node
  - This let the node run machine learning jobs
- xpack.ml.enabled enables or disables the machine learning API for the node
- Useful for running ML jobs that don't affect other tasks


```
node.ml: true|false
xpack.ml.enabled: true|false
```

**Coordination nodes**
- Coordination refers to the distribution of queries and the aggregation of results
- Useful for coordination-only nodes (for large clusters)
- Configured by disabling all other roles

```
node.master : false
node.data : false
node.ingest: false
node.ml: false
xpack.ml.enabled: false
```

**Voting-only node**
- A nodewith this role will participate in the voting of new master node.
- The node cannot be elected as the master node itself, though.
- The node cannot be elected as the master node itself, though.
- Only used for large clusters

```
node.voting_only : true|false
```


![Node Roles](noderoles.png)

In the above screenshot ***20T3S1GPC22SBS3*** is master

And also in master column we can see '*' symbol to mention, that particular node is master node.

Modifying the roles of node depends on Large Clusters
Usually we will be chaning other things like 
  - No of nodes
  - No of Shards
  - No of Replica Shards

>Only change node if you know what you are doing :smiley:

**Creating and Deleting Indices**

To create the new index
```
PUT /products
```
To delete existing index
```
DELETE /products
```
To configure number of shards and replicas while creating index
```
PUT /products
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 2
  }
}
```
>auto_create_index:true, will auto create index if the index is not present while sending request. But best practice is creating it explicitly.

To add document to index

```
POST /products/_doc
{
  "name":"Coffee Maker",
  "price":64,
  "in_stock":10
}
```
To update document into index
```
PUT /products/_doc/100
{
  "name":"Toaster",
  "price":49,
  "in_stock":4
}
```

To get document detail, should be based on Id
```
GET /products/_doc/100
```
For the above command you need to know the id.

![Product Detail](get-product.png)

To update a document based on document id
```
POST /products/_update/100
{
  "doc":{
    "in_stock":3
  }
}
```
> During post inside *doc* if we pass any field which is doesnt not exist already then that will be added as new field

```
POST /products/_update/100
{
  "doc":{
    "tags":["electronics"]
  }
}
```

**Documents are immutable**
- Elastic search documents are immutable
- We actually didn't modify the document rather we replaced the document with new values
- The Update API looked like something got updated but actually behind the scene it has created a new document and replaced the entire document with old document


**<a name="ScriptedUpdates"></a>Scripted Updates**
Scripted update is having multiple lines of code in single script.
It is not necessary to remember each and every value, everything can be updated.

```
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock--"
  }
}
```
Here in the above code we are trying to get the value of *in_stock* and then we are trying to decrement it.

```
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock = 10"
  }
}
```
Th above query will set the value to 10.
If you wantt to pass parameter to the script then *params* keyword should be used

```
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock -= params.quantity",
    "params": {
      "quantity":4
    }
  }
}
```

Here *quantity* is a parameter. So to access parameter inside the query *params.quantity* is the keyword to be used.

If you dont want certain operation to happen then we can use

```
POST /products/_update/100
{
  "script": {
    "source": """
    if(ctx._source.in_stock==0){
        ctx.op='noop';
    }
    ctx._source.in_stock--;
    """
  }
}
```
*noop* is nothing but no operation. That need to be assigned to contxt operation which is *ctx.op*. There is another operation as well which is *delete*. Delete will delete the document itself

**Upserts**
Update and Insert combination. If the document already exist then it will get updated. If the document is not found then it will create new document.


```
POST /products/_update/101
{
  "script": {
    "source": "ctx._source.in_stock++"   
  },
  "upsert":{
    "name":"Blender",
    "price":399,
    "in_stock":5
  }
}

```
In the above query, *script* part will execute if the document is present, and *upsert* part will execute if the document is not present.
