# On-Premise-project

## 네트워크 이중화 및 서버 구축 자동화 프로젝트
---
> **핵심 한 줄 요약:** Cisco 장비 기반의 무중단 고가용성(HA) 네트워크를 설계하고, Ansible과 GitHub Actions(Docker)를 목적에 맞게 분리 적용하여 서버 배포를 자동화한 인프라 통합 프로젝트입니다.

### Project Architecture
```
├── 본사 (HQ)
│   ├── Network HA
│   │   ├── L3 Switch: HSRP (게이트웨이 이중화)
│   │   ├── ASA FW: Active/Standby Failover
│   │   └── L2 Switch: STP (백업 경로)
│   └── Server Farm
│       ├── IaC (Ansible): DHCP, DNS, Nginx
│       └── CI/CD (Docker): RADIUS, NMS
│
└── VPN Tunnel (GRE over IPSec)
    │
    └── 지사 (Branch)
        └── 사설망 통신 보호 및 본사 인프라 연동
```

## 네트워크 토폴로지 개요
<img width="1883" height="1093" alt="네트워크 토폴로지-20260519 drawio" src="https://github.com/user-attachments/assets/ea57bf8c-b6dd-475e-8695-d8a03b63b31b" />



### VLAN 설계 
#### 본사

| VLAN ID | 이름 | 용도 |
|---------|------|------|
| 10 | VLAN-MGT | 관리망 |
| 20 | VLAN-SRV | 서버팜 |
| 30 | VLAN-OFFICE-1 | 사무1 |
| 40 | VLAN-OFFICE-2 | 사무2 |

#### 지사

| VLAN ID | 이름 | 용도 |
|---------|------|------|
| 10 | VLAN-OFFICE | 사무 |
| 20 | VLAN-SRV | 서버팜 |

## 구현 기술 스택
 
### Network
| 장비 | 기술 |
|------|------|
| Router (HQ-R1/R2, BR-R1) | OSPF, NAT, GRE over IPSec |
| L3 Switch (HQ-CSW1/2) | Inter-VLAN Routing, HSRP, OSPF |
| L2 Switch (HQ-ACC, BR-ACC) | STP, VLAN, Port Security, PortFast |
| ASA Firewall (x2) | Active/Standby Failover, ACL, 보안 정책 |
| 공통 | SSH, SNMP, AAA (Radius) |
 
### 2. Server & 배포 자동화
서버의 역할과 배포 방식에 따라 CI/CD 기반의 컨테이너 배포와 IaC 기반의 환경 구성으로 나누어 자동화를 구현했습니다.

* CI/CD 및 컨테이너 환경 (NMS, Radius)
  * GitHub Actions: 코드가 Push되면 내부망에 구축된 Self-hosted Runner가 동작하여 외부 노출 없이 안전하게 파이프라인을 실행합니다.
  * Docker: NMS와 Radius 등 지속적인 관리가 필요한 애플리케이션은 워크플로우를 통해 Docker 컨테이너로 자동 빌드 및 배포되도록 구성했습니다.
* IaC(Ansible) 기반 인프라 구성 (DNS, DHCP, Nginx)
  * Ansible & Jinja2: DNS, DHCP, Web(Nginx) 서버는 Ansible Playbook을 실행해 패키지 설치부터 환경 설정까지 구성합니다. 장비별 가변 데이터는 Jinja2 템플릿으로 렌더링하여 배포 유연성을 높였습니다.
 
### 도입 장비 수량 요약
- Router: 3대 (HQ-R1, HQ-R2, BR-R1)
- L3 Switch: 3대 / L2 Switch: 3대 (이상)
- ASA Firewall: 2대 (Active/Standby)
- DNS Server: 1대 / WEB(Nginx) Server: 2대
- DHCP Server: 2대 / NMS Server: 1대 / Radius Server: 1대
- 관리자PC: 1대

## 보안 포인트
 
- **ASA Failover:** HQ-ASA1(Active) ↔ HQ-ASA2(Standby) — failover link로 상태 동기화
- **ACL 정책:** 지사 → 본사 DNS/Nginx 접근만 허용 (출발지: 지사 사설 IP, 목적지: 본사 사설 IP + 서비스 포트)
- **AAA/Radius:** 모든 장비 접근 중앙 인증
- **Port Security:** Access 포트 MAC 기반 접근 제어
- **GRE over IPSec:** 본사-지사 간 암호화 터널

