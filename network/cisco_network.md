# Cisco Networking & Routing Protocols Study Notes 🌐

이 리포지토리는 Cisco 네트워크 기초, 라우터 동작 원리, IP 주소 체계부터 주요 라우팅 프로토콜(RIP, EIGRP, OSPF)의 심화 이론 및 설정까지 정리한 핵심 요약 노트입니다.

## 📚 목차 (Table of Contents)
1. [네트워크 기초 (Network Basics)](#1-네트워크-기초-network-basics)
2. [케이블링 (Cabling)](#2-케이블링-cabling)
3. [IP 주소 체계 및 서브네팅 (IP Addressing & Subnetting)](#3-ip-주소-체계-및-서브네팅-ip-addressing--subnetting)
4. [라우터 구조 및 기본 설정 (Router Architecture & Basic Config)](#4-라우터-구조-및-기본-설정-router-architecture--basic-config)
5. [라우팅 개념 및 스위칭 (Routing Concepts & Switching)](#5-라우팅-개념-및-스위칭-routing-concepts--switching)
6. [라우팅 프로토콜: RIP](#6-라우팅-프로토콜-rip-routing-information-protocol)
7. [라우팅 프로토콜: EIGRP](#7-라우팅-프로토콜-eigrp)
8. [라우팅 프로토콜: OSPF](#8-라우팅-프로토콜-ospf)
9. [자격증 정보 (Certification Info)](#9-자격증-정보-certification-info)

---

## 1. 네트워크 기초 (Network Basics)

![OSI Model](image-placeholder)

### 네트워크 개요
- **네트워크 정의:** 효율적인 데이터 전송을 위해 장비와 장비를 연결한 조직.
- **LAN vs WAN:**
  - **LAN:** 동일 네트워크(Switch 범위). 유지보수/통신 비용 낮음.
  - **WAN:** 외부 네트워크 구간. ISP 임대 회선 사용으로 비용 증가.

### OSI 계층 및 캡슐화 (Encapsulation)
- **캡슐화:** 데이터 전송을 위해 헤더(주소 정보 포함)를 추가하는 과정.
- **계층별 주소/장비:**
  - **L2:** MAC Address(48bit), Switch, Frame
  - **L3:** IP Address(32bit), Router, Packet
  - **L4:** Port Number(16bit), TCP/UDP, Segment

### 프로토콜 (Protocol)
- **TCP:** 신뢰성 중시 (HTTP, FTP, Telnet, SSH)
- **UDP:** 속도 중시 (DNS, TFTP, DHCP)
- **CDP:** 인접 Cisco 장비 정보 확인 (L2 프로토콜)

---

## 2. 케이블링 (Cabling)

![Cabling](image-placeholder)

- **Straight-Through:** PC-Switch, Switch-Router (이종 장비)
- **Crossover:** PC-PC, Switch-Switch, Router-Router (동종 장비)
- **Roll-over:** 콘솔 접속 전용

---

## 3. IP 주소 체계 및 서브네팅 (IP Addressing & Subnetting)

![IPv4 Header](image-placeholder)

### IPv4 Class
- **A Class:** 0~127 / `255.0.0.0`
- **B Class:** 128~191 / `255.255.0.0`
- **C Class:** 192~223 / `255.255.255.0`
- **사설 IP:** A(10.x), B(172.16~31), C(192.168.x)

### 서브네팅 & VLSM
- **목적:** 주소 절약 및 네트워크 분할
- **호스트 계산:** `2^n - 2`
- **Wildcard Mask:** 서브넷 마스크 반대 개념 (OSPF/ACL에서 사용)

---

## 4. 라우터 구조 및 기본 설정 (Router Architecture & Basic Config)

### 라우터 구조
- L3 장비로 최적 경로 결정
- 라우팅 테이블 없는 목적지는 Drop
- **메모리:** RAM(Running), NVRAM(Startup)

### 동작 모드
1. User Mode — `Router>`
2. Privilege Mode — `Router#`
3. Global Config — `Router(config)#`

### 기본 비밀번호/인터페이스 설정
```cisco
! Console Password
Router(config)# line console 0
Router(config-line)# password <PASSWORD>
Router(config-line)# login

! VTY (Telnet/SSH)
Router(config)# line vty 0 4
Router(config-line)# password <PASSWORD>
Router(config-line)# login

! Enable Secret (MD5)
Router(config)# enable secret <PASSWORD>
```

### Serial 인터페이스 설정
```cisco
R2(config)# interface serial 1/1
R2(config-if)# encapsulation hdlc
R2(config-if)# bandwidth 64
R2(config-if)# clock rate 64000
R2(config-if)# ip address 192.168.12.2 255.255.255.0
R2(config-if)# no shutdown
```

---

## 5. 라우팅 개념 및 스위칭 (Routing Concepts & Switching)

### Static Route
- 관리자가 직접 경로 입력
- Stub/small 네트워크 적합
```cisco
Router(config)# ip route <Network> <Mask> <Next-hop | Interface>
```

### Dynamic Routing
- 변화 자동 감지 및 최적 경로 유지
- **Distance Vector:** RIP, IGRP
- **Link State:** OSPF, IS-IS

### Cisco Switching 방식
- **Process Switching** — 모든 패킷 CPU 처리
- **Fast Switching** — 첫 패킷만 CPU 처리
- **CEF** — FIB 기반 고속 하드웨어 처리

---

## 6. 라우팅 프로토콜: RIP (Routing Information Protocol)

### 기본 특징
- Metric: Hop-count (16 = 장애)
- AD값: 120
- Timer: Update 30s, Invalid 180s, Holddown 180s, Flush 240s
- Loop 방지: Split-horizon, Route Poisoning

### RIPv1 vs RIPv2
| 항목 | RIPv1 | RIPv2 |
|------|-------|-------|
| Mask 포함 | X | O |
| 전송 방식 | Broadcast | Multicast(224.0.0.9) |
| CIDR/VLSM | 불가 | 가능 |
| 인증 | X | O |

### Cisco CLI — RIPv2
```cisco
Router(config)# router rip
Router(config-router)# version 2
Router(config-router)# no auto-summary
Router(config-router)# network 10.0.0.0
Router(config-router)# passive-interface fastethernet 0/0
```

---

## 7. 라우팅 프로토콜: EIGRP

### 기본 특징
- Hybrid Routing (DV + LS)
- Metric: BW + Delay
- AD: Internal 90 / External 170 / Summary 5
- 알고리즘: DUAL (Loop-free)
- Unequal Load Balancing 지원

### PDU
Hello, Update, Query, Reply, Ack

### DUAL 용어
- Successor — 최적 경로
- Feasible Successor — 대체 경로
- FD(Feasible Distance)

---

## 8. 라우팅 프로토콜: OSPF (Open Shortest Path First)

### 기본 특징
- Link-State 기반 LSDB 유지
- SPF(Dijkstra) 알고리즘 사용
- Cost = `100Mbps / Bandwidth`
- AD: 110
- Area 구조 — Backbone Area 0

### OSPF Packet Types
Hello, DBD, LSR, LSU, LSAck

### DR/BDR
- **Broadcast:** DR/BDR 선출 (224.0.0.5 / 224.0.0.6)
- **P2P:** DR/BDR 없음

### MD5 인증 설정
```cisco
R1(config-if)# ip ospf authentication message-digest
R1(config-if)# ip ospf message-digest-key 1 md5 cisco
```

---

## 9. 자격증 정보 (Certification Info)
- CCNA, CCNP 대비 핵심 개념 기반
- 라우팅/스위칭 실습에 Packet Tracer 또는 GNS3 추천

---

**© 2025 Cisco Networking Study Notes**
