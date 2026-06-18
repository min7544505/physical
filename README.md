# ⚡ 게이머 피지컬 측정기

**메이드 인 와리오** 식 5초짜리 순발력 미니게임 **8종**을 연속으로 클리어하고, 종합 점수로 게이머 피지컬 티어(아이언~챌린저)를 받는 단일 파일 웹앱.
각 게임은 `명령어 한 줄 → 5초 플레이 → 다음`으로 정신없이 넘어간다. 끝나면 **공유용 결과 이미지**가 나와서 친구랑 점수 대결.

## 실행

- **가장 간단:** `index.html` 더블클릭 (인터넷 불필요, 100% 로컬)
- **로컬 서버:** `node serve.mjs` → http://localhost:5566
  - Claude Code 프리뷰: `.claude/launch.json`의 `physical` 구성

## 미니게임 8종 (각 약 5초)

| | 게임 | 명령어 | 측정 | 점수 만점 기준 |
|---|------|--------|------|----------------|
| ⚡ | 반응속도 | 클릭! | 초록 되면 반응 시간 | **120ms** (0점: 420ms) |
| 🖱️ | 연타 | 연타! | 5초간 클릭수(CPS) | **14회/초** (0점: 4회/초) |
| 🎯 | 에임 | 조준! | 5초간 표적 명중수 | **14개** |
| 🧠 | 순간기억 | 외워! | 패턴 순서 재현(빠른 점멸) | **8단계 전부** |
| 🌈 | 다른색 찾기 | 다른색! | 고정 4×4·한 번 클릭 정확도(틀리면 실패) | **5문제 다 정답** |
| 🔢 | 순서대로 | 순서! | 1→5 순서 클릭(클릭당 점수) | **16클릭**(약 3세트) |
| ⏱️ | 타이밍 | 멈춰! | 중앙 가까울수록 점수(안 누르면 0점) | **정중앙** |
| 🪰 | 추적 | 따라가! | 점을 커서로 덮은 유지율 | **90% 유지** |

> 점수 철학: **모든 종목에서 100점 = 최상위 실력**으로 통일(거저 주는 만점 없음), 점수는 연속적(극단적 점프 X). 위 만점 기준 숫자만 바꾸면 난이도 조정 가능.
> 결과 카드는 각 종목의 **실제 측정값**(165ms·9.4회/초·5/6단계·오차 3% 등)과 0~100 환산 점수를 **함께** 보여준다.
> 튜닝 메모: 반응속도·에임·추적은 환산 점수에 **+5점** 보정(조금 더 후하게). 타이밍은 안 누르면 0점(가까울수록 비례). 다른색 찾기는 고정 4×4에서 **한 번만 클릭**(틀리면 실패)하는 정확도 측정.

매 측정마다 8종을 **랜덤 순서**로 플레이 → 8개 점수(0~100) 평균 → 종합 점수 → 티어. **※ 재미용 추정치.**

각 게임은 시작 전 **3·2·1 카운트다운**(약 3초)으로 명령어를 보여주고, 통과/실패 시 효과음·화면 플래시로 피드백한다. 우상단 🔊 버튼으로 소리 on/off (설정은 브라우저에 저장).

## 티어

아이언 · 브론즈 · 실버 · 골드 · 플래티넘 · 에메랄드 · 다이아몬드 · 마스터 · 그랜드마스터 · 챌린저

## 배포 (GitHub Pages)

1. 새 repo(예: `physical`) 만들고 `index.html` 푸시 → Settings → Pages → `main` 브랜치 지정
2. `index.html` 상단의 `canonical` / `og:url` / `og:image` 경로를 실제 배포 주소로 수정
3. (선택) 결과 카드를 캡처해 `og.png`로 올리면 카톡·디스코드 공유 미리보기에 썸네일이 뜬다

## 온라인 랭킹 켜기 (Supabase · 무료 · 5분)

전세계 랭킹은 점수를 모아둘 서버가 필요해요. 무료 Supabase로 켭니다. (키 넣기 전엔 측정·공유는 정상, 랭킹만 "설정 안내" 표시)

1. [supabase.com](https://supabase.com) 가입 → **New project** (무료 플랜).
2. 좌측 **SQL Editor**에 아래를 붙여넣고 **Run**:

   ```sql
   create table public.scores (
     id bigint generated always as identity primary key,
     created_at timestamptz default now(),
     nickname text not null check (char_length(nickname) between 1 and 12),
     score int not null check (score between 0 and 100),
     tier text,
     details jsonb
   );
   alter table public.scores enable row level security;
   create policy "anyone can read" on public.scores for select using (true);
   create policy "anyone can insert valid" on public.scores for insert
     with check (char_length(nickname) between 1 and 12 and score between 0 and 100);
   ```

3. **Project Settings → API**에서 두 값 복사: **Project URL**(`https://xxxx.supabase.co`)과 **anon public** 키(`eyJ...`).
4. `index.html`에서 `const RANK = { url: "", key: "" };` 를 찾아 두 값 붙여넣기:

   ```js
   const RANK = { url: "https://xxxx.supabase.co", key: "eyJ..." };
   ```

5. 저장 후 새로고침 → **🌐 전세계 랭킹** 동작! (홈의 "🌐 전세계 랭킹" 또는 결과 화면 "🌐 전세계 랭킹 올리기")

> **anon 키는 공개돼도 안전.** RLS로 *읽기* + *유효한 점수 삽입*만 허용하고 남의 기록 수정·삭제는 불가능. 점수는 0~100·닉네임 1~12자만 통과(서버에서 강제). 재미용이라 부정점수 방어는 최소 수준이며, 더 막고 싶으면 닉네임당 1일 1회 제한 등을 정책에 추가하면 됩니다.

## 파일

| 파일 | 역할 |
|------|------|
| `index.html` | 단일 파일 앱 (미니게임 프레임워크 + 9종 + 티어 + 공유 카드, 의존성 0) |
| `serve.mjs` | 의존성 없는 정적 서버 (프리뷰/로컬 확인용) |
| `.claude/launch.json` | Claude Code 프리뷰 구성 |

## 미니게임 추가하는 법

`GAMES` 배열에 객체 하나 추가하면 끝. 프레임워크가 명령어 플래시·타이머·점수 집계·티어 반영을 알아서 처리한다.

```js
{ id:"newgame", icon:"🆕", cmd:"눌러!", name:"새 게임", ds:"설명", hint:"한 줄 힌트", dur:5000,
  play(stage, ctx){
    // stage 안에 UI 구성. 조기 종료는 ctx.done(점수0~100).
    // 반환한 함수는 5초(dur) 타임아웃 시 호출되어 현재 점수를 돌려줘야 함.
    return () => 현재점수;
  } }
```

## 기록

최고 티어/기록은 브라우저 `localStorage`(`gamerPhysical.best.v2`)에만 저장된다. 홈의 **"내 기록 초기화"**로 삭제.
