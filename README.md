# EATi Infrastructure

OCI 서버에서 PostgreSQL, Redis, MongoDB를 Docker Compose로 관리하는 인프라 레포지토리입니다.

## 아키텍처

GitHub Actions에서 Docker 이미지를 빌드하여 GitHub Container Registry(GHCR)에 푸시하고, OCI 서버에서는 빌드된 이미지를 pull하여 실행하는 구조입니다.

```
[GitHub Repository]
       ↓ (push to main)
[GitHub Actions: Build Images]
       ↓
[GitHub Container Registry]
       ↓ (pull images)
[OCI Server: Deploy & Run]
```

## 디렉토리 구조

```
EATi_Infra/
├── .github/
│   └── workflows/
│       ├── build-images.yml    # Docker 이미지 빌드 워크플로우
│       └── deploy.yml          # OCI 서버 배포 워크플로우
├── postgresql/
│   ├── Dockerfile              # PostgreSQL 16 커스텀 이미지
│   └── init/                   # PostgreSQL 초기화 스크립트
├── redis/
│   ├── Dockerfile              # Redis 7.2 커스텀 이미지
│   └── redis.conf              # Redis 설정 파일
├── mongodb/
│   ├── Dockerfile              # MongoDB 7.0 커스텀 이미지
│   ├── mongod.conf             # MongoDB 설정 파일
│   └── init/                   # MongoDB 초기화 스크립트
├── docker-compose.yml          # Docker Compose 설정
├── .env.example                # 환경변수 템플릿
├── .gitignore
└── README.md
```

## 사용 기술

- PostgreSQL 16 (Alpine 기반)
- Redis 7.2 (Alpine 기반)
- MongoDB 7.0
- Docker & Docker Compose
- GitHub Actions & GitHub Container Registry

## 빠른 시작

### 1. GitHub Container Registry 패키지 공개 설정

이미지가 빌드되면 GitHub Container Registry의 패키지 설정에서 public으로 변경하거나, OCI 서버에서 인증을 설정해야 합니다.

### 2. 환경변수 설정

```bash
cp .env.example .env
```

`.env` 파일을 열어서 실제 비밀번호와 설정값을 입력하세요.

### 3. 로컬 테스트 (선택사항)

로컬에서 이미지를 빌드하고 테스트할 수 있습니다:

```bash
# PostgreSQL 이미지 빌드
docker build -t eati-postgresql:local ./postgresql

# Redis 이미지 빌드
docker build -t eati-redis:local ./redis

# MongoDB 이미지 빌드
docker build -t eati-mongodb:local ./mongodb

# 로컬 이미지로 실행 (docker-compose.yml 수정 필요)
docker compose up -d
```

## OCI 서버 설정

### 1. OCI 서버 준비

OCI 서버에 Docker와 Docker Compose를 설치하세요:

```bash
# Docker 설치
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Docker Compose 설치
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# 사용자를 docker 그룹에 추가
sudo usermod -aG docker $USER
```

### 2. 환경변수 설정

OCI 서버의 `~/eati-infra/.env` 파일을 생성하고 환경변수를 설정하세요:

```bash
mkdir -p ~/eati-infra
nano ~/eati-infra/.env
```

### 3. 방화벽 규칙 설정

OCI 콘솔에서 다음 포트를 열어야 합니다:

- PostgreSQL: 5432
- Redis: 6379
- MongoDB: 27017

## GitHub Actions 설정

### 1. GitHub Secrets 설정

GitHub 레포지토리 Settings > Secrets and variables > Actions에서 다음 secrets를 추가하세요:

- `OCI_SSH_PRIVATE_KEY`: OCI 서버 접속용 SSH 개인키
- `OCI_SERVER_IP`: OCI 서버 IP 주소
- `OCI_SERVER_USER`: OCI 서버 SSH 사용자명 (예: ubuntu, opc)

### 2. SSH 키 생성 및 설정

```bash
# 로컬에서 SSH 키 생성
ssh-keygen -t ed25519 -C "github-actions" -f ~/.ssh/oci_deploy

# OCI 서버에 공개키 복사
ssh-copy-id -i ~/.ssh/oci_deploy.pub user@your-oci-server-ip

# 개인키 내용을 복사하여 GitHub Secrets의 OCI_SSH_PRIVATE_KEY에 추가
cat ~/.ssh/oci_deploy
```

### 3. 이미지 공개 설정

첫 빌드 후 GitHub Container Registry의 패키지를 public으로 설정하세요:

1. GitHub 프로필 > Packages
2. 각 패키지(eati-postgresql, eati-redis, eati-mongodb) 선택
3. Package settings > Change visibility > Public

또는 private으로 유지하고 OCI 서버에서 GHCR 로그인:

```bash
# OCI 서버에서 실행
echo YOUR_GITHUB_PAT | docker login ghcr.io -u YOUR_USERNAME --password-stdin
```

### 4. 자동 빌드 및 배포 프로세스

Dockerfile 또는 설정 파일을 수정하고 `main` 브랜치에 푸시하면:

1. **build-images.yml**: Docker 이미지를 빌드하여 GHCR에 푸시
2. **deploy.yml**: OCI 서버에서 이미지를 pull하고 서비스를 재시작

```bash
git add .
git commit -m "Update PostgreSQL configuration"
git push origin main
```

### 5. 수동 배포

GitHub Actions 탭에서 `Deploy to OCI Server` 워크플로우를 수동으로 실행할 수 있습니다.

