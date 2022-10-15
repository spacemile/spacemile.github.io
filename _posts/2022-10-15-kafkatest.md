---
title: Kafka (Zookeeper) 시작
date: 2022-10-15
categories:
- kafka
tags:
- kafka
- data
- zookeeper
---

kafka를 실행하기 위해서는 `zookeeper` 가 필요하다. zookeeper는 kafka의 메타 데이터를 담고 사용되므로 kafka 실행전 필수로 동작 중 이어야한다. zookeeper를 사용하기 싫으면 2.8 버전 이후로 ㅠ추가된 `kraft` 모드를 사용해도 된다. kraft 모드는 zookeeper가 따로 필요하지 않지만 별도의 작업이 필요하다. kraft의 대한 내용은 다음 포스트에 담기로 하고 지금은 zookeeper와 연동을 해보겠다.

> 여기서는 zookeeper와 kafka를 localhost 환경에서 실행하는 것을 목표로 한다.
> zookeeper server 1, kafka server 1 대로 구성한다.
> zookeeper 3.7.1, kafka 3.1.2 환경에서 테스트 되었다.

### Zookeeper 시작

1. 설치
  - 아래의 링크에서 zookeeper를 다운로드 받을 수 있다. 여기선 3.7.1 버전을 다루었다.
  - https://zookeeper.apache.org/releases.html

2. 설정

zookeeper를 다운로드 받고 압축을 해제 해보자.

bin, conf, docs ... 등의 디렉토리가 등장한다.

conf에 `zoo_sample.cfg` 가 있을 것 이다. 이 파일을 수정해서 zookeeper를 설정한다.




### 카프카 설치 및 시작

https://kafka.apache.org/downloads

```
./kafka-server-start.sh [server.properties]
```
