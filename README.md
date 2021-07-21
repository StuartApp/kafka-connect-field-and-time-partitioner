## Stuart Customization

This is a fork of https://github.com/canelmas/kafka-connect-field-and-time-partitioner

We will use branch `stuart-1.0.0` as the main branch, this is based on the release 1.0.0 of the original repo 
(1.0.0 is the last release available at writing time - 2021-07-20).

#### How to create the jar
```bash
git checkout stuart-1.0.0
mvn clean package
```
The jar generated in the target folder must be added to the [external-plugins folder](https://github.com/StuartApp/kafka-connect/tree/master/plugins/external-plugins).

#### Changes applied in stuart version

- use the same kafka version used in stuart kafka cluster
- add the confluent repository to retrieve the dependencies
- change mvn compiler version to make it work with the newer versions of java
- remove a wrong error log that appears when we use avro instead of json files(`ERROR Unsupported type 'string' for partition field.`)

### Kafka Connect Field and Time Based Partitioner

- Partition initially by a custom field and then by time.
- It extends **[TimeBasedPartitioner](https://github.com/confluentinc/kafka-connect-storage-common/blob/master/partitioner/src/main/java/io/confluent/connect/storage/partitioner/TimeBasedPartitioner.java)**, so any existing time based partition config should be fine.
- In order to make it work, set `"partitioner.class"="com.canelmas.kafka.connect.FieldAndTimeBasedPartitioner"` and `"partition.field"="<custom field in your record>"`

```bash
KCONNECT_NODES=("localhost:18083" "localhost:28083" "localhost:38083")

for i in "${!KCONNECT_NODES[@]}"; do
    curl ${KCONNECT_NODES[$i]}/connectors -XPOST -H 'Content-type: application/json' -H 'Accept: application/json' -d '{
        "name": "connect-s3-sink-'$i'",
        "config": {
            "topics": "events",
            "connector.class": "io.confluent.connect.s3.S3SinkConnector",
            "tasks.max" : 4,
            "flush.size": 100,
            "rotate.schedule.interval.ms": "-1",
            "rotate.interval.ms": "-1",
            "s3.region" : "eu-west-1",
            "s3.bucket.name" : "byob-raw",
            "s3.compression.type": "gzip",
            "topics.dir": "topics",
            "storage.class" : "io.confluent.connect.s3.storage.S3Storage",
            "partitioner.class": "com.canelmas.kafka.connect.FieldAndTimeBasedPartitioner",
            "partition.duration.ms" : "3600000",
            "path.format": "YYYY-MM-dd",
            "locale" : "US",
            "timezone" : "UTC",
            "schema.compatibility": "NONE",
            "format.class" : "io.confluent.connect.s3.format.json.JsonFormat",
            "timestamp.extractor": "Record",
            "partition.field" : "appId"
        }
    }'
done
```