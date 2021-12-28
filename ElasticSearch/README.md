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

**How elastic search understand dates**
- Three formates are supported
  - Date without time
  - Date with time
  - Millisecond since the epoc(long)
- UTC timezones assumed if none is specified
- Dates must be formatted according to ISO 8601 specification

**How missing fields are handled**
- All ElasticSearch fields are optional
- Unlike relational DB we dont have to specify *NULL* values
- Some integrity checks needs to be done at application level
- Adding a field mapping does not make a field required
- Search automatically handles missing field


**Overview of mapping parameters**
Important mapping paramemters
- *Format* Parameter - Used to customize Format for date fields
  - Recommended approach is to use default format whenever possible
    - "strict_date_optional_time||epoch_millis"
  - Using Java's DateFormatter syntax
    - Eg: dd/MM/yyyy
  - Using builtin formats
    - Eg: epoch_second
- *Properties* parameter - Defines nested fields for object and nested fields
- *coerce* parameter - Used to enable or disable type coercionof values

**Introduction to doc_values**
- Elastic search makes use of several data structures
  - No single data structure serves all purpose
- Inverted indices are excellent for searching text
  - They don't perform well for many other data access patterns
- "Doc values" is another data structure used by Apache Lucene
  - Optimized for a different access pattern (document -> term)

- Essentially an "uninverted" inverted index
- Used for sorting, aggregations and scripting
- An additional data structures, not a replacement
- Elastic search automatically queries the appropriate data structure

**Disabling doc_values**
- Set the *doc_values* parameter to *false* to save disk space
  - The reason is usually doc_value will help to increase the speed of the processing by storing the desired data structure in to the disk
- Only disable doc values if you wont use aggregation, sorting or scripting
- Particularly useful for large indices, typically not worth it for small ones
- Cannot be changed without reindexing documents into new index.
  - Use with caution and try to anticipate how fields will be queried

```
PUT /sales
{
  "mappings":{
    "properties":{
      "buyer_email":{
        "type":"keyword",
        "doc_values":false
      }
    }
  }
}
```

**Understanding norms parameter**
- Normalization factos used for relevance score
- Often we don't just want filter results but also rank them
  - Eg: Search result on google in page 1 has more relevance than the result in page 5
- Norms can be disabled to save disk space
  - Useful for fields that won't be used for relevance scoring
  - The fields can still be used for filtering and aggregations

```
PUT /sales
{
  "mappings":{
    "properties":{
      "tags":{
        "type":"text",
        "norms":false
      }
    }
  }
}
```
In the above code we dont need norms whereas name field is required

**Understanding index parameter**
- Disabling indexing for a field
- Values are still stored within *_source*
- Useful if you won't use a field for search queries
- Saves disk space and slightly improve indexing throughput
- Often used for time series data
- Fields with indexing disable can still be used for aggregations

```
PUT /sales
{
  "mappings":{
    "properties":{
      "server_id":{
        "type":"integer",
        "index":false
      }
    }
  }
}
```

**Understanding null_value parameter**
- NULL value cannot be indexed or searched
- Use this parameter to replace NULL value to another value(Usually BLANK)
- Only works for explicit NULL values
- The replacement value must be of the same data type as the field
- Does not affect the value stored within _source

```
PUT /sales
{
  "mappings":{
    "properties":{
      "partner_id":{
        "type":"integer",
        "null_value":false
      }
    }
  }
}
```
**Understanding copy_to parameter**
- Used to copy multiple field values into "group fields"
- Simply specify the name of the target field as the value

- Eg: first_name and last_name >> full_name
- Values are copied, not terms/tokens
  - The analyzer of the target field is used for the values
- Target field is not part of _source

```
PUT /sales
{
  "mappings":{
    "properties":{
      "first_name":{
        "type":"text",
        "copy_to":"full_name"
      },
      "last_name":{
        "type":"text",
        "copy_to":"full_name"
      },
      "full_name":{
        "type":"text"
      }

    }
  }
}
```


**Updating existing mappings**
We cannot change existing mapping because if we do that then Elastic Search may end up changing data structure and that leads to reindexing. So the ideal way is the change the mapping and reindex from top to bottom

**Reindexing document with Reindex API**
Reindexing can be done with two ways. 
- First we need to create the new index with desired mapping
- copy all data from old index to new index. Can be done in two ways
  - Write custom program to move data
    - Eg: Python script
  - using in built elastic search feature **_reindex**. Can be done as below

Step 1: 
```
PUT /reviews_new/
{
  "mappings" : {
      "properties" : {
        "author" : {
          "properties" : {
            "email" : {
              "type" : "keyword",
              "ignore_above":256
            },
            "first_name" : {
              "type" : "text"
            },
            "last_name" : {
              "type" : "text"
            }
          }
        },
        "content" : {
          "type" : "text"
        },
        "created_at" : {
          "type" : "date"
        },
        "product_id" : {
          "type" : "keyword"
        },
        "rating" : {
          "type" : "float"
        }
    }
  }
}
```

