# ES SNAPSHOT commands


## PREPARATION

Specify VPC endpoint for the ES domain
```
VPC_ENDPOINT="http://apps.infra.internal"
```

Specify the name of the ES repo for snapshots
```
   # "cs-automated" - default name for the repo in s3 with automated ES snapshots (we don't have access to this s3 bucket);
   # "<ANY_OTHER_NAME>" - if manually configured, additional ES repo could be found for manual ES snapshots (we create this s3 bucket)

curl ${VPC_ENDPOINT}/_snapshot/ | jq

ES_REPO="cs-automated"
```



## LIST: ES repos for snapshots

List configured ES repos for snapshots
```
   # "cs-automated" - default name for the repo in s3 with automated ES snapshots (we don't have access to this s3 bucket);
   # "<ANY_OTHER_NAME>" - if manually configured, additional ES repo could be found for manual ES snapshots (we create this s3 bucket)

curl ${VPC_ENDPOINT}/_snapshot/ | jq
```



## LIST: ES snapshots

All snapshots for ES domain in specified ES repo
```
curl -X GET ${VPC_ENDPOINT}/_cat/snapshots/${ES_REPO}?v&s=id

      id                                                        status start_epoch start_time end_epoch  end_time duration indices successful_shards failed_shards total_shards
      2020-08-21t11-01-05.4a410281-a3ba-4199-abb8-f8933641b13f PARTIAL 1598007665  11:01:05   1598007701 11:01:41    35.5s     548               526           106          632
      2020-08-21t12-01-13.eb5864a9-58e1-446a-800c-581d6e407f11 PARTIAL 1598011273  12:01:13   1598011310 12:01:50      37s     548               526           106          632

```






## DESCRIBE

Describe specific ES snapshot (`snapshot_1` in this case)
```
SS_NAME_EXISTING="snapshot_1"
curl -XGET ${VPC_ENDPOINT}/_snapshot/${ES_REPO}/${SS_NAME_EXISTING} | jq

   {
     "snapshots": [
       {
         "snapshot": "snapshot_1",
         "uuid": "asdfafadfadsfasdfasdasdf",
         "version_id": 234562,
         "version": "7.4.2",
         "indices": [
           "<index_name_1>",
           "<index_name_2>",
           ...
           "<index_name_N>"
         ],
         "include_global_state": true,
         "state": "SUCCESS",
         "start_time": "2020-08-05T15:11:00.791Z",
         "start_time_in_millis": 1596640260791,
         "end_time": "2020-08-05T15:26:55.875Z",
         "end_time_in_millis": 1596641215875,
         "duration_in_millis": 955084,
         "failures": [],
         "shards": {
           "total": 631,
           "failed": 0,
           "successful": 631
         }
       }
     ]
   }
```


Describe each ES snapshot
```
curl -XGET ${VPC_ENDPOINT}/_snapshot/${ES_REPO}/_all?pretty | jq
```





## STATUS

Check if there's snapshot in process

(because you can't take a new snapshot if one is currently in progress)
```
curl -XGET ${VPC_ENDPOINT}/_snapshot/_status | jq
```




## CREATE: Automated

For AWS managed ES: we don't have ability to configure automatic snapshots.

They are done automatically each 1h.




## CREATE: Manual


[Docs](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/snapshots-take-snapshot.html)


### SS of all cluster 

Make a SS of all data streams and open indices in the cluster.

Define a name for the SS
```
SS_NAME_TO_CREATE="snapshot_3"
```

Check if there's snapshot in process

> You can't take a new snapshot if one is currently in progress)
```
curl -XGET ${VPC_ENDPOINT}/_snapshot/_status | jq
```



> During snapshot initialization, information about all previous snapshots is loaded into memory, which means that in large repositories it may take several seconds (or even minutes) for this request to return even if the `wait_for_completion` parameter is set to false.

```
curl -XPUT ${VPC_ENDPOINT}/_snapshot/${ES_REPO}/${SS_NAME_TO_CREATE}
```


### SS of specific index

Make a snapshot of only specific index

Define SS name
```
SS_NAME_TO_CREATE="snapshot_3"
```

Define a JSON file with necessary snapshot configs
```
JSON_CONFIG_NAME="/tmp/ss.json"
```

Create and fill out a JSON file with indices list and metadata
```
cat <<EOT > ${JSON_CONFIG_NAME}
{
  "indices": "dev_90_ports,test_90_vessel_technical_specification",
  "ignore_unavailable": true,
  "include_global_state": false,
  "metadata": {
    "taken_by": "Sergii Burtovyi",
    "taken_because": "Test backup attempt"
  }
}
EOT

cat ${JSON_CONFIG_NAME}
```

Take a snapshot
```
curl -XPOST -H "Content-Type: application/json" -d @${JSON_CONFIG_NAME} ${VPC_ENDPOINT}/_snapshot/${ES_REPO}/${SS_NAME_TO_CREATE}
```























