# SSL Certificates

Let's Encrypt 인증서 파일을 이 디렉토리에 배치하세요.

## 필요한 파일

```
certificates/
├── fullchain.pem    # 전체 인증서 체인 (인증서 + 중간 인증서)
├── privkey.pem      # 개인 키
└── README.md        # 이 파일
```

## Let's Encrypt 인증서 획득 방법

### 1. Certbot 설치

```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install certbot

# macOS
brew install certbot
```

### 2. 인증서 발급 (도메인이 있는 경우)

```bash
sudo certbot certonly --standalone -d yourdomain.com -d www.yourdomain.com
```

### 3. 인증서 복사

```bash
# 인증서 복사 (권한 설정 포함)
sudo cp /etc/letsencrypt/live/yourdomain.com/fullchain.pem ./certificates/
sudo cp /etc/letsencrypt/live/yourdomain.com/privkey.pem ./certificates/
sudo chown $USER:$USER ./certificates/*.pem
chmod 644 ./certificates/fullchain.pem
chmod 600 ./certificates/privkey.pem
```

## 로컬 개발용 자체 서명 인증서

개발 환경에서 테스트용:

```bash
cd certificates
openssl req -x509 -newkey rsa:4096 -keyout privkey.pem -out fullchain.pem -days 365 -nodes \
  -subj "/C=KR/ST=Seoul/L=Seoul/O=InterviewAI/CN=localhost"
```

## 주의사항

- `privkey.pem`은 절대 Git에 커밋하지 마세요 (.gitignore에 포함됨)
- 프로덕션 환경에서는 유효한 SSL 인증서를 사용하세요
- 인증서 갱신 시 파일을 교체한 후 `docker-compose restart nginx` 실행
