## ESXi Multi-Tenant Isolation Architecture

![VMware](https://img.shields.io/badge/VMware-vSphere-607078?style=flat-square&logo=vmware)
![ESXi](https://img.shields.io/badge/Hypervisor-ESXi-0066B3?style=flat-square)
![pfSense](https://img.shields.io/badge/Firewall-pfSense-212121?style=flat-square)
![WireGuard](https://img.shields.io/badge/VPN-WireGuard-88171A?style=flat-square&logo=wireguard)
![Status](https://img.shields.io/badge/Status-Completed-success?style=flat-square)

---

### Overview

본 프로젝트는 제한된 온프레미스 인프라 환경에서 다수의 독립 팀이 동시에 서비스를 운영할 수 있도록 설계한 **멀티테넌트 서비스 격리 아키텍처** 프로젝트입니다.

단순한 VM 분리를 넘어 다음 요소를 고려하여 설계하였습니다.

* 네트워크 세그멘테이션
* 스토리지 격리
* 방화벽 기반 접근 제어
* VPN 기반 보안 통신
* 리소스 경합 최소화

---

### Infrastructure Specification

#### Physical Nodes

<p align="center">
  <img src="https://github.com/user-attachments/assets/64ec0004-437a-48a3-89ae-6218dc4cfc5f" width="32%">
  <img src="https://github.com/user-attachments/assets/1df1f44a-8fea-49ee-9627-4bd60fba1b9d" width="32%">
  <img src="https://github.com/user-attachments/assets/1385a97c-3e34-48f1-9db9-228a13172b75" width="32%">
</p>


| Server | CPU | Memory | SSD | HDD | GPU |
|---------|---------|---------|---------|---------|---------|
| GSC01 | Xeon E5-2630 v4 | 128GB | 120GB | 3TB / 1TB | GT710 |
| GSC04 | Xeon E5-2630 v4 | 128GB | 120GB | 3TB / 2TB / 1TB | RTX1080 |
| GSC05 | Xeon Silver 4114 | 128GB | 120GB | 3TB / 2TB / 1TB | GeForce 210 |

#### Design Considerations

* 모든 노드 메모리 128GB 통일
* GPU 노드는 AI 및 고부하 워크로드 전용
* SSD는 ESXi 부트 및 시스템 영역 사용
* HDD는 팀별 독립 스토리지로 운영
* 통합 스토리지 대신 로컬 스토리지 채택
* 팀 간 성능 간섭 최소화 우선

---

### Storage Architecture

#### Architecture Candidates

| Option | Architecture | Diagram |
|---------|-------------|----------|
| A | Local Disk Isolation | <img src="https://github.com/user-attachments/assets/82793cfe-4569-4f6d-9dcb-7d0b2e37d881" width="350"> |
| B | VMware vSAN Shared Storage | <img src="https://github.com/user-attachments/assets/1695b649-9bd0-4cd4-8ce2-1002124a84bb" width="350"> |

#### Selected Architecture

**Option A — Local Disk Isolation**

#### Why Local Storage?

- 팀별 스토리지를 물리적으로 분리하여 I/O 간섭 최소화
- 특정 팀의 디스크 사용량 급증이 다른 팀에 영향을 주지 않음
- vSAN 클러스터 오버헤드 제거
- 제한된 온프레미스 자원 환경에 적합
- 프로젝트 목표인 "서비스 격리"에 부합

#### Trade-off Analysis

| Category | Local Storage | vSAN |
|----------|--------------|------|
| Isolation | High | Medium |
| Scalability | Low | High |
| Availability | Low | High |
| Resource Contention | Low | High |

---

### Network Segmentation

#### Address Space

```text
172.16.0.0/16
```

#### Team Networks

| Team   | Network       |
| ------ | ------------- |
| Team-A | 172.16.1.0/24 |
| Team-B | 172.16.2.0/24 |
| Team-C | 172.16.3.0/24 |
| Team-D | 172.16.4.0/24 |
| Team-E | 172.16.5.0/24 |

#### Network Policy

* VLAN 기반 논리적 분리
* 팀별 독립 서브넷 구성
* 팀 간 직접 통신 차단
* pfSense 경유 트래픽만 허용
* 관리망 별도 분리

---

### Security Architecture

#### Security Principles

* Default Deny 기반 방화벽 정책 적용
* VPN을 통한 접근만 허용
* 관리 네트워크 분리
* ESXi 직접 노출 차단
* 팀 단위 접근 제어

#### WireGuard Port Assignment

| External Port | Internal IP | Internal Port | Protocol | Team   |
| ------------- | ----------- | ------------- | -------- | ------ |
| 51820         | 192.168.0.2 | 51820         | UDP      | Team-A |
| 51821         | 192.168.0.2 | 51821         | UDP      | Team-B |
| 51823         | 192.168.0.9 | 51820         | UDP      | Team-C |
| 51824         | 192.168.0.9 | 51821         | UDP      | Team-D |

#### Security Objectives

* 팀별 VPN 포트 분리
* 포트 충돌 방지
* VPN 외 직접 접근 차단
* 중앙 방화벽 정책 적용

---

### Core Competencies Demonstrated

- 멀티테넌트 인프라 설계
- VMware ESXi 기반 가상화 환경 구축 및 운영
- 스토리지 아키텍처 의사결정
- VLAN 및 Subnet 기반 네트워크 분리 설계
- 방화벽 정책 설계 및 접근 통제
- WireGuard VPN 아키텍처 구성
- 자원 경합 최소화 전략 수립

---

### Conclusion

본 프로젝트는 제한된 온프레미스 자원 환경에서 멀티테넌트 서비스 운영을 위한 격리 중심 아키텍처를 설계한 사례입니다.

특히 스토리지, 네트워크, 보안 영역을 통합적으로 고려하여 팀 간 독립성과 운영 안정성을 확보하는 데 중점을 두었습니다.

---

### Documentation Index

| Document | Description |
|----------|-------------|
| [01. Problem Statement](./01_problem-definition.md) | 프로젝트 배경 및 기술적 과제 |
| [02. Architecture Design](./02_architecture-design.md) | 전체 인프라 설계 |
| [03. Resource Allocation](./03_resource-allocation-strategy.md) | CPU / Memory / Storage 전략 |
| [04. Network Segmentation](./04_network-segmentation.md) | VLAN 및 Subnet 설계 |
| [05. Security Architecture](./05_security-design.md) | Firewall 및 VPN 설계 |
