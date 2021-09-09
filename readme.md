# Kafka -> InfluxDB connector demo

Demo to show how to connect data in Kafka to InfluxDB using Kafka connect.

WARNING: Kafka connect is considered legacy and cannot connect Kafka to InfluxDB v2.

## Setup environment
```
docker-compose up
```

Wait for 30 seconds for the spew of logs to stop.

## Make sure that kafka is working

1. Send some data:

```
docker exec -i kafkacat kafkacat -b broker:9092 -P -t json_01 <<EOF
{ "schema": { "type": "struct", "fields": [ { "field": "tags" , "type": "map", "keys": { "type": "string", "optional": false }, "values": { "type": "string", "optional": false }, "optional": false}, { "field": "stock", "type": "double", "optional": true } ], "optional": false, "version": 1 }, "payload": { "tags": { "host": "FOO", "product": "wibble" }, "stock": 500.0 } }
EOF
```

2. Check data is in kafka:
```
docker exec kafkacat kafkacat -b broker:9092 -C -u -t json_01
```

## Setup connector

1. Create connector:
```
curl -i -X PUT -H "Content-Type:application/json" \
        http://localhost:8083/connectors/sink_influx_json_01/config \
        -d '{
            "connector.class"               : "io.confluent.influxdb.InfluxDBSinkConnector",
            "value.converter"               : "org.apache.kafka.connect.json.JsonConverter",
            "value.converter.schemas.enable": "true",
            "key.converter"                 : "org.apache.kafka.connect.storage.StringConverter",
            "topics"                        : "json_01",
            "influxdb.url"                  : "http://influxdb:8086",
            "influxdb.db"                   : "my_db",
            "measurement.name.format"       : "${topic}"
        }'
```

2. Make sure that the connector is running:
```
curl -s "http://localhost:8083/connectors?expand=info&expand=status" | python -m json.tool
```

## Check connection between kafka and influxdb

1. Send some more data with the command above.

2. Check that the data was recieved by InfluxDB:
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