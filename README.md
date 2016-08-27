This connector allows Kafka Connect to emulate a [Splunk Http Event Collector](http://dev.splunk.com/view/event-collector/SP-CAAAE6M).
This connector support receiving data and writing data to Splunk.

# Source Connector

This is under development.

# Sink Connector

The Sink Connector will transform data from a Kafka topic into a batch of json messages that will be written via HTTP to
a configured [Splunk Http Event Collector](http://dev.splunk.com/view/event-collector/SP-CAAAE6M). 

## Configuration

| Name                            | Description                                                                             | Type     | Default  | Valid Values | Importance |
|---------------------------------|-----------------------------------------------------------------------------------------|----------|----------|--------------|------------|
| splunk.auth.token               | The authorization token to use when writing data to splunk.                             | password |          |              | high       |
| splunk.remote.host              | The hostname of the remote splunk host to write data do.                                | string   |          |              | high       |
| splunk.ssl.enabled              | Flag to determine if the connection to splunk should be over ssl.                       | boolean  | true     |              | high       |
| splunk.ssl.trust.store.password | Password for the trust store.                                                           | password | [hidden] |              | high       |
| splunk.ssl.trust.store.path     | Path on the local disk to the certificate trust store.                                  | string   | ""       |              | high       |
| splunk.remote.port              | Port on the remote splunk server to write to.                                           | int      | 8088     |              | medium     |
| splunk.ssl.validate.certs       | Flag to determine if ssl connections should validate the certificateof the remote host. | boolean  | true     |              | medium     |

### Example Config

This configuration will write to Splunk over SSL but will not verify the certificate.

```
name=splunk-http-sink
topics=syslog-udp
tasks.max=1
connector.class=io.confluent.kafka.connect.splunk.SplunkHttpSinkConnector
splunk.remote.host=192.168.99.100
splunk.remote.port=8088
splunk.ssl.enabled=true
splunk.ssl.validate.certs=false
splunk.auth.token=**********
```

## Writing data to Splunk.

The Sink Connector uses the Splunk Http Event Collector as it's target to write data to Splunk. To use
this plugin you will need to [configure an endpoint](http://docs.splunk.com/Documentation/Splunk/6.4.3/Data/UsetheHTTPEventCollector).

The Sink Connector will pull over all of the fields that are in the incoming schema. If there is a timestamp field named
`date` or `time` it will be converted to a Splunk timestamp and moved to the `time` field. The `host` 
or `hostname` if it exists will be placed in the `host` field. All other fields will be copied to the `event` 
object.

Here is an example of an event generated by [Kafka Connect Syslog](https://github.com/jcustenborder/kafka-connect-syslog) written to Splunk.

```
{
  "host": "vpn.custenborder.com",
  "time": 1472342182,
  "event": {
    "charset": "UTF-8",
    "level": "6",
    "remote_address": "\/10.10.0.1:514",
    "message": "filterlog: 9,16777216,,1000000103,igb2,match,block,in,4,0x0,,64,5581,0,none,6,tcp,40,10.10.1.22,72.21.194.87,55450,443,0,A,,2551909476,8192,,",
    "facility": "16"
  }
}
```

# Running in development

```
mvn clean package
export CLASSPATH="$(find target/ -type f -name '*.jar'| grep '\-package' | tr '\n' ':')"
$CONFLUENT_HOME/bin/connect-standalone $CONFLUENT_HOME/etc/schema-registry/connect-avro-standalone.properties config/MySourceConnector.properties
```
