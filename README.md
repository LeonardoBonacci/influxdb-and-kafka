# influxdb-and-kafka

```
curl -i -X PUT -H "Content-Type:application/json" \
      http://localhost:8083/connectors/sink/config \
      -d '{
          "connector.class"               : "io.confluent.influxdb.InfluxDBSinkConnector",
          "value.converter"               : "org.apache.kafka.connect.json.JsonConverter",
          "value.converter.schemas.enable": "true",
          "key.converter"                 : "org.apache.kafka.connect.storage.StringConverter",
          "topics"                        : "json_01",
          "influxdb.url"                  : "http://localhost:8086",
          "influxdb.db"                   : "my_db",
          "measurement.name.format"       : "${topic}"
      }'

curl -i -X PUT -H "Content-Type:application/json" \
      http://localhost:8083/connectors/source/config \
      -d '{
          "connector.class"               : "io.confluent.influxdb.source.InfluxdbSourceConnector",
          "tasks.max"                     : 1,
          "influxdb.url"                  : "http://localhost:8086",
          "influxdb.db"                   : "my_db",
          "topic.prefix"                  : "inf_",
          "query"                         : "SELECT * FROM json_01"
      }'

curl localhost:8083/connectors/source/status
curl -X DELETE localhost:8083/connectors/source
```

```
docker exec -i kafkacat kafkacat -b broker:29092 -P -t json_01 <<EOF
{ "schema": { "type": "struct", "fields": [ { "field": "tags" , "type": "map", "keys": { "type": "string", "optional": false }, "values": { "type": "string", "optional": false }, "optional": false}, { "field": "stock", "type": "double", "optional": true } ], "optional": false, "version": 1 }, "payload": { "tags": { "host": "FOO", "product": "wibble" }, "stock": 500.0 } }
EOF

docker exec kafkacat kafkacat -b broker:29092 -C -u -t json_01
```

```
$ docker exec -it influxdb influx -execute 'show measurements on "my_db"'
name: measurements
name
----
json_01

$ docker exec -it influxdb influx -execute 'show tag keys on "my_db"'
name: json_01
tagKey
------
host
product

$ docker exec -it influxdb influx -execute 'SELECT * FROM json_01 GROUP BY host, product;' -database "my_db"
name: json_01
tags: host=FOO, product=wibble
time                stock
----                -----
1579779810974000000 500
1579779820028000000 500
1579779833143000000 500
```

## References

* https://rmoff.net/2020/01/23/notes-on-getting-data-into-influxdb-from-kafka-with-kafka-connect/
* https://github.com/confluentinc/demo-scene/tree/master/influxdb-and-kafka
* https://docs.confluent.io/kafka-connect-influxdb/current/influx-db-sink-connector/influx_d_b_sink_connector_config.html
* https://docs.confluent.io/kafka-connect-influxdb/current/influx-db-source-connector/influx_db_source_connector_config.html
