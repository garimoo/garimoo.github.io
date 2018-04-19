---
layout: post
title:  "[ORACLE] 데이터베이스 유저 보안 관리"
date:   2018-04-19 20:31:00
author: garimoo
categories: Database
---
<br/>
## 데이터베이스 유저 계정

* 고유 username
* 인증 방식
* 기본 테이블스페이스
* 임시 테이블스페이스
* 유저 프로파일: 유저에게 할당되는 리소스 및 암호 제한 집합
* 초기 Consumer Group: 리소스 관리자가 사용
* 계정 상태: "open"상태의 계정에만 액세스 할 수 있다.

### 스키마

* 데이터베이스 유저가 소유하는 데이터베이스 객체의 모음
* 데이터베이스의 데이터를 직접 참조하는 논리적 구조
* 테이블, 뷰, 시퀀스, 내장 프로시저, 동의어, 인덱스, 클러스터 및 데이터베이스 링크가 있다.
* 어플리케이션이 데이터베이스에 생성하는 모든것이 포함됨.

## (미리 정의된) 관리 계정

### SYS

* DBA 롤이 부여
* ADMIN OPTION에 대한 모든 권한 포함.
* 데이터 딕셔너리 소유.
* 연결하려면 데이터베이스 Instance에 대해 `AS SYSDBA`절을 사용, ASM Instance에 대해 `AS SYSASM`을 사용해야 함.
* 권한 있는 유저만이 Instance를 시작 및 종료할 수 있다.

### SYSTEM

* DBA 권한이 없다.
* AQ\_ADMINISTRATOR\_ROLE 및 MGMT\_USER 롤이 부여\.

### DBSNMP

* Enterprise Manager의 관리 에이전트가 데이터베이스를 모니터 및 관리하는 데 사용

### SYSMAN

* Oracle Enterprise Manager의 관리 작업을 수행하는 데 사용

## 관리자 인증

### 운영 체제 보안

* DBA는 파일을 생성하고 삭제하기 위한 OS 권한을 가져야 함.
* 일반 유저는 데이터베이스 파일을 생성, 삭제 할 수 있는 OS 권한을 가질 수 없음.

### 관리자 보안

* SYSDBA, SYSOPER, SYSASM 유저에 대한 연결은 Password File을 사용한 검증을 수행한 이후에만 인증.
* OS 인증이 사용되는 경우 Username과 암호를 사용하지 않는다.

## 권한

특정 유형의 SQL문을 실행하거나 다른 유저의 객체에 액세스 할 수 있는 권한.

### 시스템 권한

특정 데이터베이스에 대한 작업을 수행 가능

* **RESTRICTED SESSION**: 데이터베이스가 제한 모드로 열려 있어도 로그인 가능
* **SYSDBA / SYSOPER**: 데이터베이스에서 종료, 시작, recovery 등 관리 작업 수행 가능
    * STARTUP / SHUTDOWN
    * CREATE SPFILE
    * ALTER DATABASE OPEN/MOUNT/BACKUP
    * ALTER DATABASE ARCHIVELOG
    * ALTER DATABASE RECOVER
    * RESTRICTED SESSION
* **SYSASM**: ASM Instance를 시작, 종료 및 관리 가능
* **DROP ANY object**: 다른 스키마 유저가 소유한 객체를 삭제 가능
* **CREATE, MANAGE, DROP 및 ALTER TABLESPACE**: 테이블스페이스 속성 생성, 삭제 등 테이블스페이스 관리
* **CREATE LIBRARY**: C 라이브러리와 같은 외부 코드 생성 및 호출 가능. PL/SQL에서 실행할 수 있는 임의 코드 라이브러리 생성 가능
* **CREATE ANY DIRECTORY**: 유저가 해당 디렉토리의 외부 프로시저에 액세스 가능(비보안 코드 객체 호출 가능). redo 로그와 같은 데이터베이스 파일에 대해 직접 읽기/쓰기 시도 가능.
* **GRANT ANY OBJECT PRIVILEGE**: 자신이 소유하지 않은 객체에 대해 사용 권한 부여 가능
* **ALTER DATABSE / ALTER SYSTEM**: **(강력)** 데이터 파일 이름 변경, 버퍼 캐시 비우기 등 Oracle Instance 수정 작업 수행 가능.

### ADMIN OPTION을 사용한 시스템 권한 취소

