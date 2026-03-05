#  HairFlow 프로젝트 정리
---

## 목차

- [[#1. 프로젝트 개요]]
- [[#2. 기술 스택]]
- [[#3. 프로젝트 구조]]
- [[#4. 데이터베이스 스키마]]
- [[#5. 핵심 아키텍처 패턴]]
- [[#6. AI 이미지 생성 파이프라인]]
- [[#7. 유용한 프롬프트 모음]]
- [[#8. 트러블슈팅 & 교훈]]
- [[#9. 바이브코딩 팁]]

---

## 1. 프로젝트 개요


### 주요 기능

| 기능        | 설명                                       | 사용 AI                          |
| --------- | ---------------------------------------- | ------------------------------ |
| 5면 사진 분석  | 고객 얼굴형/두상/모발 상태 진단                       | GPT-4o-mini Vision             |
| AI 스타일 추천 | 얼굴형에 맞는 헤어스타일 추천 + 이미지 생성                | GPT-4o-mini + Flux Kontext Pro |
| 시술 레시피    | 현재→목표 모발 비교 분석 후 단계별 레시피                 | GPT-4o Vision                  |
| 구독/결제     | Free/Basic/Enterprise 플랜 + Toss Payments | -                              |
| 매장 관리     | Organization + 멤버 초대코드 시스템               | -                              |

---

## 2. 기술 스택

### 프론트엔드
```
Next.js 16 (App Router) + React 19
TailwindCSS v4 + shadcn/ui
Framer Motion (애니메이션)
Lucide React (아이콘)
Sonner (토스트 알림)
TypeScript 5
```

### 백엔드/인프라
```
Supabase (Auth, PostgreSQL, Storage, RLS)
Vercel (배포, Cron Jobs)
Toss Payments SDK (결제)
```


### AI/ML
```
OpenAI GPT-4o-mini / GPT-4o (Vision 분석)
Fal.ai Flux Kontext Pro (이미지 편집 — 헤어스타일 변경)
Fal.ai Flux Pro v1.1 (Text-to-Image)
Fal.ai IP-Adapter Face ID (얼굴 합성, 폴백용)
```


---

## 3. 프로젝트 구조
```
hairflow/
├── src/
│   ├── app/
│   │   ├── (auth)/          # 로그인 페이지 (그룹 라우트)
│   │   ├── (main)/          # 인증 필요 페이지들 (layout.tsx에서 Navbar 포함)
│   │   │   ├── customers/   # 고객 관리 (목록 + 상세)
│   │   │   ├── dashboard/   # 대시보드
│   │   │   ├── pricing/     # 요금제
│   │   │   ├── recipe/      # 시술 레시피
│   │   │   ├── salon/       # 매장 관리
│   │   │   ├── settings/    # 설정
│   │   │   └── timeline/    # 타임라인
│   │   ├── api/             # API Routes (12개)
│   │   │   ├── auth/        # Supabase Auth 콜백
│   │   │   ├── customers/   # 고객 CRUD + AI 분석 엔드포인트
│   │   │   ├── payment/     # Toss Payments 결제 확인
│   │   │   ├── cron/        # 자동 결제 갱신 (Vercel Cron)
│   │   │   └── ...
│   │   ├── onboarding/      # 신규 사용자 온보딩
│   │   ├── portfolio/       # 공개 포트폴리오 페이지
│   │   └── page.tsx         # 랜딩 페이지
│   ├── components/
│   │   ├── ui/              # shadcn/ui 기본 컴포넌트 (Button, Card, Dialog 등)
│   │   ├── salon/           # 매장 관련 컴포넌트
│   │   ├── settings/        # 설정 관련 컴포넌트
│   │   ├── Navbar.tsx       # 네비게이션 바
│   │   ├── ImageUploader.tsx # 이미지 업로드
│   │   ├── RecipeCard.tsx   # 레시피 카드
│   │   └── TimelineView.tsx # 타임라인 뷰
│   ├── lib/
│   │   ├── supabase/        # Supabase 클라이언트 (server/client/admin)
│   │   ├── auth.ts          # 인증 + 사용량 관리
│   │   ├── fal.ts           # Fal.ai 이미지 생성 래퍼
│   │   ├── openai.ts        # OpenAI 클라이언트 싱글톤
│   │   ├── prompts.ts       # 모든 AI 프롬프트 중앙 관리
│   │   ├── storage.ts       # AI 생성 이미지 → Supabase Storage 캐싱
│   │   ├── subscription.ts  # 구독 유틸리티 (D-Day, 권한, 가격)
│   │   ├── hairStyles.ts    # 헤어스타일 데이터 카탈로그
│   │   └── replicate.ts     # Replicate API (대안)
│   ├── types/
│   │   └── index.ts         # 전체 TypeScript 타입 정의
│   └── middleware.ts        # 인증 + 온보딩 가드
├── supabase/
│   ├── schema.sql           # 초기 DB 스키마
│   └── migrations/          # 마이그레이션 파일들 (17개)
├── vercel.json              # Cron Jobs 설정
├── next.config.ts           # 외부 이미지 도메인 허용 목록
└── package.json

```


## 4. 데이터베이스 스키마

### RLS (Row Level Security) 핵심 규칙
```
sql

-- 본인 데이터만 접근 (모든 테이블 공통 패턴)

CREATE POLICY "own_data" ON {table}

  FOR SELECT USING (user_id = auth.uid());
```

TIP
RLS가 활성화된 테이블에서 **서버 측 admin 작업**이 필요하면 `createAdminClient()`를 사용. Admin Client는 `SUPABASE_SERVICE_ROLE_KEY`를 사용하여 RLS를 우회함.

---

## 5. 핵심 아키텍처 패턴

### 인증 플로우
```
미들웨어(middleware.ts)

  ├─ 미인증 + 보호경로 → /login 리다이렉트

  ├─ 인증 + /login → /dashboard 리다이렉트

  └─ 인증 + 미온보딩 → /onboarding 리다이렉트
```
### API 응답 표준 형식
```typescript

// 모든 API는 이 형식을 따름
interface ApiResponse<T> {

  data: T | null;

  error: { code: string; message: string } | null;

}
```
### 사용량 제한 (Free 플랜)

```typescript

// 일일 3회 제한, 날짜 변경 시 자동 리셋

const DAILY_LIMITS = { free: 3, basic: 999999, enterprise: 999999 };

// Enterprise 조직 멤버는 자동으로 basic 권한으로 업그레이드

if (orgPlan === 'enterprise' && userPlan === 'free') {

  userPlan = 'basic'; // 무제한

}
```

### Vercel 배포 설정

```typescript

// next.config.ts — 외부 이미지 도메인 허용 필수

images: {

  remotePatterns: [

    { hostname: "*.supabase.co" },   // Supabase Storage

    { hostname: "fal.media" },        // Fal.ai 생성 이미지

    { hostname: "replicate.delivery" } // Replicate (대안)

  ]

}

// API Route 타임아웃 확장

export const maxDuration = 60; // Vercel Pro 기준 최대 60초
```
---
## 6. AI 이미지 생성 파이프라인

### 전체 흐름

![](Pasted%20image%2020260304200107.png)

### Flux Kontext Pro 최적 설정 (원본 보존 최대화)

```typescript

const result = await fal.subscribe('fal-ai/flux-pro/kontext', {

  input: {

    image_url: accessibleUrl,

    prompt: `Change ONLY the hairstyle of this person to ${style}. 

             Keep the exact same face, facial features, skin tone, 

             expression, pose, body, clothing, background, and lighting. 

             Do NOT alter anything except the hair. 

             The result must look like the same photograph with only 

             the hair restyled.`,

    guidance_scale: 2,      // 낮을수록 원본 보존 ↑ (기본 3.5)

    output_format: 'jpeg',

    seed: 42,               // 일관된 결과

  },

});
```
### 이미지 접근성 문제 해결 패턴

```typescript

// 문제: Fal.ai가 Supabase Storage URL에 직접 접근 불가

// 해결: Fal CDN에 프록시 업로드

async function uploadToFalCdn(imageUrl: string): Promise<string> {

  const response = await fetch(imageUrl);

  const blob = await response.blob();

  const file = new File([blob], 'input.jpg', { type: 'image/jpeg' });

  return await fal.storage.upload(file); // Fal CDN URL 반환

}
```

### AI 모델 비교 (이미지 생성)

| 모델                   | 용도                        | 원본 보존 | 품질    | 비용  |
| -------------------- | ------------------------- | ----- | ----- | --- |
| **Flux Kontext Pro** | 헤어스타일 편집 (image-to-image) | ⭐⭐⭐⭐  | ⭐⭐⭐⭐  | $$  |
| Flux Pro v1.1        | Text-to-Image (폴백)        | ❌     | ⭐⭐⭐⭐⭐ | $$  |
| IP-Adapter Face ID   | 얼굴 합성                     | ⭐⭐    | ⭐⭐⭐   | $   |
| Gemini Imagen        | 이미지 편집                    | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | $$$ |


**`guidance_scale`은 원본 보존의 핵심 파라미터**

- 높으면 (3~5): 프롬프트를 강하게 따름 → 원본과 괴리 ↑
- 낮으면 (1~2): 원본에 가깝게 유지 → 변경 최소화
- 헤어스타일만 바꿀 때는 **2 이하** 추천

---

## 7. 유용한 프롬프트 모음

### GPT Vision — 모발 상태 분석

```당신은 20년 경력의 최고급 헤어 디자이너이자 모발 과학 전문가입니다.
고객의 앞면 사진 1장을 정밀 분석하여, 얼굴형, 두상 형태, 모발 상태, 손상도를 진단합니다.
반드시 아래 JSON 형식으로만 응답하세요.
```

### GPT Vision — 시술 레시피 생성
```
두 장의 사진(현재 모발 + 레퍼런스 스타일)을 비교 분석하여,
주니어 헤어 디자이너도 따라할 수 있는 정확한 시술 레시피를 제공합니다.
약제 브랜드는 한국 시장 기준(웰라, 로레알, 밀본 등)으로 추천하세요.
```

### Flux Kontext Pro — 헤어스타일 변경 (원본 보존)
```
Change ONLY the hairstyle of this person to {스타일명}.
Keep the exact same face, facial features, skin tone, expression,
pose, body, clothing, background, and lighting.
Do NOT alter anything except the hair.
The result must look like the same photograph with only the hair restyled.
```

### Flux Pro — 고품질 헤어 이미지 (Text-to-Image)
```
Ultra-realistic professional salon portrait photograph.
{스타일 설명}.
Shot with Canon EOS R5, 85mm f/1.4 lens.
Soft studio lighting, individual hair strands visible.
Fashion magazine quality, no text or watermarks.
```

---

## 트러블슈팅 & 교훈

###  Fal.ai가 Supabase URL 접근 불가

- **원인**: Supabase Storage의 private 버킷 URL은 외부에서 접근 불가
- **해결**: uploadToFalCdn() — 이미지를 다운로드 후 Fal CDN에 재업로드
- **교훈**: 외부 AI 서비스에 이미지를 전달할 때는 반드시 **공개 접근 가능한 URL** 필요

###  Vercel 빌드 시 API 키 없어서 실패

- **원인**: 빌드 타임에 `process.env.OPENAI_API_KEY`를 직접 참조하면 에러
- **해결**: getOpenAI() 싱글톤 패턴으로 **런타임에만 초기화**
- **교훈**: 환경변수가 필요한 코드는 **빌드타임 vs 런타임** 구분 필수

```typescript

// ❌ 빌드 타임에 에러 발생

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY! });

// ✅ 런타임에 지연 초기화

export function getOpenAI(): OpenAI {

  if (!_openai) {

    _openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

  }

  return _openai;

}
```

###  AI 생성 이미지 URL 만료

- **원인**: Fal.ai/Replicate가 반환하는 URL은 임시 URL (수 시간 후 만료)
- **해결**: cacheGeneratedImage() — 생성 직후 Supabase Storage에 영구 저장
- **교훈**: AI 생성 이미지는 **반드시 자체 스토리지에 캐싱**

###  이미지 생성 시 원본과 괴리감

- **원인**: `guidance_scale`이 너무 높으면 프롬프트를 과도하게 따름
- **해결**: `guidance_scale: 2` + 보존 요소 명시적 나열 + `seed` 고정
- **교훈**: Image-to-Image 편집에서 **guidance_scale은 변경 강도**를 제어

###  Next.js Image 컴포넌트 외부 이미지 에러

- **원인**: 허용되지 않은 hostname의 이미지 사용
- **해결**: next.config.ts의 `remotePatterns`에 도메인 추가
- **교훈**: 새로운 외부 이미지 소스 추가 시 항상 next.config.ts 업데이트

###  API Route 타임아웃 (이미지 생성 느림)

- **원인**: Vercel 기본 타임아웃 10초, AI 이미지 생성은 20~45초
- **해결**: `export const maxDuration = 60;` + `Promise.race()` 타임아웃
- **교훈**: AI 호출이 포함된 API는 `maxDuration` 설정 + 클라이언트 측 타임아웃 필수

---

##  환경변수 관리 체크리스트


# .env.local (로컬 개발)

```
NEXT_PUBLIC_SUPABASE_URL=         # 프론트+백 공통

NEXT_PUBLIC_SUPABASE_ANON_KEY=    # 프론트+백 공통

SUPABASE_SERVICE_ROLE_KEY=        # 서버 전용 (Admin)

OPENAI_API_KEY=                   # 서버 전용

FAL_KEY=                          # 서버 전용

TOSS_PAYMENTS_SECRET_KEY=         # 서버 전용

NEXT_PUBLIC_TOSS_CLIENT_KEY=      # 프론트 전용

```

###  Supabase 활용 패턴


```typescript

// 3종류의 Supabase 클라이언트 사용 구분

import { createClient } from '@/lib/supabase/server';  // SSR (쿠키 기반 인증)

import { createBrowserClient } from '@/lib/supabase/client'; // CSR

import { createAdminClient } from '@/lib/supabase/admin';    // RLS 우회

// Admin Client 사용 시점:

// - AI 생성 결과를 DB에 저장할 때 (사용자 쿠키 없이)

// - Signed URL 생성할 때

// - Cron Job에서 자동 처리할 때
```


### 💡 성능 최적화 팁

- `Promise.allSettled()` — 여러 AI 이미지 병렬 생성 (하나 실패해도 나머지는 성공)
- `Promise.race()` — AI 호출에 타임아웃 설정 (무한 대기 방지)
- `dynamic import` — AI 라이브러리는 사용하는 라우트에서만 로드
- 이미지 캐싱 — AI 생성 이미지를 영구 스토리지에 저장 (재생성 방지)

---

#HairFlow #NextJS #Supabase #FalAI #OpenAI 