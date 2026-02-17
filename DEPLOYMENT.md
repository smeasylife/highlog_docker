# HTTPS 배포 가이드

이 문서는 nginx 리버스 프록스를 사용하여 HTTPS를 설정하는 방법을 안내합니다.

## 📋 사전 준비 사항

### 1. 도메인 및 DNS 설정
- 도메인이 있어야 합니다 (예: `yourdomain.com`)
- DNS A 레코드가 VM의 공인 IP를 가리키고 있어야 합니다

### 2. 방화벽 포트 오픈
VM의 방화벽에서 다음 포트가 열려 있어야 합니다:
```bash
# AWS Security Group 또는 방화벽 규칙
HTTP  (80)   - open
HTTPS (443)  - open
```

### 3. Docker 및 Docker Compose 설치
```bash
# Docker 설치 확인
docker --version
docker-compose --version
```

---

## 🚀 배포 단계

### Step 1: 코드 배포

```bash
# 프로젝트 디렉토리로 이동
cd /path/to/interview-ai

# Git pull (코드 업데이트)
git pull origin main
```

### Step 2: SSL 인증서 발급 (Let's Encrypt)

#### 2-1. Certbot 설치

```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install certbot

# CentOS/RHEL
sudo yum install certbot
```

#### 2-2. 인증서 발급

**방법 A: Standalone (nginx를 중지하고 발급)**

```bash
# 1. 기존에 nginx가 실행 중이면 중지
sudo systemctl stop nginx  # 또는 docker-compose down

# 2. 인증서 발급
sudo certbot certonly --standalone \
  -d yourdomain.com \
  -d www.yourdomain.com \
  --email your-email@example.com \
  --agree-tos \
  --non-interactive

# 3. 발급된 인증서 복사
sudo cp /etc/letsencrypt/live/yourdomain.com/fullchain.pem ./certificates/
sudo cp /etc/letsencrypt/live/yourdomain.com/privkey.pem ./certificates/

# 4. 권한 설정
sudo chown $USER:$USER ./certificates/*.pem
chmod 644 ./certificates/fullchain.pem
chmod 600 ./certificates/privkey.pem
```

**방법 B: Webroot (nginx 실행 중에 발급 - 권장)**

```bash
# 1. docker-compose로 nginx 시작 (HTTP만)
docker-compose up -d nginx

# 2. webroot 플러그인으로 인증서 발급
sudo certbot certonly --webroot \
  -w /var/www/html \
  -d yourdomain.com \
  -d www.yourdomain.com \
  --email your-email@example.com \
  --agree-tos \
  --non-interactive

# 3. 발급된 인증서 복사
sudo cp /etc/letsencrypt/live/yourdomain.com/fullchain.pem ./certificates/
sudo cp /etc/letsencrypt/live/yourdomain.com/privkey.pem ./certificates/

# 4. 권한 설정
sudo chown $USER:$USER ./certificates/*.pem
chmod 644 ./certificates/fullchain.pem
chmod 600 ./certificates/privkey.pem

# 5. nginx 재시작
docker-compose restart nginx
```

### Step 3: 환경 변수 설정

`.env` 파일에서 도메인에 맞게 CORS 설정을 변경하세요:

```bash
# .env 파일 수정
vim .env
```

```env
# 실제 배포 도메인으로 변경
CORS_ORIGINS=["https://yourdomain.com","https://www.yourdomain.com"]
```

### Step 4: Docker Compose 실행

```bash
# 전체 서비스 시작
docker-compose up -d

# 로그 확인
docker-compose logs -f nginx

# 또는 전체 로그
docker-compose logs -f
```

### Step 5: 동작 확인

```bash
# 1. 컨테이너 상태 확인
docker-compose ps

# 2. HTTP → HTTPS 리다이렉트 테스트
curl -I http://yourdomain.com
# 결과: 301 Moved Permanently -> https://...

# 3. HTTPS 테스트
curl -I https://yourdomain.com
# 결과: 200 OK

# 4. API 엔드포인트 테스트
curl https://yourdomain.com/health
curl https://yourdomain.com/api/health
curl https://yourdomain.com/ai/health
```

