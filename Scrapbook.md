
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

# Setting up tap-github

* Reference [https://github.com/singer-io/tap-github](tap-github)
* Install, python3-virtualenv, create a virtual environment and install tap-github via pip
```
 sudo apt install python3-virtualenv
 virtualenv -p python venv
 source venv/bin/activate
 pip install tap-github
```

* Login to github > Settings > Developer Settings > Personal Access Tokens. Generate a new access token.
* Create a _config.json_ as specified. 
  * Update _"access_token"_ with the one generated in previous step
  * Update _"repository"_ with the one you need to read from
  * Update _"start_date"_ to the time from which you need to read updates from
```
{"access_token": "your-access-token",
 "repository": "abyphil/scratch",
 "start_date": "2021-01-01T00:00:00Z",
 "request_timeout": 300
}
```

* Run _tap-github_ in discover mode to generate properties.json
```
tap-github --config config.json --discover > properties.json
```
* Edit the _properties.json_ to add _"selected": true_ to any streams you are interested. 
* Now run the command to fetch the stream
```
tap-github --config config.json --properties properties.json
```
* This should print a list of json records that show all changes in the stream, for _repository_, since _start_date_.

# TODOs
* Get a singer tap running
* Get a singer target running

# Ideas
* Create a java adapter that can read from any singer tap
* Add an iterator interface for the adapter. Records read off the stdout, can in turn be read from the iterator.
* Create a java based SDK for singer taps
* Singer tap can be ran from command line to examine the content. Instead of command line, may be create a Repl. 
  * Load the tap
  * Examine the configuration, state etc. 