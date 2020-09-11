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


## STATUS

Check if there's snapshot in process

(because you can't take a new snapshot if one is currently in progress)
```
curl -XGET ${VPC_ENDPOINT}/_snapshot/_status | jq
```








