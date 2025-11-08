# Quickstart: 소셜 피드 개발 환경 설정

**Feature**: 001-social-feed
**Date**: 2025-11-09
**Target**: 개발자 로컬 환경 설정 및 실행 (Docker 기반)

## Prerequisites

다음 도구들이 설치되어 있어야 합니다:

- **Node.js**: v20.19.5 (v20 LTS) ⭐ **필수 버전**
- **pnpm**: v8.0.0 이상 (또는 npm, yarn)
- **Docker**: v20.10 이상
- **Docker Compose**: v2.0 이상
- **Git**: 버전 관리
- **Firebase 계정**: Firebase Storage 및 FCM 설정용

### 설치 확인

```bash
node --version        # v20.19.5
pnpm --version        # v8.0.0+
docker --version      # v20.10+
docker-compose --version  # v2.0+
git --version         # any version
```

### Docker 설치

**macOS**:
```bash
# Homebrew 사용
brew install --cask docker

# 또는 Docker Desktop 다운로드
# https://www.docker.com/products/docker-desktop
```

**Linux**:
```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 사용자를 docker 그룹에 추가 (sudo 없이 docker 실행)
sudo usermod -aG docker $USER
```

---

## 1. 프로젝트 클론 및 브랜치 체크아웃

```bash
# 저장소 클론
git clone https://github.com/your-org/feed-app.git
cd feed-app

# Feature 브랜치로 체크아웃
git checkout 001-social-feed
```

---

## 2. Docker 환경 설정

### 2.1. docker-compose.yml 생성

프로젝트 루트에 `docker-compose.yml` 파일을 생성합니다:

```yaml
version: '3.9'

services:
  # PostgreSQL Database
  db:
    image: postgres:15-alpine
    container_name: crossfeed-db
    restart: unless-stopped
    environment:
      POSTGRES_USER: crossfeed
      POSTGRES_PASSWORD: dev_password_change_in_production
      POSTGRES_DB: crossfeed_dev
    ports:
      - "5432:5432"
    volumes:
      # 데이터 지속성: 호스트의 ./docker-volumes/postgres-data에 저장
      - ./docker-volumes/postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U crossfeed"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Next.js Application
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    container_name: crossfeed-app
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://crossfeed:dev_password_change_in_production@db:5432/crossfeed_dev
      - NEXTAUTH_URL=http://localhost:3000
      - NEXTAUTH_SECRET=${NEXTAUTH_SECRET}
      # Firebase env vars (from .env.local)
      - NEXT_PUBLIC_FIREBASE_API_KEY=${NEXT_PUBLIC_FIREBASE_API_KEY}
      - NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=${NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN}
      - NEXT_PUBLIC_FIREBASE_PROJECT_ID=${NEXT_PUBLIC_FIREBASE_PROJECT_ID}
      - NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=${NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET}
      - NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=${NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID}
      - NEXT_PUBLIC_FIREBASE_APP_ID=${NEXT_PUBLIC_FIREBASE_APP_ID}
      - FIREBASE_ADMIN_PROJECT_ID=${FIREBASE_ADMIN_PROJECT_ID}
      - FIREBASE_ADMIN_CLIENT_EMAIL=${FIREBASE_ADMIN_CLIENT_EMAIL}
      - FIREBASE_ADMIN_PRIVATE_KEY=${FIREBASE_ADMIN_PRIVATE_KEY}
    volumes:
      # 소스 코드 hot reload
      - .:/app
      - /app/node_modules
      - /app/.next
    depends_on:
      db:
        condition: service_healthy
    command: pnpm dev

volumes:
  postgres-data:
```

### 2.2. Dockerfile.dev 생성

프로젝트 루트에 `Dockerfile.dev` 파일을 생성합니다:

```dockerfile
FROM node:20.19.5-alpine

# pnpm 설치
RUN npm install -g pnpm@latest

# 작업 디렉터리 설정
WORKDIR /app

# package.json 및 pnpm-lock.yaml 복사
COPY package.json pnpm-lock.yaml* ./

# 의존성 설치
RUN pnpm install --frozen-lockfile

# 소스 코드 복사 (volume mount로 덮어씌워짐)
COPY . .

# Prisma Client 생성
RUN pnpm prisma generate

# 포트 노출
EXPOSE 3000

# 개발 서버 실행 (docker-compose.yml에서 override)
CMD ["pnpm", "dev"]
```

### 2.3. .dockerignore 생성

불필요한 파일을 Docker 이미지에서 제외합니다:

```
node_modules
.next
.git
.env.local
.env*.local
docker-volumes
*.log
.DS_Store
```

---

