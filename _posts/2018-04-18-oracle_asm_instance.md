---
layout: post
title:  "[ORACLE] ASM Instance"
date:   2018-04-18 20:08:00
author: garimoo
categories: Database
---
<br/>
## ASM 사용 이점

### ASM에서 불필요한 작업

* I/O 성능 튜닝: 자동 리밸런싱 작업
* 데이터 파일 및 이동 재구성
* 파일 이름 관리
* 논리 볼륨, 파일 시스템, 클러스터 파일 시스템, Raw Device 관리

### ASM을 사용한 추가적인 이점

* LUN(Logical Unit Number) 관리: 더 적은 수와 더 큰 LUN 사용
* 시스템 관리자 ↔ 데이터베이스 관리자 종속성 줄어듬.
* 수동 유지 관리 작업과 관련된 오류 발생 간으성 줄어듬.

## ASM Instance

![Image](/assets/20180418/1.png)

* ASM/데이터베이스가 시작될 때마다 SGA(System Global Area) 할당, ASM과 백그라운드 프로세스 시작.
* Oracle ASM Instance = 백그라운드 프로세스 + SGA
* ASM Instance SGA는 데이터베이스 Instance SGA와 메모리 할당 및 사용 방법이 _다름_.
    * Shared Pool: 메타 데이터 정보
    * Large Pool: 병렬 작업에 사용
    * ASM 캐시: 리밸런스 작업 중 읽기 및 쓰기 블록에 사용
    * 사용 가능한 메모리: 사용 가능한 할당 해제된 메모리

### ASM 기본 프로세스

* **RBAL**: 검색 중에 모든 장치 파일을 열고 리밸런스 작업 조정. ASM 디스크에서 Global 열기 수행.
* **ARBn**: ASM Instance에서 실제 리밸런스 데이터 Extent 이동 수행. 한번에 많은 프로세스 가능 _(ARB0, ARB1, ...)_
* **GMON**: 삭제 또는 오프라인과 같은 디스크 레벨 작업 관리. ASM 디스크 그룹 호환성 개선
* **MARK**: 오프라인 디스크에 기록을 실패하면 Allocation Unit을 stale로 표시.
* **Onnn**: 메시지 교환을 위해 ASM Instance에 대한 연결 풀을 형성하며, 필요할 때만 표시.
* **PZ9n**: 클러스터화된 ASM 설치의 데이터를 GV$뷰에서 fetch하는 데 사용되는 하나 이상의 병렬 슬레이브 프로세스

### ASM Instance 초기화 파라미터

* **INSTANCE_TYPE**: `ASM`으로 설정되어야 함. _(데이터베이스 Instance 에 대해서는 RDBMS로 설정)_
* **ASM\_POWER\_LIMIT**: 리밸런스 작업의 속도를 제어. 기본은 1, 11이 제일 빠르다.
* **ASM_DISKSTRING**: 검색에 고려되는 디스크 셋 제한. 기본은 NULL
* **ASM_DISKGROUPS**: ASM instance에서 마운트 할 디스크 그룹의 이름 리스트.
* **ASM\_PREFERRED\_READ\_GROUPS**: preferred read 디스크를 포함하는 failure 그룹.
* **DIAGNOSTIC_DEST**: 홈의 위치
* **LARGE\_POOL\_SIZE**: Large pool 할당 힙의 크기를 바이트 단위로 지정
* **REMOTE\_LOGIN\_PASSWORDFILE**: 오라클 소프트웨어가 Password file을 검사할지의 여부. 기본값은 EXCLUSIVE

### 데이터베이스 Instance와 ASM 사이의 상호 작용

![Image](/assets/20180418/2.png)

```
1. 데이터베이스가 파일 생성 요청
2. ASM 프로세스가 COD(Continuing Operation Directory) 항목 생성, 디스크 그룹에서 새 파일에 대한 공간 할당
3. ASMB 데이터베이스 프로세스가 새 파일에 대한 Extent 맵 수신
4. 파일이 열리고, 데이터베이스 프로세스가 파일을 직접 초기화
5. 초기화가 끝나면 데이터베이스 프로세스가 파일 생성을 커밋하도록 요청. 이렇게 되면 ASM 포그라운드 프로세스가 COD 항목을 지우고 파일을 생성된 것으로 표시.
6. 파일 커밋이 확인되면 파일이 암시적으로 닫힘. 데이터베이스 Instance는 이후 I/O를 수행할 때 파일을 다시 열어야 함.
```

