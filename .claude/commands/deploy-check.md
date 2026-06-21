# 배포 전 체크리스트

이 프로젝트를 배포하기 전에 아래 항목들을 순서대로 점검합니다.

## 점검 순서

### 1. 환경 변수 확인

`.env` 또는 배포 플랫폼의 환경 변수에서 아래를 확인하세요:

- `AUTH_SECRET`이 기본값(`your-secret-here`)이 아닌 안전한 랜덤 값인지
- `AUTH_URL`이 실제 배포 도메인으로 설정됐는지 (`http://localhost:3000`이 아닌지)
- `DATABASE_URL`이 프로덕션 DB(PostgreSQL 등)를 가리키는지

### 2. 빌드 성공 여부

```bash
npm run build
```

빌드 에러가 없는지 확인하세요. TypeScript 타입 오류와 ESLint 경고도 점검합니다.

```bash
npx eslint .
npx tsc --noEmit
```

### 3. Prisma 마이그레이션 상태

프로덕션 DB에 마이그레이션이 모두 적용됐는지 확인:

```bash
npx prisma migrate status
```

배포 시 자동 마이그레이션이 필요하면:

```bash
npx prisma migrate deploy
```

### 4. Prisma Client 생성

```bash
npx prisma generate
```

빌드 전에 반드시 실행해야 `src/generated/prisma`가 최신 상태를 유지합니다.

### 5. 보안 점검

- `prisma/schema.prisma`에 민감한 데이터가 하드코딩되지 않았는지 확인
- `.env` 파일이 git에 커밋되지 않았는지 확인:

```bash
git status --short | grep ".env$"
```

결과가 나오면 즉시 `.gitignore`에 추가하고 git history에서 제거해야 합니다.

### 6. Next.js 프로덕션 최적화 확인

`next.config.js` (또는 `next.config.ts`)에서:

- 불필요한 `console.log`가 남아있지 않은지
- `output: 'standalone'` 설정 여부 (컨테이너 배포 시 권장)

### 7. 최종 점검 요약

현재 프로젝트 파일들을 읽고 아래 항목의 실제 상태를 확인해주세요:

- [ ] `npm run build` 성공
- [ ] `AUTH_SECRET` 안전한 값으로 설정
- [ ] `AUTH_URL` 프로덕션 URL로 설정
- [ ] `DATABASE_URL` 프로덕션 DB로 설정
- [ ] Prisma 마이그레이션 최신 상태
- [ ] `.env`가 `.gitignore`에 포함
- [ ] TypeScript/ESLint 오류 없음

누락된 항목이 있으면 해결 방법을 구체적으로 안내해주세요.
