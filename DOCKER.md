# Docker Compose 사용 가이드

## 🚀 빠른 시작

### 1. 환경 설정

```bash
# .env.example 파일을 복사하여 .env 파일 생성
cp .env.example .env

# .env 파일에 필요한 값 입력 (필수 항목)
# - AWS S3 credentials
# - Google API Key
# - Email credentials
```

### 2. Docker 컨테이너 시작

```bash
# 모든 서비스 시작 (PostgreSQL, Redis, Spring Boot, FastAPI)
docker-compose up -d

# 로그 보기
docker-compose logs -f

# 특정 서비스 로그만 보기
docker-compose logs -f ai-service
docker-compose logs -f highlog-backend
```

### 3. 서비스 중지

```bash
# 모든 서비스 중지
docker-compose down

# 볼륨까지 모두 삭제 (데이터 초기화)
docker-compose down -v
```

## 📁 포트 매핑

| 서비스 | 내부 포트 | 외부 포트 | 접속 |
|--------|----------|----------|------|
| PostgreSQL | 5432 | 5432 | localhost:5432 |
| Redis | 6379 | 6379 | localhost:6379 |
| HighLog Backend | 8080 | 8080 | http://localhost:8080 |
| AI Service | 8000 | 8000 | http://localhost:8000 |

## 🔧 환경변수 설정

### 데이터베이스

```bash
POSTGRES_HOST=postgres          # Docker 내부 네트워크 호스트명
POSTGRES_PORT=5432
POSTGRES_DB=highlog
POSTGRES_USER=ksm9203
POSTGRES_PASSWORD=ksm1234
```

### AWS S3

```bash
AWS_ACCESS_KEY_ID=your_key
AWS_SECRET_ACCESS_KEY=your_secret
AWS_REGION=ap-northeast-2
AWS_S3_BUCKET_NAME=highlog-records
```

### Google AI

```bash
GOOGLE_API_KEY=your_google_api_key
```

### Azure Speech (TTS / Viseme)

```bash
AZURE_SPEECH_KEY=your_azure_speech_key
AZURE_SPEECH_REGION=koreacentral
AZURE_SPEECH_VOICE_NAME=ko-KR-InJoonNeural
AZURE_SPEECH_LANGUAGE=ko-KR
```

### JWT

```bash
JWT_SECRET=your_secret_key
```

## 🔄 로컬 개발 시 DB 접속

### PostgreSQL

```bash
# Docker 컨테이너 접속
docker exec -it highlog-postgres psql -U ksm9203 -d highlog

# 로컬에서 접속 (psql 설치 필요)
psql -h localhost -p 5432 -U ksm9203 -d highlog
```

### Redis

```bash
# Docker 컨테이너 접속
docker exec -it highlog-redis redis-cli

# 로컬에서 접속 (redis-cli 설치 필요)
redis-cli -h localhost -p 6379
```

## 🐛 문제 해결

### 포트 충돌

```bash
# 5432 포트 사용 중인 프로세스 확인
lsof -i :5432

# 6379 포트 사용 중인 프로세스 확인
lsof -i :6379
```

### 컨테이너 재시작

```bash
# 특정 서비스 재시작
docker-compose restart ai-service
docker-compose restart highlog-backend

# 전체 재시작
docker-compose restart
```

### 로그 확인

```bash
# 전체 로그
docker-compose logs

# 최근 100줄
docker-compose logs --tail=100

# 실시간 로그
docker-compose logs -f
```

## 📊 데이터 볼륨 관리

```bash
# 볼륨 목록 확인
docker volume ls

# postgres_data 볼륨 삭제 (데이터 초기화)
docker volume rm interview-ai_postgres_data

# redis_data 볼륨 삭제
docker volume rm interview-ai_redis_data
```

## 🔄 기존 로컬 DB 사용하기

로컬에서 이미 사용 중인 PostgreSQL과 Redis가 있다면:

```bash
# .env 파일에서 호스트 설정 변경
POSTGRES_HOST=host.docker.internal  # Docker에서 로컬 호스트 접근
REDIS_HOST=host.docker.internal
```

또는 docker-compose.yml에서 포트 매핑만 사용하고 서비스 정의를 주석처리:

```yaml
services:
  # postgres:
  #   ... (주석 처리)

  # redis:
  #   ... (주석 처리)

  highlog-backend:
    # ...
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://host.docker.internal:5432/highlog
      REDIS_HOST: host.docker.internal
    # depends_on 제거
```

## 🚀 프로덕션 배포 시

1. **보안**: 모든 비밀번호/키 변경 필수
2. **네트워크**: 내부 네트워크만 사용하도록 설정
3. **리소스 제한**: CPU/메모리 제한 설정
4. **데이터 백업**: 볼륨 백업 전략 수립
5. **로그 설정**: JSON 로그 드라이버 사용

## 📝 주요 파일

- `.env` - 환경변수 설정 (git 제외)
- `.env.example` - 환경변수 예시
- `docker-compose.yml` - Docker Compose 설정
- `db-init/init.sql` - DB 초기화 스크립트
- `highLog/Dockerfile` - Spring Boot 빌드
- `ai-service/Dockerfile` - FastAPI 빌드