Step 2:
```
POST /_reindex
{
  "source": {
    "index": "reviews"
  },
  "dest": {
    "index": "reviews_new"
  }
}
```

Step 3: (Optional - To view data in new index)
```
GET /reviews_new/_search
{
  "query": {
    "match_all": {}
  }
}
```

- Changing the data type will not affect how the values are stored in _source 
- Its common to use _source values from search results
  - You would probably expect a string for a keyword field
- We can modify a source value during reindexing

To remove the existing index following query will help

```
POST /_reindex
{
  "source": {
    "index": "reviews"
  },
  "dest": {
    "index": "reviews_new"
  },
  "script": {
    "source": """
    if (ctx._source.product_id !=null){
      ctx._source.product_id = ctx._source.product_id.toString();
    }
    """
  }
}
```

To delete that index

```
POST /reviews_new/_delete_by_query
{
  "query":{
    "match_all":{}
  }
}
```
To get all data from new index review_new

```
POST /reviews_new/_delete_by_query
{
  "query":{
    "match_all":{}
  }
}
```
**Advanced reindexing Queries**
To reindex only subset of source index then following query can be used

```
POST /_reindex
{
  "source": {
    "index": "reviews",
    "query": {
      "range": {
        "rating": {
          "gte": 4.0
        }
      }
    }
  },
  "dest": {
    "index": "reviews_new"
  }
}
```

**Removing fields**
- Field mappings cannot be deleted
- Fields can be left out when indexing document
- Maybe we want to reclaim disk space used by a field
  - Already index values still take up disk space
  - For larger dataset this may be a worthwhile
    - Assuming that we no longer need the values

```
POST /_reindex
{
  "source": {
    "index": "reviews",
    "_source": ["content","created_at", "rating"]
  },
  "dest": {
    "index": "reviews_new"
  }
}
```

only "content","created_at" and "rating" fields will be reindexed. 
If a field need to be renamed while reindexing


```
POST /_reindex
{
  "source": {
    "index": "reviews",
    "_source": ["content","created_at", "rating"]
  },
  "dest": {
    "index": "reviews_new"
  },
  "script": {
    "source": """
    # Rename 'content' field to 'comment' field
    ctx._source.comment = ctx._source.remove("content");
    """
  }
}
```
if you dont want certain data to be indexed based on values

```
POST /_reindex
{
  "source": {
    "index": "reviews"
  },
  "dest": {
    "index": "reviews_new"
  },
  "script": {
    "source": """
    # Dont index values which has rating lesser than 4
    if(ctx._source.rating < 4.0>){
      ctx.op = "noop"; # Can also be set to delete
    }
    """
  }
}
```

> Query parameter is always performance efficient than using ctx.op

> Reindex api creates snapshot before reindexing data.

- Reindex api performns operations in batches
  - Just like the Update by Query or Delete by Query
  - It uses the Scroll API internally.
  - This is how millions of documents can be reindexed efficiently
- Throttling can be configured to limit the performance impact
  - Useful for production clusters

#### Defining field aliases
To rename just a field name without reindexing then following query can be used

- Point to remember is it wil not change the field name. It just add the calling name

```
POST /reviews/_mapping
{
  "properties": {
    "comment": {
      "type": "alias",
      "path": "content"
    }
  }
}
```

Now *Comment* is *content*. But if you think that internally content is changed to comment no.

```
GET /reviews/_search {
  "query": {
    "match": {
      "content": "outstanding"
    }
  }
}
```

```
GET /reviews/_search {
  "query": {
    "match": {
      "comment": "outstanding"
    }
  }
}
```

Both will yield same result. It means that data has not got changed just the calling name got added as one more.

- Field aliases can be updated
- if you want to change that to different field, simply update *path* parameter.
- Similar to field aliases, Elasticserch also supports index aliases.

> Aggregating cannot be run on Text datatype. It can be run only on *Keyword* datatype.

> More than one datatype can also be assigned to the particular field. That will allow us to perform different types of searches.

Following queries will help

```
PUT /multi_field_test
{
  "mappings": {
    "properties": {
      "description": {
        "type": "text"
      },
      "ingredients": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      }
    }
  }
}

POST /multi_field_test/_doc
{
  "description": "This is the test description for first item",
  "ingredients": ["Macroon","Carrot","Peas", "Chilli", "Onion", "Turmeric"]
}

GET /multi_field_test/_search
{
  "query": {
    "match": {
      "ingredients": "macroon"
    }
  }
}

GET /multi_field_test/_search
{
  "query": {
    "term": {
      "ingredients.keyword": "Macroon"
    }
  }
}
```
Here in last two GET first GET has no case sensitivity and the second GET has case sensitivity. In the second GET we are using Keyword part so case sensitivity is important.
Text fields are analyzed and inverted index was populated for the terms that are made by. But for *ingredients* field we are using multiple field type, both text and keyword. So for Text text analyzer will be used and for keyword, keyword analyzer will be used. So two indexes will be created only for that field alone.

