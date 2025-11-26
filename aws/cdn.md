# AWS CloudFront Performance Optimization Test

이 문서는 **Amazon CloudFront(CDN)**를 도입하여 대용량 정적 콘텐츠(이미지)의 전송 속도를 개선하고, 캐싱(Caching) 성능을 비교 분석한 테스트 리포트입니다.


## 📋 프로젝트 개요

### 🎯 목표
1.  **속도 개선**: Origin(EC2) 직접 접속 대비 CDN 경유 접속 시 로딩 속도 비교.
2.  **캐시 검증**: Cache Miss(최초 접속)와 Cache Hit(재접속) 상황에서의 성능 차이 분석.
3.  **부하 감소**: 엣지 로케이션(Edge Location) 캐싱을 통한 오리진 서버 부하 감소 확인.

### 💻 테스트 환경
* **Origin Server**: Amazon Linux 2 (Apache Web Server)
* **CDN**: AWS CloudFront
* **Target File**: 고해상도 이미지 파일 (`test.jpg`, 약 10MB)

---

## 📊 성능 비교 테스트 (Performance Benchmark)

브라우저 개발자 도구(Network 탭)를 사용하여 동일한 리소스에 대한 로딩 시간을 측정했습니다.

### 1. Origin Server 직접 접속 (Baseline)
* **상황**: CloudFront를 거치지 않고 EC2 공인 IP로 직접 요청.
* **결과**:
    * **Time**: `8.64 s`
    * **분석**: 서버의 물리적 위치와 대역폭 제한으로 인해 데이터 다운로드에 긴 시간이 소요됨.

### 2. CloudFront 최초 접속 (Cache Miss)
* **상황**: CDN 도메인으로 접속했으나, 엣지 로케이션에 캐시 파일이 없는 상태.
* **결과**:
    * **Time**: `5.53 s`
    * **분석**: 오리진에서 데이터를 가져와야 하므로 시간은 소요되나, AWS 백본 네트워크 활용으로 Origin 직접 접속보다는 단축됨.

### 3. CloudFront 재접속 (Cache Hit)
* **상황**: 엣지 로케이션(또는 브라우저)에 캐시된 데이터를 불러오는 상태.
* **결과**:
    * **Time**: `0 ms` (Memory Cache) / `475 ms` (Network Fetch)
    * **분석**: **캐시 적중(Cache Hit)**. 오리진 서버와의 통신 없이 즉시 데이터를 렌더링하여 가장 빠른 속도를 기록.

---

## 📈 성능 요약 표

| 테스트 케이스 | 접속 방식 | 로드 시간 (Time) | 비고 |
| :--- | :--- | :--- | :--- |
| **Case 1** | Origin Server (EC2) | **8.64 s** | Slowest (네트워크 지연 발생) |
| **Case 2** | CloudFront (Miss) | **5.53 s** | Origin Fetch 발생 |
| **Case 3** | CloudFront (Hit) | **0 ms ~ Instant** | **Fastest (캐싱 적용 완료)** |

---

## 💡 핵심 운영 개념 (Operations)

### TTL (Time to Live)
* 콘텐츠가 엣지 서버에 머무르는 유효 기간.
* 정적 파일(이미지 등)은 긴 TTL(ex: 24시간~1년)을 설정하여 캐시 효율을 높임.

### Cache Invalidation (캐시 무효화)
* 오리진의 원본 파일이 변경되었을 때, TTL 만료 전 강제로 엣지 캐시를 삭제하는 작업.
* `aws cloudfront create-invalidation` 명령어로 수행 가능.

---

## ✅ 결론 (Conclusion)

* **사용자 경험(UX) 개선**: 대용량 리소스 로딩 시간이 8초대에서 0초대(캐시 적중 시)로 획기적으로 단축되었습니다.
* **서버 부하 감소**: 반복적인 요청을 엣지 로케이션에서 처리함으로써 오리진 서버의 트래픽 비용과 리소스 사용량을 절감할 수 있습니다.

---
