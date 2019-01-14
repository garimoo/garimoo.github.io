---
layout: post
title:  "[ORACLE] Library Lock, Pin and Load Lock"
date:   2019-01-14 15:24:00
author: garimoo
categories: Database
---

## Lock, Pin and Load Lock
### Library cache lock
- object handle의 lock을 획득하여 클라이언트 간의 library cache에 대한 동시성을 제어한다.
    - 같은 object에 대해 한 client와 다른 client가 액세스하지 못하게 할 수 있다.
    - 한 클라이언트가 오랫동안 dependency를 유지하게 할 수 있다.
- library cache에서 object를 찾기 위해 확보된다.
- 이 잠금은 SQL 또는 PL/SQL문의 구문 분석이나 컴파일 중에 참조되는 object에서 가져온다. lock은 컴파일이 끝날 때 해제된다.

### Library cache pin
- library cache의 동시성을 관리한다.
    - 개체를 pinning 하면 heap이 메모리에 로드된다.
    - 클라이언트가 object를 수정, 검사하려는 경우 client는 lock 후에 pin을 확보해야 한다.
    - pin은 **NULL, SHARE, EXCLUSIVE** 모드로 획득할 수 있다.
    - library cache pin을 기다리는 것은 다른 세션이 pin을 호환되지 않는 모드로 유지한다는 것을 의미한다.

### 두 종류의 lock이 필요한 이유
- library cache의 object에 액세스하기 위해서 lock과 pin이 모두 필요하다.
- **lock은 프로세스간의 동시성을 관리, pin은 캐시의 일관성을 관리** 한다.
- PL/SQL 의 컴파일, 파싱 등을 위해 Library Cache Lock, Library Cache Pin이 필요한 이유는 컴파일, 파싱 작업 중 object의 정의를 변경하고, object를 삭제하고, 새로 작성하는 동안 아무도 그 object를 사용하지 않도록 하기 위함이다.
- SQL문이 세션에 의해 하드파싱 될 때에도 다른 세션이 object에 액세스하거나 수정하지 못하도록 Library Cache Lock을 획득해야 한다.
- 하드 파싱과 별개로, 세션이 SQL에 지정된 object의 정의를 변경하거나 수정을 한다면 Libray cache Pin과 함께 Library Cache Lock을 획득해야 한다.

### Library Cache Load Lock
- 세션은 object를 load하기 위해서 object에 대한 load lock을 찾으려고 시도함.
- load lock은 항상 exclusive mode에서 획득되므로, 다른 프로세스가 동일한 객체를 load할 수 없음. 세션은 잠금이 사용 가능해 질 때 까지 이벤트를 기다림.
- 동일한 object의 load를 요청하는 multiple process를 방지하기 위해, object를 메모리에 로드할 때 lock이 사용중이면 다른 세션은 library cache load lock을 기다려야 한다.

### Library Cache pin과 Library Load lock의 관계
- PL/SQL 의 컴파일 등의 과정에서 Library Cache Pin과 load lock이 발생할 수 있다.
- 컴파일은 항상 명시적이고, object의 invalidation에 의해 object의 재컴파일이 일어날 수 있다.
<br/>
- object가 invalidation 된 후, oracle은 object에 처음 access할 때 object를 재컴파일 하려고 시도한다.
- 이 때 다른 세션이 library cache에 객체를 pin 시키면 문제가 발생할 수 있다.
- 객체의 재 컴파일을 기다리는 동안 액세스를 시도하는 모든 세션을 차단하는 데 몇 시간이 걸릴 수도 있다.

---------------

## 참고 문서
- [Automatic Locks in DDL Operations](https://docs.oracle.com/cd/E11882_01/server.112/e41084/ap_locks002.htm#SQLRF55509)
- [Schema Object Dependencies](https://docs.oracle.com/cd/B28359_01/server.111/b28318/dependencies.htm#BEGIN)
- [Document 444560.1](https://support.oracle.com/epmos/faces/DocumentDisplay?_afrLoop=396224682066885&parent=SrDetailText&sourceId=3-18443773491&id=444560.1&_afrWindowMode=0&_adf.ctrl-state=1bcu4fowxv_195)