**Index Templates**
Index Templates are used to create a settings for indices which matches one or more pattern. Patterns may include wild card patterns (*)

```
PUT /_template/access-logs
{
  "index_patterns": ["access-logs-*"], //This matches pattern
  "settings": {
    "number_of_shards": 2 //Number of shards
  }, 
  "mappings": {
    "properties": {
      "@timestamp": {
        "type": "date"
      },
      "url.original": {
        "type": "keyword"
      },
      "http.request.referrer": {
        "type": "keyword"
      },
      "http.response.status_code": {
        "type": "long"
      }
    }
  }
}
```

With this pattern new index with following name created like access-logs-2021-04, then automatically this settings will be applied.

If we overwrite it with following code. The specific settings will get overridden.

```
PUT /access-logs-2021-11
{
  "settings": {
    "number_of_shards": 3
  }
}
```
If there are duplicates then create index request configuration will take the precedence over the default one.

![Index Tenplate](index_templates.png)

**Elastic common schema**
- A specification of common fields and how they should be mapped.
- Before ECS there was no ohesion between fields names
- Ingesting logs from nginx would give different field names than Apache
- ECS means thatcommon fields are named the same thing
  - E.g. *@timestamp*
- Use-case independent
- Groups of fields are referred to as field sets.

Refer ECS documentation online to see more standard fields.

- In, ECS documents are called as Events
  - ECS doesnt provide fields for non-events(Eg.Products)
  Product is use case specific, whereas ECS is applicable only on generic world
- ECS is useful for standard events
  - WebServer logs
  - Operating System Metrics
  - etc
 - ECS is automatically handled by Elastic search products
  - If you use them you dont have to actively deal with them

**Introduction to Dynamic Mapping**

![Dynamic Mapping](dynamic_mapping.png)

![Dynamic Mapping Rules](dynamic_mapping_rules.png)

**Combining explicit and dynamic mapping**
Combining explicit mapping and dynamic mapping is a best choice. 
```
PUT /people
{
  "mappings":{
    "properties": {
      "first_name": {
        "type": "text"
      }
    }
  }
}

POST /people/_doc
{
  "first_name": "Vinoth",
  "last_name": "Suresh"
}

GET /people/_mapping

DELETE /people
```

The mapping output is as follows
```json
{
  "people" : {
    "mappings" : {
      "properties" : {
        "first_name" : {
          "type" : "text"
        },
        "last_name" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
      }
    }
  }
}
```
**Configuring dynamic mapping**
Dynamic mapping auto creates fields based on data and assigns types which best suits
But if we turn off dynamic mapping like shown below
```
PUT /people
{
  "mappings":{
    "dynamic":false, //Dynamic mapping turned off
    "properties": {
      "first_name": {
        "type": "text"
      }
    }
  }
}
```

Following are the consequenses of turning Dynamic mapping off
- New fields are ignored
  - They are not indexed but still part of _source
- No inverted index is created for the last_name field
  - Queriying field gives no results
- Fields cannot be indexed without mapping
  - When enabled, dynamic mapping creats one before indexing values
- New fields should be mapped explicitly

*Another way*
- ..setting *dynamic* to "strict"
- Elasticsearch will reject unmapped fields
  - All fields must be mapped explicitly
  - Similar to the behavior of relational database

```
PUT /people
{
  "mappings":{
    "dynamic":"strict",
    "properties": {
      "first_name": {
        "type": "text"
      }
    }
  }
}

POST /people/_doc
{
  "first_name": "Vinoth",
  "last_name": "Suresh"
}
```

In case if the above post query executed, then following error occurs
```json
{
  "error": {
    "root_cause": [
      {
        "type": "strict_dynamic_mapping_exception",
        "reason": "mapping set to strict, dynamic introduction of [last_name] within [_doc] is not allowed"
      }
    ],
    "type": "strict_dynamic_mapping_exception",
    "reason": "mapping set to strict, dynamic introduction of [last_name] within [_doc] is not allowed"
  },
  "status": 400
}
```
![Dynamic Mapping Strict VS True](dynamic_mapping_config.png)

![Dynamic Numeric Detection](numeric_detection.png)

![Dynamic Date Detection](date_detection.png)

**Dynamic Templates**

![Matching Types](matching_types.png)


![Dynamic template types example 1](dynamic_template_types_example_1.png)


