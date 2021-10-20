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

**Replacing the documents**
Replacing document is as easy as just replacing the verb from *POST* to *PUT*
```
PUT /products/_doc/100
{
  "script":{
    "name":"Toaster_New",
    "price":79,
    "in_stock":20
  }
}
```

**Delete the document**
Deleting the document is as easy as changing the verb to DELETE

```
DELETE /products/_doc/101
```

**Understanding Routing**
- How does elastic search knows which Shard the data needs to be stored?
- How does elastic search knows which Shard the data needs to be extracted from?

Routing is the answer for above questions. Routing is the process of resolving a shard for the document.

When we index a document elastic search uses a simple formula to select Shard to place data.

>shard_num= hash(_routing) % num_primary_shards

Because of the above formula we will not be able to change the number of Shards once a document is indexed. If number of Shards changes then based on above forumla even the location where it resides also change will result in different shard number.

*It is possible to customize routing.*

**How elastic search reads data?**
Whenever a get request is raised the request will be received by Node which coordination and get the data for requestor.
A node which coordination is called Coordination node.
Refer the image below.

![ES_Data_Read](es_data_read.png)

When coordination node picks document from the Shards, it will try to evaluate which copy of document is health among replica shards
**How elastic search writes data?**
Request goes to coordination node and then it will be passed to Primary Shards as shown below.

![ES_Writes_Data](es_writes_data.png)

Once Primary shard has validated the request then same document will be replicated in Replication shards.

This operation will succeed even if the replication is failed once it is successful by Primary Shards.

Elastic search is distributed, so every document when it is indexed locally then it replicates the same index to Replica Shards as well. During copying of index to Replica Shards if Primary Shard goes down then it will Elastic search will go to recovery mode.

>In recovery mode if the Primary Shard is down then one of the Replica Shard will be promoted as Primary Shard.

**Primary terms and Sequenece Number**
- A way to distinguish between old and new prmiary shards
- Essentially a counter for how many times the primary shard has changed
- The primary term is appended to write operations

**Recovery when a primary shard fails**
- Primary terms and sequence numbers are the key when elastic search needs to recover from a primary shard failure
  - Enables Elastic Search to more efficiently figure out which write operations need to be applied
- For large indices, this process is really expensive.

**Global and local checkpoints**
- Essentially Sequence Numbers
- Each replication group has a global checkpoint
- Each replica shard has a local checkpoint
- Global Checkpoints 
  - The Sequence numbers that all active shards within a replication group has been aligned at least up to
- Local Checkpoints
  - The sequence number of the last write operation that was performed.

**Understanding document versioning**
Elastic search keeps the versioning of the document. 
- Not a version history of document but just the last operation
- Elastic search stores an _version metadata field with every document
  - The value is an integer
  - It is incremented by one when modifying a document
  - The value is retained for 60 seconds when deleting a document
    - Configured with the index.gc_deletes settings
  - The _version field is returned when retrieving document
 ![ES_Document_Versioning](es_document_versioning.png)
  
**Types of versioning**
- Default Versioning - Internal Versioning
- External Versioning Type
  - Useful when versions are maintained outside of Elastic search
  - E.g. when documents are all stored in RDBMS
 ![ES_Version_Type](es_version_type.png)

**Use of Versioning?**
- You can tell how many times the document is modified
- It was previously a way to optimise the concurrency control

**Optimistic concurrency control**
- Prevent overwriting documents inadvertently due to concurrent operations
- There are many scenarios in which that can happen
- If two customers checksout the same product at a same time then *in_stock* field need to be updated for the second person.
- We always make sure the updated document is fetched.
- This is where versioning comes in to picture
![ES_Version_Use](es_version_use.png)

The old approach is to use *_version* parameter along with the query parameter. In this case we can easily identify if the version is not matching with the one which is present in source system then it will throw error.

It works well in usual cases but there are few flaws. That is why in ES we are using two fields. 
- Primary Terms
- Sequence No's

