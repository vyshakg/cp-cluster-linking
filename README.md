


--SOURCE--
export CONFLUENT_HOME=
export CONFLUENT_CONFIG=$CONFLUENT_HOME/etc/kafka

cp $CONFLUENT_CONFIG/zookeeper.properties $CONFLUENT_CONFIG/zookeeper-clusterlinking_src.properties
sed -i '' -e "s/zookeeper/zookeeper-clusterlinking_src/g" $CONFLUENT_CONFIG/zookeeper-clusterlinking_src.properties


cp $CONFLUENT_CONFIG/server.properties $CONFLUENT_CONFIG/server-clusterlinking_src.properties

sed -i '' -e "s/#listeners=PLAINTEXT/listeners=PLAINTEXT/g" $CONFLUENT_CONFIG/server-clusterlinking_src.properties
sed -i '' -e "s/#advertised.listeners=PLAINTEXT/advertised.listeners=PLAINTEXT/g" $CONFLUENT_CONFIG/server-clusterlinking_src.properties

sed -i '' -e "s/your.host.name:9092/:9092/g" $CONFLUENT_CONFIG/server-clusterlinking_src.properties

sed -i '' -e "s/kafka-logs/kafka-logs-1/g" $CONFLUENT_CONFIG/server-clusterlinking_src.properties
echo "inter.broker.listener.name=PLAINTEXT" >> $CONFLUENT_CONFIG/server-clusterlinking_src.properties
echo "confluent.reporters.telemetry.auto.enable=false" >> $CONFLUENT_CONFIG/server-clusterlinking_src.properties
echo "confluent.cluster.link.enable=true" >> $CONFLUENT_CONFIG/server-clusterlinking_src.properties

echo "confluent.http.server.listeners=http://localhost:8090" >> $CONFLUENT_CONFIG/server-clusterlinking_src.properties


----

export CONFLUENT_HOME=
export CONFLUENT_CONFIG=$CONFLUENT_HOME/etc/kafka

cp $CONFLUENT_CONFIG/zookeeper.properties $CONFLUENT_CONFIG/zookeeper-clusterlinking_dest.properties


sed -i '' -e "s/2181/2183/g" $CONFLUENT_CONFIG/zookeeper-clusterlinking_dest.properties


sed -i '' -e "s/zookeeper/zookeeper-clusterlinking_dest/g" $CONFLUENT_CONFIG/zookeeper-clusterlinking_dest.properties


cp $CONFLUENT_CONFIG/server.properties $CONFLUENT_CONFIG/server-clusterlinking_dest.properties

sed -i '' -e "s/#listeners=PLAINTEXT/listeners=PLAINTEXT/g" $CONFLUENT_CONFIG/server-clusterlinking_dest.properties


sed -i '' -e "s/9092/9094/g" $CONFLUENT_CONFIG/server-clusterlinking_dest.properties

sed -i '' -e "s/2181/2183/g" $CONFLUENT_CONFIG/server-clusterlinking_dest.properties


sed -i '' -e "s/kafka-logs/kafka-logs-2/g" $CONFLUENT_CONFIG/server-clusterlinking_dest.properties


echo "confluent.http.server.listeners=http://localhost:8091" >> $CONFLUENT_CONFIG/server-clusterlinking_dest.properties


echo "password.encoder.secret=mypassword" >> $CONFLUENT_CONFIG/server-clusterlinking_dest.properties




kafka-cluster-links --bootstrap-server localhost:9092 --create --link dev-link --config bootstrap.servers=broker-old:29094


----


kafka-cluster-links --bootstrap-server broker-dr:9093 --create --link dev-link --config bootstrap.servers=broker:9092
kafka-mirrors --create --mirror-topic VYS_SWAG --link demo-link --bootstrap-server localhost:9092
kafka-console-producer --topic demo --bootstrap-server broker:9092
kafka-console-consumer --topic demo --from-beginning --bootstrap-server broker-dr:9093




kafka-topics --create --topic VYS_SWAG --bootstrap-server broker-old:9094
kafka-console-producer --topic VYS_SWAG --bootstrap-server localhost:29094
kafka-cluster-links --bootstrap-server broker:9092 --create --link vyshak-link --config bootstrap.servers=broker-old:9094


kafka-console-consumer --topic VYS_SWAG --from-beginning --bootstrap-server localhost:9094 --group dev-group

kafka-console-consumer --topic VYS_SWAG  --bootstrap-server localhost:9092 --group dev-group

kafka-console-producer --topic VYS_SWAG --bootstrap-server localhost:9094


kafka-configs --bootstrap-server localhost:9092 \
                  --alter \
                  --cluster-link dev-link \
                  --add-config consumer.offset.group.filters={ \"groupFilters\": [ { \"name\": \"dev-group\", \"patternType\": \"LITERAL\", \"filterType\": \"INCLUDE\" } ] }


kafka-configs --bootstrap-server localhost:9092 \
                  --describe \
                  --cluster-link dev-link

echo "consumer.offset.group.filters={\"groupFilters\": [ \
  { \
    \"name\": \"dev-group\", \
    \"patternType\": \"LITERAL\", \
    \"filterType\": \"INCLUDE\" \
  } \
]}" > newFilters.properties


consumer.offset.sync.enable=true


kafka-configs --bootstrap-server localhost:9092 --alter --cluster-link dev-link --add-config-file newFilters.properties