*match* and *unmatch* parameters
- Used to specify conditions for field names
- Field names must match condition specifiedby the match parameter
- *unmatch* is used to exclude fields that were matched by the *match* parameter
- Both parameters supports patterns with wildcard(*)
![Match Unmatch parameters](match_unmatch_parameter.png)
![Match Unmatch Regular Expression](match_unmatch_regex.png)

- These parameters evaluate full field paths
  - Not just the field names
- This is dot notation that you saw earlier
  - Eg: name.first_name


![Match Unmatch CopyTo](match_unmatch_copyto.png)
![Dynamic Template - Dynamic Type](dynamic_template_dynamic_type.png)

- Index templates apply mappings and index settings for matching indices
  - This happens when indices are created and their namesatch a pattern

**Mapping Recommendations**
- Dynamic mappings is convinent but often not a good idea in production
- Save disk space with optimized mappings when storing many documents
- Set "dynamic" to "strict" not false
  - Avoid surprises and unexpected results
- Don't always map stringsas both *text* and *keyword*
  - Typically only one is needed
  - Each mapping requires disk space
- Do you need to perform full-text searches?
  - Add a text mapping
- Do you need to do aggregations, sorting or filtering on exact values?
  - Add a keyword mapping
- Coercion forgives you for not doing the right thing.
- Try to do the right thing instead
- Always use correct data types whenever possible
- Use appropriate numeric data types
- For whole numbers the integerdata type might be enough
  - long can store large numbers, but also uses more disk space
- For decimal numbers, the floated data type might be precise enough
  - double stores numbers with high precision but uses 2x disk space
  - Usually, float provides enough precision

**Mapping Parameters**
- Set *doc_values* to *false* if you don't need sorting, aggregations and scripting
- Set *norms* to *false* if you don't need relevance scoring.
- Set *index* to *false* if you dont need to filter on values
  - You can still do aggregations, e.g for time series data
- Probably only worth the effort when sorting lots of documents
  - Otherwise its probably an over complication
- Worst case scenario you will need to reindex documents

**Stemming and Stop words**
![Without Stemming](without_stemming.png)

- Stemming is the process of reducing the words to their root form
  - E.g *Loved* -> *love* and *drinking* -> *drink*

What happens if we apply stemming to the whole sentence. Consider following sentence
> I Loved drinking bottles of wine on last year's vacation.

The above stemmed sentences will get converted to 

> I love drink bottle of wine on last year vacation

**Introduction to stop words**
- Words that are filtered our during text analysis
  - Common words such as "a", "the", "at", "of", "on", etc
- They provide little or no value for relevance scoring
- Fairly common to remove such words
  -  Less common in elastic search today than in past
    - The relevance algorithm has been improved significantly
- Not removed by default, and i generally dont recommend doing so




**Analyzers and search queries**
![Stemming analyzer](stemming_analyzer.png)

**Build-in analyzers**
*Standard Analyzer*
- Splittext at word boundaries and remove punctuation
  - Done by the standard tokenizer
- Lowercases letters with the lowercase token filter
- Contains the stop token filter (disabled by default)

![Standard analyzer](standard_analyzer.png)

**Simple analyzer**

- Similar to standard analyzer
  - Splits into tokens when encountering anything else than letters
- Lowercases letters with the lowercase tokenizer
  - Unusual performance hack
![Simple analyzer](simple_analyzer.png)

**Whitespace analyzer**
- Splits text into tokens by whitespace
- Does not lowercase letters

![Whitespace Analyzer](whitespace_analyzer.png)

**Keyword analyzer**
- No-op analyzer that leaves the input text intact
  - It simply outputs it as single token
- Used for keyword fields by default
  - Used for exact matching

![Keyword analyzer](keyword_analyzer.png)

**Pattern Analyzer**
- A regular expression is used to match token seperators
  - It should match whatever should split the text into tokens
- This analyzer is flexible
- the default matches all non word characters (\W+)
- Lowercase letters by default

![Pattern Analyzer](pattern_analyzer.png)

[Built-In Analyzers references](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-analyzers.html)

![Built In Analyzer Example 1](builtin_analyzer_example_1.png)

![Built In Custom Extension](built_in_extend.png)

![Using custom builtin analyzer](custom_builtin_usage.png)


**Custom Analyzer**
# Creating custom analyzers

#### Remove HTML tags and convert HTML entities
```
POST /_analyze
{
  "char_filter": ["html_strip"],
  "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"
}
```

#### Add the `standard` tokenizer
```
POST /_analyze
{
  "char_filter": ["html_strip"],
  "tokenizer": "standard",
  "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"
}
```

#### Add the `lowercase` token filter
```
POST /_analyze
{
  "char_filter": ["html_strip"],
  "tokenizer": "standard",
  "filter": [
    "lowercase"
  ],
  "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"
}
```

#### Add the `stop` token filter

