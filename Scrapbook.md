
# Setting up development environment

Install python3

```bash
choco install python3
```

# Setting up kafka

* Download kafka binary. Refer [here](https://kafka.apache.org/quickstart)

```bash
cd ​C:\home\kafka\kafka_2.13-3.0.0\bin\windows
```

Running kafka natively on windows is not a smooth affair. Zookeeper runs fine but running kafka is not straightforward. With WSL, this is easier than before but is not a smooth ride as in linux. Instead, we can use docker for the purpose.
We still need the the kafka tar file so that we can use the scripts to create topics, run kafka producer and consumer.

# Setting up docker based kafka on windows

* Go to [dockerhub](https://hub.docker.com/r/bitnami/kafka)
As per instructions download and create docker-compose.yml

```base
// use bash (cmder)
curl -sSL https://raw.githubusercontent.com/bitnami/bitnami-docker-kafka/master/docker-compose.yml > docker-compose.yml
```

Add the following lines in it.
```
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CLIENT:PLAINTEXT,EXTERNAL:PLAINTEXT
      - KAFKA_CFG_LISTENERS=CLIENT://:9092,EXTERNAL://:9093
      - KAFKA_CFG_ADVERTISED_LISTENERS=CLIENT://kafka:9092,EXTERNAL://localhost:9093
      - KAFKA_INTER_BROKER_LISTENER_NAME=CLIENT
```

Start kafka on docker using docker-compose
```bash
 docker-compose up -d
```

Run the following from powershell. The kafka scripts for windows should be used. Use separate windows for running producer and consumer. 
```bash

cd C:\home\kafka\kafka_2.13-3.0.0\bin\windows\
.\kafka-topics.bat --create --topic test --bootstrap-server localhost:9093 --partitions 1
.\kafka-topics.bat --list --bootstrap-server localhost:9093
.\kafka-console-producer.bat  --broker-list 127.0.0.1:9093 --topic test
.\kafka-console-consumer.bat --bootstrap-server 127.0.0.1:9093 --topic test --from-beginning
```
This will open the producer and consumer consoles. Anything typed in the producer console would be received in the consumer console too.


# Ideas
