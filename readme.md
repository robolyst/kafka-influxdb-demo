# Kafka -> InfluxDB connector demo

Demo to show how to connect data in Kafka to InfluxDB using Telegraf.

## Setup environment
```
docker-compose up
```

Wait for 30 seconds for the spew of logs to stop.

## Send data to Influxdb

1. Send some data:

```
docker exec -i kafkacat kafkacat -b broker:9092 -P -t telegraf <<EOF
{"metric": "weather", "tags": {"location": "Sydney", "season": "spring"}, "temperature": 20, "timestamp": "$(date +"%Y-%m-%dT%H:%M:%S%z")"}
EOF
``` 

2. Navigate to `<YOURMACHINE>:8086` and login with username: `my-user` and password: `my-password`. Check the data in the bucket `my-bucket` and you'll see a measurement in there for `weather`.