---
title: Kafka 인증 Scram
date: 2022-10-21
categories:
- kafka
tags:
- kafka
- auth
---

카프카의 인증 방식은 `SCRAM, PLAIN, OAUTHBEARER, GSSAPI` 가 있다. 그 중 이번에는 SCRAM 방식을 사용해 카프카 인증을 적용해 보려고한다.

## SCRAM

SCRAM(Salted Challenge Response Authentication Mechanism)은 `username`과 `password`로 이루어진 SASL 메커니즘의 인증 방식이다.

카프카에서는 `SCRAM-SHA-256` , `SCRAM-SHA-512` 두 가지의 방식을 지원한다.

SCRAM 방식은 주키퍼에 자격 증명을 저장하여 사용하므로 자격 증명을 등록하는 과정이 필요로 하다.


## 브로커 실행중 SCRAM 계정을 등록하는 방법

`브로커가 동작중` 계정정보를 등록해야한다면 아래의 방법을 이용해 등록할 수 있다.

만약 카프카를 처음 설정하고, 동작한다면 사전에 계정정보를 주키퍼에 등록할 수 있다. 아래의 __브로커 멈춤 상태에 SCRAM 계정을 등록하는 방법__ 을 이용하고 2번 부터 시작하면 문제 없이 할 수 있다. 


1. kafka-configs.sh를 이용한 계정 등록

kafka-configs를 통해서 계정 정보를 등록할 수 있다. 아래의 가이드를 참고해 계정을 등록해 보자.

`SCRAM-SHA-256` 및 `SCRAM-SHA-512` 에 대한 패스워드를 지정해주고 마지막에 아이디 정보를 넣는다.

```bash
./kafka-configs.sh --bootstrap-server localhost:9092 --alter --add-config 'SCRAM-SHA-256=[password=admin-secret],SCRAM-SHA-512=[password=admin-secret]' --entity-type users --entity-name admin
Completed updating config for user admin.
```

그리고 카프카의 서버 로그를 보면 아래와 같이 admin 계정이 성공적으로 만들어진 것을 확인할 수 있다.

```bash
[2022-10-22 00:13:36,451] INFO Processing notification(s) to /config/changes (kafka.common.ZkNodeChangeNotificationListener)
[2022-10-22 00:13:36,456] INFO Processing override for entityPath: users/admin with config: HashMap(SCRAM-SHA-512 -> [hidden], SCRAM-SHA-256 -> [hidden]) (kafka.server.DynamicConfigManager)
[2022-10-22 00:13:36,457] INFO Removing PRODUCE quota for user admin (kafka.server.ClientQuotaManager)
[2022-10-22 00:13:36,458] INFO Removing FETCH quota for user admin (kafka.server.ClientQuotaManager)
[2022-10-22 00:13:36,458] INFO Removing REQUEST quota for user admin (kafka.server.ClientRequestQuotaManager)
[2022-10-22 00:13:36,458] INFO Removing CONTROLLER_MUTATION quota for user admin (kafka.server.ControllerMutationQuotaManager)
```

주키퍼 `zkCli.sh` 에 /{cluster_name}/config/users로 접근해보면 아이디 목록을 확인해 볼 수 있다.

```bash
[zk: localhost:2181(CONNECTED) 1] ls /test/config/users
[admin]
```

2. server.properties 설정

인증도 걸었겠다! 그렇다면 topic을 생성해보자!

```bash
# 생성 명령어
./kafka-topics.sh --bootstrap-server localhost:9092 --create --topic test

# List 확인
./kafka-topics.sh --bootstrap-server localhost:9092 --list         
test
```

인증을 걸었는데 이상하게도 아무 문제없이 토픽이 생성되고 리스트를 확인할 수 있다..

사실 아직 계정 정보만 주키퍼에 등록했을 뿐 카프카에는 아무런 인증 정보를 넣지 않았다.

단일 카프카를 사용하면 server.properties 변경을 위해서 모두 내려야한다.

하지만 3개 이상의 브로커를 사용하면 하나씩 server.properties를 업데이트 해줌으로써 무중단으로 사용이 가능할 것 같다. (해보지는 않았다.)

그럼 server.properties를 설정해 보자.

```bash
# 변경
listeners=SASL_PLAINTEXT://localhost:9092
sasl.enabled.mechanisms=SCRAM-SHA-256
sasl.mechanism.inter.broker.protocol=SCRAM-SHA-256
security.inter.broker.protocol=SASL_PLAINTEXT
#inter.broker.listener.name=SASL_PLAINTEXT
listener.name.sasl_plaintext.scram-sha-256.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required \
   username="admin" \
   password="admin-secret";
```

`listeners` : 기존 PLAINTEXT를 SASL_PLAINTEXT로 사용함으로 인증 프로토콜을 사용한다.
`sasl.enabled.mechanisms` : SCRAM-SHA-256을 활성화 한다.
`sasl.mechanism.inter.broker.protocol` : 브로커 통신 메커니즘을 정한다.
`security.inter.broker.protocol` : 브로커 통신 프로토콜을 정한다.
`listener.name.sasl_plaintext.scram-sha-256.sasl.jaas.config` : scram login 모듈을 설정한다. 여기서 username,password를 작성해준다.

이렇게 간단하게 server.properties를 작성하면 인증을 작동하는데 준비가 모두 끝났다.


3. 테스트

그리고 위에서 사용했던 topic 리스트를 출력하는 명령을 날리면 아래와 같이 인증이 되지 않은 오류 로그를 볼 수 있다.

```bash
[2022-10-22 00:32:52,659] INFO [SocketServer listenerType=ZK_BROKER, nodeId=0] Failed authentication with /127.0.0.1 (Unexpected Kafka request of type METADATA during SASL handshake.) (org.apache.kafka.common.network.Selector)
[2022-10-22 00:32:53,071] INFO [SocketServer listenerType=ZK_BROKER, nodeId=0] Failed authentication with /127.0.0.1 (Unexpected Kafka request of type METADATA during SASL handshake.) (org.apache.kafka.common.network.Selector)
[2022-10-22 00:32:53,481] INFO [SocketServer listenerType=ZK_BROKER, nodeId=0] Failed authentication with /127.0.0.1 (Unexpected Kafka request of type METADATA during SASL handshake.) (org.apache.kafka.common.network.Selector)
```

그럼 어떻게 접근을 해야할까? client 접근 설정을 하면된다!

```bash
vi client.properties

# 아래의 내용을 입력
sasl.mechanism=SCRAM-SHA-256
security.protocol=SASL_PLAINTEXT
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required \
  username="admin" \
  password="admin-secret";
  
:wq!
```

위의 설정을 마쳤으면 다시 한 번 토픽 리스트를 출력해 보자.

kafka-topics.sh 에서 client.properties를 적용하기 위해서는 `--command-config {properties_name}`을 지정해 주면 된다.

```bash
./kafka-topics.sh --bootstrap-server localhost:9092 --list --command-config client.properties
test
```

성공적으로 인증을 걸었다!


## 브로커 멈춤 상태에 주키퍼에 등록하는 방법 

```bash
./kafka-configs --zookeeper localhost:2181 --alter --add-config 'SCRAM-SHA-256=[iterations=8192,password=alice-secret],SCRAM-SHA-512=[password=alice-secret]' --entity-type users --entity-name alice

./kafka-configs --zookeeper localhost:2181 --alter --add-config 'SCRAM-SHA-256=[password=admin-secret],SCRAM-SHA-512=[password=admin-secret]' --entity-type users --entity-name admin
```
