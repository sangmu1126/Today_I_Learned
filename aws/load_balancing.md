# 🚀 AWS ELB Load Balancing 실습 보고서

Application Load Balancer(ALB) / Network Load Balancer(NLB) 구성 및
트래픽 분산 검증

## 📌 1. 실습 개요

### 목표

AWS의 핵심 로드 밸런서 **ALB(L7)**, **NLB(L4)** 를 직접 구성하고 HTTP &
UDP 트래픽 분산을 실습 및 검증

### 구성 환경

-   **Client:** Linux EC2\
-   **Web Servers:** Apache + SNMP Agent EC2 3대\
-   **Load Balancers:**
    -   ALB --- HTTP 80\
    -   NLB --- UDP 161

## ☁️ 1.1 아키텍처 다이어그램

``` mermaid
graph TD
    User(Client/User) -->|Internet| IGW(Internet Gateway)
    IGW --> ALB(ALB: HTTP 80)
    IGW --> NLB(NLB: UDP 161)

    subgraph VPC [AWS Cloud (VPC)]
        ALB -->|Round Robin| TG_Web[Target Group: Web Servers]
        NLB -->|Hash Flow| TG_SNMP[Target Group: SNMP Agents]

        subgraph AZ_A [Availability Zone A]
            TG_Web --> Server1
            TG_SNMP --> Server1
        end

        subgraph AZ_C [Availability Zone C]
            TG_Web --> Server2
            TG_Web --> Server3
            TG_SNMP --> Server2
            TG_SNMP --> Server3
        end
    end
```

## 🛠️ 2. 트러블슈팅(Troubleshooting)

### 2.1 보안 그룹 문제

**현상:** curl 요청 → 무응답(timeout)\
**원인:** EC2 SG에서 HTTP(80) 미허용\
**해결:** HTTP 80 / 0.0.0.0/0 규칙 추가

### 2.2 VPC 간 통신 불가

서로 다른 VPC 간 Private IP 접근 불가\
➡️ **Public IP / DNS 사용하여 해결**

### 2.3 Cross-Zone Load Balancing

특정 AZ로 트래픽 편중 위험\
➡️ **Cross-Zone LB 활성화**

------------------------------------------------------------------------

## 🌐 3. ALB 테스트 결과 (HTTP / L7)

### 3.1 Round-Robin 트래픽 분산

``` bash
ALB=ALB-1900379046.ap-northeast-2.elb.amazonaws.com

curl $ALB
<h1>ELB LAB Web Server-3</h1>

curl $ALB
<h1>ELB LAB Web Server-2</h1>

curl $ALB
<h1>ELB LAB Web Server-1</h1>
```

✔ 정상적으로 1 → 2 → 3 순서로 분산

### 3.2 X-Forwarded-For -- 클라이언트 IP 확인

Apache 설정에 아래 헤더 추가:

    %{X-Forwarded-For}i

서버 로그 결과:

    3.38.214.31 ... "GET / HTTP/1.1" 200

✔ ALB → 실제 Client IP 추출 성공

------------------------------------------------------------------------

## ⚡ 4. NLB 테스트 결과 (UDP / L4)

### 4.1 SNMP 요청 테스트 (UDP Load Balancing)

``` bash
NLB=NLB-a0d31b2a19e53769.elb.ap-northeast-2.amazonaws.com

snmpget -v2c -c public $NLB 1.3.6.1.2.1.1.5.0
# SERVER2

snmpget -v2c -c public $NLB 1.3.6.1.2.1.1.5.0
# SERVER1
```

✔ UDP 또한 여러 서버로 정상 분산됨

### 4.2 패킷 분석 (Client IP 보존)

    tcpdump udp port 161 -nn

결과:

    IP 3.38.214.31 > 10.40.1.10.161

✔ NLB는 Client IP 그대로 전달 (Preserve Source IP)

------------------------------------------------------------------------

## 🔍 5. ALB vs NLB --- Client IP 전달 방식

  구분             ALB                         NLB
  ---------------- --------------------------- ---------------------
  계층             L7 (HTTP)                   L4 (TCP/UDP)
  Client IP 전달   X-Forwarded-For 헤더 필요   원본 IP 그대로 전달
  서버 로그 설정   필요                        불필요
  특징             정교한 라우팅               초고성능·저지연

------------------------------------------------------------------------

## 🧾 6. 결론

-   **ALB = L7 기반 라우팅에 최적화**
-   **NLB = 고성능 UDP/TCP 처리, Client IP 보존**
-   보안 그룹 구성 필수\
-   Cross-Zone LB 활성화로 가용성 증가

------------------------------------------------------------------------

## 📘 Appendix: Cheat Sheet

### 🔧 1. 기본 테스트 명령어

``` bash
curl -v http://[DNS]
snmpget -v2c -c public [NLB_DNS] 1.3.6.1.2.1.1.5.0
```

### 🔎 2. 서버 상태 모니터링

``` bash
sudo systemctl status httpd
sudo systemctl reload httpd
tail -f /var/log/httpd/access_log | grep -v "ELB-HealthChecker"
```

### 📡 3. 패킷 분석

``` bash
tcpdump tcp port 80 -nn
tcpdump udp port 161 -nn
```
