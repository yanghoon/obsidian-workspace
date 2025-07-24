#flink #iceberg

## Architecture

### Flink, Iceberg와 Hadoop 라이브러리 의존성

- **Flink는 기본적으로 Hadoop 없이도 실행 가능**하지만, HDFS 같은 Hadoop 기반 스토리지 접근이나 YARN 리소스 관리 사용 시에는 Hadoop 라이브러리가 필요합니다. Hadoop 라이브러리는 보통 `flink-shaded-hadoop-2-uber` 같은 Uber JAR 형태로 제공되거나 별도 Hadoop 클라이언트 JAR 형태로 포함됩니다.

- **Iceberg Flink Runtime(iceberg-flink-runtime-1.20)** 은 Flink와 Iceberg 통합에 필요한 라이브러리로, Hadoop MapReduce 라이브러리를 필수로 포함하지는 않습니다. 하지만 Iceberg가 내부적으로 Parquet 파일을 처리할 때 Hadoop MapReduce 기반 API(`FileInputFormat` 등)를 사용하므로, Hadoop 관련 라이브러리는 환경에 따라 반드시 포함되어야 합니다.

- Iceberg에서 S3와 같은 오브젝트 스토리지의 Parquet 파일을 읽을 때도 Hadoop 파일 시스템 API와 MapReduce의 입출력 포맷에 의존하기 때문에, **hadoop-mapreduce-client-core 등 Hadoop 라이브러리를 함께 제공해야 정상 작동**합니다.

---

#### 라이브러리 의존 상황 및 필요한 라이브러리

| 상황                        | 필요한 라이브러리 예시                               | 설명                                   |
| ------------------------- | ------------------------------------------ | ------------------------------------ |
| Flink 독립 실행               | Flink Core 라이브러리                           | Hadoop 없이도 동작 가능                     |
| Hadoop 스토리지 접근 (HDFS, S3) | flink-shaded-hadoop-2-uber 또는 하둡 클라이언트 JAR | 파일 시스템 연동 및 Parquet 처리 지원            |
| Iceberg- Flink 통합 사용      | iceberg-flink-runtime, iceberg-aws-bundle  | Iceberg와 Flink 연동, S3 및 Glue 카탈로그 지원 |
| Parquet 파일 입출력            | hadoop-mapreduce-client-core               | Parquet 파일 읽기 위한 MapReduce API       |

- S3 사용 시 `iceberg-aws-bundle`이 AWS 인증 및 S3 접속 기능을 제공합니다.
- Hadoop 관련 JAR들이 Flink 컨테이너 내에 없으면 파일 및 Parquet 작업 중 클래스 오류가 발생할 수 있습니다.

---

#### Flink Docker 이미지에서 Hadoop 의존성 설정 방법

1. **플러그인 디렉토리 설정**  
   - `FLINK_PLUGINS_DIR` 환경변수에 플러그인 JAR들이 위치한 경로 지정(예: `/opt/flink/plugins`)  
   - Docker Compose 또는 `docker run` 시 호스트의 플러그인 디렉토리를 컨테이너에 마운트합니다.

2. **Hadoop 라이브러리 포함**  
   - Flink 기본 Docker 이미지에는 Hadoop 클라이언트 라이브러리가 없음  
   - `flink-shaded-hadoop-2-uber.jar` 또는 필수 하둡 클라이언트 JAR들을 직접 플러그인 디렉토리에 추가  
   - 또는 Hadoop이 포함된 커스텀 Docker 이미지 사용 가능

3. **환경변수로 플러그인 활성화 제어**  
   - `ENABLE_BUILT_IN_PLUGINS=true` (기본값)로 내장 플러그인을 활성화  
   - Hadoop/S3 관련 플러그인 JAR 별도 추가 후 플러그인 폴더에 위치시키기  

4. **AWS S3 연동 시 필요한 설정**  
   - 컨테이너에 `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION` 환경변수 지정  
   - Iceberg 설정 파일에 S3 관련 옵션 및 endpoint 설정  

#### Iceberg Hadoop 의존성 추가 (in Flink Docker Container)

