


## GCP 에 카프카 설치하기 
- VM 인스턴스 생성 
https://cloud.google.com/compute/docs/quickstart-linux?hl=ko
- SSH 설정 
https://jybaek.gitbook.io/with-gcp/appendix/gce_to_ssh
- 방화벽 설정 
inbound 22, 9092, 2181
https://gusrb.tistory.com/50


- java 및 kafka 설치 (우분투)
```bash
$ sudo apt update
$ sudo apt-get install -y openjdk-8-jdk
$ java -version

# kafka
$ wget https://archive.apache.org/dist/kafka/2.5.0/kafka_2.12-2.5.0.tgz
$ tar xvf kafka_2.12-2.5.0.tgz
$ ls kafka_2.12-2.5.0
```

- kafka heap memory 설정 
기본적으로 카프카, 주키퍼를 설치하면 카프카는 1G, 주키퍼는 0.5G 를 사용함 
ㄴ 우리는 1G VM 장비이기 때문에 카프카 힙 메모리를 줄여야 한다 
```bash
$ vi .bashrc
export KAFKA_HEAP_OPTS="-Xmx400m -Xms400m"

$ source .bashrc
$ echo $KAFKA_HEAP_OPTS
```

카프카 메모리 설정 확인 - bin/kafka-server-start.sh
```bash
if [ "x$KAFKA_HEAP_OPTS" = "x" ]; then
    export KAFKA_HEAP_OPTS="-Xmx1G -Xms1G"
fi
```

- 카프카 브로커 실행 옵션 설정 
config/server.properties 에서 카프카 브로커 실행 옵션을 지정할 수 있음 
advertised.listeners 에 현재 VM의 IP 를 추가하여 접속 가능하도록 세팅 
```bash
$ vi config/server.properties

# 주석을 해제 + VM IP 지정 
advertised.listeners=PLAINTEXT://[VM IP]:9092
```

- 주키퍼 실행 
```bash
$ bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
# check
$ jps -vm
18025 QuorumPeerMain config/zookeeper.properties -Xmx400m -Xms400m -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+ExplicitGCInvokesConcurrent -XX:MaxInlineLevel=15 -Djava.awt.headless=true -Xloggc:/home/koungho632/kafka_2.12-2.5.0/bin/../logs/zookeeper-gc.log -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=100M -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Dkafka.logs.dir=/home/koungho632/kafka_2.12-2.5.0/bin/../logs -Dlog4j.configuration=file:bin/../config/log4j.properties
18062 Jps -vm -Dapplication.home=/usr/lib/jvm/java-8-openjdk-amd64 -Xms8m
```

- 카프카 브로커 실행 
```bash
$ bin/kafka-server-start.sh -daemon config/server.properties

# check
$ jps -m
18423 Kafka config/server.properties
18025 QuorumPeerMain config/zookeeper.properties
18493 Jps -m
$ tail -f logs/server.log
[2021-05-02 02:57:57,122] INFO [TransactionCoordinator id=0] Starting up. (kafka.coordinator.transaction.TransactionCoordinator)
# ..
```

- 로컬에서 VM 카프카 호출해보기  
```bash
# 호출을 위해선 먼저 카프카를 설치 
$ wget https://archive.apache.org/dist/kafka/2.5.0/kafka_2.12-2.5.0.tgz
$ tar xvf kafka_2.12-2.5.0.tgz

# VM 장비 버전 확인하기 
$ cd kafka_2.12-2.5.0
$ bin/kafka-broker-api-versions.sh --bootstrap-server [VM IP]:9092

# 로컬에 도메인 지정 
$ sudo vi /etc/hosts
# VM 의 공인 IP 를 my-kafka 세팅 
VM IP   my-kafka

$ bin/kafka-broker-api-versions.sh --bootstrap-server my-kafka:9092
```

## 카프카 CLI 실습 
### kafka-topic.sh
CLI 환경에서 토픽과 관련된 명령을 실행 

- 토픽 생성 
토픽을 생성하는 방법은 2가지가 있는데, 명시적으로 CLI 를 통해서 생성하는 방법과 프로듀서, 컨슈머에 의해 자동으로 생성되는 방법이 있음 
ㄴ 명시적으로 생성하는 방법을 추천하고 있음 
```bash
$ bin/kafka-topics.sh \
    --create \
    # 접속 정보 
    --bootstrap-server my-kafka:9092 \
    # 파티션 수 
    --partitions 3 \
    # 레플리케이션 수 
    --replication-factor 1 \
    # 데이터 유지 시간 (2일)
    --config retention.ms=172800000 \ 
    --topic hello.kafka
```

- 토픽 리스트 조회 
```bash
$ bin/kafka-topics.sh --bootstrap-server my-kafka:9092 --list
hello.kafka
```

- 토픽 조회 
```bash
$ bin/kafka-topics.sh --bootstrap-server my-kafka:9092 --describe --topic hello.kafka
Topic: hello.kafka	PartitionCount: 3	ReplicationFactor: 1	Configs: segment.bytes=1073741824,retention.ms=172800000
	Topic: hello.kafka	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: hello.kafka	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
	Topic: hello.kafka	Partition: 2	Leader: 0	Replicas: 0	Isr: 0
```

