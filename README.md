| Elasticsearch | RDBMS |
| --- | --- |
| Cluster | Database |
| Shard | Shard |
| Index | Table |
| Field | Column |
| Document | Row |

### Create index
```
curl -X PUT "localhost:9200/users"
```
### Delete Index
```
curl -X DELETE "localhost:9200/users"
```

### Create index and add document
```
curl -X POST "localhost:9200/users/_doc/1" -H 'Content-Type: application/json' -d'
{
    "firstName": "Ali",
    "lastName": "Veli",
    "email": "ali.veli@gmail.com",
    "age": 32,
    "city": "Konya",
    "roles": [
        "role-1",
        "role-2"
    ]
}
'
```

### Bulk create
Performs multiple indexing or delete operations in a single API call. This reduces overhead and can greatly increase indexing speed.
> requests.txt
```
'{ "index" : { "_index" : "users", "_id" : "1" } }
{ "firstName" : "Ali", "lastName" : "Yılmaz", "email" : "ali@mail.com", "age" : 22, "city" : "Ankara", "roles" : ["role-1"]}
{ "index" : { "_index" : "users", "_id" : "2" } }
{ "firstName" : "Veli", "lastName" : "Öztürk", "email" : "veli@mail.com", "age" : 26, "city" : "İstanbul", "roles" : ["role-1","role-2"]}
{ "index" : { "_index" : "users", "_id" : "3" } }
{ "firstName" : "Mustafa", "lastName" : "Gezer", "email" : "mustafa@mail.com", "age" : 30, "city" : "İzmir", "roles" : ["role-3","role-1"]}
{ "index" : { "_index" : "users", "_id" : "4" } }
{ "firstName" : "Fatih", "lastName" : "Tekin", "email" : "fatih@mail.com", "age" : 28, "city" : "Ankara", "roles" : ["role-4"]}
{ "index" : { "_index" : "users", "_id" : "5" } }
{ "firstName" : "Mehmet", "lastName" : "Aytekin", "email" : "mehmet@mail.com", "age" : 38, "city" : "Antalya", "roles" : ["role-2","role-5"]}'
```
```
curl -XPOST localhost:9200/_bulk -s -H "Content-Type: application/x-ndjson" --data-binary "@requests.txt";
```

### Get document
```
curl -X GET "localhost:9200/users/_doc/1"
```

### Update document
```
curl -X POST "localhost:9200/users/_update/1" -H 'Content-Type: application/json' -d'
{
    "doc": {
        "age": 33
    }
}'
```
### Delete Document
Removes a JSON document from the specified index.
#### 1. Delete single document using document id
```
curl -X DELETE "localhost:9200/users/_doc/1"
```
#### 2. Delete all documents from the index
```
curl -X POST "localhost:9200/users/_delete_by_query?conflicts=proceed"
{
    "query": {
        "match_all": {}
    }
}
```
#### 3. Delete documents based on the query or specific criteria
```
curl -X POST "localhost:9200/users/_delete_by_query"
{
    "query": {
        "range": {
            "id": {
                "gte": 1
            }
        }
    }
}
```
```
curl -X POST "localhost:9200/users/_delete_by_query"
{
    "query": {
        "match": {
            "firstName": "Ali"
        }
    }
}
```


## Mapping
Mapping is the process of defining how a document, and the fields it contains, are stored and indexed.
### Dynamic mapping
When Elasticsearch detects a new field in a document, it dynamically adds the field to the type mapping by default. The dynamic parameter controls this behavior.
### Explicit mapping
Explicit mapping allows you to precisely choose how to define the mapping definition.
### 1. Create an index with an explicit mapping
```
curl -X PUT "localhost:9200/school" -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "properties": {
      "age":    { "type": "integer" },  
      "email":  { "type": "keyword"  }, 
      "name":   { "type": "text"  }     
    }
  }
}'
```
### 2. Add a field to an existing mapping
```
curl -X PUT "localhost:9200/school/_mapping" -H 'Content-Type: application/json' -d'
{
  "properties": {
    "employee-id": {
      "type": "keyword",
      "index": false
    }
  }
}'
```
### 3. View the mapping of an indexedit
```
curl -X GET "localhost:9200/users/_mapping"
```