GRANT 명령으로 직접 부여된 시스템 권한은 `REVOKE` SQL문을 이용해 취소 가능.
연쇄적으로 취소되지 않음.

```
REVOKE <system_privilege> FROM <grantee clause>
```

### 객체 권한

객체 권한: 테이블, 뷰, 시퀀스, 프로시저 등의 특정 객체에 대한 작업을 수행 가능

```
SQL> GRANT <object_privilege> ON <object> TO <grantee clause> [WITH GRANT OPTION]
```

### GRANT OPTION을 사용한 객체 권한 취소

DML 작업과 관련된 시스템 권한을 취소할 때 **연쇄적 결과**를 관찰 가능.

## 롤 사용 시의 이점

* 권한 관리 용이
    * 유저에게 권한을 부여하는 것이 아닌, 롤을 부여한 후 롤에 권한을 부여
* 동적 권한 관리
    * 롤에 관한 권한을 수정하면 그 롤을 부여받은 유저는 즉시 수정된 권한 얻는다.
* 권한의 선택적 가용성
    * 롤을 활성화/비활성화 하여 권한을 일시적으로 설정/해제 가능

### 롤 특성

* 롤에도 권한 부여/취소 가능
* 롤을 유저나 다른 롤에 부여/취소 가능
* 시스템 및 객체 권한으로 구성
* 롤을 활성화 하려면 암호가 필요할 수 있음.
* 롤은 유저가 소유하지 않으며 스키마에 존재하지 않음.

## 프로파일 및 유저

* 프로파일은 데이터베이스 사용 및 instance 리소스에 명령된 리소스 제한 집합을 적용.
* 계정 상태를 관리, 유저의 암호 길이, 만료 시간 등의 제한 지정
* 모든 유저에게 프로파일 할당, 언제나 하나의 프로파일에만 속할 수 있음.
* 프로파일을 변경하면 다음 로그인부터 적용.
* 제어 가능 리소스
    * **CPU**: CPU 리소스를 세션 또는 호출 단위로 제한 가능
    * **네트워크/메모리**: 각 데이터베이스 세션은 시스템 메모리 리소스 및 네트워크 리소스를 소모
        * Connect Time: 허용되는 연결 시간
        * Idle Time: idle 상태를 유지할 수 있는 시간.
        * Concurrent Sessions: 동시 세션 수
        * Private SGA: SGA 내의 공간 양 제한
    * **디스크 I/O**: 세션 단위 레벨 또는 호출 단위 레벨에서 읽을 수 있는 데이터 양 제한.

## 암호 보안 기능

* 유저가 지정한 횟수 내에 로그인 못하면 일정 시간 동안 **계정을 LOCK**
    * FAILED\_LOGIN\_ATTEMPTS: 만료되기 전 까지의 로그인 시도 실패 횟수
    * PASSWORD\_LOCK\_TIME: Lock이 유지되는 기간
    * PASSWORD\_LIFE\_TIME: 암호가 만료되기 전에 사용할 수 있는 실행 시간
    * PASSWORD\_GRACE\_TIME: 암호 변경을 위한 유예 기간
* 암호 변경 횟수 동안 **암호를 재사용**하지 않았는지 확인하기 위한 새 암호 검사
    * PASSWORD\_REUSE\_TIME: 주어진 일 수동안 암호를 재사용 할 수 없도록 지정
    * PASSWORD\_REUSE\_MAX: 현재 암호를 재사용할 수 있기 전에 필요한 암호 변경 횟수
* **암호의 복잡성**을 검사하여 특정 규칙 확인

### VERIFY\_FUNCTION\_11G

* 8자 이상
* Username과 관련된 형태가 아님
* 데이터베이스 이름과 관련된 형태가 아님
* 최소 하나의 영문자와 하나의 숫자를 포함하는 문자열
* 이전 암호와 3자리 이상이 다름

## 최소 권한의 원칙

유저가 작업을 효율적으로 완료하는 데 **필요한 권한**만 유저에게 부여

* 데이터 딕셔너리 보호
    * _07\_DICTIONARY\_ACCESSIBILITY = FALSE_
* PUBLIC에서 불필요한 권한 취소
* ACL을 사용하여 네트워크 단에서 제어
* 유저가 액세스할 수 있는 디렉토리 제한
* 관리 권한을 갖는 유저 제한
* 원격 데이터베이스 인증 제한
    * _REMOTE\_OS\_AUTHENT = FALSE_