This removes English stop words by default.
```
POST /_analyze
{
  "char_filter": ["html_strip"],
  "tokenizer": "standard",
  "filter": [
    "lowercase",
    "stop"
  ],
  "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"
}
```

#### Add the `asciifolding` token filter

Convert characters to their ASCII equivalent.
```
POST /_analyze
{
  "char_filter": ["html_strip"],
  "tokenizer": "standard",
  "filter": [
    "lowercase",
    "stop",
    "asciifolding"
  ],
  "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"
}
```

#### Create a custom analyzer named `my_custom_analyzer`
```
PUT /analyzer_test
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "char_filter": ["html_strip"],
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "stop",
            "asciifolding"
          ]
        }
      }
    }
  }
}
```

#### Configure the analyzer to remove Danish stop words

To run this query, change the index name to avoid a conflict. Remember to remove the comments. :wink:
```
PUT /analyzer_test
{
  "settings": {
    "analysis": {
      "filter": {
        "danish_stop": {
          "type": "stop",
          "stopwords": "_danish_"
        }
      },
      "char_filter": {
        # Add character filters here
      },
      "tokenizer": {
        # Add tokenizers here
      },
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "char_filter": ["html_strip"],
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "danish_stop",
            "asciifolding"
          ]
        }
      }
    }
  }
}
```

#### Test the custom analyzer
```
POST /analyzer_test/_analyze
{
  "analyzer": "my_custom_analyzer", 
  "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"
}
```

**Adding analyzers to existing indices**
If you try to add analyzer to existing index then following error will appear

```
PUT /analyzer_test/_settings
{
  "analysis": {
    "analyzer": {
      "my_second_analyzer": {
        "type": "custom",
        "tokenizer": "standard",
        "char_filter": ["html_strip"],
        "filter": [
          "lowercase",
          "stop",
          "asciifolding"
        ]
      }
    }
  }
}
```


```json
{
  "error": {
    "root_cause": [
      {
        "type": "illegal_argument_exception",
        "reason": "Can't update non dynamic settings [[index.analysis.analyzer.my_second_analyzer.tokenizer, index.analysis.analyzer.my_second_analyzer.type, index.analysis.analyzer.my_second_analyzer.char_filter, index.analysis.analyzer.my_second_analyzer.filter]] for open indices [[analyzer_test/eN5VeIooQlyawrSELU5P-A]]"
      }
    ],
    "type": "illegal_argument_exception",
    "reason": "Can't update non dynamic settings [[index.analysis.analyzer.my_second_analyzer.tokenizer, index.analysis.analyzer.my_second_analyzer.type, index.analysis.analyzer.my_second_analyzer.char_filter, index.analysis.analyzer.my_second_analyzer.filter]] for open indices [[analyzer_test/eN5VeIooQlyawrSELU5P-A]]"
  },
  "status": 400
}
```


- An open index is available for indexing and searching requests
- A closed index will refuse requests
  - Read and write requests are blocked
  - Necessary for performing operation

```
POST /analyzer_test/_close
```
```
PUT /analyzer_test/_settings
{
  "analysis": {
    "analyzer": {
      "my_second_analyzer": {
        "type": "custom",
        "tokenizer": "standard",
        "char_filter": ["html_strip"],
        "filter": [
          "lowercase",
          "stop",
          "asciifolding"
        ]
      }
    }
  }
}
```

```
POST /analyzer_test/_open
```

*Dynamic and static settings*
- Dynamic settings can be changed withoutclosing the index first
  - Requires no downtime
Static settings requires the index to be closed first
- The index will be briefly unavailable
- `Analyzers settings are static settings`

>When the index is closed it will not be avialable for any search and this results in downtime.

use below query to get details
```
GET /analyzer_test/_settings
```
*Opening and closing index*
- Fairly quick but might not be an option for production clusters
  - Eg Mission critical systems where downtime is acceptable
- Alternatively reindex documents into a new index
  - Create a new index with the updated settings
  - Use an index alias for the transition

**Updating Analyzers**
Sometimes you may have to update an existing analyzer

#### Add `description` mapping using `my_custom_analyzer`
```
PUT /analyzer_test/_mapping
{
  "properties": {
    "description": {
      "type": "text",
      "analyzer": "my_custom_analyzer"
    }
  }
}
```

#### Index a test document
```
POST /analyzer_test/_doc
{
  "description": "Is that Peter's cute-looking dog?"
}
```

#### Search query using `keyword` analyzer
```
GET /analyzer_test/_search
{
  "query": {
    "match": {
      "description": {
        "query": "that",
        "analyzer": "keyword"
      }
    }
  }
}
```

#### Close `analyzer_test` index
```
POST /analyzer_test/_close
```

