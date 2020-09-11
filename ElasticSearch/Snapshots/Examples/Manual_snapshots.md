# ES: MANUAL SNAPSHOTS


# Documentation
[ES official](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/snapshots-take-snapshot.html)

[AWS ES official](https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-managedomains-snapshots.html)


# 1. Preparation

## Enable VPN for appropriate env

## 1.1. Define "VPC endpoint" of the ES
```
VPC_ENDPOINT="https://vpc-esrch-k8s-logs-noticeably-pr-gzibkcrjyduidqngcyh6kk7lla.eu-west-1.es.amazonaws.com"
```

## 1.2. View configured repository for manual snapshots

> s3 and Role should be already configured for the ES cluster
> during one-time pre-requisite steps for manual snapshoting

> Remember the name of the previously configured repository for manual snapshots
> it's on the "cs-automated" JSON level, if configured
```
curl ${VPC_ENDPOINT}/_snapshot/ | jq
```
```
ES_MAN_REPO="amazon-do-not-delete"
```

## 1.3. Status

Check status of spanshoting
Because you can't take a snapshot if one is currently in progress
```
curl -XGET ${VPC_ENDPOINT}/_snapshot/_status | jq
```

## 1.4. List 

### All snapshots of your domain
```
curl -XGET ${VPC_ENDPOINT}/_snapshot/${ES_MAN_REPO}/_all?pretty | jq
```

#### Specific snapshot
```
SS_NAME_EXISTING="snapshot_1"
curl -XGET ${VPC_ENDPOINT}/_snapshot/${ES_MAN_REPO}/${SS_NAME_EXISTING} | jq
```


# 2. Take a manual snapshot

[Taking manual ss](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/snapshots-take-snapshot.html)

## 2.1. SS of all cluster (all data streams and open indices in the cluster)
```
SS_NAME_TO_CREATE="snapshot_3"
```

> During snapshot initialization, information about all previous snapshots is loaded into memory, which means that in large repositories it may take several seconds (or even minutes) for this request to return even if the wait_for_completion parameter is set to false.
```
curl -XPUT ${VPC_ENDPOINT}/_snapshot/${ES_MAN_REPO}/${SS_NAME_TO_CREATE}
```

## 2.2. SS of specific index

### Define snapshot name
```
SS_NAME_TO_CREATE="snapshot_3"
```

### Define a JSON file with necessary snapshot configs
```
JSON_CONFIG_NAME="/tmp/ss.json"
```

### Create and fill out a JSON file with indices list and metadata
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

### Take a snapshot
```
curl -XPOST -H "Content-Type: application/json" -d @${JSON_CONFIG_NAME} ${VPC_ENDPOINT}/_snapshot/${ES_MAN_REPO}/${SS_NAME_TO_CREATE}
```




# Restore
[snapshots-restore-snapshot](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/snapshots-restore-snapshot.html)

[es-managedomains-snapshots](https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-managedomains-snapshots.html)