### DSL Query
Elasticsearch provides a full Query DSL (Domain Specific Language) based on JSON to define queries. 
### 1. Match query
Returns documents that match a provided text, number, date or boolean value. The provided text is analyzed before matching.
```
curl -X GET "localhost:9200/school/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {
      "firstName": "ali"
    }
  }
}'
```

### 2. Term query
Returns documents that contain an exact term in a provided field.
```
curl -X GET "localhost:9200/school/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "term": {
      "firstName": "Ali"
    }
  }
}'
```

### 3. Range query
Returns documents that contain terms within a provided range.
```
curl -X GET "localhost:9200/school/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "range": {
      "age": {
        "gte": 30,
        "lte": 40
      }
    }
  }
}'
```

### 4. Query string query
Returns documents based on a provided query string, using a parser with a strict syntax.

```
curl -X GET "localhost:9200/school/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "query_string": {
      "query": "(ali) OR (veli)",
      "fields": [ "firstName"]
    }
  }
}'
```

### 5. Match all query
The most simple query, which matches all documents, giving them all a _score of 1.0.
```
curl -X GET "localhost:9200/school/_search" -H 'Content-Type: application/json' -d'
{
    "query": {
        "match_all": {}
    }
}'
```

### 6. Compound queries
Compound queries wrap other compound or leaf queries, either to combine their results and scores, to change their behaviour, or to switch from query to filter context.
```
curl -X GET "localhost:9200/school/_search" -H 'Content-Type: application/json' -d'
{
    "query": {
        "bool": {
            "must": {
                "term": { "firsname": "ali" }
            },
            "filter": {
                "term": { "age": 32 }
            },
            "should": [
                { "term": { "city": "ankara" } },
                { "term": { "city": "konya" } }
            ],
            "minimum_should_match": 1
        }
    }
}'
```
### 6. IDs queries
Returns documents based on their IDs. This query uses document IDs stored in the _id field.
```
curl -X GET "localhost:9200/school/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "ids" : {
      "values" : ["1", "4", "5"]
    }
  }
}'
```

### 7. Prefix query
Returns documents that contain a specific prefix in a provided field.
```
curl -X GET "localhost:9200/school/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "prefix" : { "firstName" : "Me" }
    }
  }
}'
```

### 8. Wildcard query
Returns documents that contain terms matching a wildcard pattern.
```
curl -X GET "localhost:9200/school/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "wildcard": {
      "firstName": { "value": "Me*m" }
    }
  }
}'
```

### 9. Fuzzy query
Returns documents that contain terms similar to the search term.
```
curl -X GET "localhost:9200/school/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "fuzzy": {
      "firstName": { "value": "Mem", "fuzziness": "AUTO", "transpositions": true }
    }
  }
}'
```

### Paginate search resultsedit
By default, searches return the top 10 matching hits. To page through a larger set of results, you can use the search API's from and size parameters
```
curl -X GET "localhost:9200/school/_search" -H 'Content-Type: application/json' -d'
{
  "from": 5,
  "size": 20,
  "query": {
    "match": {
      "match_all": {}
    }
  }
}'
```

### Sort search results
Allows you to add one or more sorts on specific fields. Each sort can be reversed as well.
```
curl -X GET "localhost:9200/school/_search" -H 'Content-Type: application/json' -d'
{
   "query" : {
      "term" : { "firstName" : "ali" }
   },
   "sort" : [
      {"age" : {"order" : "asc"} }
   ]
}'
```

### Aggregations
An aggregation summarizes your data as metrics, statistics, or other analytics. Aggregations help you answer questions like:

- What’s the average load time for my website?
- Who are my most valuable customers based on transaction volume?
- What would be considered a large file on my network?
- How many products are in each product category?
```
curl -X GET "localhost:9200/school/_search" -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "my-agg-name": {
      "avg": {
        "field": "age", "missing": 0
      }
    }
  }
}'
```