## 3. 환경 변수 설정

### 3.1. `.env.local` 파일 생성

프로젝트 루트에 `.env.local` 파일을 생성합니다:

```bash
# Database (Docker PostgreSQL)
DATABASE_URL="postgresql://crossfeed:dev_password_change_in_production@localhost:5432/crossfeed_dev"

# NextAuth.js
NEXTAUTH_URL="http://localhost:3000"
NEXTAUTH_SECRET="your-super-secret-key-change-this-in-production"

# Firebase Client (공개 키 - 프론트엔드에서 사용)
NEXT_PUBLIC_FIREBASE_API_KEY="your-firebase-api-key"
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN="your-app.firebaseapp.com"
NEXT_PUBLIC_FIREBASE_PROJECT_ID="your-project-id"
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET="your-app.appspot.com"
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID="123456789"
NEXT_PUBLIC_FIREBASE_APP_ID="1:123456789:web:abcdef"

# Firebase Admin SDK (서버 전용 - 비공개 키)
FIREBASE_ADMIN_PROJECT_ID="your-project-id"
FIREBASE_ADMIN_CLIENT_EMAIL="firebase-adminsdk@your-project.iam.gserviceaccount.com"
FIREBASE_ADMIN_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n"
```

### 3.2. 환경 변수 값 얻기

**NEXTAUTH_SECRET 생성**:
```bash
openssl rand -base64 32
```

