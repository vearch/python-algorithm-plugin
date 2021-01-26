# Quickstart

*This  Quickstart suit for those who want to search something but do not know how to extract image or text to features. Others refer to [APILowLevel.md](APILowLevel.md) .* 

Vearch is aimed to build a simple and fast image retrieval system. Through this system, a image retrieval system could be easily built, including image object detection,  feature extraction and similarity search. This quickstart demonstrates how to use it.

![images/main_process.gif](images/main_process.gif)



## Before you begin

1. Deploy Vearch system referred to [Deploy.md](https://github.com/vearch/vearch/blob/master/docs/Deploy.md).

 For testing you can download  [coco data](https://pjreddie.com/media/files/val2014.zip), or  use the images in images folder we choose from [coco data](https://pjreddie.com/media/files/val2014.zip). For more details, you can refer test folder in `plugin.src`

## Different from APILowLevel.md

This API is similar to [APILowLevel.md](https://github.com/vearch/vearch/blob/master/docs/APILowLevel.md),  and plugin can perfectly adapt to it, Any method defined in APILowLevel.md can be used by plugin. However, if vector has existed, the APILowLevel.md is the fisrt choice.

The difference:

- The name of db can not be one of  ['_cluster', 'list', 'db', 'space'].
- Can not use `_msearch` method.
- Replace the feature field with the object requiring the feature, refer to insert or search demo.


## Deploy a plugin service

This requires only two operations:

1. Modify parameters in `src/config.py`;
2. Execution script:
    For image, `bash ./bin/run.sh image` ;
    For video, `bash ./bin/run.sh video`;
    For text, `bash ./bin/run.sh text` ;


## Create a database and space

Before inserting and searching, a database and space should be creating firstly. Use the following `curl` command to create a new database and space.

```shell
# create a db which name test
curl -XPUT -H "content-type:application/json" -d '{"name": "test"}' http://127.0.0.1:4101/db/_create

# create a space in test db which name test too.
## for image
curl -XPUT -H "content-type: application/json" -d '{"name": "test", "partition_num": 1, "replica_num": 1, "engine": {"name": "gamma", "index_size": 70000, "max_size": 10000000, "id_type": "String", "retrieval_type": "IVFPQ", "retrieval_param": { "metric_type": "InnerProduct", "ncentroids": 256, "nsubvector": 32  } }, "properties": {  "url": { "type": "keyword", "index": true }, "feature1": { "type": "vector", "dimension":512, "format": "normalization"  } }}  }' http://127.0.0.1:4101/space/test/_create

## for text dimension is 768
curl -XPUT -H "content-type: application/json" -d '{"name": "test", "partition_num": 1, "replica_num": 1, "engine": {"name": "gamma", "index_size": 70000, "max_size": 10000000, "id_type": "String", "retrieval_type": "IVFPQ", "retrieval_param": { "metric_type": "InnerProduct", "ncentroids": 256, "nsubvector": 32  } }, "properties": {  "text": { "type": "keyword", "index": true }, "feature1": { "type": "vector", "dimension":768, "format": "normalization"  } }}  }' http://127.0.0.1:4101/space/test/_create
```

A successful response looks like this:

```shell
# create db
{"code":200,"msg":"success","data":{"id":1,"name":"test"}}

# create space
{"code":200,"msg":"success","data":{"id":2,"name":"test","version":2,"db_id":1,"enabled":true,"partitions":[{"id":3,"space_id":2,"db_id":1,"partition_slot":0,"replicas":[1]},{"id":4,"space_id":2,"db_id":1,"partition_slot":2147483647,"replicas":[1]}],"partition_num":2,"replica_num":1,"properties":{ "url": { "type": "keyword", "index":true}, "feature1": { "type": "vector", "dimension":512, "format": "normalization" }},"engine":{"name":"gamma","index_size":100000,"max_size":100000,"nprobe":-1,"metric_type":"InnerProduct","ncentroids":-1,"nsubvector":-1,"nbits_per_idx":-1}}}
```



## Delete a database and space

If the database and space is not need again, Use the following `curl` command to delete a database and space.

```shell

curl -XDELETE http://127.0.0.1:4101/space/test/test
curl -XDELETE http://127.0.0.1:4101/db/test
```

A successful response looks like this:

```shell
{"code":200,"msg":"success"}
```



## Insert data into space

Use the following `curl` command to insert single data into space.

The method of single import demo:

```shell
# single insert

## image
curl -XPOST -H "content-type: application/json"  -d' { "url": "../images/COCO_val2014_000000123599.jpg", "feature1":{"feature":"../images/COCO_val2014_000000123599.jpg"}} ' http://127.0.0.1:4101/test/test/AW63W9I4JG6WicwQX_RC

## text
curl -XPOST -H "content-type: application/json"  -d' { "text": "感谢大家", "feature1":{"feature":"感谢大家"} } ' http://127.0.0.1:4101/test/test/AW63W9I4JG6WicwQX_RC'
```

A successful response like this:

```shell
{"_index":"test","_type":"test","_id":"AW63W9I4JG6WicwQX_RC","status":201,"_version":1,"_shards":{"total":0,"successful":1,"failed":0},"result":"created","_seq_no":1,"_primary_term":1}
```

## Get record by ID

Use the following `curl` command to get a record by ID

```shell
# request
curl -XGET http://127.0.0.1:4101/test/test/AW63W9I4JG6WicwQX_RC

# response
{"_index":"test","_type":"test","_id":"AW63W9I4JG6WicwQX_RC","found":true,"_version":1,"_source":{"url":"../images/COCO_val2014_000000123599.jpg"}}
```



## Delete record by ID

Use the following `curl` command to delete a record by ID

```shell
# request
curl -XDELETE http://127.0.0.1:4101/test/test/AWz2IFBSJG6WicwQVTog

# response
{"_index":"test","_type":"test","_id":"AW63W9I4JG6WicwQX_RC","status":200,"_version":0,"_shards":{"total":0,"successful":1,"failed":0},"result":"unknow","_seq_no":1,"_primary_term":1}
```




## Search similar result from space

Searching by an image URI for an publicly accessible online image or an image stored in images folders.

Searching by an image stored in images folders or image URI on Internet, using the following `curl` command to search  similar result from space

```shell
curl -H "content-type: application/json" -XPOST -d '{ "query": { "sum": [{"feature":"../images/COCO_val2014_000000123599.jpg", "field":"feature1"}]}}' http://127.0.0.1:4101/test/test/_search

```
**Note**：If no index is created, you need add a parameter in request to enforce brute force search rather than index search, eg:
```shell
curl -H "content-type: application/json" -XPOST -d '{ "query": { "sum": [{"feature":"../images/COCO_val2014_000000123599.jpg", "field":"feature1"}]},"is_brute_search":1}' http://127.0.0.1:4101/test/test/_search

```

A successful response looks like this:

```shell
{"took":14,"timed_out":false,"_shards":{"total":2,"failed":0,"successful":2},"hits":{"total":1,"max_score":0.9999997615814209,"hits":[{"_index":"test","_type":"test","_id":"AW8OftTLJG6WicwQyAt2","_score":0.9999997615814209,"_extra":{"vector_result":[{"field":"feature1","source":"","score":0.9999997615814209}]},"_version":1,"_source":{"url":"../images/COCO_val2014_000000123599.jpg"}}]}}
```

search result look like this

![images/COCO_val2014_000000123599.jpg](images/COCO_val2014_000000123599.jpg)

![images/result.jpg](images/result.jpg)


# Docker

* build: `docker build . -t vearch/plugin:latest`
  * add `-f Dockerfile.NonChina` to not use the Chinese docker index
* run: `docker run -p 4101:4101 vearch/plugin`
