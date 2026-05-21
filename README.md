# On-Premise-project

## 네트워크 이중화 및 서버 구축 자동화 프로젝트
---
> **핵심 한 줄 요약:** Cisco 장비 기반의 이중화 네트워크를 구축하고, Ansible + Docker로 서버 배포 전체를 자동화한 인프라 프로젝트

## 네트워크 토폴로지 개요
<img width="1813" height="1121" alt="네트워크 토폴로지-20260521" src="https://github.com/user-attachments/assets/af0d3a47-aea5-451d-b007-50ab92f72328" />


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

## 🔧 구현 기술 스택
 
### Network
| 장비 | 기술 |
|------|------|
| Router (HQ-R1/R2, BR-R1) | OSPF, NAT, GRE over IPSec |
| L3 Switch (HQ-CSW1/2) | Inter-VLAN Routing, HSRP, OSPF |
| L2 Switch (HQ-ACC, BR-ACC) | STP, VLAN, Port Security, PortFast |
| ASA Firewall (x2) | Active/Standby Failover, ACL, 보안 정책 |
| 공통 | SSH, SNMP, AAA (Radius) |

### 도입 장비 수량 요약
- Router: 3대 (HQ-R1, HQ-R2, BR-R1)
- L3 Switch: 3대 / L2 Switch: 3대 (이상)
- ASA Firewall: 2대 (Active/Standby)
- DNS Server: 1대 / WEB(Nginx) Server: 2대
- DHCP Server: 2대 / NMS Server: 1대 / Radius Server: 1대
- 관리자PC: 1대
 
### Server / Automation
| 기술 | 역할 |
|------|------|
| **Ansible** | 네트워크 장비 및 서버 자동 구성 |
| **Jinja2** | 서버 설정파일 템플릿 관리 |
| **Docker** | 서비스 컨테이너화 (DNS, DHCP, Nginx, DHCP) |

## 🔒 보안 포인트
 
- **ASA Failover:** HQ-ASA1(Active) ↔ HQ-ASA2(Standby) — failover link로 상태 동기화
- **ACL 정책:** 지사 → 본사 DNS/Nginx 접근만 허용 (출발지: 지사 사설 IP, 목적지: 본사 사설 IP + 서비스 포트)
- **AAA/Radius:** 모든 장비 접근 중앙 인증
- **Port Security:** Access 포트 MAC 기반 접근 제어
- **GRE over IPSec:** 본사-지사 간 암호화 터널

