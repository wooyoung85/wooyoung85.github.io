> Kafka를 주제로 발표했던 자료 정리한 내용입니다.  
Custom Partitioner 구현한 예제는 따로 블로깅하도록 하겠습니다 ^^


# Kafka 소개
## Kafka란?
- Apache Kafka was originated at <span style="font-weight:bold">LinkedIn</span> and later became an <span style="font-weight:bold">open sourced</span> Apache project in 2011
- Kafka is written in <span style="font-weight:bold">Scala</span> and <span style="font-weight:bold">Java</span>. 
- Apache Kafka is <span style="color:green;font-weight:bold">publish</span>-<span style="color:green;font-weight:bold">subscribe</span> based fault tolerant <span style="color:red;font-weight:bold">messaging system</span>. 
- It is fast, <span style="font-weight:bold">scalable</span> and <span style="font-weight:bold">distributed</span> by design.
- 대용량의 실시간 로그처리에 특화

> **Messaging System은 왜 사용할까?**
> - 애플리케이션/시스템 간의 통신
> - 서버 부하가 많은 작업
> - 부하분산
> - 데이터 손실 방지

## 구성요소
- **Topic**
    - 특정 범주에 속하는 메세지 스트림
    - 데이터는 Topic에 저장됨!!!
- Partition
    - Topic을 Partitioning하면 많은 양의 데이터를 처리 가능함
- Partition offset
    - partitioned message의 유니크한 sequence id
- Replicas of partition
    - partion의 백업일뿐 데이터를 읽거나 쓸 수 없음
- Brokers
- Kafka Cluster
- **Producer** (publish)
- **Consumer** (subscribe)
- Leader
- Follower

## Kafka Architecture


# [QuickStart](https://kafka.apache.org/quickstart)
- 카프카를 다운로드 한후, 압축을 풀면 주키퍼와 카프카가 포함되어 있음
- 카프카는 JVM위에서 동작 (당연히 Java 설치가 필요함)

### Broker 구성하기 (Single)

- 주키퍼 실행  
    ```shell
    $> kafka_2.11-2.0.0/bin/zookeeper-server-start.sh kafka_2.11-2.0.0/config/zookeeper.properties
    ```
- 카프카 실행  
    ```shell
    $> kafka_2.11-2.0.0/bin/kafka-server-start.sh kafka_2.11-2.0.0/config/server.properties
    ```

### Topic 만들기

- Topic 생성  
    ```shell
    $> kafka_2.11-2.0.0/bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
    ```

- Topic 리스트 확인  
    ```shell
    $> kafka_2.11-2.0.0/bin/kafka-topics.sh --list --zookeeper localhost:2181
    ```

### Pub-Sub

- 메세지 보내기  
    ```shell
    $> kafka_2.11-2.0.0/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
    ```

- 메세지 가져오기  
    ```shell
    $> kafka_2.11-2.0.0/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
    ```

# 좀 더 이해하기
## 😵 Producer 비율이 Consumer 비율보다 높거나 Consumer의 처리속도가 느리다면 ???


## 👉 Topic Partition을 활용하거나 Topic을 잘 설계해야 함(세분화)


## Topic Partition 
- Topic은 하나의 머신에선 확장성이 없기 때문에 Topic Partition이라는 개념이 나옴
- <span style="font-weight:bold">Topic은 여러 Partition으로 분할될 수 있음</span>
- Partition 데이터는 클러스터의 다른 머신에 저장될 수 있음
- 각 파티션은 소비자 그룹의 다른 Consumer에 의해 소비되어 질 수 있음
- 기본적으로 Producer는 라운드로빈 방식으로 파티션에 메세지를 분배함
- 커스텀 파티셔너(비지니스 로직)로 이를 바꿀 수 있음

## Offset란? 
- 파티션안에 데이터 위치를 유니크한 숫자로 표시한 것 
- 컨슈머는 자신이 어디까지 데이터를 가져갔는지 offset을 이용해서 관리함

## Topic Partition 메세지 순서 이해하기
### Partition이 하나인 경우

### Partition이 여러 개인 경우

> **카프카는 각각의 Partition에 대해서만 순서를 보장함**    
Partition이 1개인 경우에는 Producer가 보낸 순서대로 가져올 수 있지만,  
Partition이 여러 개일 경우 Producer가 보낸 순서대로 메시지를 가져올 수 없음

## Consumer Group
- Consumer Instance들(프로세스, 서버)을 대표하는 그룹 

### Consumer Group이 왜 필요한가?
- 가용성 (Availability)
- Consumer Group 별로 offset 관리를 하기 때문에

### 만약에 Consumer Group이 없다면???

### Consumer Group이 있을 때

## Custom Partitioner 구현하기
1. 다수의 Partition을 가지는 Topic 만들기
2. Producer Application 실행
3. Consumer Application들을 실행해서 작업이 잘 분배되는지 확인하기

# 어떻게 활용할 것인가?

## Kafka를 사용 중인 회사

#### runs in production in thousands of companies

## Use Case
- Kafka Messaging
- Website Activity Tracking
- Kafka Metrics
- Kafka Log Aggregation
- Stream Processing
- Kafka Event Sourcing
- Commit Log

## Kafka Event Sourcing
#### Building Event-driven Microservices Using CQRS


## 참고자료
[Apache Kafka](https://kafka.apache.org/)  
[Kafka 운영자가 말하는 처음 접하는 Kafka](https://www.popit.kr/kafka-%EC%9A%B4%EC%98%81%EC%9E%90%EA%B0%80-%EB%A7%90%ED%95%98%EB%8A%94-%EC%B2%98%EC%9D%8C-%EC%A0%91%ED%95%98%EB%8A%94-kafka/)  
[[Apache Kafka] 1. 소개및 아키텍처 정리](http://epicdevs.com/17)  
[Kafka & Couchbase Integration Patterns](https://www.slideshare.net/ManuelHurtado1/kafka-couchbase-integration-patterns)  
[Write An Apache Kafka Custom Partitioner](https://howtoprogram.xyz/2016/06/04/write-apache-kafka-custom-partitioner/)