**Firebase 설정**:
1. [Firebase Console](https://console.firebase.google.com/) 접속
2. 프로젝트 생성
3. 프로젝트 설정 → General → Your apps → 웹 앱 추가
4. SDK setup and configuration → Config 값 복사 → `NEXT_PUBLIC_FIREBASE_*`
5. 프로젝트 설정 → Service accounts → Generate new private key
6. JSON 파일 다운로드 → 값 추출 → `FIREBASE_ADMIN_*`

---

## 4. Docker 컨테이너 시작

### 4.1. Docker Compose 실행

```bash
# 백그라운드에서 모든 서비스 시작
docker-compose up -d

# 로그 확인
docker-compose logs -f

# 특정 서비스 로그만 확인
docker-compose logs -f app
docker-compose logs -f db
```

### 4.2. 컨테이너 상태 확인

```bash
# 실행 중인 컨테이너 확인
docker-compose ps

# 예상 출력:
# NAME                COMMAND                  SERVICE   STATUS    PORTS
# crossfeed-app       "docker-entrypoint.s…"   app       Up        0.0.0.0:3000->3000/tcp
# crossfeed-db        "docker-entrypoint.s…"   db        Up        0.0.0.0:5432->5432/tcp
```

---

## 5. 데이터베이스 설정 (Docker 컨테이너 내)

### 5.1. Prisma 마이그레이션 실행

**옵션 1: 로컬에서 실행** (호스트 → Docker DB 연결):
```bash
# 의존성 설치 (로컬에서)
pnpm install

# Prisma Client 생성
pnpm prisma generate

# 마이그레이션 실행
pnpm prisma migrate dev --name init
```

**옵션 2: Docker 컨테이너 내에서 실행**:
```bash
# app 컨테이너에 접속
docker-compose exec app sh

# 컨테이너 내에서 실행
pnpm prisma migrate dev --name init
pnpm prisma generate

# 컨테이너 종료
exit
```

### 5.2. Seed 데이터 생성 (선택)

```bash
# 로컬 또는 컨테이너 내에서 실행
pnpm prisma db seed
```

**`prisma/seed.ts` 예제** (프로젝트 루트에 생성):

```typescript
import { PrismaClient } from '@prisma/client';
import bcrypt from 'bcrypt';

const prisma = new PrismaClient();

async function main() {
  const passwordHash = await bcrypt.hash('password123', 10);

  const user1 = await prisma.user.create({
    data: {
      email: 'alice@example.com',
      passwordHash,
      username: 'alice',
      displayName: 'Alice Kim',
      bio: '사진을 좋아합니다 📸',
    },
  });

  const user2 = await prisma.user.create({
    data: {
      email: 'bob@example.com',
      passwordHash,
      username: 'bob',
      displayName: 'Bob Lee',
    },
  });

  const post = await prisma.post.create({
    data: {
      authorId: user1.id,
      imageUrl: 'https://via.placeholder.com/600',
      caption: '테스트 포스트입니다',
    },
  });

  console.log('✅ Seed data created!');
  console.log({ user1, user2, post });
}

main()
  .catch((e) => {
    console.error('❌ Seed failed:', e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

**`package.json`에 seed 스크립트 추가**:
```json
{
  "prisma": {
    "seed": "tsx prisma/seed.ts"
  },
  "devDependencies": {
    "tsx": "^4.0.0"
  }
}
```

---

## 6. 애플리케이션 접속

### 브라우저에서 접속

- **Frontend**: [http://localhost:3000](http://localhost:3000)
- **API**: [http://localhost:3000/api](http://localhost:3000/api)

### 데이터베이스 접속

**Prisma Studio** (GUI):
```bash
# 로컬에서 실행
pnpm prisma studio

# 또는 Docker 컨테이너 내에서
docker-compose exec app pnpm prisma studio
```

**psql (CLI)**:
```bash
# 호스트에서 Docker DB 접속
psql postgresql://crossfeed:dev_password_change_in_production@localhost:5432/crossfeed_dev

# 또는 Docker exec로 접속
docker-compose exec db psql -U crossfeed -d crossfeed_dev
```

---

## 7. Firebase 설정

### 7.1. Firebase Storage Rules

Firebase Console → Storage → Rules:

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /posts/{imageId} {
      allow create: if request.auth != null
                    && request.resource.size < 10 * 1024 * 1024
                    && request.resource.contentType.matches('image/(jpeg|png|webp)');
      allow read: if true;
      allow delete: if request.auth != null;
    }
  }
}
```

### 7.2. Firebase Cloud Messaging (FCM)

**`public/firebase-messaging-sw.js` 생성**:

```javascript
importScripts('https://www.gstatic.com/firebasejs/10.7.0/firebase-app-compat.js');
importScripts('https://www.gstatic.com/firebasejs/10.7.0/firebase-messaging-compat.js');

firebase.initializeApp({
  apiKey: "your-api-key",
  projectId: "your-project-id",
  messagingSenderId: "your-sender-id",
  appId: "your-app-id"
});

const messaging = firebase.messaging();

messaging.onBackgroundMessage((payload) => {
  console.log('Background message:', payload);

  self.registration.showNotification(
    payload.notification.title,
    {
      body: payload.notification.body,
      icon: '/icon.png',
      data: payload.data
    }
  );
});

self.addEventListener('notificationclick', (event) => {
  event.notification.close();
  const url = event.notification.data?.click_action || '/';
  event.waitUntil(clients.openWindow(url));
});
```

---

## 8. 개발 워크플로우

### 일반적인 명령어

```bash
# 서비스 시작
docker-compose up -d

# 서비스 중지
docker-compose down

# 서비스 재시작
docker-compose restart

# 로그 확인 (실시간)
docker-compose logs -f

# app 컨테이너에 접속
docker-compose exec app sh

# 데이터베이스 컨테이너 접속
docker-compose exec db psql -U crossfeed -d crossfeed_dev

# 컨테이너 및 볼륨 모두 삭제 (⚠️ 데이터 손실)
docker-compose down -v
```

### 의존성 추가

```bash
# 로컬에서 패키지 추가
pnpm add <package-name>

# Docker 이미지 재빌드
docker-compose build app

# 서비스 재시작
docker-compose up -d app
```

### 데이터베이스 스키마 변경

```bash
# 1. schema.prisma 수정

# 2. 마이그레이션 생성 및 적용
pnpm prisma migrate dev --name add_new_field

# 3. Prisma Client 재생성
pnpm prisma generate

# 4. Docker 컨테이너 재시작 (필요 시)
docker-compose restart app
```

### 코드 변경 시 Hot Reload

Docker Compose volume mount 덕분에 코드 변경 시 자동으로 반영됩니다 (hot reload):
- `src/` 디렉터리 수정 → 자동 재로드
- `package.json` 변경 → 컨테이너 재빌드 필요
- `prisma/schema.prisma` 변경 → 마이그레이션 + 재시작 필요

---

## 9. 테스트 실행

```bash
# 로컬에서 테스트 실행
pnpm test

# Watch 모드
pnpm test:watch

# 커버리지
pnpm test:coverage

# Docker 컨테이너 내에서
docker-compose exec app pnpm test
```

---

## 10. 코드 품질 검사

```bash
# ESLint
pnpm lint

# Prettier
pnpm format

# TypeScript 타입 체크
pnpm type-check

# 모든 검사 실행
pnpm lint && pnpm type-check
```

---

## 11. 데이터 지속성 및 백업

### 11.1. 데이터 위치

PostgreSQL 데이터는 호스트의 `./docker-volumes/postgres-data`에 저장됩니다:

```bash
# 데이터 디렉터리 확인
ls -la docker-volumes/postgres-data

# 디스크 사용량 확인
du -sh docker-volumes/postgres-data
```

### 11.2. 데이터 백업

```bash
# 데이터베이스 백업
docker-compose exec db pg_dump -U crossfeed crossfeed_dev > backup-$(date +%Y%m%d-%H%M%S).sql

# 데이터 복원
docker-compose exec -T db psql -U crossfeed -d crossfeed_dev < backup-20250109-120000.sql
```

### 11.3. 데이터 초기화

```bash
# ⚠️ 주의: 모든 데이터 삭제
docker-compose down -v

# 새로 시작
docker-compose up -d

# 마이그레이션 재실행
pnpm prisma migrate dev

# Seed 데이터 재생성
pnpm prisma db seed
```

---

## 12. 문제 해결 (Troubleshooting)

### Docker 컨테이너가 시작되지 않음

```bash
# 로그 확인
docker-compose logs db
docker-compose logs app

# 컨테이너 재빌드
docker-compose build --no-cache

# 모든 컨테이너 및 볼륨 삭제 후 재시작
docker-compose down -v
docker-compose up -d
```

### 포트 충돌 (3000번 또는 5432번 사용 중)

```bash
# 사용 중인 포트 확인
lsof -i :3000
lsof -i :5432

# docker-compose.yml에서 포트 변경
# ports:
#   - "3001:3000"  # 호스트:컨테이너
```

### 데이터베이스 연결 실패

```bash
# DB 헬스체크 확인
docker-compose ps

# DB 로그 확인
docker-compose logs db

# 수동으로 DB 접속 테스트
docker-compose exec db psql -U crossfeed -d crossfeed_dev -c "SELECT version();"
```

### Prisma Client 생성 오류

```bash
# Docker 컨테이너 내에서 재생성
docker-compose exec app pnpm prisma generate

# 또는 로컬에서
pnpm prisma generate

# node_modules 삭제 후 재설치
docker-compose down
rm -rf node_modules
pnpm install
docker-compose up -d --build
```

### Hot Reload가 작동하지 않음

```bash
# 1. volume mount 확인 (docker-compose.yml)
# volumes:
#   - .:/app

# 2. Next.js 개발 서버 재시작
docker-compose restart app

# 3. 컨테이너 재빌드
docker-compose up -d --build app
```

---

## 13. 프로덕션 배포 (Vercel)

### 로컬에서 프로덕션 빌드 테스트

```bash
# Docker 컨테이너 내에서
docker-compose exec app pnpm build
docker-compose exec app pnpm start

# 또는 로컬에서
pnpm build
pnpm start
```

### Vercel 배포

Docker는 개발 환경 전용입니다. 프로덕션 배포는 Vercel을 사용합니다:

1. **Vercel 프로젝트 연결**:
   ```bash
   pnpm add -g vercel
   vercel login
   vercel link
   ```

2. **환경 변수 설정**: Vercel 대시보드에서 설정

3. **배포**:
   ```bash
   # Preview
   vercel

   # Production
   vercel --prod
   ```

4. **GitHub Actions 자동 배포**: `main` 브랜치 병합 시 자동

---

## 14. 프로젝트 구조

```text
feed-app/
├── src/                     # 소스 코드
│   ├── app/                # Next.js App Router
│   ├── entities/           # FSD: 도메인 엔티티
│   ├── features/           # FSD: 사용자 기능
│   ├── widgets/            # FSD: 페이지 조합
│   └── shared/             # FSD: 공통 코드
├── prisma/
│   ├── schema.prisma       # DB 스키마
│   ├── migrations/         # 마이그레이션
│   └── seed.ts             # Seed 데이터
├── public/
│   └── firebase-messaging-sw.js
├── docker-volumes/         # Docker 데이터 (gitignore)
│   └── postgres-data/
├── docker-compose.yml      # Docker Compose 설정
├── Dockerfile.dev          # 개발용 Dockerfile
├── .dockerignore
├── .env.local              # 환경 변수 (gitignore)
├── next.config.js
├── tsconfig.json
├── package.json
└── README.md
```

---

## 15. 다음 단계

✅ 개발 환경 설정 완료 후:

1. **User Story 1 (P1) 구현**: 포스트 생성 기능
2. **API 테스트**: Postman/Thunder Client로 엔드포인트 테스트
3. **UI 구현**: FSD 구조에 맞게 컴포넌트 작성
4. **테스트 작성**: Jest + React Testing Library

**참고 문서**:
- [spec.md](./spec.md) - 기능 명세
- [plan.md](./plan.md) - 구현 계획
- [data-model.md](./data-model.md) - 데이터 모델
- [contracts/](./contracts/) - API 계약

**질문이나 문제가 있으면 팀에 문의하세요!** 🚀
