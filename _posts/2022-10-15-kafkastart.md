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




```
./kafka-server-start.sh [server.properties]
```
