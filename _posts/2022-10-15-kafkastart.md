---
title: Kafka 시작
date: 2022-10-15
categories:
- kafka
tags:
- kafka
---

이번에는 kafka를 zookeeper와 연동해서 실행하는 방법을 소개한다. zookeeper를 사전에 설치하지 않았다면 zookeeper 설치 글을 보고 설치부터 하고 와야한다.

### 카프카 설치 및 시작

#### 1. 설치

설치는 아래의 링크에서 가능하다.

https://kafka.apache.org/downloads

#### 2. 설정

다운로드 받은 kafka 압축을 해제하면,

zookeeper와 같이 bin, config ... 등의 디렉토리가 존재한다.

kafka의 서버 세팅은 `config/server.properties` 에서 한다. 

##### listener 설정

kafka를 localhost 환경에서 동작하게 할 것 이므로 listener 설정을 한다.

우선 server.properties를 열고 listener 설정을 추가 하거나 주석을 해제한후 값을 다음과 같이 입력해 주자.

통신이 `PLAINTEXT` 그리고 localhost의 9092를 통해서 이루어진다는 뜻 이다.

```properties
listeners=PLAINTEXT://localhost:9092
```

PLAINTEXT는 별도의 인증을 사용하지 않는 프로토콜이며 인증을 사용하기 위해서는 `SASL_PLAINTEXT` 과 같은 프로토콜을 입력해야하지만 다른 추가적인 설정을 해야할 것이 많으므로 다음번에 글을 작성 하겠다.

##### zookeeper.connect 설정

그리고 kafka를 사용하기 위해서 zookeeper를 설치 하였다.

zookeeper 역시 localhost에 설치를 하였기 때문에 아래처럼 설정한다.

zookeeper의 기본 client port는 2181 이다.

```properties
zookeeper.connect=localhost:2181
```

2가지 설정을 함으로써 localhost kafka를 사용할 수 있게 되었다.


#### 3. 실행

kafka의 bin 디렉토리에 들어가보자. 다양한 스크립트들이 있다.

여기서 `kafka-server-start.sh` 를 이용해 서버를 시작할 것이다.

아래의 명령어처럼 config/server.properties를 지정해서 kafka를 실행할 수 있다.

--daemon 옵션을 추가하면 백그라운드 환경에서 동작한다.

```
./kafka-server-start.sh [server.properties]
```

위 명령어를 실행하고 아래의 로그가 보인다면 실행에 성공한 것이다.

```
INFO [KafkaServer id=0] started (kafka.server.KafkaServer)
```

### Topic Test

kafka를 설치하고 실행하였으면 간단한 명령어로 동작을 확인할 수 있다.

아래의 명령어는 Topic을 생성하는 명령어이다. bin 디렉토리에서 실행해 보자.

test라는 이름의 Topic을 생성하는 명령어다.

```
kafka-topics.sh --bootstrap-server localhost:9092 --create --topic test
```

`Created topic test.` 메시지가 뜨면 정상적으로 Topic을 생성한 것이다.

생성된 Topic을 확인해 보자.

```
kafka-topics.sh --bootstrap-server localhost:9092 --list

test
```

위와 같이 test 토픽이 생성된 것을 확인할 수 있다.

### Producer

bin 디렉토리에서 `kafka-console-producer.sh` 를 사용하면 producer도 테스트가 가능하다.

아래의 명령어는 만들어진 test Topic에 producing 할 수 있는 명령어다.

```
./kafka-console-producer.sh --bootstrap-server localhost:9092 --topic test
```

이 명령어를 치면 `>` 문자가 나타나는데 여기서 Topic에 메시지를 전송할 수 있다.

```
> hello kafka
>
```

전송은 되었는 것 같은데 어디서 확인할 수 있을까? 다음은 consumer 이다.

### Consumer

consumer는 producer가 전송한 로그를 읽어오는데 사용된다.

`kafka-console-consumer.sh`를 
consumer는 producer가 전송한 로그를 읽어오는데 사용된다.ㅅㅏ용
consumer는 producer가 전송한 로그를 읽어오는데 사용된다.