* 데이터베이스 Instance와 ASM Instance는 연동된 방식으로 함께 작동. ASM Instance에 데이터베이스 파일을 `매핑`하기 위해 상호작용 하는 것. 또한 데이터베이스 Instance는 ASM Extent를 Lock하거나 이동할 수 있는 ASM 작업과 관련한 메시지 스트림을 계속 수신.
* 데이터베이스 I/O는 ASM Instance를 통해 채널링 되지 않음. **4** 번처럼 데이터베이스가 ASM 파일에 대해 I/O 작업을 직접 수행.

### Dynamic Performance 뷰

* 모든 Instance의 주요 기능 중 하나는 메모리 기반 메타 데이터 테이블을 저장하는 기능.
* 일반적인 Dynamic Performance view
    * V$ASM_ALIAS
    * V$ASM_ATTRIBUTE
    * V$ASM_CLIENT
    * V$ASM_DISK
    * V$ASM\_DISK\_IOSTAT
    * V$ASM\_DISK\_STAT
    * V$ASM_DISKGROUP
    * V$ASM\_DISKGROUP\_STAT
    * V$ASM_FILE
    * V$ASM_OPERATION
    * V$ASM_TEMPLATE

### ASM 시스템 권한

| **ASM 권한** | **권한 그룹(권장)** | **권한** |
| ------ | --------- | --- |
| SYSASM | OSASM<br>(asmadmin) | 모든 관리 권한 |
| SYSDBA | OSDBA<br>(asmdba) | ASM에 저장된 데이터에 대한 액세스 권한<br>및 현재 릴리스의 SYSASM |
| SYSOPER | OSOPER<br>(asmoper) | 비파괴적인 ALTER DISKGROUP 명령과 함께<br>ASM Instance를 시작 및 정지할 수 있는 제한된 권한 |

## 디스크 그룹

![Image](/assets/20180418/3.png)

* ASM이 하나의 모음으로 관리하는 하나 이상의 디스크에 대한 논리적 그룹.
* 디스크 그룹 == 논리 볼륨
* 중복성 설정
    * **External redundancy**: 미러링을 제공하지 않으며, 디스크가 매우 안정적.
    * **Normal redundancy**: 양방향 미러링을 지원하여 안정성이 낮은 저장 영역에 대한 데이터 무결성 보장.
    * **High redundancy**: 세방향 미러링을 지원하여, 더 높은 레벨의 데이터 무결성 보장.

### ASM 디스크

* 클러스터의 모든 노드에서 ASM 소유자가 읽기 및 쓰기 액세스 할 수 있어야 함.
* ASM 내에서 사용되는 물리적 디스크를 다른 어플리케이션과 공유하지 않는 것을 권장.
* **AU(Allocation Unit)**: ASM 디스크 내에서 공간을 나누는 단위. 기본 크기는 1MB. 변경할 수 없다.

### ASM 파일

* AU로 구성되는 ASM Extent의 모음.
* 데이터베이스 커널에는 일반 파일로 표시.
* 파일 이름이 '+'로 시작.
* SAME(stripe and mirror everything) 정책을 사용하여 디스크 그룹의 ASM 디스크에 고르게 분산됨.

### Extent 맵

![Image](/assets/20180418/4.png)

* Extent 맵은 파일에 있는 데이터 Extent를 AU에 매핑하는 테이블.
* 큰 AU와 함께 가변 크기 Extent를 사용하여 매우 큰 ASM 파일을 수용할 수 있다.

### 스트라이핑 세분성

* ASM에서 스트라이핑이 목적으로 갖는 것
    * 디스크 그룹 내의 모든 디스크에서 I/O 로드 밸런싱
    * I/O 대기 시간 개선

#### 상위 단계(coarse-grain) 스트라이핑

* AU를 분산 → 디스크 그룹에 대한 로드 밸런싱
ex) 8개의 디스크를 포함하는 external redundancy 디스크 그룹에서 5개의 디스크에 5개 AU가 스트라이핑 된 파일,
![Image](/assets/20180418/5.png)

#### 하위 단계(fine-grain) 스트라이핑

* 대기 시간 개선을 위해 AU 그룹에 128KB 스트라이핑 단위 사용.
* 기본적으로 control 파일, 온라인 redo 로그 파일에 사용.
ex) 1MB를 8개 디스크에 분산.
![Image](/assets/20180418/6.png)

### ASM Failure 그룹

