---
layout: post
title:  "[ORACLE] 네트워크 환경 구성"
date:   2018-04-19 20:31:00
author: garimoo
categories: Database
---
<br/>
## Oracle Net

![Image](/assets/20180419/1.png)

* 클라이언트 또는 Middle-tier 응용 프로그램에서 Oracle 서버로의 네트워크 연결을 활성화
* 클라이언트 어플리케이션과 데이터베이스 서버 간의 메시지 교환뿐 아니라, 연결을 설정, 유지 관리.
* 클라이언트 컴퓨터에서 Oracle Net은 DB에 대한 어플리케이션 연결의 백그라운드 구성 요소
* 데이터베이스 서버에서 Oracle Net은 `Oracle Net 리스너`를 포함한다.

**리스너**

* 로컬이 아닌 모든 유저 연결을 위한 Oracle Instance의 게이트웨이. 단일 리스너는 수천개의 클라이언트 연결 지원.
## Net 연결 설정

### 이름 분석(Name Resolution)

아래와 같은 연결 정보를 확인하는 절차.

* 리스너가 실행중인 호스트
* 리스너가 모니터중인 포트
* 리스너가 사용 중인 프로토콜
* 리스너가 처리 중인 서비스 이름

### 연결 설정

* user process의 연결 요청이 리스너로 전달.
* 리스너는 CONNECT 패킷을 수신한 후, 이 패킷이 적합한 Oracle Net 서비스 이름을 요청하고 있는지 검사.
* 부적합한 경우 User Process에 오류 코드 전송

### 유저 세션

* CONNECT 패킷이 적합한 서비스 이름을 요청한 경우 리스너는 **연결을 처리할 새 프로세스 생성 : 서버프로세스**
* 리스너는 이 프로세스에 연결해서 user process의 주소 정보를 포함한 초기화 정보 전달
    * 이 때부터 리스너는 연결을 처리하지 않고 서버프로세스로 전달.
* 서버 프로세스는 유저 인증서를 확인, 인증서가 적합한 경우 `세션` 생성.
* Dedicated server 프로세스
    * 어플리케이션을 통해 실행된 모든 SQL문의 구문 분석, 실행
    * SQL문을 수해하는 데 필요한 데이터 블록의 데이터베이스 버퍼 캐시 검사
    * SGA에 데이터 블록이 없는 경우 디스크의 데이터 파일에서 SGA의 데이터베이스 버퍼 캐시로 필요한 데이터 블록 읽음.
    * 정렬 작업 관리 (PGA에 있음)
    * 어플리케이션이 정보를 처리할 수 있는 방식으로 User Process에 결과 반환
    * 감사(audit) 대상에 user process 보고

## Oracle 네트워크 구성 및 관리 도구

* Enterprise Manager: 통합된 환경에서 Oracle Net 서비스 구성, 관리 가능.
* Oracle Net Manager: 로컬 클라이언트나 서버 호스트에서 Oracle Net 서비스 구성하는 GUI 인터페이스 제공.
    * 이름 지정
    * 이름 지정 방식
    * 프로파일
    * 리스너
* Oracle Net Configuration Assistant: 오라클 DB에 대한 수신 프로토콜 주소와 서비스 정보 구성.

## Listener Control

* Oracle Net 리스너는 `lsnrctl` 유틸리티 사용해서 제어 가능.

### Listener Control 유틸리티

Instance가 시작되면 리스너 프로세스가 오라클 데이터베이스에 대한 통신 경로 설정. 이 후 리스너가 디비 연결 요청 받아들일 수 있다.

* 리스너 시작
* 리스너 정지
* 리스너 상태 검사
* 구성 파일 파라미터에서 리스너 다시 초기화
* 여러 리스너 동적 구성
* 리스너 암호 변경

![Image](/assets/20180419/2.png)

### SRVCTL

`SRVCTL`을 이용해 Oracle Restart로 관리되는 모든 리스너를 시작, 정지, 상태를 볼 수 있음.

```
srvctl start listener
srvctl stop listener
srvctl status listener [-l mylistener]
```

## 이름 지정 방식

### Easy Connect

![Image](/assets/20180419/3.png)

* 연결 문자열의 일부로서 Oracle Net 연결에 필요한 모든 정보 제공 가능.
* `<username>/<password>@<hostname>:<listener port>/<service name>`

### 로컬 이름 지정

![Image](/assets/20180419/4.png)

* 클라이언트 쪽 이름 분석(Names Resolution) 파일 필요
* 모든 Oracle Net 프로토콜 지원
* 짧은 alias만 기억하면 돼서 쉽다.
* tnsnames.ora 에 저장됨.
* Oracle Net 서비스 구성이 자주 변경되지 않는 조직에 적합.

### 디렉토리 이름 지정

![Image](/assets/20180419/5.png)

