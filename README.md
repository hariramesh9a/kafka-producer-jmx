	
	
KAFKA On MAC
brew cask install java
 brew install kafka
 brew services start zookeeper
 brew services start kafka

zookeeper-server-start /usr/local/etc/kafka/zookeeper.properties

Mac collecting JMX:


Download JMX exporter from 
https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.3.0/jmx_prometheus_javaagent-0.3.0.jar


My jar is in: 
/Users/harumughan/Downloads/jmx_prometheus_javaagent-0.3.0.jar


lsof -i -P | grep -i "listen" | grep 7070
	



wget https://downloads.apache.org/kafka/2.4.1/kafka_2.12-2.4.1.tgz

-javaagent:/opt/jmx-exporter/jmx-exporter.jar=7070:/etc/jmx-exporter/zookeeper.yml

/Users/harumughan/Downloads/kafka_2.12-2.4.1/bin/zookeeper-server-start.sh /Users/harumughan/Downloads/kafka_2.12-2.4.1/config/zookeeper.properties


 

export EXTRA_ARGS=“-javaagent:jmx_prometheus_javaagent.jar=7070:zookeeper.yml”


Working steps:
———————

Download JMX exporter from 
https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.3.0/jmx_prometheus_javaagent-0.3.0.jar
Download  Kafka yaml for JMX extractor

Download Kafka server
wget https://downloads.apache.org/kafka/2.4.1/kafka_2.12-2.4.1.tgz
Tar kafka*
Cd kafka*

Start ZK:
/Users/harumughan/Downloads/kafka_knowledge/kafka_2.12-2.4.1/bin/zookeeper-server-start.sh /Users/harumughan/Downloads/kafka_knowledge/kafka_2.12-2.4.1/config/zookeeper.properties

Start Kafka:
export KAFKA_OPTS='-javaagent:/Users/harumughan/Downloads/kafka_knowledge/kafka_2.12-2.4.1/kafka-jmx/jmx_prometheus_javaagent-0.3.0.jar=7072:/Users/harumughan/Downloads/kafka_knowledge/kafka_2.12-2.4.1/kafka-jmx//kafka.yml'
export JMX_PORT=7081
￼
/Users/harumughan/Downloads/kafka_knowledge/kafka_2.12-2.4.1/bin/kafka-server-start.sh /Users/harumughan/Downloads/kafka_knowledge/kafka_2.12-2.4.1/config/server.properties


Create topic
=========


bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic logs


Produce message:
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic logs
=========

Check the ports:
netstat -tlnpu |grep 7072 or on MAC
lsof -i -P | grep -i "listen" | grep 7072
Output:
java      57235 harumughan  101u  IPv6 0x6a31d625314bbe3b      0t0  TCP *:7072 (LISTEN)
Or got jconsole and see 
￼


Install : Prometheus

On Mac download  Darwin prometheus-2.17.1.darwin-amd64.tar.gz
Untar -xvcf prometheus-2.17.1.darwin-amd64.tar.gz


 > create config.yml

global:
  scrape_interval:     15s
  external_labels:
    monitor: 'producer'

scrape_configs:
  - job_name: 'prod'
    scrape_interval: 5s
    static_configs:
      - targets: [localhost:7072]


cd /Users/harumughan/Downloads/kafka-knowledge/kafka-jmx/prometheus-2.17.1.darwin-amd64
./prometheus --config.file=kafka.yml


Now you can view promethue UI serving on port 9090

￼

￼

And you can see Kafka producer metrics being captured in prometheus

￼


Get grafana
—————

sudo apt-get install -y adduser libfontconfig1
wget https://dl.grafana.com/oss/release/grafana_7.0.3_amd64.deb
sudo dpkg -i grafana_7.0.3_amd64.deb



wget https://grafana.com/api/dashboards/721/revisions/1/download
wget https://dl.grafana.com/oss/release/grafana_5.4.2_amd64.deb
sudo apt-get install -y adduser libfontconfig
sudo dpkg -i grafana_5.4.2_amd64.deb
sudo systemctl start grafana-server

/Users/harumughan/Downloads/kafka-jmx/grafana-6.7.2/bin
./grafana-server start


￼


Getting producer Logs:

export KAFKA_OPTS=‘-javaagent:/Users/harumughan/Downloads/kafka_knowledge/kafka_2.12-2.4.1/kafka-jmx/jmx_prometheus_javaagent-0.3.0.jar=10102:/Users/harumughan/Downloads/kafka_knowledge/kafka_2.12-2.4.1/kafka-jmx/kafka.yml’
JMX_PORT=10102 bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test

export KAFKA_OPTS="-javaagent:/Users/harumughan/Downloads/kafka_knowledge/kafka_2.12-2.4.1/kafka-jmx/jmx_prometheus_javaagent-0.3.0.jar=10103:/Users/harumughan/Downloads/kafka_knowledge/kafka_2.12-2.4.1/kafka-jmx/jmx-producer.yml"
JMX_PORT=10102 bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test


Type something




Or use a custom app to generate messages

Once your producer message you can see jconsole started tracking producer metrics as well

￼

￼


Check prometheus target to make sure prometheus started scraping producer metrics as well

￼

￼

Starting consumer:

bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning

