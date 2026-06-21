# 환경 변수 설정 가이드

이 프로젝트(Next.js 16 + Prisma v7 + NextAuth v5)에 필요한 환경 변수를 단계별로 안내합니다.

## 1. .env 파일 확인

프로젝트 루트의 `.env` 파일을 읽고 현재 설정 상태를 확인하세요.

현재 필요한 환경 변수 목록:

| 변수명 | 설명 | 예시 |
|--------|------|------|
| `DATABASE_URL` | Prisma DB 연결 문자열 | `file:./dev.db` (SQLite) |
| `AUTH_SECRET` | NextAuth 세션 암호화 키 (필수) | 32자 이상 랜덤 문자열 |
| `AUTH_URL` | 앱 배포 URL | `http://localhost:3000` |

## 2. AUTH_SECRET 생성

아직 설정하지 않았다면 아래 명령으로 안전한 시크릿을 생성하세요:

```bash
# Node.js로 생성
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"

# 또는 npx로 생성 (NextAuth v5 공식 방법)
npx auth secret
```

## 3. 데이터베이스 설정

SQLite(개발용) 기본 설정:
```env
DATABASE_URL="file:./dev.db"
```

PostgreSQL(프로덕션)으로 전환 시 `prisma/schema.prisma`의 `provider`도 함께 변경해야 합니다:
```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

## 4. .env 파일이 없다면

`.env.example`을 복사해서 시작하세요:

```bash
cp .env.example .env
```

## 5. 체크리스트

현재 `.env` 파일을 읽어서 아래 항목을 확인해주세요:

- [ ] `DATABASE_URL` 설정됨
- [ ] `AUTH_SECRET` 설정됨 (기본값 `your-secret-here` 그대로면 반드시 변경)
- [ ] `AUTH_URL` 실제 URL로 설정됨
- [ ] `.env`가 `.gitignore`에 포함됨 (민감 정보 노출 방지)

`.gitignore`에서 `.env` 제외 여부를 확인하고, 누락된 변수가 있으면 추가 방법을 안내해주세요.