#### Update `my_custom_analyzer` (remove `stop` token filter)
```
PUT /analyzer_test/_settings
{
  "analysis": {
    "analyzer": {
      "my_custom_analyzer": {
        "type": "custom",
        "tokenizer": "standard",
        "char_filter": ["html_strip"],
        "filter": [
          "lowercase",
          "asciifolding"
        ]
      }
    }
  }
}
```

#### Open `analyzer_test` index
```
POST /analyzer_test/_open
```

#### Retrieve index settings
```
GET /analyzer_test/_settings
```

#### Reindex documents
```
POST /analyzer_test/_update_by_query?conflicts=proceed
```
## Introduction to Searching
We can query through two ways 
- Query DSL
- Request URI

### Searching with the request URI

#### Matching all documents

```
GET /products/_search?q=*
```

#### Matching documents containing the term `Lobster`

```
GET /products/_search?q=name:Lobster
```

#### Matching documents containing the tag `Meat`

```
GET /products/_search?q=tags:Meat
```

#### Matching documents containing the tag `Meat` _and_ name `Tuna`

```
GET /products/_search?q=tags:Meat AND name:Tuna
```

### Introduction to Query DSL

![Compound Query](compound_query.png)

![Shard Process](shard_process.png)

### Understanding relevance score

![Relevance Scores](relevance_scores.png)


Relevance scoring algorithm used are:
- It was using TF/IDF - Term Frequency/Inverse Document Frequency
- Currently it is using Okapi BM25

Term Frequency: How many times does the term appears in the field for a given document

Inverse Document Frequency: How often does the term appear within the index

Field length norm: The longer the field the less likely it has relevance score. The idea is the shorter the term is much easy it is to search.

`BM25` algorithm differs slightly improvised than TF/IDF because BM25 is better in managing Stop words.

Nonlinear Term Frequency Saturation: Will help to normalize the curve. It means that if term occurs more frquently then it plays almost no importance. For Eg: If the word occured 40 times is given very low importance as 1000 times. This will help algorithm to filter out stop words

![NonLinear Term Frequency Saturation](ntfs.png)

Field length norm is also improved in BM25 algorithm
The shorter the field the more significant the term is considered.

```
GET /products/default/_search?explain
{
  "query": {
    "term": {
      "name": "lobster"
    }
  }
}
```
`explain` will help us understand the query result in detail. Checkout the following query

```
GET /products/_doc/1/_explain
{
  "query": {
    "term": {
      "name": "lobster"
    }
  }
}
```
for this following resulted

```json
{
  "_index" : "products",
  "_type" : "_doc",
  "_id" : "1",
  "matched" : false,
  "explanation" : {
    "value" : 0.0,
    "description" : "no matching term",
    "details" : [ ]
  }
}

```

In this `explanation` part it shows matching term is not found. This gives clear meaning why data is not found. If it is complex query it will be very much helpful.

### Query Contexts

Filter context: Filter context are usually used in Dates/Status/ Ranges etc,

Query Context: Affects Relevance. It checks if document matches or not and uses Filter Context if required.


**When to use Query Context/ Filter Context**
If you want to calculate `more relevance` then use `query context`. It uses relevance and try to find most suitable. If you want to find the `values directly` irrelevant to relevance, use `Filter context`. It will be much faster, because it avoids Relevance score calculation

> Filters can be cached

### Full text queries vs term level queries


- Term Queries  will by default look for exact keyword. And in the first screenshot as expected values is small case `lobsters` and the query returns value
![Term Level Query Correct](term_query_1.png)
- Term Queries  will by default look for exact keyword. And in the second screenshot as expected values is small case `lobsters` and the query doesnt returns value
![Term Level Query Wrong](term_query_2.png)
- Match Queries  will analyze and convert to lowercase.  And in the third screenshot as expected values is neither small or capital case `lobsters` and the query will returns value because it will be automatically converted
![Match Level Query Wrong](match_query.png)

- Comparison between Term and Match queries
![Match vs term Query](term_match_query.png)

> Term level queries are useful to search Enum, dates, numbers etc

### Introduction to term level queries
Pretty much useful for status field to check whether its 1 or 0. Not be very useful for searching through descriptions
```
GET /products/_doc/_search
{
  "query": {
    "term": {
      "is_active": true
    }
  }
}
```
**(or)**
```
GET /products/_doc/_search
{
  "query": {
    "term": {
      "is_active": {
        "value": true
      }
    }
  }
}

```

The above query will fetch all status with value true

**Multiple values**
```
GET /products/_doc/_search
{
  "query": {
    "terms": {
      "tags.keyword": [
        "Soup",
        "Cake"
      ]
    }
  }
}
```
This query will return all the matching items which satisfies multiple values.

**Matching documents with range values**
`gte` and `lte` are the important factos for range queries. It doesnt necessary that both values should be present. Either one is also fine