```
{
  ...
  "_primary_term":1,
  "_seq_no":71
}
```
**How do we handle failures?**
- Sending write requests to Elasticsearch concurrently may overwrite changes made by other concurrent process
- Traditionally, _the version field was used to prevent this
- Today we use the _primary_term and _seq_no fields
- Elastic search will reject write operation if it contains the wrong primary term or sequence number
  - This should be handled at application level
  ```
  POST /products/_update/102?if_primary_term=5&if_seq_no=17
  {
    "script": {
      "source": "ctx._source.in_stock = 101"
    }
  }
  ```

  if we try to update the same thing again with the same primary term and *Seq_no* then following error will occur because when it was updated first time *seq_no* would have changed.
  >{
  "error" : {
    "root_cause" : [
      {
        "type" : "version_conflict_engine_exception",
        "reason" : "[102]: version conflict, required seqNo [17], primary term [5]. current document has seqNo [18] and primary term [5]",
        "index_uuid" : "xBHWarPPSeGxRhjBUvj52w",
        "shard" : "0",
        "index" : "products"
      }
    ],
    "type" : "version_conflict_engine_exception",
    "reason" : "[102]: version conflict, required seqNo [17], primary term [5]. current document has seqNo [18] and primary term [5]",
    "index_uuid" : "xBHWarPPSeGxRhjBUvj52w",
    "shard" : "0",
    "index" : "products"
  },
  "status" : 409
}

**Update by Query**
So far we have been updating single document with the help of Id. But now we are going to look how to update multiple documents.
All queries are based on three concepts
- Primary Terms
- Sequenece Numbers
- Optimistic concurrency control

```
POST /products/_update_by_query
{
  "script": {
    "source": "ctx._source.in_stock--"
  },
  "query": {
    "match_all": {}
  }
}
```
![ES_Update_Query](es_update_query.png)
Whenever a query is sent for modiying document following process takes place
- A snapshot of index will be taken first
- All bulk update will be run against each and every Indexes one by one
- Incase if any query failed in any shards then it will not roll back entire update query
  - Thats why we have Optimistic Concurrency control
  - Primary Terms and Sequence Number will come in to picture
- Snapshot which taken initially will be referred, if there is any Shards where this numbers are different then it will not update those shards. Only matching will be updated during retrying..
- Every query will be retried 10 times before Aborting

**Delete by query**
Deleting multiple items based on pattern. *As of now the pattern is just everything*

```
POST /products/_delete_by_query
{
  "query": {
    "match_all": {}
  }
}
```

**Batch Processing**
Batch processing is insertng, updating, deleting multiple documents at a time, that is done with the bulk API.

Bulk API expects the format **NDJSON**.

```
POST /_bulk
{ "index": {"_index": "products", "_id":200} }
{ "name": "Apple Airpord", "price": 8999, "in_stock":2000 }
{ "create": {"_index": "products", "_id":201} }
{ "name": "Apple iwatch", "price": 13860, "in_stock":5000 }
```
Here difference between *index* and *create* is during *index* if record found with that id, document will be replaced or else it will create. Whereas in *create* if document exists it will throw error

```
POST /_bulk
{ "update": {"_index":"products","_id":201} }
{ "doc": {"price":14999} }
{ "delete": {"_index":"products", "_id":200} }
```
> For Delete we dont have to enter second line

The same query can be modified like below if we are targeting all these bulk queries only on one single index then below query can be used

```
POST /products/_bulk
{ "update": {"_id":201} }
{ "doc": {"price":14999} }
{ "delete": {"_id":200} }
```


**Things to be aware of when using Bulk API**
- The HTTP Content-Type header should be set as follows
  - Content-Type: application /x-ndjson
  - application/json is accepted but thats not the correct way
- The console tool handles this for us
  - The Elasticsearch SDKs also handles this for us
  - Using HTTP clients, we need ot handle this by ourselves
- Each line must end with a newline character (\n or \r\n)
  - This includes the last line
    - In a text editor, this means that the last line should be empty
  - Automatically handled with the Console tool
  - Typically a script will generate bulk file, in which case you need to handle this
- A failed action will not affect other actions
  - Neither will the bulk request as a whole be aborted
- The bulk API returns details information about each action
  - Insepect each item by *items* key to see if a given action succeed 
    - The order is the same as the actions with the request
  - The errors key conveniently tells us if any error occured.

**When should we use Bulk API**
- When you need to perform lots of write operations at the same time
  - E.g when importing data or modifying lots of data
- The Bulk API is more efficient than sending individual write requests.
  - A lot of round trip are avoided
- Routing is used to resolve a document's shard
  - The routing can be customized if necessary
The Bulk API supports optimistic concurrency control
  - Include the if_primary_term and if_seq_no parameters within the action metadata.

