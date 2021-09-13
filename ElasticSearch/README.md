#Elastic Search
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
