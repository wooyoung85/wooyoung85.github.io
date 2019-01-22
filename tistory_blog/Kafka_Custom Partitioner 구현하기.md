> [Kafka 정리](https://wooyoung85.tistory.com/32) 내용을 보신 후 실습해 보시기 바랍니다.

## Custom Partitioner란?
기본적으로 Producer는 RoundRobin 방식으로 파티션에 메세지를 분배하는데 이 방식을 Customizing 하여 특정 Partition에 특정 데이터를 넣어주는 것

## 예제 코드 작성
> Producer는 사용자 정보를 불러와서 UserMessage라는 Topic을 Publish하고 Consumer는 UserMessage Topic을 Subscribe 한다.


1. maven 프로젝트 생성 후 `pom.xml` 파일 아래와 같이 작성
	```xml
	<project xmlns="http://maven.apache.org/POM/4.0.0"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
		<modelVersion>4.0.0</modelVersion>

		<groupId>com.wooyoung</groupId>
		<artifactId>kafka-demo</artifactId>
		<version>0.0.1-SNAPSHOT</version>
		<packaging>jar</packaging>

		<name>kafka-demo</name>
		<url>http://maven.apache.org</url>

		<properties>
			<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
			<maven.compiler.source>1.8</maven.compiler.source>
			<maven.compiler.target>1.8</maven.compiler.target>
		</properties>

		<dependencies>
			<dependency>
				<groupId>org.apache.kafka</groupId>
				<artifactId>kafka-clients</artifactId>
				<version>0.9.0.1</version>
				<scope>provided</scope>
			</dependency>
		</dependencies>
	</project>
	```

2. UserService 구현
- `IUserService`
	```java
	package com.wooyoung.kafka_demo;

	import java.util.List;

	public interface IUserService {

		public Integer findUserId(String userName);
		public List<String> findAllUsers();
	}

	```
- `UserServiceImpl`
	```java
	package com.wooyoung.kafka_demo;

	import java.util.ArrayList;
	import java.util.HashMap;
	import java.util.List;
	import java.util.Map;

	public class UserServiceImpl implements IUserService {

		private Map<String, Integer> usersMap;

		public UserServiceImpl() {
			usersMap = new HashMap<String, Integer>();

			usersMap.put("wooyoung_0", 0);
			usersMap.put("hobi_0", 1);
			usersMap.put("tayo_0", 2);
			usersMap.put("ddiddibbo_0", 3);
			usersMap.put("bbororo_0", 4);
			usersMap.put("dudadakoong_0", 5);
			usersMap.put("wooyoung_1", 0);
			usersMap.put("hobi_1", 1);
			usersMap.put("tayo_1", 2);
			usersMap.put("ddiddibbo_1", 3);
			usersMap.put("bbororo_1", 4);
			usersMap.put("ddudadakoong_1", 5);
			usersMap.put("wooyoung_2", 0);
			usersMap.put("hobi_2", 1);
			usersMap.put("tayo_2", 2);
			usersMap.put("ddiddibbo_2", 3);
			usersMap.put("bbororo_2", 4);
			usersMap.put("dudadakoong_2", 5);
		}

		public Integer findUserId(String userName) {
			return usersMap.get(userName);
		}

		public List<String> findAllUsers() {
			return new ArrayList<String>(usersMap.keySet());
		}
	}

	```

3. Producer 구현

	```java
	package com.wooyoung.kafka_demo;

	import java.io.IOException;
	import java.util.List;
	import java.util.Properties;

	import org.apache.kafka.clients.producer.Callback;
	import org.apache.kafka.clients.producer.KafkaProducer;
	import org.apache.kafka.clients.producer.ProducerRecord;
	import org.apache.kafka.clients.producer.RecordMetadata;

	public class Producer {

		public static void main(String[] args) throws IOException {
			String brokers = "localhost:9092";
			String topic = "UserMessage";
			IUserService userService = new UserServiceImpl();

			Properties configs = new Properties();
			configs.put("bootstrap.servers", brokers);
			configs.put("acks", "all");
			configs.put("retries", 0);
			configs.put("batch.size", 16384);
			configs.put("linger.ms", 1);
			configs.put("buffer.memory", 33554432);
			configs.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
			configs.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
			// 이 부분을 주석처리하고 실행하면 RoundRobin방식으로 파티션에 메세지를 분배함
			configs.put("partitioner.class", "com.wooyoung.kafka_demo.CustomPatitioner");

			// Producer 생성
			KafkaProducer<String, String> producer = new KafkaProducer<String, String>(configs);

			System.out.println("Produces messages");
			List<String> users = userService.findAllUsers();

			for (final String user : users) {
				final String msg = "Hello " + user;
				// Topic 설정하여 메세지 Publish
				// send 메소드에 Callback함수를 인자로 넘길 수 있음
				producer.send(new ProducerRecord<String, String>(topic, user, msg), new Callback() {
					public void onCompletion(RecordMetadata metadata, Exception e) {
						if (e != null) {
							e.printStackTrace();
						}
						System.out.println("Sent:" + msg + ", User: " + user + ", Partition: " + metadata.partition());
					}
				});

			}

			// closes producer
			producer.flush();
			producer.close();
		}
	}

	```

4. Consumer 구현
	```java
	package com.wooyoung.kafka_demo;

	import java.io.IOException;
	import java.util.Arrays;
	import java.util.Properties;

	import org.apache.kafka.clients.consumer.ConsumerRecord;
	import org.apache.kafka.clients.consumer.ConsumerRecords;
	import org.apache.kafka.clients.consumer.KafkaConsumer;

	public class Consumer {
		public static void main(String[] args) throws IOException {
			String brokers = "localhost:9092";
			String groupId = "group01";
			String topic = "UserMessage";

			Properties configs = new Properties();
			configs.put("bootstrap.servers", brokers);
			configs.put("group.id", groupId);
			configs.put("enable.auto.commit", "true");
			configs.put("auto.commit.interval.ms", "1000");
			configs.put("session.timeout.ms", "30000");
			configs.put("auto.offset.reset", "earliest");
			configs.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
			configs.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

			// Consumer 생성
			KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(configs);

			// Subscribe 할 Topic 설정
			consumer.subscribe(Arrays.asList(topic));

			while (true) {
				ConsumerRecords<String, String> records = consumer.poll(100);
				for (final ConsumerRecord record : records) {
					System.out.println("Receive message: " + record.value() + ", Partition: " + record.partition()
							+ ", Offset: " + record.offset());
				}
			}
		}
	}

	```

## 실행 순서
1. Zookeeper / Kafka 실행
2. Topic 생성
3. Producer Application 실행
4. Consumer Application 실행

## 참고자료
[Write An Apache Kafka Custom Partitioner](https://howtoprogram.xyz/2016/06/04/write-apache-kafka-custom-partitioner/)