---

## 🔧 경로 라우팅 구조

| 경로 | 대상 서비스 | 설명 |
|------|-----------|------|
| `/api/*` | Spring Boot (8080) | 메인 백엔드 API |
| `/ai/*` | FastAPI (8000) | AI 서비스 |
| `/health` | nginx | 헬스체크 |
| `/` | nginx | 안내 메시지 |

---

## 🔄 SSL 인증서 자동 갱신 설정

Let's Encrypt 인증서는 90일마다 갱신해야 합니다.

### crontab 등록

```bash
# crontab 편집
crontab -e

# 매월 1일 새벽 3시에 갱신 시도
0 3 1 * * certbot renew --quiet --post-hook "cd /path/to/interview-ai && cp /etc/letsencrypt/live/yourdomain.com/fullchain.pem ./certificates/ && cp /etc/letsencrypt/live/yourdomain.com/privkey.pem ./certificates/ && docker-compose restart nginx"
```

또는 갱신 스크립트 생성:

```bash
#!/bin/bash
# /path/to/interview-ai/renew-ssl.sh

certbot renew --quiet --post-hook "cd /path/to/interview-ai && \
  cp /etc/letsencrypt/live/yourdomain.com/fullchain.pem ./certificates/ && \
  cp /etc/letsencrypt/live/yourdomain.com/privkey.pem ./certificates/ && \
  docker-compose restart nginx"
```

---

## 🐛 트러블슈팅

### nginx가 시작하지 않을 때

```bash
# nginx 로그 확인
docker-compose logs nginx

# 설정 테스트
docker-compose exec nginx nginx -t

# 인증서 파일 존재 확인
ls -la ./certificates/
```

### 502 Bad Gateway 에러

```bash
# 백엔드 서비스 상태 확인
docker-compose ps
docker-compose logs highlog-backend
docker-compose logs ai-service

# 네트워크 확인
docker network ls
docker network inspect interview-ai_app-network
```

### CORS 에러

`.env` 파일의 `CORS_ORIGINS`에 정확한 도메인이 포함되어 있는지 확인하세요.

### SSL 인증서 오류

```bash
# 인증서 유효기간 확인
openssl x509 -in ./certificates/fullchain.pem -text -noout | grep "Not After"

# 인증서와 키가 매칭되는지 확인
CERT_MOD=$(openssl x509 -noout -modulus -in ./certificates/fullchain.pem | openssl md5)
KEY_MOD=$(openssl rsa -noout -modulus -in ./certificates/privkey.pem | openssl md5)
echo "Cert: $CERT_MOD"
echo "Key:  $KEY_MOD"
# 두 값이 같아야 함
```

---

## 📝 체크리스트

배포 전 확인:

- [ ] 도메인 DNS 설정 완료
- [ ] 방화벽 80, 443 포트 오픈
- [ ] SSL 인증서 발급 및 `./certificates/`에 배치
- [ ] `.env` 파일 `CORS_ORIGINS` HTTPS로 변경
- [ ] Docker & Docker Compose 설치 확인
- [ ] `docker-compose up -d` 실행
- [ ] HTTP → HTTPS 리다이렉트 확인
- [ ] API 엔드포인트 동작 확인
- [ ] SSL 자동 갱신 crontab 등록

---

## 🆘 긴급 시나리오

### SSL 인증서가 만료되었을 때

```bash
# 즉시 갱신
sudo certbot renew --force-renewal

# 파일 복사
sudo cp /etc/letsencrypt/live/yourdomain.com/fullchain.pem ./certificates/
sudo cp /etc/letsencrypt/live/yourdomain.com/privkey.pem ./certificates/

# nginx 재시작
docker-compose restart nginx
```

### 컨테이너가 계속 재시작될 때

```bash
# 전체 재시작
docker-compose down
docker-compose up -d

# 로그 확인
docker-compose logs -f --tail=100

# 특정 서비스만 재시작
docker-compose restart nginx
docker-compose restart highlog-backend
docker-compose restart ai-service
```
