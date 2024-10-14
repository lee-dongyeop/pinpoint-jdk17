# pinpoint-jdk17

- Pinpoint APM 구축기(Using JDK 17) - [참고자료](https://coor.tistory.com/66)
- Mac OS를 기준으로 작성되었습니다.

<Br/>

## 1. HBase
### 1-1. HBase 설치
```bash
curl -O https://archive.apache.org/dist/hbase/2.5.8/hbase-2.5.8-bin.tar.gz
tar xzf hbase-2.5.8-bin.tar.gz
```

### 1-2. HBase 실행
```bash
ln -s hbase-2.5.8 hbase
hbase/bin/start-hbase.sh

# JAVA_HOME이 설정되어있지 않을 경우, 아래와 같이 현재 터미널 세션에서만 설정도 가능.
export JAVA_HOME=$(/usr/libexec/java_home -v 17)
```

### 1-3. HBase 테이블 생성
```bash
curl -O https://raw.githubusercontent.com/pinpoint-apm/pinpoint/master/hbase/scripts/hbase-create.hbase
hbase/bin/hbase shell ./hbase-create.hbase\n
```

<br/>

## 2.  Collector
### 2-1. Collector 설치 
```bash
curl -O https://github.com/pinpoint-apm/pinpoint/releases/tag/v3.0.0/pinpoint-collector-3.0.0-exec.jar
```

### 2-2. Collector 실행
```bash
chmod +x pinpoint-collector-3.0.0-exec.jar
nohup java -jar -Dpinpoint.zookeeper.address=localhost pinpoint-collector-3.0.0-exec.jar >/dev/null 2>&1 &

# 본인의 경우 jar 파일 실행에 오류가 발생하였고, 웹 브라우저에서 아래 주소에 직접 접속하여 다운로드 하였음.
# https://github.com/pinpoint-apm/pinpoint/releases/tag/v3.0.0/pinpoint-collector-3.0.0-exec.jar
```

<br/>

## 3. Web
### 3-1. Web 설치
```bash
curl -O https://github.com/pinpoint-apm/pinpoint/releases/tag/v3.0.0/pinpoint-web-3.0.0-exec.jar
```

### 3-2. Web 실행
```bash
chmod +x pinpoint-web-3.0.0-exec.jar
nohup java -jar -Dpinpoint.zookeeper.address=localhost pinpoint-web-3.0.0-exec.jar >/dev/null 2>&1 &\n

# 본인의 경우 jar 파일 실행에 오류가 발생하였고, 웹 브라우저에서 아래 주소에 직접 접속하여 다운로드 하였음.
# https://github.com/pinpoint-apm/pinpoint/releases/tag/v3.0.0/pinpoint-web-3.0.0-exec.jar
```

<br/>

## 4. Agent
### 4-1. Agent 설치
```bash
curl -O https://github.com/pinpoint-apm/pinpoint/releases/tag/v3.0.0/pinpoint-agent-3.0.0.tar.gz
tar xvzf pinpoint-agent-3.0.0.tar.gz
```

### 4-2. Agent 설정
```bash
# 이동
cd pinpoint-agent-3.0.0

# config 파일 수정
vi pinpoint-root.config

# ip와 rate 수정
profiler.transport.grpc.collector.ip={pinpoint ip} # default : 127.0.0.1이므로 Collector가 설치된 서버의 공인IP 입력
profiler.sampling.counting.sampling-rate=100       # default : 100 (=100% 전송)
```

### 4-3. Agent와 함께 Spring Boot 실행
- `-Dpinpoint.agentId`
  - Pinpoint 에이전트의 ID를 설정하는 옵션입니다.
  - 이 ID는 고유한 에이전트 인스턴스를 식별하는 데 사용되며, 동일한 서버나 여러 서버에서 여러 에이전트를 구분할 수 있습니다.
- `-Dpinpoint.applicationName`
  - 모니터링할 애플리케이션의 이름을 지정합니다.
  - Pinpoint UI에서 해당 애플리케이션을 구분하기 위해 사용됩니다.

```bash
nohup java -jar -javaagent:/{$PINPOINT_AGENT_PATH}/pinpoint-agent-3.0.0/pinpoint-bootstrap-3.0.0.jar \
-Djasypt.encryptor.password={$JASYPT_PASSWORD} -Dpinpoint.agentId=local-MG-health \
-Dpinpoint.applicationName=local-MG-health-01 \
{$JAR_PATH}/songareeit.api.health.jar > nohup.out 2>&1 &
```

<br/>

## 5. 실행 결과
<img width="1740" alt="스크린샷 2024-10-11 오후 3 10 02 1" src="https://github.com/user-attachments/assets/9c8adac3-d2a6-4f8f-b290-54cb650d7acf">

<img width="1840" alt="스크린샷 2024-10-11 오후 3 14 41" src="https://github.com/user-attachments/assets/c2a88f88-d8ce-4b61-b8c5-e4a63c374d0d">

<br/>

## 6. 남은 할일
- Insepctor 활성화를 위한 Flink 연동
- 계정(Administration) 추가 및 관리
- 알림 추가 및 관리
