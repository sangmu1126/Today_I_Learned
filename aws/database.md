
# AWS RDS Integration: HA & Read Replica Test

이 프로젝트는 Amazon Linux 2 기반의 EC2 웹 서버와 AWS RDS(MySQL 8.0)를 연동하여, 기본적인 데이터 처리뿐만 아니라 **Multi-AZ 페일오버(Failover)** 및 **읽기 전용 복제본(Read Replica)**의 동작 원리를 검증한 기술 문서입니다.

## 📋 프로젝트 개요

* **목표**:
  1. **기본 연동**: EC2와 RDS 간의 연결 및 CRUD 동작 확인.
  2. **장애 조치(Failover)**: Multi-AZ 환경에서 Primary 인스턴스 장애 시 Standby로 전환 및 DNS 변경 확인.
  3. **부하 분산(Read Replica)**: 읽기 전용 복제본(RR)을 통한 데이터 복제 및 쓰기 방지 정책 검증.
* **환경**:
  * **OS**: Amazon Linux 2 (AL2)
  * **Database**: AWS RDS (MySQL 8.0.43)
  * **Client**: MySQL CLI (`mysql` Ver 15.1)

## 🛠️ 사전 설정 (Prerequisites)

### 환경 변수 설정

```bash
RDS1=rds1.crquyssi02m2.ap-northeast-2.rds.amazonaws.com      # Primary (Multi-AZ)
RDS1RR=rds1-rr.crquyssi02m2.ap-northeast-2.rds.amazonaws.com # Read Replica
```

## 🧪 테스트 1: Multi-AZ 페일오버 (Failover)

**시나리오**: Multi-AZ가 구성된 `RDS1`을 재부팅(Failover)하여, DNS 엔드포인트가 새로운 Primary IP로 변경되는 과정과 다운타임을 측정합니다.

### 1. 장애 발생 및 연결 끊김

```text
rds1...amazonaws.com has address 10.6.3.24
Wed Nov 26 11:39:30 KST 2025
ERROR 2003 (HY000): Can't connect to MySQL server... (4)
...
Wed Nov 26 11:39:33 KST 2025
ERROR 2003 (HY000): Can't connect to MySQL server... (111)
```

### 2. 페일오버 완료 및 서비스 복구

약 4~5초 후 DNS가 Standby 인스턴스의 IP(`10.6.2.194`)로 변경되며 서비스가 재개됩니다.

```text
rds1...amazonaws.com has address 10.6.2.194
Wed Nov 26 11:39:34 KST 2025
+------+------+---------+
| ID   | NAME | ADDRESS |
+------+------+---------+
|    1 | Son  | UK      |
+------+------+---------+
```

* **결과**: 데이터 손실 없이 자동으로 새로운 인스턴스로 연결 전환 확인.

## 🧪 테스트 2: Read Replica (RR) 동작 검증

**시나리오**: Primary DB에 데이터를 입력했을 때 RR로 복제되는지 확인하고, RR에 직접 쓰기를 시도했을 때 차단되는지 검증합니다.

### 1. 데이터 복제(Replication) 확인

```sql
-- Primary에 데이터 입력
INSERT INTO EMPLOYEES VALUES ('2','Park','Suwon');

-- Replica에서 조회
mysql -h $RDS1RR -uroot -p -e "USE sample;SELECT * FROM EMPLOYEES;"
```

**결과**:

```text
+------+------+---------+
| ID   | NAME | ADDRESS |
+------+------+---------+
|    1 | Son  | UK      |
|    2 | Park | Suwon   |
+------+------+---------+
```

### 2. 읽기 전용(Read-Only) 정책 확인

```bash
mysql -h $RDS1RR -uroot -p -e "USE sample;INSERT INTO EMPLOYEES VALUES ('3','Lee','China');"
```

**결과 (에러 발생)**:

```text
ERROR 1290 (HY000) at line 1: The MySQL server is running with the --read-only option so it cannot execute this statement
```

* **결론**: Read Replica는 쓰기 불가, Primary로부터의 복제만 가능.

## 🖼 RDS 아키텍처 구성도 (Multi-AZ + Read Replica)

```mermaid
graph TD
    Client[EC2 Web Server / App Server] -->|Read/Write| RDS_Primary[Primary DB (Multi-AZ)]
    
    subgraph RDS_MultiAZ[Multi-AZ 구성]
        RDS_Primary --> RDS_Standby[Standby DB (Failover)]
    end

    Client -->|Read Only| RDS_RR[Read Replica]

    style RDS_Primary fill:#f9f,stroke:#333,stroke-width:2px
    style RDS_Standby fill:#ff9,stroke:#333,stroke-width:2px
    style RDS_RR fill:#9f9,stroke:#333,stroke-width:2px
```

## ✅ 종합 결론

1. **가용성 (Availability)**: Multi-AZ 환경에서 장애 발생 시 수 초 내에 자동 복구.
2. **확장성 (Scalability)**: Read Replica를 통해 읽기 트래픽 분산 가능.
3. **데이터 무결성 (Integrity)**: Failover 후에도 데이터 유지, RR은 쓰기 불가로 데이터 일관성 보장.

---

### 🛠 Tech Stack

* Cloud Provider: AWS (EC2, RDS)
* OS: Amazon Linux 2
* Database: MySQL 8.0
* Tools: MySQL CLI