To get all stocks greater than 1 and lesser than 5
```
GET /products/_doc/_search
{
  "query": {
    "range": {
      "in_stock": {
        "gte":1,
        "lte":5
      }
    }
  }
}

```
To get all products which has created greater than 1st jan 2010 and 31st Dec 2021
```
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2010/01/01",
        "lte": "2010/12/31"
      }
    }
  }
}
```

To specify date format `format` will help.

```
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "01-01-2010",
        "lte": "31-12-2010",
        "format": "dd-MM-yyyy"
      }
    }
  }
}
```

**Working with relative dates (date math)**
This will be very much helpful to write equation
For dates it is

|Term  |Notation|
|------|--------|
|Day   |d       |
|Month |M       |
|Year  |y       |
|Week  |w       |

```
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2010/01/01||-1y"
      }
    }
  }
}
```

We should write it with `||` and then expression. In the above query it is mentioned as `-1y` which means it is reduce the date by 1 year which is `01-01-2009`

To subtract 1 year from `2010/01/01` and rounding by month

```
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2010/01/01||-1y/M"
      }
    }
  }
}
```

Rounding by month before subtracting one year from `2010/01/01`

```
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2010/01/01||/M-1y"
      }
    }
  }
}
```

**Rounding by month before subtracting one year from the current date**

```
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "now/M-1y"
      }
    }
  }
}
```

**Matching documents with a `created` field containing the current date or later**

```
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "now"
      }
    }
  }
}
```

### Matching document with non-null values
Just `exists` will help
```
GET /products/_search
{
  "query": {
    "exists": {
      "field": "tags"
    }
  }
}
```
> Empty string means null in Elastic search

### Matching based on prefixes
Matching documents containing a tag beginning with Vege

```
GET /products/_search
{
  "query": {
    "prefix": {
      "tags.keyword": "Vege"
    }
  }
}
```

### Searching with wildcards
Adding an asterisk for any characters (zero or more)

`*` means `zero or more`
`?` means 'single character
 
The below querry returns all the products which contains item in array for tags 
```
GET /products/_search
{
  "query": {
    "wildcard": {
      "tags.keyword": "Veg*ble"
    }
  }
}
```
The below querry returns all the products which contains item in array for tags 
```
GET /products/_search
{
  "query": {
    "wildcard": {
      "tags.keyword": "Veg?ble"
    }
  }
}
```
The above query will not return any values.
Whereas the q query will

```
GET /products/_search
{
  "query": {
    "wildcard": {
      "tags.keyword": "Veget?ble"
    }
  }
}
```
Because only one character is missing and `?` is doing justification.


**Searching with regular expressions**

We can even search with regular expression as well.

```
GET /products/_search
{
  "query": {
    "regexp": {
      "tags.keyword": "Veg[a-zA-Z]+ble"
    }
  }
}
```

### Introduction to Full text Queries

> Elastic search don't have array data type because field may contain one or more type of data.

**Flexible matching with match query**
Match query lookout for relevance score
For eg
```
GET /recipe/_doc/_search
{
  "query":{
    "match": {
      "title": "Recipes with Pasta or Spaghetti"
    }
  }
}
```

In the above query the search text will be broken in to pieces and removes all joining words like `with`, `or`, `and`, `they` etc

So the result will be the one which matches 
`Recipes` or `Pasta` or `Spaghetti`

If we need to include `AND` instead of `OR` then we have to use `operator` attribute

```
GET /recipe/_doc/_search
{
  "query":{
    "match": {
      "title": {
        "query": "Recipes with Pasta or Spaghetti",
        "operator": "AND"
      }
    }
  }
```

**Matching phrases**
Matching exact phrase is what it means

```
GET /recipe/_search
{
  "query": {
    "match_phrase": {
      "title": "spaghetti puttanesca"
    }
  }
}
```

It will pull the result which has `spaghetti puttanesca` in the title. 

> Note: If the order is changed it will not the yield the expected result. Because in match phrase order matters

```
GET /recipe/_search
{
  "query": {
    "match_phrase": {
      "title": "puttanesca spaghetti"
    }
  }
}
```
This will not yield the same result as previous query.

**Searching multiple field**
To search across multiple fields then `multi_match` should be used

```
GET /recipe/_search
{
  "query": {
    "multi_match": {
      "query": "pasta",
      "fields": [ "title", "description" ]
    }
  }
}
```
Relevance score is used to match.:smile:


### Introduction to compound queries
Leaf queries are very simple query where it will search only one query at a time. Which is mostly used in term queries or match queries. Compound queries are used to write advanced queries which contain multiple leaf queries.


**Querying with boolean logic**
Bool queries are like Where clause in SQL but with extra feature of using `Relevance scores`

```
GET /recipe/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        },
        {
          "range": {
            "preparation_time_minutes": {
              "lte": 15
            }
          }
        }
      ]
    }
  }
}
```