* 새 서비스 이름이 LDAP 디렉토리에 추가되는 즉시 유저가 연결 가능.
* Oracle Net 서비스 구성이 자주 변경되는 구성에 적합.

### 외부 이름 지정 방식

![Image](/assets/20180419/6.png)

* 지원되는 비오라클 이름 지정 서비스 사용
    * NIS(Network Information Service) 외부 이름 지정
    * DCE(Distributed Computing Environment)의 CDS(Cell Directory Service)

## 유저 세션

### Dedicated Server 프로세스

![Image](/assets/20180419/7.png)

* dedicated server 프로세스에서는 한 개의 서버 프로세스에 한 개의 user process가 연결.
* 각 서버 프로세스는 CPU clock 및 메모리를 포함하는 시스템 리소스 사용
* 로드가 많아지면 시스템에 악영향을 끼칠 수 있고, 이 때 두가지의 해결책이 있다.
    * 메모리, CPU 용량 늘려 시스템 리소스 증가
    * Oracle Shared Server 프로세스 구조 사용

### Shared Server 프로세스

![Image](/assets/20180419/8.png)

* 각 서비스에는 디스패처 프로세스가 한 개(일반적으로 두개 이상씩) 있음.
* 연결 요청이 도착하면 서버 프로세스를 생성하지 않고, 디스패처에 대한 연결 로드(동시 연결 수)와 함께 각 서비스 이름에 사용할 수 있는 디스패처 리스트를 유지 관리.
* 단일 디스패처는 수백 개의 유저 세션 관리
* 디스패처는 유저 요청 작업을 실제로 처리하지 않고, SGA의 Shared Pool에 위치한 Common Queue에 유저 요청 전달.
* Shared Server 프로세스는 Common Queue에서 요청을 가져와 완료될 때까지 처리하여 Dedicated Server 프로세스 작업의 대부분을 넘겨 받음.

### SGA와 PGA

Dedicated Server를 사용하는지, Shared Server를 사용하는지 여부에 따라 SGA와 PGA의 내용이 달라짐.
![Image](/assets/20180419/9.png)
**Shared Server일 때에는 유저 세션 데이터가 SGA에 저장.**

* 모든 SQL문의 텍스트와 구문이 분석된 Form은 SGA에 저장.
* Cursor State는 검색한 행과 같은 SQL문에 대한 런타임 메모리 값을 포함.
* 유저 세션 데이터는 보안과 리소스 사용 정보를 포함.
* 스택 공간은 프로세스 로컬 변수를 포함.

### Shared Server : 연결 풀링

* 데이터베이스 서버는
    * idle한 세션을 물리적으로 만료
    * 논리적으로는 계속 남겨둠
    * 해당 세션에서 다음 요청이 올 때 자동으로 물리적 세션을 설정.
    * 웹 어플리케이션이 기존 하드웨어를 사용하여 `더 많은 동시 유저 수용` 가능하다.

### Shared Server를 사용하지 않는 경우

* shared server 구조는 메모리를 효율적으로 사용할 수 있는 모델이지만, 모든 연결에 적합하지는 않다.
**단점**
* Common Request Queue 처리
* 많은 유저가 디스패처 Response Queue를 공유할 수 있으므로
    * Shared Server는 _웨어하우스 query_ 또는 _일괄 처리_ 와 같은 **큰 데이터 집합을 다루어야 하는 작업**과 함께 수행하면 제대로 작동하지 않음.

**사용하지 않는 경우**

* Instance 시작 및 종료
* 테이블스페이스, 데이터 파일 생성
* 인덱스 및 테이블 유지 관리
* 통계 분석
* DBA 수행 작업
    * 모든 DBA 세션은 Dedicated Server를 선택해야 함.

## 데이터베이스 간의 통신 구성

* 데이터베이스 링크: 다른 데이터베이스의 객체에 액세스 할 수 있도록 하는 데이터베이스의 스키마 객체.
    * 비오라클 시스템에 액세스하려면 \_Oracle Heterogeneous Services\_를 사용해야 함\.
* **Pricate** 데이터베이스 링크를 생성하려면 **CREATE DATABASE LINK** 시스템 권한을 가져야 함.
* **Public** 데이터베이스 링크를 생성하려면 **CREATE PUBLIC DATABASE LINK** 시스템 권한을 가져야 함.

```
CREATE DATABASE LINK <remote_global_name>
CONNECT TO <user> IDENTIFIED BY <pwd>
USING '<connect_string_for_remote_db>';
```

* 데이터베이스 링크를 생성한 후에는 이를 사용하여 다른 데이터베이스의 테이블 및 뷰를 참조 가능.

```
CONNECT hr/hr@orc1;

CREATE DATABASE LINK remote
CONNECT TO HR IDENTIFIED BY HR
USING 'REMOTE_ORCL';

SELECT * FROM employees@remote
```
