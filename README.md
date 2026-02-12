# 🛡️ Cloud Security Fortress

Oracle Cloud Always Free Tier(AMD E2.1.Micro 추천)에서  
🔒 **WireGuard** + 🛡️ **AdGuard Home** 을 구축하는 현실적이고 투명한 가이드입니다.

- 전체 트래픽 VPN 터널링 (네이버·쿠팡 등 한국 사이트 접속 최적화)
- 네트워크 전체 광고·추적·악성 차단
- Docker Compose + setup.sh 제공 (직접 확인 가능, 투명함)
- 한국 현실 반영: 사이트 접속 문제 경고 + 대처법 포함

**2026년 기준 최신 방법** (wg-easy:latest + AdGuard Home:latest) 🚀

## ⚠️ 치명적 주의사항 – 한국 사용자 필독! ⚠️

AdGuard Home의 강력한 차단 기능 때문에 **네이버·다음·쿠팡·쇼핑몰 등이 로딩 안 되거나 타임아웃** 되는 문제가 자주 발생합니다 😩

- 원인 후보: 과도한 blocklist, upstream DNS(Quad9 등) + 한국 CDN 충돌, EDNS Client Subnet, 일부 ISP(LG U+) 호환성
- 실제 사례: 네이버 앱/웹에서 "dns.adguard" 오류 또는 무한 로딩 보고됨

**즉시 대처 순서** (문제 발생 시 꼭 따라하세요!):
1. AdGuard UI → **Filters → DNS blocklists** → 한국 무관/과도 리스트 비활성화
2. **Settings → DNS settings** → **EDNS client subnet** ❌ 체크 해제
3. Upstream DNS 변경 테스트: **[Cloudflare](https://1.1.1.1/dns-query)** 또는 **[Google](https://dns.google/dns-query)**
4. 그래도 안 되면 AdGuard Home 중지 (`docker compose down`) → WireGuard만 사용 (광고차단 포기)
5. 최후: AdGuard 대신 Pi-hole이나 Unbound 고려

많은 유저가 이 문제로 AdGuard를 **선택적**으로만 씁니다. 미리 테스트 필수! ⚠️

## 🎯 목표 아키텍처
```bash
🌐 인터넷
   ↕
🔒 WireGuard 클라이언트 (폰/PC)
   │ 암호화 터널 (UDP 51820)
   ↓
OCI 공인 IP
   │
   └─→ 🛡️ AdGuard Home DNS (10.8.0.1:53)
         ↓
   🌍 외부 인터넷 (서버 IP 위장)
```

## 준비물 ✅

- Oracle Cloud 계정 (Always Free Tier)
- SSH 클라이언트
- WireGuard 앱 (미리 설치)

## 🖥️ 1. Oracle Cloud 인스턴스 생성

1. https://cloud.oracle.com → Compute → Instances → Create instance
2. OS: **Ubuntu 24.04 LTS** (최신 추천)
3. Shape: **VM.Standard.E2.1.Micro** (AMD)
4. Public IP: ✅ 할당
5. SSH 키: 생성 또는 업로드 (private key 보관!)
6. Boot volume: **50~100GB** (무료 한도 총 200GB 내 – 100GB로 해도 OK, 하지만 나중 추가 스토리지 여유 고려)
7. Create → 공인 IP 복사

## 🧱 2. OCI Security List 포트 열기

| 프로토콜 | Source CIDR   | Port Range | 용도                     |
|----------|---------------|------------|--------------------------|
| TCP      | 0.0.0.0/0     | 22         | 🔑 SSH                   |
| UDP      | 0.0.0.0/0     | 51820      | 🔒 WireGuard 터널        |
| TCP      | 0.0.0.0/0     | 51821      | 🖥️ WireGuard UI         |
| TCP      | 0.0.0.0/0     | 3000       | 🛡️ AdGuard 초기 설정 (후 삭제 추천) |
| TCP/UDP  | 0.0.0.0/0     | 53         | 🌐 DNS 질의              |

## 🚀 3. 서버 접속 & setup.sh 실행

로컬 PC:

```bash
# 개인키 권한 설정 (400 추천 – 실행 권한 차단)
chmod 400 your-key-name.key

# 서버 접속
ssh -i your-key-name.key ubuntu@<YOUR_ORACLE_IP>
```

## 🚀 4. 저장소 클론 (또는 직접 파일 생성)

git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git
cd YOUR_REPO

chmod +x setup.sh
./setup.sh
setup.sh가 Docker 설치·IP 포워딩·폴더 생성 등을 자동으로 해줍니다.

## ⚙️ 5. 서비스 실행 

```bash
# WireGuard
cd wireguard
docker compose up -d

# AdGuard Home
cd ../adguard
docker compose up -d
```

## 🌍 6. 초기 설정 (브라우저) 
```bash
AdGuard Home: http://YOUR_IP:3000
→ 계정 생성 → DNS settings
Upstream 추천 (문제 최소화):
https://1.1.1.1/dns-query
https://dns.google/dns-query
EDNS client subnet: ❌ 비활성화
DNSSEC: ✅ (문제 시 꺼보기)

WireGuard UI: http://<YOUR_ORACLE_IP>:51821
→ docker-compose.yml에서 설정한 PASSWORD로 로그인
→ New Client → QR/파일 다운로드
```

## 📱💻 7. 클라이언트 연결 & 테스트

WireGuard 앱 → QR/파일 임포트 → 연결

curl ifconfig.me               # 서버 IP 나와야 함
nslookup doubleclick.net       # 0.0.0.0 또는 차단됨

사이트 안 열릴 때 → 위 치명적 주의사항 대처법 따라가세요!

## 🔐 8. 사후 관리 및 접속 정보 (Post-Installation)

설치가 완료되면 아래 주소로 접속하여 서비스를 관리할 수 있습니다.

### 🛡️ AdGuard Home 접속
- **주소:** `http://<YOUR_ORACLE_IP>:3000`
- **관리자 설정:** 최초 접속 시 본인만의 ID/PW를 생성해야 합니다.
 
### 🛰️ WireGuard 접속
- **주소:** `http://<YOUR_ORACLE_IP>:51821`
- **비밀번호:** `PASSWORD는 docker-compose.yml에서 설정한 값`
- **특징:** 최신 v14+ UI, QR 코드 생성 및 클라이언트 관리.

### 🔄 유지보수 명령어 
```bash
sudo docker ps #상태 확인
sudo docker compose pull && docker compose up -d # 각 폴더에서
```

## 🔐 보안 & 팁

- 설정 끝난 후 OCI Security List에서 포트 3000 삭제
- WireGuard UI 비밀번호: 12자 이상 + 특수문자
- 서버 IP 절대 공개 금지!
- 정기 업데이트: sudo apt update && sudo apt upgrade
