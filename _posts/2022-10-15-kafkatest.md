---
title: Zookeeper 시작
date: 2022-10-15
categories:
- zookeeper
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

#### 1. 설치

아래의 링크에서 zookeeper를 다운로드 받을 수 있다. 여기선 3.7.1 버전을 다루었다.

https://zookeeper.apache.org/releases.html

#### 2. 설정

zookeeper를 다운로드 받고 압축을 해제 해보자.

bin, conf, logs ... 등의 디렉토리가 등장한다.

1. conf는 zookeeper 관련 설정파일이 들어있다. 서버를 시작하기전에 설정을 다루어야한다.
2. bin에는 zookeeper 관련 바이너리 파일이 들어있다. zkCli에 접속하거나 서버를 실행할 수 있다.

그러면 zookeeper를 설정하고 실행해 보자.

우선 conf 디렉토리를 보자, 그럼 `zoo_sample.cfg` 가 있는 것을 확인할 수 있다. 이 파일을 수정해서 zookeeper를 설정한다. 우리는 localhost 환경에서 zookeeper를 설정할 것이므로
`zoo_sample.cfg` 파일을 `zoo.cfg` 로 복사 만 하면 된다.

```bash
cp zoo_sample.cfg zoo.cfg
```

#### 3. 시작

bin 파일에 가서 `zkServer.sh` 를 찾아보자. zookeeper 서버를 실행하는 스크립트다.

서버의 실행은 아래의 명령어를 통해서 가능하다. zkServer.sh에 config 옵션으로 디렉토리를 지정해 주면 된다. 그리고 마지막에 start, stop 같은 서버 동작 명령을 내려준다.

```bash
./zkServer.sh --config ../conf start
```

그리고 아래의 명령어를 통해서 시작되었는지 찾아보자. logs 디렉토리에서 사용한다. grep으로 started 되었는지만 확인하는 명령어다.

```bash
tail -f [log file name] | grep started
```

아래의 로그처럼 나왔다면 시작이 된 것 이다.

```
INFO  [ProcessThread(sid:0 cport:2181)::PrepRequestProcessor@137] - PrepRequestProcessor (sid:0) started, reconfigEnabled=false
```

그리고 zookeeper 쉘을 통해서 시작되었는지 확인이 가능하다. bin 디렉토리로 이동한다. 그리고 아래의 명령어를 실행한다.

```bash
./zkCli.sh
```

아래처럼 zkCli에서 `ls /` 명령어를 입력해 보자

```
[zk: localhost:2181(CONNECTED) 0] ls /
```

그럼 `/` 위치에 `zookeeper`를 확인가능하다.

log 및 zkCli.sh 를 통해서 zookeeper를 정상적으로 설정 시작이 되었는지 확인해 보았다.


