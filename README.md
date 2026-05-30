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
│   │   └── L2 Switch: PVST+ (백업 경로)
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
<img width="1883" height="1093" alt="네트워크 토폴로지-20260519 drawio" src="https://github.com/user-attachments/assets/ea57bf8c-b6dd-475e-8695-d8a03b63b31b" />

## Cisco 장비 사진
<img width="480" alt="IMG_6292" src="https://github.com/user-attachments/assets/78be18ce-bb80-4dd6-aec4-afaec876fb34" />



### VLAN 설계
#### 본사

| VLAN ID | 이름 | 서브넷 | 주요 서버 |
|---------|------|--------|----------|
| 10 | VLAN-MGT | 172.16.10.0/27 | NMS(.10), RADIUS(.20) |
| 20 | VLAN-SRV | 172.16.20.0/27 | DNS(.10), DHCP(.20), Nginx LB(.29), Nginx BE(.30) |
| 30 | VLAN-OFFICE-1 | 172.16.30.0/24 | 사무 단말 (DHCP 할당) |
| 40 | VLAN-OFFICE-2 | 172.16.40.0/24 | 사무 단말 (DHCP 할당) |

#### 지사

| VLAN ID | 이름 | 서브넷 |
|---------|------|--------|
| 10 | VLAN-OFFICE | 192.168.10.0/24 |
| 20 | VLAN-SRV | 192.168.20.0/27 |

## 구현 기술 스택

### Network
| 장비 | 기술 |
|------|------|
| Router (HQ-R1/R2, BR-R1) | OSPF, NAT, GRE over IPSec |
| L3 Switch (HQ-CSW1/2, BR-CSW) | Inter-VLAN Routing, HSRP, OSPF |
| L2 Switch (HQ-ACC1/2, BR-ACC) | PVST+, VLAN, Port Security, PortFast |
| ASA Firewall (HQ-ASA1/2) | Active/Standby Failover, ACL |
| 공통 | SSH v2, SNMP v2c, AAA (RADIUS) |

### Server & 배포 자동화
서버의 역할과 배포 방식에 따라 CI/CD 기반의 컨테이너 배포와 IaC 기반의 환경 구성으로 나누어 자동화를 구현했습니다.

| 서버 | 배포 방식 | 패키지/이미지 | 역할 |
|------|----------|--------------|------|
| DHCP Server | Ansible + Jinja2 | dhcp-server (ISC DHCP) | VLAN30/40 단말 IP 할당 |
| DNS Server | Ansible + Jinja2 | bind, bind-utils (BIND9) | team2.com 내부 도메인 해석 |
| Nginx LB | Ansible + Jinja2 | nginx | Least Connections 로드밸런싱 (port 80) |
| Nginx Backend | Ansible + Jinja2 | nginx | 정적 페이지 서빙 (port 8080) |
| RADIUS Server | GitHub Actions + Docker | freeradius/freeradius-server:latest | 네트워크 장비 AAA 인증 |
| NMS Server | GitHub Actions + Docker | net-snmp, net-snmp-utils | SNMP 수집 / Trap 수신 |

### CI/CD 및 컨테이너 환경 (NMS, RADIUS)
  * GitHub Actions: 코드가 Push되면 내부망에 구축된 Self-hosted Runner가 동작하여 외부 노출 없이 안전하게 파이프라인을 실행합니다.
  * Docker: NMS와 RADIUS 등 지속적인 관리가 필요한 애플리케이션은 워크플로우를 통해 Docker 컨테이너로 자동 빌드 및 배포되도록 구성했습니다.

### IaC(Ansible) 기반 인프라 구성 (DNS, DHCP, Nginx)
  * Ansible & Jinja2: DNS, DHCP, Web(Nginx) 서버는 Ansible Playbook을 실행해 패키지 설치부터 환경 설정까지 구성합니다. 장비별 가변 데이터는 Jinja2 템플릿으로 렌더링하여 배포 유연성을 높였습니다.

### 도입 장비 수량 요약

| 장비 유형 | 대수 | 모델 | IOS / OS |
|----------|------|------|----------|
| Router | 3대 (HQ-R1, HQ-R2, BR-R1) | Cisco 2811 | IOS 12.4 |
| L3 Switch | 3대 (HQ-CSW1/2, BR-CSW) | Catalyst 3750G (WS-C3750G-24TS-1U) | IOS 12.2 |
| L2 Switch | 3대 (HQ-ACC1/2, BR-ACC) | Catalyst 2950 | IOS 12.1 |
| ASA Firewall | 2대 (Active/Standby) | Cisco ASA 5505 | ASA OS 8.2(5) |

## 보안 포인트

- **ASA Failover:** HQ-ASA1(Active) ↔ HQ-ASA2(Standby) — failover link로 상태 동기화
- **ACL 정책:** 지사 → 본사 DNS/Nginx 접근만 허용 (출발지: 지사 사설 IP, 목적지: 본사 사설 IP + 서비스 포트)
- **AAA/Radius:** 모든 장비 접근 중앙 인증
- **Port Security:** Access 포트 MAC 기반 접근 제어
- **GRE over IPSec:** 본사-지사 간 암호화 터널
