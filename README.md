# n8n HTTPS 호스팅 (Caddy + DuckDNS)

VPS에 n8n을 HTTPS로 셀프 호스팅하기 위한 Docker Compose 설정입니다.

## 주요 기능

- ✅ **자동 HTTPS**: Caddy가 Let's Encrypt 인증서 자동 발급/갱신
- ✅ **자동 DDNS**: DuckDNS가 5분마다 공인 IP 자동 업데이트
- ✅ **데이터 영속성**: Docker volumes로 n8n 데이터 및 인증서 보존
- ✅ **WebSocket 지원**: n8n webhook 및 실시간 기능 완벽 지원
- ✅ **자동 재시작**: 서버 재부팅 시 컨테이너 자동 시작

## 아키텍처

```
인터넷 → DuckDNS → VPS:80/443 → Caddy (HTTPS) → n8n:5678
```

## 사전 준비

1. **DuckDNS 계정 및 토큰**
   - https://www.duckdns.org/ 에서 무료 계정 생성
   - 원하는 서브도메인 생성 (예: myapp → myapp.duckdns.org)
   - 토큰 발급 받기

2. **VPS 요구사항**
   - Docker 및 Docker Compose 설치
   - 공인 IP 주소
   - 포트 80, 443 오픈 필요

## 설치 방법

### 1. 저장소 클론 (또는 파일 다운로드)

```bash
git clone https://github.com/genju83/n8n.git
cd n8n
```

### 2. 설정 파일 생성

`.env.example`과 `Caddyfile.example` 파일을 복사하여 실제 설정 파일을 생성하세요:

```bash
# 환경 변수 파일 생성
cp .env.example .env

# Caddy 설정 파일 생성
cp Caddyfile.example Caddyfile
```

### 3. 환경 변수 설정

`.env` 파일을 편집하여 다음 값을 설정하세요:

```bash
nano .env  # 또는 vi, vim 등
```

설정할 값:

```bash
# DuckDNS 토큰 및 서브도메인 입력
DUCKDNS_TOKEN=your-duckdns-token-here
DUCKDNS_SUBDOMAIN=your-subdomain

# 도메인 설정 (your-subdomain을 실제 서브도메인으로 변경)
DOMAIN=your-subdomain.duckdns.org
N8N_HOST=your-subdomain.duckdns.org
N8N_PROTOCOL=https
WEBHOOK_URL=https://your-subdomain.duckdns.org

# 타임존 설정 (필요시 변경)
GENERIC_TIMEZONE=Asia/Seoul

# SMTP 설정 (선택사항 - Resend 사용 시)
N8N_EMAIL_MODE=smtp
N8N_SMTP_HOST=smtp.resend.com
N8N_SMTP_PORT=587
N8N_SMTP_USER=resend
N8N_SMTP_PASS=your-resend-api-key
N8N_SMTP_SENDER=onboarding@resend.dev
```

### 4. Caddyfile 도메인 설정

`Caddyfile`을 편집하여 도메인을 실제 서브도메인으로 변경하세요:

```bash
nano Caddyfile
```

도메인 변경:
```caddy
your-subdomain.duckdns.org {
    reverse_proxy n8n:5678
}
```

예시 (DuckDNS 서브도메인이 `myapp`인 경우):
```caddy
myapp.duckdns.org {
    reverse_proxy n8n:5678
}
```

### 5. 방화벽 설정

VPS에서 포트 80, 443을 열어야 합니다:

```bash
# Ubuntu/Debian (ufw 사용 시)
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw reload

# CentOS/RHEL (firewalld 사용 시)
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

### 6. Docker Compose 실행

```bash
# 백그라운드에서 컨테이너 실행
docker-compose up -d

# 로그 확인
docker-compose logs -f
```

### 7. 접속 확인

브라우저에서 `https://your-subdomain.duckdns.org` 로 접속하여 n8n 초기 설정을 진행하세요.

## 관리 명령어

### 컨테이너 상태 확인
```bash
docker-compose ps
```

### 로그 확인
```bash
# 전체 로그
docker-compose logs -f

# 특정 서비스 로그
docker-compose logs -f caddy
docker-compose logs -f n8n
docker-compose logs -f duckdns
```

### 컨테이너 재시작
```bash
docker-compose restart
```

### 컨테이너 중지
```bash
docker-compose down
```

### 볼륨 데이터까지 완전 삭제 (주의!)
```bash
docker-compose down -v
```

## 데이터 백업

n8n 데이터는 Docker volume에 저장됩니다. 백업 방법:

```bash
# n8n 데이터 백업
docker run --rm -v n8n_n8n_data:/data -v $(pwd):/backup ubuntu tar czf /backup/n8n-backup.tar.gz /data

# 복원
docker run --rm -v n8n_n8n_data:/data -v $(pwd):/backup ubuntu tar xzf /backup/n8n-backup.tar.gz -C /
```

## 문제 해결

### 인증서 발급 실패

도메인이 올바르게 설정되었는지 확인:
```bash
nslookup your-subdomain.duckdns.org
```

Caddy 로그 확인:
```bash
docker-compose logs caddy
```

### DuckDNS IP 업데이트 안됨

DuckDNS 로그 확인:
```bash
docker-compose logs duckdns
```

토큰이 올바른지 확인하고, 필요시 컨테이너 재시작:
```bash
docker-compose restart duckdns
```

### n8n 접속 불가

1. 컨테이너 상태 확인:
   ```bash
   docker-compose ps
   ```

2. n8n 로그 확인:
   ```bash
   docker-compose logs n8n
   ```

3. 방화벽 설정 확인:
   ```bash
   sudo ufw status
   ```

## 파일 구조

```
.
├── docker-compose.yml    # Docker Compose 설정
├── Caddyfile            # Caddy 리버스 프록시 설정 (Git에 커밋 안됨)
├── Caddyfile.example    # Caddy 설정 템플릿
├── .env                 # 환경 변수 (토큰, 도메인 등) - Git에 커밋 안됨
├── .env.example         # 환경 변수 템플릿
├── .gitignore           # Git 제외 파일
└── README.md            # 이 파일
```

**주의**: `Caddyfile`과 `.env` 파일은 `.gitignore`에 포함되어 Git에 커밋되지 않습니다.

## 업데이트

최신 이미지로 업데이트:

```bash
# 이미지 다운로드
docker-compose pull

# 컨테이너 재생성 및 시작
docker-compose up -d
```

## 보안 권장사항

1. `.env`와 `Caddyfile` 파일은 절대 Git에 커밋하지 마세요 (이미 .gitignore 처리됨)
2. n8n 초기 설정 시 강력한 비밀번호 사용
3. 정기적으로 Docker 이미지 업데이트
4. 필요시 Caddy에 기본 인증(Basic Auth) 추가 고려
5. SMTP API 키는 절대 외부에 노출하지 마세요

## 라이선스

이 설정은 자유롭게 사용 가능합니다.
