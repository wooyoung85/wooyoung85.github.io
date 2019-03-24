> Spark 2.4.0 기준으로 문서 작성하였으니 참고하시기 바랍니다 ^^

# Apache Spark란?
> Apache Spark™ is a unified analytics engine for large-scale data processing.

# 특징
### Speed
- DAG scheduler, query optimizer 등을 사용하여 빠른 데이터 처리 속도를 제공
- In‑Memory 컴퓨팅 (물론 Disk기반도 가능)
- Hadoop보다 100배나 빠르다고 주장함
### Ease of Use
- Java, Scala, Python, R, SQL 등의 언어를 사용하여 분산 처리 어플리케이션을 빠르게 만들 수 있음
### Generality(범용성)
- Spark이 제공하는 SQL과 DataFrames, MLlib(머신러닝), GraphX, Spark Streaming 등의 라이브러리들을 하나의 어플리케이션에서 완벽하게 조합하여 사용할 수 있음
   
### Runs Everywhere
- Spark 실행환경  
standalone cluster mode를 기본적으로 사용할 수 있고, EC2, Hadoop, YARN, Apache Mesos, Kubernetes 등을 활용하여 실행 가능함
- Data Sources  
HDFS, Alluxio, Apache Cassandra, Apache HBase, Apache Hive 등 수많은 Data Sources에 접근 가능함

# 무엇을 할 수 있나?
- 대용량 데이터를 실시간 혹은 배치로 ETL(Extract, Transform, Load), 분석, 머신러닝, 그래프 처리 등을 할 수 있음
- 데이터 양이 많으면서 다양한 형태의 데이터를 분석해야 할 경우 Spark을 사용하기 매우 적합함  
~~(단순히 oracle db의 데이터를 분석할꺼면 그냥 쿼리를 사용해 분석하면 됨)~~

# Cluster Mode Overview
Spark을 Local(사용자 PC) Thread에서 실행할 수도 있지만 Cluster 환경에서 실행하는 것이 일반적임 

### 구성요소
- Spark 어플리케이션은 Cluster에서 독립적인 프로세스들의 집합으로 실행되고 이들은 Driver Program의 Spark Context를 통해 조정됨
- Cluster 환경에서 실행하기 위해 Spark Context는 몇 가지 타입의 cluster manager(어플리케이션 간 리소스를 할당)들에 접속할 수 있음
- 일단 연결되면, Spark는 Cluster의 노드에서 Executor(사용자가 만든 애플리케이션에 대한 계산을 실행하고 데이터를 저장하는 프로세스)를 요구함
- 다음으로, 사용자가 작성한 어플리케이션 코드를 Executor에게 보냄
- 마지막으로, SparkContext가 Executor들에게 task들을 보내 실행함

### 주요 용어
| 용어 | 의미 | 
|---|---|
| **Application** | Spark에 기반한 사용자 프로그램<br> Driver Program과 Cluster 상의 Executor들로 구성되어 있음 |
| **Application jar** | 사용자의 Spark Application이 담겨있는 jar 파일<br> 경우에 따라서 uber jar(Application의 의존성을 포함한 jar)를 만들기 원할 수도 있음<br> 사용자의 jar 파일은 절대 Hadoop 또는 Spark 라이브러리를 포함하지 않고 런타임 시에 추가됨 |
| **Driver Program** | main 함수가 실행되는 프로세스이고 SparkContext를 생성함 |
| **Cluster Manager** | 클러스터로부터 리소스를 얻기 위한 외부 서비스 |
| **Deploy mode** | Driver Process가 실행되는 위치를 구별함<br> **Cluster mode** : Framework이 Cluster 내부에서 Driver를 실행함<br> **Client mode** : 제출자가 Cluster 밖에서 Driver를 실행함 |
| **Worker Node** | 클러스터에서 응용 프로그램 코드를 실행할 수있는 모든 노드 |
| **Executor** | Application 실행을 위해 worker node에서 실행되는 프로세스는 작업을 실행하고 데이터를 메모리 또는 디스크 저장소에 저장함<br> 각 어플리케이션은 자신의 executor들을 가짐 |
| **Task** | 하나의 Executor로 보내지게 될 작업 단위 |
| **Job** | Spark Action(e.g. save, collect)에 의해 생생된 여러 작업으로 이뤄진 병렬 계산 <br>이 용어는 Driver 쪽 log에서 볼 수 있음 |
| **Stage** | Each job gets divided into smaller sets of tasks called stages that depend on each other (similar to the map and reduce stages in MapReduce); you'll see this term used in the driver's logs. |

### Standalone Cluster 실행순서
0. `bin/spark-submit`을 이용하여 Application 제출
1. Driver Program이 실행되고 SparkContext(Spark Cluster와의 연결)가 생성됨
2. Driver가 Cluster Manager에게 Executor 리소스 요청
3. Cluster Manager가 Worker들에게 Executor를 띄우도록 명령함
4. Driver Program이 DAG Scheduling을 통해 작업을 Task 단위로 분할하여 할당된 Executor에게 전송
5. Executor가 여러 스레드에서 Task들을 실행한 후 결과를 Driver Program에 보냄
6. Application이 종료되면 Cluster Manager에게 리소스 반납


## 참고자료
[Apache Spark™ - Unified Analytics Engine for Big Data](https://spark.apache.org/)  
[Overview - Spark 2.4.0 Documentation](https://spark.apache.org/docs/latest/index.html)  
[Understand the Spark Deployment Modes](https://trongkhoanguyen.com/spark/understand-the-spark-deployment-modes/)

