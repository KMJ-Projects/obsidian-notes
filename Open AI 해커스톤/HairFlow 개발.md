## 🎯 MVP 

### 필수 화면 (5개만)

```
1. 랜딩 페이지        → 서비스 소개 + 시작하기 버튼
2. 로그인/회원가입     → Supabase Auth (구글 로그인)
3. AI 시술 레시피      → 사진 2장 업로드 → 레시피 카드 결과
4. AI 미래 타임라인    → 시술 완료 사진 → 변화 예측 이미지
5. 구독 플랜 페이지    → Free/Basic/Pro + 결제 버튼
```
### 핵심 기능

#### 기능 1: AI 시술 레시피

```
[입력] 
- 고객 레퍼런스 사진 (원하는 연예인 머리 스타일)
- 고객 현재 모발 사진

[AI 처리]
- GPT-4o Vision: 두 사진 비교 분석
- GPT-4o: 시술 레시피 텍스트 생성 (헤어디자이너가 이해할 수 있도록 친숙하고 전문적인 형태)

[출력 - 레시피 카드]
- 현재 모발 상태 요약
- 목표 스타일 분석
- 시술 방법 (컬러/커트/펌)
- 약제 배합 비율
- 시술 순서 및 시간
- 주의사항
```

#### 기능 2: AI 미래 타임라인

```
[입력]
- 시술 완료 사진
- 시술 종류 선택 (염색/커트/펌)
  고객 두피 상태에 따라 머리카락 자라는 속도가 다르지만 평균 사람들의 자라는 속도에 맞추어 예측

[AI 처리]
- GPT-4o Vision: 현재 상태 분석
- DALL-E 3: 시간별 변화 이미지 생성

[출력 - 타임라인]
- 현재 (시술 직후)
- 2주 후 예측 이미지
- 4주 후 예측 이미지
- 8주 후 예측 이미지
- 추천 재방문 시점 표시
```

## 🛠 기술 스택

| 영역      | 기술                                     | 설치 명령어                                                        |
| ------- | -------------------------------------- | ------------------------------------------------------------- |
| 프론트엔드   | Next.js 14 + TypeScript + Tailwind CSS | `npx create-next-app@latest hairflow --typescript --tailwind` |
| 백엔드     | Next.js API Routes (별도 백엔드 불필요!)       | -                                                             |
| AI      | OpenAI API (GPT-4o Vision + DALL-E 3)  | `npm install openai`                                          |
| DB      | Supabase (PostgreSQL)                  | `npm install @supabase/supabase-js`                           |
| 인증      | Supabase Auth (구글 로그인)                 | (Supabase에 포함)                                                |
| 결제      | Toss Payments (테스트 모드)                 | `npm install @tosspayments/payment-sdk`                       |
| 배포      | Vercel                                 | `npm install -g vercel`                                       |
| UI 컴포넌트 | shadcn/ui                              | `npx shadcn-ui@latest init`                                   |
## 📁 프로젝트 구조

```
hairflow/
├── app/
│   ├── page.tsx                    # 랜딩 페이지
│   ├── login/page.tsx              # 로그인
│   ├── dashboard/page.tsx          # 대시보드
│   ├── recipe/page.tsx             # AI 시술 레시피
│   ├── timeline/page.tsx           # AI 미래 타임라인
│   ├── pricing/page.tsx            # 구독 플랜
│   └── api/
│       ├── recipe/route.ts         # 레시피 API
│       ├── timeline/route.ts       # 타임라인 API
│       └── auth/route.ts           # 인증 API
├── components/
│   ├── ui/                         # shadcn 컴포넌트
│   ├── ImageUploader.tsx           # 사진 업로드 컴포넌트
│   ├── RecipeCard.tsx              # 레시피 결과 카드
│   ├── TimelineView.tsx            # 타임라인 뷰
│   └── Navbar.tsx                  # 네비게이션
├── lib/
│   ├── openai.ts                   # OpenAI 클라이언트
│   ├── supabase.ts                 # Supabase 클라이언트
│   └── prompts.ts                  # AI 프롬프트 모음
├── .env.local                      # API 키 (절대 깃에 안 올림)
└── package.json
```
## 🔑 필요한 API 키 / 계정

- [ ]  OpenAI API Key → platform.openai.com
- [ ]  Supabase 프로젝트 → supabase.com
- [ ]  Vercel 계정 → vercel.com
- [ ]  Toss Payments 테스트 키 → developers.tosspayments.com
- [ ]  GitHub 레포지토리 생성

## 🎬 3분 시연 영상 구성

```
0:00-0:20  "헤어 디자이너 30만명의 두 가지 치명적 문제"
0:20-0:40  "HairFlow가 해결합니다" (서비스 소개)
0:40-1:30  [데모] AI 시술 레시피 시연
           - 레퍼런스 사진 + 현재 모발 사진 업로드
           - 3초 후 레시피 카드 생성되는 장면
1:30-2:20  [데모] AI 미래 타임라인 시연
           - 시술 완료 사진 업로드
           - 2주/4주/8주 변화 예측 이미지 생성
2:20-2:40  구독 플랜 + 결제 플로우 시연
2:40-3:00  비즈니스 모델 + 시장 규모 + 마무리
```