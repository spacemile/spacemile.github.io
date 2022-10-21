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

만약 카프카를 처음 설정하고, 동작한다면 사전에 계정정보를 주키퍼에 등록할 수 있다. 아래의 ` 브로커 멈춤 상태에 SCRAM 계정을 등록하는 방법 ` 을 이용하고 ?번 부터 시작하면 문제 없이 할 수 있다. 


1. kafka-configs.sh를 이용한 계정 등록

kafka-configs를 통해서 계정 정보를 등록할 

```bash
./kafka-configs.sh --bootstrap-server localhost:9092 --alter --add-config 'SCRAM-SHA-256=[password=admin-secret],SCRAM-SHA-512=[password=admin-secret]' --entity-type users --entity-name admin
Completed updating config for user admin.
```

kafka server log

```bash
[2022-10-22 00:13:36,451] INFO Processing notification(s) to /config/changes (kafka.common.ZkNodeChangeNotificationListener)
[2022-10-22 00:13:36,456] INFO Processing override for entityPath: users/admin with config: HashMap(SCRAM-SHA-512 -> [hidden], SCRAM-SHA-256 -> [hidden]) (kafka.server.DynamicConfigManager)
[2022-10-22 00:13:36,457] INFO Removing PRODUCE quota for user admin (kafka.server.ClientQuotaManager)
[2022-10-22 00:13:36,458] INFO Removing FETCH quota for user admin (kafka.server.ClientQuotaManager)
[2022-10-22 00:13:36,458] INFO Removing REQUEST quota for user admin (kafka.server.ClientRequestQuotaManager)
[2022-10-22 00:13:36,458] INFO Removing CONTROLLER_MUTATION quota for user admin (kafka.server.ControllerMutationQuotaManager)
```

zookeeper

```bash
[zk: localhost:2181(CONNECTED) 1] ls /test/config/users
[admin]
```

topic 아직은 생성됨

```bash
./kafka-topics.sh --bootstrap-server localhost:9092 --create --topic test

./kafka-topics.sh --bootstrap-server localhost:9092 --list         
test

```

server.properties

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

topic 위에서 토픽 list 명령어 날리면 kafka에서 이런 반응이 옴

```bash
[2022-10-22 00:32:52,659] INFO [SocketServer listenerType=ZK_BROKER, nodeId=0] Failed authentication with /127.0.0.1 (Unexpected Kafka request of type METADATA during SASL handshake.) (org.apache.kafka.common.network.Selector)
[2022-10-22 00:32:53,071] INFO [SocketServer listenerType=ZK_BROKER, nodeId=0] Failed authentication with /127.0.0.1 (Unexpected Kafka request of type METADATA during SASL handshake.) (org.apache.kafka.common.network.Selector)
[2022-10-22 00:32:53,481] INFO [SocketServer listenerType=ZK_BROKER, nodeId=0] Failed authentication with /127.0.0.1 (Unexpected Kafka request of type METADATA during SASL handshake.) (org.apache.kafka.common.network.Selector)
[2022-10-22 00:32:53,892] INFO [SocketServer listenerType=ZK_BROKER, nodeId=0] Failed authentication with /127.0.0.1 (Unexpected Kafka request of type METADATA during SASL handshake.) (org.apache.kafka.common.network.Selector)
[
```

client 설정을 해야함

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

그리고 토픽 리스트 찍어보기

```bash
./kafka-topics.sh --bootstrap-server localhost:9092 --list --command-config client.properties
test
```


## 브로커 멈춤 상태에 주키퍼에 등록하는 방법 

```bash
./kafka-configs --zookeeper localhost:2181 --alter --add-config 'SCRAM-SHA-256=[iterations=8192,password=alice-secret],SCRAM-SHA-512=[password=alice-secret]' --entity-type users --entity-name alice

./kafka-configs --zookeeper localhost:2181 --alter --add-config 'SCRAM-SHA-256=[password=admin-secret],SCRAM-SHA-512=[password=admin-secret]' --entity-type users --entity-name admin
```