### Mapping and Analysis
Text analysis is analyzing text while indexing document. The result is stored in data structure that are efficient for searching.
The *_source* object cannot be used for searching document because which contains raw data.
**Analyzer** helps in analyzing the text value and used to store in searchable data structure
  - Character Filters
  - Token Filters
  - Tokenizer
  
**Character Filters**
- Adds, removes or change characters
- Analyzer contain zero or more character filters
- Character filters are specified in orders which they are specified
- Example *html_strip* filter
  - Input: "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp; and I <strong>love</strong> you!"
  - Output: I'm in a good mood - and i love you!

**Tokenizer**
An analyzer contain atleast one Tokenizer. Tokenizer splits string in to tokens.
Example:
**Input:** "I REALLY like beer!"
**Output:** ["I","REALLY","like","beer"]

**Token Filters**
-  Receive the output of the tokenizer as a input (i.e.tokens)
- A token filter can add remove or modify tokens
- An analyzer contains zero or more token filters
- Token filters are applied in the order in which they are specified
- Example (lowercase filter)
  - **Input:** ["I","REALLY","like","the","beer"]
  - **Output:** ["I","really","like","the","beer"]


There are lot of filtes and tokenizers available. If we want even we can build custom filters as well

**Using the analyze API**
Below is the API we can use it to check
```
POST /_analyze
POST /_analyze
{
  "text": ["This is just a simple,Text"],
  "analyzer": "standard"
}

```
Here the analyzer used is *standard*. We can even put some other analyzer if needed.

The algorithm is not splitting string based on white space, actually it does more than that.


The below query does similar to the above one

```
POST /_analyze
{
  "text": ["This is just a simple,TEXT"],
  "char_filter": [],
  "tokenizer": "standard",
  "filter": ["lowercase"]
}
```

**Understanding inverted indices**
This is created and maintained by Apache Lucene and not Elastic Search.
- The datastructure depends on fields datatype
- Ensures efficient data access - E.g. Searches
- Elasticsearch makes sure that more than one data structure used because of that ES is able to provide better search.
  - For Eg, Searching for given term is handled differently than Aggregating data
- Inverted index is the essentially the mapping between terms
  - Terms means token that we created with Analyzer
  - Token terminology is used in Analyzer
![ES_Inverted_Index](es_inverted_index.png)
- ES also has relevance score stored for Inverted indices
- For every text field inverted indices will be created.
![ES_Document_Text_Index](es_document_text_index.png)
- Only text fields are in this format. If it is numeric, date or special fields then it is stored in BKD trees datastructure.

**Introduction to Mapping**
Mapping defines the structure of document and how they are indexed and stored. 
- Also used to configure how values are indexed.
- Similar to a table's schema in a relational database

Basic elastic search datatypes
- **object** - Any json object. Objects can be nested. *Properties* are used in place of types for other datatypes. Apache Lucene does not supportobject. Objects are transformed into compatible json for indexing.
- **boolean** - 
- **text**
- **integer**
- **long**
- **date**
- **float**
- **short**
- **double**
- **ip**

**Nested DataTypes** are nothing but placing the related object inside main json object.
``` json
{
  "name":"Coffee Maker",
  "reviews":[
    {
      "rating": 5.0,
      "author": "Average Joe",
      "description":"Haven't slept for days .. Amazing"
    },
    {
      "rating": 3.5,
      "author": "John doe",
      "description":"Could be better :("
    }
  ]
}
```

Here nested objects are stored as *Hidden documents*. These documents wont show up in query. Each adn every nested objects are stored in hidden document format. And there will also be main documents where these hidden documents are refferred

**Keyword** datatype is used for exact matching. Since this is used for exact mathing this datatype is used for Aggregation, Sorting and Filtering.

E.g Searching for articles with a status of PUBLISHED.

> If you want to perform FULL TEXT search then *text* data type should be used.

**How Keyword datatype works?**
- *keyword* fields are analyzed with *keyword analyzer*
- The *keyword* analyzer is a no-op analyzer
  - It outputs the unmodified string

In *text* data type each and every word is broken and stored in inverted indices. Whereas in *keyword* data type each and every sentence is considered as one complete token and stored as one index. 

> In *Keyword* data type even symbols will not be removed. It wont convert from uppercase to lowercase. Mainly used for searching email address, product numbers etc.

**Understanding type coercion**
- Datatypes are inspected while indexing a data
  - They are validated and some invalid values are rejected
  - Trying to index an object for a text field
