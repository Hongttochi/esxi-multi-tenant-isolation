# 02. 아키텍처 설계

<img width="1692" height="692" alt="image" src="https://github.com/user-attachments/assets/dbde382c-61ea-4af3-b974-b7ca1b668e00" />

## 2.1 전체 구성

본 시스템은 가상화 기반 인프라 위에서 네트워크 분리와 보안 경계를 명확히 하는 구조로 설계되었다. 핵심 구성 요소는 다음과 같다.

- **Hypervisor**: VMware ESXi
- **Firewall / Gateway**: pfSense
- **VPN Access**: WireGuard
- **Network Segmentation**
  - Internal Network
  - Management Network
  - Internet Network

전체 트래픽은 pfSense를 중심으로 제어되며, 외부 접근은 VPN을 통해서만 허용된다.

---

## 2.2 물리 구성

<img width="753" height="534" alt="image" src="https://github.com/user-attachments/assets/2522d068-0e7c-4eb8-becc-464470f87fc1" />

본 환경은 총 3대의 ESXi 서버로 구성되어 있으며, 각 서버는 팀 단위 워크로드를 분산 처리하도록 설계되었다.

- GSC01
- GSC04
- GSC05

각 ESXi 호스트에는 서로 다른 팀의 VM이 분산 배치되어 있으며, 단일 호스트 장애가 전체 서비스에 미치는 영향을 최소화하도록 구성하였다.

또한 네트워크 안정성과 보안을 강화하기 위해 Internet Switch와 Management Switch를 물리적으로 분리하였다. 이를 통해 관리 트래픽과 외부 트래픽의 경로를 명확히 분리하고, 관리망의 외부 노출을 원천적으로 차단하였다.

---

## 2.3 설계 원칙

본 아키텍처는 운영 편의성보다 보안성과 격리성을 우선으로 설계되었다. 주요 설계 원칙은 다음과 같다.

- Single Point of Control (pfSense 중심 제어 구조)
- Default Deny 정책 기반 접근 제어
- VM 단위 격리로 테넌트 간 영향 최소화
- 외부 접근은 WireGuard VPN을 통한 인증 기반 접속만 허용
- Management Network는 외부 네트워크와 완전 분리

---

## 2.4 설계 방향 (가용성 vs 격리)

본 시스템은 고가용성(HA) 중심의 엔터프라이즈 운영 환경이 아니라, 제한된 물리 자원 내에서 다수 팀이 독립적으로 서비스를 운영할 수 있는 **격리 중심 환경**으로 설계되었다.

이에 따라 vSAN 기반 공유 스토리지 및 클러스터 HA 기능 대신 로컬 디스크 기반 운영 구조를 채택하였다. 이를 통해 다음과 같은 효과를 얻는다.

- 팀별 자원 독립성 확보
- 스토리지 경합 및 성능 간섭 최소화
- 장애 영향 범위의 물리적 분리

즉, 고가용성보다는 실사용 환경에서의 격리성과 예측 가능한 성능을 우선한 구조이다.

---

## 2.5 아키텍처 목표

본 아키텍처의 최종 목표는 클라우드 환경과 유사한 멀티테넌시 구조를 온프레미스 환경에서 구현하는 것이다.

- 클라우드형 멀티테넌시 구조 구현
- 팀 간 네트워크 및 자원 간섭 방지
- 보안 중심의 폐쇄형 운영 모델 구축
- VPN 기반 접근 통제로 외부 공격 표면 최소화
- 물리/가상 계층 모두에서 명확한 격리 경계 확보
