# Cisco Networking & CCNA Study Notes 🌐

이 리포지토리는 Cisco 네트워크 기초, 라우터 동작 원리, IP 주소 체계 및 CCNA 자격증 준비를 위한 핵심 요약 노트입니다.

## 📚 목차 (Table of Contents)

1. [네트워크 기초 (Network Basics)](#1-네트워크-기초-network-basics)
2. [IP 주소 체계 및 서브네팅 (IP Addressing & Subnetting)](#2-ip-주소-체계-및-서브네팅-ip-addressing--subnetting)
3. [라우터 구조 및 동작 (Router Architecture & Operation)](#3-라우터-구조-및-동작-router-architecture--operation)
4. [기본 설정 및 명령어 (Basic Configuration & Commands)](#4-기본-설정-및-명령어-basic-configuration--commands)
5. [케이블링 (Cabling)](#5-케이블링-cabling)
6. [자격증 정보 (Certification Info)](#6-자격증-정보-certification-info)

---

## 1. 네트워크 기초 (Network Basics)

### 네트워크 개요
* [cite_start]**네트워크 정의:** 효율적인 데이터 전송을 위해 장비와 장비를 연결한 조직[cite: 87].
* **LAN vs WAN:**
    * [cite_start]**LAN (Local Area Network):** 자신이 포함된 동일 네트워크(Switch 범위 내), 구축 비용 높음, 유지보수 비용 낮음[cite: 91, 92].
    * [cite_start]**WAN (Wide Area Network):** 라우터를 통해 연결된 외부 네트워크, ISP 임대 회선 사용, 구축 비용 낮음, 유지보수 비용 높음[cite: 93, 94].


### OSI 계층 및 캡슐화 (Encapsulation)
* [cite_start]**캡슐화:** 데이터 전송을 위해 기존 데이터에 헤더(Source/Dest Address 등)를 추가하는 과정[cite: 88, 89].
* **계층별 주소 및 장비:**
    * [cite_start]**L2 (Data Link):** MAC Address (48bit), Switch, Frame 단위[cite: 100].
    * [cite_start]**L3 (Network):** IP Address (32bit), Router, Packet 단위[cite: 99].
    * [cite_start]**L4 (Transport):** Port Number (16bit), TCP/UDP, Segment 단위[cite: 96].

### 프로토콜 (Protocol)
* [cite_start]**TCP:** 신뢰성 기반, 연결 지향 (3-way handshake), HTTP(80), FTP(20/21)[cite: 97].
* [cite_start]**UDP:** 신속성 기반, 비연결형, DNS(53), DHCP(67/68)[cite: 98].
* [cite_start]**CDP (Cisco Discovery Protocol):** 직접 연결된 시스코 장비의 정보를 확인하는 2계층 프로토콜 (60초 주기 전송)[cite: 157, 159].

---

## 2. IP 주소 체계 및 서브네팅 (IP Addressing & Subnetting)


### IPv4 Class
* [cite_start]**A Class:** 0 ~ 127 / 대형망 / `255.0.0.0`[cite: 43].
* [cite_start]**B Class:** 128 ~ 191 / 중형망 / `255.255.0.0`[cite: 44].
* [cite_start]**C Class:** 192 ~ 223 / 소형망 / `255.255.255.0`[cite: 44].
* [cite_start]**사설 IP:** 내부 네트워크용 (A: 10.x.x.x, B: 172.16.x.x~172.31.x.x, C: 192.168.x.x)[cite: 50, 51].

### 서브네팅 (Subnetting) & VLSM
* [cite_start]**목적:** IP 주소 낭비를 최소화하기 위해 네트워크를 분할[cite: 52, 59].
* [cite_start]**계산 공식:** 사용 가능 호스트 수 = $2^n - 2$ (n=호스트 비트 수)[cite: 48].
* [cite_start]**Wildcard Mask:** 서브넷 마스크의 반대 개념(0이 공통, 1이 비공통), ACL이나 라우팅 프로토콜(OSPF, EIGRP) 범위 지정에 사용[cite: 114, 117].

---

## 3. 라우터 구조 및 동작 (Router Architecture & Operation)

### 라우터의 역할
* [cite_start]L3 장비로 이기종 네트워크 연결 및 패킷의 최적 경로 결정(Routing)[cite: 13].
* [cite_start]라우팅 테이블에 없는 목적지는 Drop, Next-hop까지만 통신 책임[cite: 15, 16].

### [cite_start]주요 메모리 [cite: 32]
| 메모리 | 특징 | 저장 내용 | 명령어 |
| :--- | :--- | :--- | :--- |
| **RAM** | 휘발성 | 실행 중인 설정 (Running-Config) | `show running-config` |
| **NVRAM** | 비휘발성 | 저장된 설정 (Startup-Config) | `show startup-config` |

### [cite_start]동작 모드 (Modes) [cite: 25, 29, 30]
1.  **User Mode (`Router>`):** 기본 상태 확인만 가능 (Privilege Level 1).
2.  **Privilege Mode (`Router#`):** 관리자 모드, 모든 정보 확인 및 저장/삭제 가능 (Level 15).
3.  **Global Mode (`Router(config)#`):** 전체 설정 모드.

---

## 4. 기본 설정 및 명령어 (Basic Configuration & Commands)

### 패스워드 설정 (Security)
접속 방식에 따라 비밀번호를 설정하여 보안을 강화합니다.
* [cite_start]**권장 사항:** 대/소문자, 숫자, 특수문자 조합 7자 이상 (안전 시 13자 이상)[cite: 4, 7].

```cisco
! Console Password (직접 접속) [cite: 5]
Router(config)# line console 0
Router(config-line)# password <PASSWORD>
Router(config-line)# login

! VTY Password (Telnet/SSH 원격 접속) [cite: 8]
Router(config)# line vty 0 4
Router(config-line)# password <PASSWORD>
Router(config-line)# login

! Enable Secret (관리자 권한 암호화 저장 - MD5) [cite: 12]
Router(config)# enable secret <PASSWORD>
인터페이스 설정 (Interface)

Ethernet: LAN 연결, L2 표준이므로 IP 할당 및 no shutdown만 필요.



Serial: WAN 연결, bandwidth, clock rate (DCE단) 설정 필요, L2 프로토콜(HDLC 등) 지정.


Cisco CLI

! Ethernet 설정 예시 [cite: 132]
R1(config)# interface fastethernet 0/0
R1(config-if)# ip address 192.168.1.254 255.255.255.0
R1(config-if)# no shutdown

! Serial 설정 예시 (DCE) [cite: 143]
R2(config)# interface serial 1/1
R2(config-if)# encapsulation hdlc
R2(config-if)# bandwidth 64
R2(config-if)# clock rate 64000
R2(config-if)# ip address 192.168.12.2 255.255.255.0
R2(config-if)# no shutdown
5. 케이블링 (Cabling)
장비 간 연결에 사용하는 UTP 케이블 종류입니다.


Straight-Through (Direct): 다른 계층 장비 연결 (PC-Switch, Switch-Router).


Crossover: 같은 계층 장비 연결 (PC-PC, Switch-Switch, Router-Router).


Roll-over (Console): PC에서 라우터/스위치 콘솔 포트 접속 시 사용.