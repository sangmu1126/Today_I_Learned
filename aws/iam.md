# AWS IAM Security & Authorization Lab

이 문서는 AWS의 핵심 보안 기능인 **IAM(Identity and Access Management)**의 작동 방식을 검증한 테스트 리포트입니다.
**명시적 거부(Explicit Deny)**와 **묵시적 거부(Implicit Deny)**의 차이를 실습을 통해 확인하고, EC2 인스턴스 프로파일을 통한 안전한 자격 증명 관리를 검증합니다.

---

## 📋 프로젝트 개요

### 🎯 목표

1. **EC2 제어**: AWS CLI를 사용하여 원격으로 인스턴스를 제어(Reboot)하고 상태를 확인.
2. **Explicit Deny (명시적 거부)**: 관리자 권한(`Admin`)이 있어도 특정 정책에 의해 작업이 차단되는지 검증.
3. **Instance Profile (Role)**: Access Key 하드코딩 없이 역할을 통해 권한을 위임받는 방식 검증.
4. **Implicit Deny (묵시적 거부)**: 허용되지 않은 작업은 기본적으로 차단되는 원칙 확인.

### 💻 테스트 환경

* **Controller EC2**: `BasicEC2` (Amazon Linux 2, Admin User)
* **Target Instance**: `S3IAMRoleEC2` (Amazon Linux 2, Instance Role Attached)

---

## 🧪 Scenario 1: 명시적 거부 (Explicit Deny) 테스트

**상황**: 관리자(`admin`) 권한을 가진 사용자가 특정 인스턴스를 재부팅하려고 시도합니다. 처음에는 성공하지만, **Deny Policy** 적용 후 실패하는 것을 확인합니다.

### 1. 정상 재부팅 확인 (Allow)

* **Command**:

```bash
aws ec2 reboot-instances --instance-ids i-0c253093b24439b59
```

* **Result**: Ping 패킷 유실 후 복구됨.

```
64 bytes from 10.0.0.253: icmp_seq=3 ttl=255 time=0.363 ms
... (Rebooting) ...
64 bytes from 10.0.0.253: icmp_seq=17 ttl=255 time=0.382 ms
```

### 2. 차단 정책 적용 (Explicit Deny Policy)

아래와 같은 **Explicit Deny** 정책을 admin 유저(또는 그룹)에 인라인으로 추가.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyRebootSpecificInstance",
            "Effect": "Deny",
            "Action": "ec2:RebootInstances",
            "Resource": "arn:aws:ec2:ap-northeast-2:309866937539:instance/i-0c253093b24439b59"
        }
    ]
}
```

### 3. 재부팅 차단 확인

* **Command**:

```bash
aws ec2 reboot-instances --instance-ids i-0c253093b24439b59
```

* **Result**:

```
An error occurred (UnauthorizedOperation) ... with an explicit deny in an identity-based policy.
```

* **분석**: AWS IAM 평가 로직에서 **Deny가 최우선**이므로 Admin 권한이 있어도 수행 불가.

---

## 🧪 Scenario 2: 인스턴스 프로파일 & 묵시적 거부 (Implicit Deny)

**상황**: `S3IAMRoleEC2` 인스턴스 내부에서 AWS CLI를 사용. 이 인스턴스는 S3 권한만 가진 IAM Role을 사용 중.

### 1. 자격 증명 확인 (Instance Profile)

```bash
aws configure list
```

```
NAME        : VALUE                 : TYPE      : LOCATION
access_key  : ****************LKAF  : iam-role  :
```

* **분석**: Access Key 설정 없이 인스턴스 프로파일로 IAM Role 자격 증명 자동 수신.

### 2. 허용된 작업 (S3 Access)

```bash
aws s3 mb s3://paka9999
aws s3 rb s3://paka9999
```

* **Result**: 성공

### 3. 거부된 작업 (EC2 Control)

```bash
aws ec2 describe-vpcs
```

```
An error occurred (UnauthorizedOperation) ... because no identity-based policy allows the ec2:DescribeVpcs action
```

* **분석 (Implicit Deny)**: Role에 EC2 관련 Allow가 없으므로 기본적으로 모든 요청이 거부됨.

---

## 🧩 IAM 권한 평가 흐름도 (Authorization Flow)

```
[Request]
     ↓
[Explicit Deny?] → Yes → Deny
     ↓ No
[Allow?] → Yes → Allow
     ↓ No
[Implicit Deny]
```

---

## ✅ 핵심 요약 (Key Takeaways)

| 구분         | Explicit Deny (명시적 거부)  | Implicit Deny (묵시적 거부)                    |
| ---------- | ----------------------- | ----------------------------------------- |
| **정의**     | 정책에 명확히 `Deny` 선언       | Allow가 없으면 자동 거부                          |
| **우선순위**   | **최상위 (모든 Allow 무시)**   | Allow가 없을 때 적용                            |
| **테스트 사례** | Admin도 인스턴스 재부팅 실패      | S3 Role 서버가 EC2 API 사용 불가                 |
| **에러 메시지** | `with an explicit deny` | `because no identity-based policy allows` |

---
