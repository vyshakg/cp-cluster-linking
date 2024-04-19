# Destination Cluster - MAIN


## Cluster linking
```
kafka-cluster-links --bootstrap-server localhost:9092 --create --link dev-link --config bootstrap.servers=broker-old:29094
```

## Topic mirrioring
```
kafka-mirrors --create --mirror-topic VYS_SWAG --link dev-link --bootstrap-server localhost:9092
```

## Describe link
```
kafka-configs --bootstrap-server localhost:9092 --describe --cluster-link dev-link
```
```
...
consumer.offset.sync.enable=true
 consumer.offset.group.filters={"groupFilters": [   {     "name": "dev-group",     "patternType": "LITERAL",     "filterType": "INCLUDE"   } ]} sensitive=false synonyms={}
...
```

### preparing for consumer syncing

```
echo "consumer.offset.group.filters={\"groupFilters\": [ \
  { \
    \"name\": \"dev-group\", \
    \"patternType\": \"LITERAL\", \
    \"filterType\": \"INCLUDE\" \
  } \
]}" > newFilters.properties
```



```
kafka-configs --bootstrap-server localhost:9092 --alter --cluster-link dev-link --add-config-file newFilters.properties
```

### Update the offset migration filters to remove the group from the migration process.

```
echo "consumer.offset.group.filters={\"groupFilters\": [ \
  { \
    \"name\": \"*\", \
    \"patternType\": \"LITERAL\", \
    \"filterType\": \"INCLUDE\" \
  }, \
  { \
    \"name\": \"dev-group\", \
    \"patternType\": \"LITERAL\", \
    \"filterType\": \"EXCLUDE\" \
  } \
]}" > newFilters.properties
```

[Migrating consumer groups from source to destination cluster](https://docs.confluent.io/platform/current/multi-dc-deployments/cluster-linking/commands.html#migrating-consumer-groups-from-source-to-destination-cluster)

----



# Source Cluster - OLD


```
kafka-console-producer --topic VYS_SWAG --bootstrap-server localhost:9094

kafka-console-consumer --topic VYS_SWAG --from-beginning --bootstrap-server localhost:9094 --group dev-group

```