In the above query we have bool query context and inside that we are going to provide `must` array. `Match` will get relevance scores and `range` will not.

```
GET /recipe/_search
GET /recipe/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "preparation_time_minutes": {
              "lte": 15
            }
          }
        }
      ]
    }
  }
}
```

Its the same previous query but range is added to filter context rather than query context because of two reasons
- If we place range query in Query context it will return constant score of 1 for the matching range. Whereas if it is in Filter context it will not add any score.
- The query inside filter context can be cached, whereas query inside query context will not be cached.

> Adding filters in filter context will have added edge in performance

to compare the result between range in query context and filter context see the sample scores from both

*Query context*
```json
  "_index" : "recipe",
  "_type" : "_doc",
  "_id" : "1",
  "_score" : 2.3795729,
  "_ignored" : [
    "description.keyword",
    "steps.keyword"
  ],
```

*Filter context*

```json
  "_index" : "recipe",
  "_type" : "_doc",
  "_id" : "1",
  "_score" : 1.3795729,
  "_ignored" : [
    "description.keyword",
    "steps.keyword"
  ],
```
If you observe `_score` key the value is reduced in filter context because there is no score added for range in filter context.

`must_not`
``` json
GET /recipe/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "ingredients.name": "tuna"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "preparation_time_minutes": {
              "lte": 15
            }
          }
        }
      ]
    }
  }
}
```

`must_not` is to tell that dont include results which is matching `tuna`. the highlight is must_not will not include any score though its in query because it is running in filter context through it is query context. So even must_not will be cached.

`should`
Should key is like better to have concept. It will boost the relevance score

```json
GET /recipe/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "ingredients.name": "tuna"
          }
        }
      ],
      "should": [
        {
          "match": {
            "ingredients.name": "parsley"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "preparation_time_minutes": {
              "lte": 15
            }
          }
        }
      ]
    }
  }
}
```
But mainly to boost relevance score only.

>`should` behavior depends on where it is.

For example in the previous query should is like good to have. It was optional in the above query, there is no `must` so `should` will act like `must` - a `mandatory` key

```json
GET /recipe/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        }
      ]
    }
  }
}
```

**Debugging boolean query**
>How to debug queries? 
By using Explain query. 

But here we are going to do with Named Query
```
GET /recipe/_search
{
    "query": {
        "bool": {
          "must": [
            {
              "match": {
                "ingredients.name": {
                  "query": "parmesan",
                  "_name": "parmesan_must"
                }
              }
            }
          ],
          "must_not": [
            {
              "match": {
                "ingredients.name": {
                  "query": "tuna",
                  "_name": "tuna_must_not"
                }
              }
            }
          ],
          "should": [
            {
              "match": {
                "ingredients.name": {
                  "query": "parsley",
                  "_name": "parsley_should"
                }
              }
            }
          ],
          "filter": [
            {
              "range": {
                "preparation_time_minutes": {
                  "lte": 15,
                  "_name": "prep_time_filter"
                }
              }
            }
          ]
        }
    }
}
```
![Debugging boolean query](boolean_debugging_matching_query.png)

This will help us identifies how the matching happened. It is easy to debug why this data has been returned.

**How match query works?**
Match query converts internally to term-bool query after analyzing tohe query with any of the query analyzer specified in match query. 
By default all match query will be `Should` query 
Below two queries will yield same result

```
GET /recipe/_search
{
  "query": {
    "match": {
      "title": "pasta carbonara"
    }
  }
}
```

```
GET /recipe/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "title": "pasta"
          }
        },
        {
          "term": {
            "title": "carbonara"
          }
        }
      ]
    }
  }
}
```

And if we use `AND` operator in match query `should` will be replaced to `must`.
Below two queries will yield same result. Note we have used `AND` operator in Match query.

```
GET /recipe/_search
{
  "query": {
    "match": {
      "title": {
        "query": "pasta carbonara",
        "operator": "and"
      }
    }
  }
}
```

```
GET /recipe/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "title": "pasta"
          }
        },
        {
          "term": {
            "title": "carbonara"
          }
        }
      ]
    }
  }
}
```
Now the important thing to note here is Match query does tokenization and analyzer will help to analyze query. Where as term query will not do that. So how both are yielding same result? Because both are using small case. Even Match query after analysis it will be converted to smaller case.

Below two query will not yield same result.
```
GET /recipe/_search
{
  "query": {
    "match": {
      "title": {
        "query": "Pasta carbonara",
        "operator": "and"
      }
    }
  }
}
```

```
GET /recipe/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "title": "Pasta"
          }
        },
        {
          "term": {
            "title": "carbonara"
          }
        }
      ]
    }
  }
}
```

The reason is because the match query after analysis it will search for `pasta` and `carbonara`. But term query as there is no analysis it will be searched as `Pasta`(Upper case `P`) and `carbonara`, which will yield different result.
