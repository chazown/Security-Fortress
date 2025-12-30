# 🛡️ Oracle Cloud Security Fortress

개인용 클라우드 환경에서 WireGuard 와 AdGuard Home 을 통합 구축하는 마스터 가이드입니다.

## 🧱 1. 오라클 클라우드 콘솔 설정 (Ingress Rules)
서버 접속 전, 아래 포트들을 반드시 개방하십시오.

| 프로토콜 | 포트 | 용도 |
| :--- | :--- | :--- |
| **TCP** | `51821`, `3000`, `53`, `853` | 관리 UI 및 서비스 |
| **UDP** | `51820`, `53` | VPN 터널 및 DNS 질의 |

## 🚀 2. 빠른 설치 방법
서버에 SSH로 접속한 뒤, 아래 명령어를 실행하여 자동 설치 스크립트를 가동하십시오.

```bash
chmod +x setup.sh
./setup.sh
