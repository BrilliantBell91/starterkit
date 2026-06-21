# 성능 최적화 제안

이 프로젝트(Next.js 16 + Prisma v7 + NextAuth v5 + shadcn/ui)의 성능 최적화 포인트를 분석하고 제안합니다.

## 분석 대상

아래 파일들을 읽어서 현재 코드 상태를 파악한 뒤 구체적인 개선 사항을 제안해주세요:

- `src/app/` 하위의 페이지 컴포넌트들
- `src/lib/` 또는 `src/utils/` 하위의 유틸리티
- `prisma/schema.prisma`
- `next.config.*`

## 점검 영역

### 1. Next.js 렌더링 전략

- 불필요하게 `"use client"`가 붙은 컴포넌트가 있는지 확인 (서버 컴포넌트로 전환 가능한 경우)
- 데이터 페칭이 `fetch`의 캐싱 옵션을 활용하는지 확인
  - `fetch(url, { cache: 'force-cache' })` — 정적 데이터
  - `fetch(url, { next: { revalidate: 60 } })` — ISR 방식
- 동적 라우트에서 `generateStaticParams` 활용 가능한지 확인

### 2. Prisma 쿼리 최적화

- N+1 문제: `include` 또는 `select`로 관련 데이터를 한 번에 가져오는지 확인
- 필요한 필드만 `select`로 제한하는지 확인 (전체 row 조회 대신)
- 자주 조회하는 필드에 인덱스(`@@index`)가 있는지 `schema.prisma` 확인

예시 개선:
```typescript
// 비효율
const posts = await prisma.post.findMany()
// 개선
const posts = await prisma.post.findMany({
  select: { id: true, title: true, createdAt: true },
  where: { published: true },
  orderBy: { createdAt: 'desc' },
  take: 20,
})
```

### 3. 이미지 최적화

- `<img>` 태그 대신 Next.js `<Image>` 컴포넌트를 사용하는지 확인
- `priority` 속성이 LCP(최대 콘텐츠 페인트) 이미지에 설정됐는지 확인

### 4. 번들 크기

- shadcn/ui 컴포넌트를 필요한 것만 import하는지 확인 (트리 쉐이킹)
- 무거운 라이브러리가 있다면 dynamic import 적용 가능한지 확인:

```typescript
const HeavyComponent = dynamic(() => import('@/components/HeavyComponent'), {
  loading: () => <p>로딩 중...</p>,
})
```

### 5. NextAuth 세션 최적화

- 세션 조회가 매 요청마다 DB를 히트하는지, JWT 전략이 더 적합한지 확인
- `auth()` 호출이 중복되는 레이아웃 구조가 있는지 확인

## 출력 형식

현재 코드를 분석한 뒤 아래 형식으로 답해주세요:

1. **발견된 문제** — 파일 경로와 구체적인 코드 위치 포함
2. **개선 방법** — 실제 수정 코드 예시 포함
3. **예상 효과** — 수정 시 기대되는 성능 향상 설명

쉽게 적용할 수 있는 항목부터 우선순위 순으로 제안해주세요.