* [Elephant Below the Waterline: Hadoop Dependencies in Iceberg](https://exceptionfactory.com/posts/2024/11/08/elephant-below-the-waterline-hadoop-dependencies-in-iceberg/)
* [examples / catalogs / flink-iceberg-hive / flink / Dockerfile](https://github.com/decodableco/examples/blob/main/catalogs/flink-iceberg-hive/flink/Dockerfile)

```bash
mkdir -p ./lib/hadoop && \
    curl https://repo1.maven.org/maven2/org/apache/commons/commons-configuration2/2.1.1/commons-configuration2-2.1.1.jar -o ./lib/hadoop/commons-configuration2-2.1.1.jar && \
    curl https://repo1.maven.org/maven2/commons-logging/commons-logging/1.1.3/commons-logging-1.1.3.jar -o ./lib/hadoop/commons-logging-1.1.3.jar && \
    curl https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-auth/3.3.4/hadoop-auth-3.3.4.jar -o ./lib/hadoop/hadoop-auth-3.3.4.jar && \
    curl https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-common/3.3.4/hadoop-common-3.3.4.jar -o ./lib/hadoop/hadoop-common-3.3.4.jar && \
    curl https://repo1.maven.org/maven2/org/apache/hadoop/thirdparty/hadoop-shaded-guava/1.1.1/hadoop-shaded-guava-1.1.1.jar -o ./lib/hadoop/hadoop-shaded-guava-1.1.1.jar && \
    curl https://repo1.maven.org/maven2/org/codehaus/woodstox/stax2-api/4.2.1/stax2-api-4.2.1.jar -o ./lib/hadoop/stax2-api-4.2.1.jar && \
    curl https://repo1.maven.org/maven2/com/fasterxml/woodstox/woodstox-core/5.3.0/woodstox-core-5.3.0.jar -o ./lib/hadoop/woodstox-core-5.3.0.jar && \
    curl https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-hdfs-client/3.3.4/hadoop-hdfs-client-3.3.4.jar -o ./lib/hadoop/hadoop-hdfs-client-3.3.4.jar && \
    curl https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-mapreduce-client-core/3.3.4/hadoop-mapreduce-client-core-3.3.4.jar -o ./lib/hadoop/hadoop-mapreduce-client-core-3.3.4.jar
```
## Deploy

### Flink Docker Compose

#### Usage

* Flink JobManager UI - http://localhost:8081

```bash
docker compose up -d
```

```bash
docker ps
```

#### Docker Compose File

```yaml
services:
  jobmanager:
    image: apache/flink:1.20.2
    environment:
      - JOB_MANAGER_RPC_ADDRESS=jobmanager
      - FLINK_PLUGINS_DIR=/opt/flink/plugins
    ports:
      - "8081:8081"
    volumes:
      - ./plugins:/opt/flink/plugins
      - ./conf/flink-conf.yaml:/opt/flink/conf/flink-conf.yaml:ro
    command: jobmanager

  taskmanager:
    image: apache/flink:1.20.2
    profiles:
      - task
    environment:
      - JOB_MANAGER_RPC_ADDRESS=jobmanager
      - FLINK_PLUGINS_DIR=/opt/flink/plugins
    depends_on:
      - jobmanager
    volumes:
      - ./plugins:/opt/flink/plugins
      - ./conf/flink-conf.yaml:/opt/flink/conf/flink-conf.yaml:ro
    command: taskmanager
    deploy:
      replicas: 1
```

#### Flink Config File

```yaml
# Path : conf/flink-conf.yaml
# Iceberg REST Catalog 설정 예시
iceberg.catalog-type: rest
iceberg.catalog-impl: org.apache.iceberg.rest.RESTCatalog
iceberg.uri: http://localhost:8181
iceberg.warehouse: s3://your-bucket/warehouse

# AWS S3 인증 및 endpoint (필요 시)
s3.access-key: YOUR_ACCESS_KEY
s3.secret-key: YOUR_SECRET_KEY
s3.endpoint: https://maxio.com
s3.path-style-access: true
```

#### References

- [Building a Local Flink Environment with Docker and Submitting Your First Job (tim santeford)](https://www.timsanteford.com/posts/building-a-local-flink-environment-with-docker-and-submitting-your-first-job/)  
- [Apache Flink SQL Client on Docker (DEV Community)](https://dev.to/ftisiot/apache-flink-on-docker-4kij)  
- [Setup Local Development Environment for Apache Flink and Spark Using EMR Container Images (Jaehyeon Kim)](https://jaehyeon.me/blog/2023-12-07-flink-spark-local-dev/)  