* 저장 영역 또는 DBA가 ASM Mirroring이 수행되는 하드웨어 경계를 지정하는 방법.
* 개별 디스크, 디스크 컨트롤러, I/O 네트워크 구성 요소, 전체 저장 영역 시스템이 `다른 영역의(?)` failure로부터 보호된다.
![Image](/assets/20180418/7.png)

### 스트라이핑 & Failure 예제

![Image](/assets/20180418/8.png)
Primary(빨간색)이 기본 Extent, Secondary(하늘색)이 무결성을 위한 보조 복사본. 따라서 모든 읽기는 기본 Extent에서 수행.

![Image](/assets/20180418/9.png)
Disk H가 죽었을 때, 거기에 있던 primary 5와 secondary 3이 각각 다른 디스크로 복사됨.

## ASM 디스크 그룹 생성 및 삭제

* ASM Instance의 주된 목적은 디스크 그룹 관리, 데이터 보호.
* ASM Instance는 파일 레이아웃을 데이터베이스 Instance에 전달 → 데이터베이스 Instance는 디스크 그룹에 저장된 파일에 직접 액세스

### 디스크 그룹 생성 예제

```
SQL> CREATE DISKGROUP dgroupA NORMAL REDUNDANCY
FAILGROUP controller1 DISK
    '/devices/A1' NAME diskA1 SIZE 120G FORCE,
    '/devices/A2',
FAILGROUP controller2 DISK
    '/devices/B1',
    '/devices/B2';
```

* DGROUPA라는 디스크 그룹을 CONTROLLER1과 CONTROLLER2라는 두 개의 failure 그룹으로 구성.
* normal redundancy 사용.
* 선택적으로 디스크 이름 및 크기 지정, 없다면 ASM은 기본 이름을 생성, 크기 결정.
* force는 지정된 디스크가 이미 ASM 디스크 그룹의 멤버로 포맷되었어도 지정된 디스크 그룹에 추가되어야 함을 의미.
    * 포맷되지 않은 디스크에 FORCE 옵션 사용하면 오류.

### 디스크 그룹 삭제 예제

```
DROP DISKGROUP dgroupA INCLUDING CONTENTS;
```

* 디스크 그룹을 모든 파일과 함께 삭제 가능.
* 디스크 그룹은 삭제할 순서대로 마운트.

### 디스크 그룹에 디스크 추가 예제

```
ALTER DISKGROUP dgroupA ADD DISK
    '/dev/sde1' NAME A5,
    '/dev/sdf1' NAME A6,
    '/dev/sdg1' NAME A7,
    '/dev/sdh1' NAME A8;
```

* 디스크 그룹에 네개의 새 디스크 추가

```
ALTER DISKGROUP dgroupA ADD DISK '/devices/A*';
```

* A4를 DGROUPA 디스크 그룹에 추가. 디스크 그룹에 디스크를 추가하면 ASM Instance는 디스크의 주소를 지정하고 사용 가능하게 함.
* 리밸런스 프로세스는 모든 파일의 Extent를 새 디스크로 이동하므로 시간이 많이 걸린다.
    * 리밸런싱은 데이터베이스 작업을 차단하지 않는다. 리밸런스 프로세스는 주로 **시스템의 I/O로드**에 영향을 줌. 리밸런스 성능이 높을 수록 시스템에 더 많은 I/O 로드가 적재.

### 디스크 그룹에서 디스크 제거

```
ALTER DISKGROUP dgroupA DROP DISK A5;
```

### 디스크 삭제 작업 취소

```
ALTER DISKGROUP dgroupA UNDOP DISKS;
```

### ASM 디스크 그룹 호환성

* **RDBMS 호환성**: ASM과 RDBMS Instance 간에 교환되는 메시지 형식 지정. 따라서 ASM Instance는 각기 다른 호환성 설정으로 실행되는 서로 다른 RDBMS 클라이언트 지원 가능.
* **ASM 호환성**: 디스크에서 ASM 메타데이터의 데이터 구조 형식을 제어하기 위한 Persistent 호환성 설정.
* **ADVM 호환성**: 디스크 그룹에 Oracle ASM 볼륨이 포함될 수 있는지 여부 결정.

### ASM Fast Mirror Resync

![Image](/assets/20180418/10.png)

* ASM Fast Mirror Resync를 사용하면 디스크의 Transient Failure를 재동기화하는데 필요한 시간이 줄어듬.
* transient Failure 후에 디스크가 오프라인 상태가 되면 ASM은 정전되어 있는 동안 수정된 Extent를 추적.
* Transient Failure가 복구되면 정전 시 영향을 받은 ASM 디스크 Extent만 신속하게 `재동기화`