- Sometimes providing the wrong data type is ok
```
PUT /coercion_test/_doc/1
{
  "price":5.4
}
```
Here the above datattype will default to Float
```
PUT /coercion_test/_doc/1
{
  "price":"5.4"
}
```
In the above query still it will index 5.4 as Float but _source will show it as string.
```
PUT /coercion_test/_doc/1
{
  "price":"5.4m"
}
```
In the above query it will throw error that it is not able to convert in to Float.

> By default it will always try to do coercison if dataype is not supplied. This is called dynamic mapping. The preferred method is to use proper data type so that proper mapping will take place.

**Understanding Arrays**
There is no such thing as arrays in Elastic Search.

Whatever is there inside the array will be concatenated and tokenized

```
POST /_analyze
{
  "text":["This is text1","this is TEXT2"],
  "analyzer": "standard"
}
```

the result will be 
```json
{
  "tokens" : [
    {
      "token" : "this",
      "start_offset" : 0,
      "end_offset" : 4,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "is",
      "start_offset" : 5,
      "end_offset" : 7,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "text1",
      "start_offset" : 8,
      "end_offset" : 13,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "this",
      "start_offset" : 14,
      "end_offset" : 18,
      "type" : "<ALPHANUM>",
      "position" : 3
    },
    {
      "token" : "is",
      "start_offset" : 19,
      "end_offset" : 21,
      "type" : "<ALPHANUM>",
      "position" : 4
    },
    {
      "token" : "text2",
      "start_offset" : 22,
      "end_offset" : 27,
      "type" : "<ALPHANUM>",
      "position" : 5
    }
  ]
}
```

- All of the values within the array should have same data type
- We can actually mix and match datatype only if we have an option to coerce
For Eg:
  - [1,2,3] - Correct
  - [1,"2",3] - Correct
  - [1,2.3,3] - Correct // This will take only first segment and digits beyond decimal will be ignored
  - [1,"two",3] - Incorrect // Will not be possible to Coerce
- Coercion only work if the fields are already mapped
- Nested Array is also possible
  - For Eg: [1,[2,3]] becomes [1,2,3]

> Remember to use the Nested Array only if you need to query the objects independently. If you dont want to query data independently then *object* datatype must be used.

**Adding explicit mapping**
Creating the explicit mapping
```
PUT /reviews
{
  "mappings":{
    "properties": {
      "rating": {"type": "float"},
      "content": {"type":"text"},
      "product_id": {"type": "integer"},
      "author": {
        "properties": {
          "first_name": {"type":"text"},
          "last_name": {"type":"text"},
          "email": {"type":"keyword"}
        }
      }
    }
  }
}
```
Indexing document
```
PUT /reviews/_doc/1
{
  "rating": 5.0,
  "content": "Outstanding work.... Commendable",
  "product_id": 123,
  "author": {
      "first_name": "John",
      "last_name": "Doe",
      "email": {}
  }
}
```

If you see above query we have given object in email which is going to throw following error

>failed to parse field [author.email] of type [keyword] in document with id '1'. Preview of field's value: '{}'

Strict data type

```
PUT /reviews/_doc/1
{
  "rating": 5.0,
  "content": "Outstanding work.... Commendable",
  "product_id": 123,
  "author": {
      "first_name": "John",
      "last_name": "Doe",
      "email": "johndoe123@example.com"
  }
}
```
This will be successfully indexed.

**Retrieving Mapping**
To see the existing mapping

```
GET /reviews/_mapping /*This will give you entire index mapping*/
GET /reviews/_mapping/field/content /*This will get you mapping only for that particular field*/
GET /reviews/_mapping/field/author.first_name /*This will give you the content of the nested object with dot('.') operator*/
```

**Using dot notation in field name**
We can simply use dot notation as well instead of doing nested operation like below, which will yield similar result

```
PUT /reviews_dot_notation
{
  "mappings":{
    "properties": {
      "rating": {"type": "float"},
      "content": {"type":"text"},
      "product_id": {"type": "integer"},
      "author.first_name": {"type":"text"},
      "author.last_name": {"type":"text"},
      "author.email": {"type":"keyword"}
    }
  }
}
```

**Adding mapping to the exisitng mapping**
If we want to add new field mapping to already created index then we can directly give properties instead of mapping like shown below

```
PUT /reviews/_mapping
{
  "properties":{
    "created_at":{
      "type":"date"
    }
  }
}
```