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

### SCRAM

SCRAM(Salted Challenge Response Authentication Mechanism)은 `username`과 `password`로 이루어진 SASL 메커니즘의 인증 방식이다.

카프카에서는 `SCRAM-SHA-256` , `SCRAM-SHA-512` 두 가지의 방식을 지원한다.

SCRAM 방식은 주키퍼에 자격 증명을 저장하여 사용하므로 자격 증명을 등록하는 과정이 필요로 하다.

```bash
./kafka-configs --zookeeper localhost:2181 --alter --add-config 'SCRAM-SHA-256=[iterations=8192,password=alice-secret],SCRAM-SHA-512=[password=alice-secret]' --entity-type users --entity-name alice

./kafka-configs --zookeeper localhost:2181 --alter --add-config 'SCRAM-SHA-256=[password=admin-secret],SCRAM-SHA-512=[password=admin-secret]' --entity-type users --entity-name admin
```