- 토픽 옵션 수정 
토픽의 옵션을 수정할 때는 kafka-topics.sh, kafka-configs.sh 두 개를 이용함 
```bash
$ bin/kafka-topics.sh --bootstrap-server my-kafka:9092 \
    --topic hello.kafka \
    # 파티션 개수 수정을 위한 옵션 
    --alter \
    --partitions 4
# 조회 
$ bin/kafka-topics.sh --bootstrap-server my-kafka:9092 \
    --topic hello.kafka \
    --describe
Topic: hello.kafka	PartitionCount: 4	ReplicationFactor: 1	Configs: segment.bytes=1073741824,retention.ms=172800000
	Topic: hello.kafka	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: hello.kafka	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
	Topic: hello.kafka	Partition: 2	Leader: 0	Replicas: 0	Isr: 0
	Topic: hello.kafka	Partition: 3	Leader: 0	Replicas: 0	Isr: 0


$ bin/kafka-configs.sh --bootstrap-server my-kafka:9092 \
    --entity-type topics \
    --entity-name hello.kafka \
    # 토픽의 옵션 수정 
    --alter -add-config retention.ms=86400000
# 조회
$ bin/kafka-config.sh --bootstrap-server my-kafka:9092 \
    --entity-type topics \
    --entity-name hello.kafka \
    --describe
Dynamic configs for topic hello.kafka are:
  retention.ms=86400000 sensitive=false synonyms={DYNAMIC_TOPIC_CONFIG:retention.ms=86400000}
```

- 프로듀서 콘솔 
kafka-console-producer.sh 를 이용해서 토픽에 데이터를 넣을 수 있음 
```bash
$ bin/kafka-console-producer.sh --bootstrap-server my-kafka:9092 \
    --topic hello.kafka 
> hello
> kafka
> 0
> 1
```
별도로 key 를 지정하지 않으면 key 는 null 로 세팅됨 
각 key, value 들은 ByteArraySerializer 로 직렬화 된다 

key 를 갖는 메시지를 전송
```bash
$ bin/kafka-console-producer.sh --bootstrap-server my-kafka:9092 \
    --topic hello.kafka \
    # message key 를 위한 옵션 
    --property "parse.key=true" \
    # 구분자 
    --property "key.separator=:" 

> key1:no1
> key2:no2
> key3:no3
```

- 컨슈머 콘솔
kafka-console-consumer.sh 로 토픽에 대한 데이터를 확인할 수 있음 
```bash
$ bin/kafka-console-consumer.sh --bootstrap-server my-kafka:9092 \
    --topic hello.kafka \
    --from-beginning
no1
...
```
kafka-console-producer.sh 로 전송한 데이터들을 확인 할 수 있음 

key, value 모두 확인 
```bash
$ bin/kafka-console-consumer.sh --bootstrap-server my-kafka:9092 \
    --topic hello.kafka \
    --property "print.key=true" \
    --property "key.separator=-" \
    # 컨슈머 그룹 생성 - offset 이 커밋됨 
    --group hello-group \
    --from-beginning
key1-no1
key2-no2
```
컨슈머 그룹을 이용해서 consume 하면 commit 이 일어나며 offset 이 증가한다 

- 컨슈머 그룹 콘솔
kafka-consumer-groups.sh 로 생성된 컨슈머 그룹을 확인할 수 있다 
```bash
# 리스트 확인 
$ bin/kafka-consumer-groups.sh --bootstrap-server my-kafka:9092 \
    # kafka-console-consumer.sh 의 --group 으로 생성한 그룹이름 
    --list hello-group
hello-group

# 컨슈머 그룹의 대한 상세 정보 
$ bin/kafka-consumer-groups.sh --bootstrap-server my-kafka:9092 \
    # kafka-console-consumer.sh 의 --group 으로 생성한 그룹이름 
    --group hello-group \
    --describe

Consumer group 'hello-group' has no active members.

GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
hello-group     hello.kafka     3          2               2               0               -               -               -
hello-group     hello.kafka     2          0               0               0               -               -               -
hello-group     hello.kafka     1          1               1               0               -               -               -
hello-group     hello.kafka     0          2               2               0               -               -               -
```

- 토픽의 삭제 
kafka-delete-records.sh 는 이미 적재된 토픽의 데이터 중 가장 오래된 데이터부터 특정 offset 까지 삭제할 때 사용 
```bash
# 삭제하고자 하는 정보를 파일로 저장 
$ vi delete-topic.json
{"partitions": [{"topic": "test", "partition": 0, "offset": 50}], "version": 1}

$ bin/kafka-delete-records.sh --bootstrap-server my-kafka:9092 \
    --offset-json-file delete-topic.json
```
토픽은 특정 레코드만 삭제할 수 없고 **파티션에 존재하는 가장 오래된 데이터** 부터 삭제됨 