## 서비스 접속 정보

### PostgreSQL

```bash
# 로컬 접속
psql -h localhost -p 5432 -U postgres -d eati

# Docker 컨테이너 접속
docker exec -it eati-postgresql psql -U postgres -d eati
```

### Redis

```bash
# 로컬 접속
redis-cli -h localhost -p 6379 -a your_password

# Docker 컨테이너 접속
docker exec -it eati-redis redis-cli
```

### MongoDB

```bash
# 로컬 접속
mongosh "mongodb://admin:your_password@localhost:27017"

# Docker 컨테이너 접속
docker exec -it eati-mongodb mongosh -u admin -p your_password
```

## 빌드된 Docker 이미지

GitHub Container Registry에 다음 이미지들이 자동으로 빌드됩니다:

- `ghcr.io/[username]/eati-postgresql:latest` (PostgreSQL 16 + 한국 시간대)
- `ghcr.io/[username]/eati-redis:latest` (Redis 7.2 + 최적화된 설정)
- `ghcr.io/[username]/eati-mongodb:latest` (MongoDB 7.0 + 한국 시간대)

각 이미지는 다음과 같은 태그가 생성됩니다:
- `latest`: 가장 최신 빌드
- `16` / `7.2` / `7.0`: 버전 태그
- `main-{SHA}`: 커밋 해시별 태그

## 초기화 스크립트

### PostgreSQL 초기화

`postgresql/init/` 디렉토리에 `.sql` 또는 `.sh` 파일을 추가하고 이미지를 다시 빌드하세요.

예시 (`postgresql/init/01-init.sql`):

```sql
CREATE TABLE IF NOT EXISTS users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### MongoDB 초기화

`mongodb/init/` 디렉토리에 `.js` 파일을 추가하고 이미지를 다시 빌드하세요.

예시 (`mongodb/init/01-init.js`):

```javascript
db = db.getSiblingDB('eati');

db.createCollection('users');

db.users.createIndex({ username: 1 }, { unique: true });
```

**중요**: 초기화 스크립트를 추가하거나 수정한 후에는 반드시 커밋하고 푸시하여 이미지를 다시 빌드해야 합니다.

## 백업 및 복구

### PostgreSQL 백업

```bash
docker exec eati-postgresql pg_dump -U postgres eati > backup_$(date +%Y%m%d).sql
```

### Redis 백업

```bash
docker exec eati-redis redis-cli --rdb /data/backup.rdb SAVE
docker cp eati-redis:/data/backup.rdb ./backup_$(date +%Y%m%d).rdb
```

### MongoDB 백업

```bash
docker exec eati-mongodb mongodump --username admin --password your_password --authenticationDatabase admin --out /backup
docker cp eati-mongodb:/backup ./mongodb_backup_$(date +%Y%m%d)
```

## 모니터링

### 서비스 상태 확인

```bash
# 모든 서비스 상태
docker compose ps

# 특정 서비스 로그
docker compose logs postgresql
docker compose logs redis
docker compose logs mongodb

# 실시간 로그
docker compose logs -f
```

### 리소스 사용량 확인

```bash
docker stats
```

## 트러블슈팅

### 컨테이너가 시작되지 않는 경우

```bash
# 로그 확인
docker compose logs

# 컨테이너 재시작
docker compose restart

# 완전히 재배포
docker compose down
docker compose up -d
```

### 포트 충돌

`.env` 파일에서 포트 번호를 변경하세요:

```bash
POSTGRES_PORT=5433
REDIS_PORT=6380
MONGO_PORT=27018
```

### 볼륨 권한 문제

```bash
# 볼륨 재생성
docker compose down -v
docker compose up -d
```

## 커스텀 이미지 특징

### PostgreSQL 16
- Alpine Linux 기반 경량 이미지
- 한국 시간대(Asia/Seoul) 설정
- UTF-8 인코딩 기본 설정
- 헬스체크 포함

### Redis 7.2
- Alpine Linux 기반 경량 이미지
- 한국 시간대(Asia/Seoul) 설정
- AOF(Append Only File) 영속성 설정
- 메모리 관리 정책(LRU) 적용
- 슬로우 로그 설정

### MongoDB 7.0
- WiredTiger 스토리지 엔진
- 한국 시간대(Asia/Seoul) 설정
- 인증 활성화
- 슬로우 쿼리 프로파일링 설정

## 보안 고려사항

1. `.env` 파일은 절대 커밋하지 마세요
2. 강력한 비밀번호를 사용하세요
3. GHCR 이미지를 public으로 설정하거나, private 유지 시 OCI 서버에서 인증 설정
4. 프로덕션 환경에서는 방화벽 규칙을 엄격하게 설정하세요
5. 정기적으로 백업을 수행하세요
6. SSH 키를 안전하게 관리하세요

## 트러블슈팅

### GHCR 이미지 pull 실패

OCI 서버에서 GHCR에 로그인이 필요한 경우:

```bash
echo YOUR_GITHUB_PAT | docker login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin
```

### 이미지 빌드 실패

GitHub Actions 탭에서 빌드 로그를 확인하세요. 플랫폼 호환성 문제가 있다면 `.github/workflows/build-images.yml`의 `platforms` 설정을 조정하세요.

### 배포 후 서비스가 시작되지 않음

```bash
# OCI 서버에서 로그 확인
cd ~/eati-infra
docker compose logs

# 환경변수 확인
cat .env
```

## 라이선스

Private
