# 운동기록 앱 — 다운로드 홈페이지 만들기 가이드 (데스크탑 + 안드로이드(출시예정) + 아이폰(웹앱))

AI에게 오늘 한 운동을 말하거나 러닝 앱·클라이밍·헬스 앱 기록 사진을 올리면, 정해진 형식으로 폰 안의 DB에 기록하고 대시보드를 자동으로 만들어 주는 안드로이드 앱입니다.

앱 이름은 **Caldron AI**입니다. 같은 앱 코드(`www`)를 아래 여러 곳에 배포합니다.

| 플랫폼 | 설치 방법 | 상태 |
|---|---|---|
| 데스크탑 (Windows) | Microsoft Store: https://apps.microsoft.com/detail/9P68C75Z2QTP | 출시 |
| 안드로이드 | 홈페이지의 **APK 다운로드** | 베타 (Google Play 출시 예정) |
| 아이폰 | 홈페이지의 **아이폰에 설치** → Safari에서 홈 화면에 추가 (웹앱/PWA) | 베타 |
| 웹 브라우저 | `https://내아이디.github.io/workout-app/app/` | 베타 |

> Microsoft Store 앱과 (출시 예정인) Google Play 앱은 배포된 웹 주소를 불러오는 방식이라, 홈페이지에 새 버전을 올리면 **스토어 재심사 없이 자동으로 업데이트**됩니다. APK는 앱 안에 코드가 들어 있어서 새 APK를 다시 설치해야 합니다.

---

## 전체 구조

```
[앱]  채팅·사진 ──▶ [Supabase Edge Function `ai`] ──▶ [Google Gemini API]
  │                  (지시문 부착, 하루 한도, 호출 기록)
  └─ 운동 기록은 기기 안에만 저장 (APK: SQLite / 웹·아이폰·스토어 앱: 브라우저 저장소)
```

- **기본(무료) 모드**: 사용자는 키 입력 없이 바로 사용합니다. 익명 로그인 후 서버 함수가 Gemini를 대신 호출하며, 1인당 하루 횟수 제한(기본 10회)이 있습니다.
- **AI 지시문(시스템 프롬프트)은 서버에만** 있습니다. 앱은 사용자 입력·사진·최근 기록만 보내므로, 누가 앱의 공개 키를 가져가도 운동 기록 추출 외 용도로 쓸 수 없습니다.
- **내 API 키 모드(선택)**: 설정 → AI 연결 방식에서 OpenAI 또는 OpenRouter 키를 입력하면 기기에서 직접 호출합니다 (한도 없음, 요금은 본인 계정).

---

## 준비물 체크리스트

| 준비물 | 어디서 | 비용 |
|---|---|---|
| 이 폴더 (zip 압축 풀기) | 받은 파일 | - |
| GitHub 계정 | github.com → Sign up | 무료 |
| Supabase 프로젝트 | supabase.com → New project | 무료 (Free 플랜) |
| Google AI Studio API 키 | aistudio.google.com → Get API key | 무료 등급 사용 (결제 연결 안 하면 요금 없음) |

> **API 키·비밀값은 저장소나 앱에 넣지 않습니다.** Gemini 키는 Supabase Secrets에만 저장합니다. 앱에 들어가는 Supabase URL과 publishable key(`www/js/config.js`)는 공개용이라 저장소를 Public으로 만들어도 안전합니다 (데이터는 RLS로 보호).

---

## 1단계. GitHub 저장소 만들기

1. https://github.com 에 로그인합니다.
2. 오른쪽 위 **+** → **New repository**를 누릅니다.
3. Repository name에 `workout-app`을 적습니다.
4. **Public**을 선택합니다.
   - 무료 계정에서 홈페이지(GitHub Pages)와 다운로드 링크가 누구에게나 열리려면 Public이어야 합니다.
5. 아래 옵션(README 추가 등)은 아무것도 체크하지 말고 **Create repository**를 누릅니다.

## 2단계. 홈페이지 켜기 (파일 올리기 전에!)

1. 저장소 위쪽 **Settings** 탭 → 왼쪽 메뉴 **Pages**를 누릅니다.
2. **Build and deployment → Source**를 **GitHub Actions**로 바꿉니다. (저장 버튼 없이 바로 적용됩니다)

## 3단계. AI 서버 만들기 (Supabase + Gemini)

1. **Supabase 프로젝트 만들기**: supabase.com → New project (지역: Seoul 권장).
2. **익명 로그인 켜기**: Authentication → Sign In / Providers → **Allow anonymous sign-ins** 켜기.
3. **테이블·함수 만들기**: SQL Editor → `supabase/schema.sql` 내용을 붙여넣고 **Run**.
   - `ai_usage`(하루 사용 횟수), `ai_logs`(호출 기록) 테이블과 `use_ai_quota` / `refund_ai_quota` 함수가 생깁니다.
4. **Gemini 키 발급**: aistudio.google.com → **Get API key** → Create API key.
5. **Secrets 등록**: Edge Functions → **Secrets**에 아래 값을 추가합니다.

   | 이름 | 값 | 필수 |
   |---|---|---|
   | `GEMINI_API_KEY` | AI Studio에서 받은 키 | 필수 |
   | `GEMINI_MODEL` | 사용할 모델. 쉼표로 여러 개 적으면 앞에서부터 시도 (예: `gemini-3.5-flash-lite`) | 선택 (기본 `gemini-3.8-flash,gemini-3.5-flash-lite`) |
   | `GEMINI_REASONING` | 답하기 전 생각 정도: `none` / `minimal` / `low` / `medium` / `high` / `default` | 선택 (기본 `low`). 빠르게 하려면 `minimal` |
   | `DAILY_LIMIT` | 1인당 하루 AI 사용 횟수 | 선택 (기본 10) |

6. **함수 배포**: Edge Functions → **Deploy a new function → Via Editor** → 이름 `ai` → `supabase/functions/ai/index.ts` 내용을 붙여넣고 **Deploy**.
7. **앱에 주소 연결**: Project Settings → API에서 Project URL과 publishable key를 복사해 `www/js/config.js`에 넣습니다.

> **무료 등급 참고**
> - 무료 한도는 모델마다 다르며 **요청 횟수** 기준입니다 (사진 여러 장도 보내기 1번 = 1회). 정확한 숫자는 https://aistudio.google.com/rate-limit 에서 확인하세요. Flash-Lite가 하루 한도가 넉넉하고 빠릅니다.
> - 무료 등급으로 보낸 내용은 Google의 AI 학습·서비스 개선에 활용될 수 있습니다. 앱 공지와 개인정보처리방침(`www/privacy.html`)에 이 내용이 들어 있습니다.
> - Google이 모델을 내리면(예: 신규 사용자에게 `gemini-2.5-flash` 중단) 오류 메시지에 새 모델 이름이 나옵니다. `GEMINI_MODEL` Secret만 바꾸면 되고, 재배포는 필요 없습니다.

## 4단계. 파일 올리기

1. 저장소 첫 화면(**Code** 탭)에서 **uploading an existing file** 링크를 누릅니다.
   (또는 **Add file → Upload files**)
2. zip을 푼 `workout-app` 폴더를 열고, **안에 있는 것 전부**(`.github`, `www`, `site`, `resources`, `supabase` 폴더와 `package.json` 등)를 선택해서 브라우저 창으로 끌어다 놓습니다.
3. 아래 **Commit changes**를 누릅니다.
4. 올라간 목록에 **`.github`** 폴더가 보이는지 확인합니다.
   - 안 보이면 → 아래 "문제 해결 1"을 따라 하세요.

## 5단계. 자동 빌드·배포 기다리기

1. 저장소 위쪽 **Actions** 탭을 누릅니다.
2. `Build apps`가 노란색(진행 중)으로 돌고 있으면 기다립니다. **약 5~10분**.
   - 목록이 비어 있으면: 왼쪽 **Build apps** → 오른쪽 **Run workflow** → 초록 버튼 **Run workflow**.
3. 그 줄을 눌러 보면 3개 작업이 있습니다.
   - `Android APK` ✓ → APK가 만들어져 **Releases**에 올라감
   - `Homepage (GitHub Pages)` ✓ → 홈페이지와 웹앱(`/app/`)이 열림
   - `iOS build check` → 실패해도 상관없습니다.

## 6단계. 설치하기

홈페이지 **`https://내아이디.github.io/workout-app/`** 은 접속한 기기를 판별해 맞는 버튼을 보여 줍니다.

- **Windows**: **Microsoft Store** 버튼 → 스토어에서 설치.
- **안드로이드**: **APK 다운로드** → 알림창(또는 다운로드 폴더)에서 `workout-log.apk` 실행.
  - "출처를 알 수 없는 앱" 안내 → **설정** → 이 출처 허용 → 뒤로 → **설치**
  - Play 프로텍트 경고 → **세부정보 더보기** → **무시하고 설치**
  - Google Play 출시 후에는 스토어의 "설치" 버튼 하나로 바뀝니다.
- **아이폰**: 아래 "아이폰" 항목 참고.
- PC에서 열면 QR 코드가 나와요. 폰 카메라로 찍으면 바로 열립니다.

### 업데이트할 때
고친 파일을 같은 방법으로 다시 올리면(같은 이름은 덮어쓰기) 자동으로 새 버전(1.0.N)이 만들어집니다.
- 웹·아이폰·Microsoft Store 앱: 다음에 앱을 열 때 자동 적용.
- APK: 홈페이지에서 다시 받아 설치하면 **기록은 그대로** 두고 앱만 바뀝니다. (고정 서명키 `resources/debug.keystore` 덕분이니 이 파일은 지우지 마세요.)
- 서버 함수(`supabase/functions/ai/index.ts`)를 고쳤다면 Supabase에서 따로 다시 배포해야 합니다.

## 7단계. 처음 실행

1. 앱을 열면 **이용 안내 공지**가 한 번 뜹니다 (AI 데이터 처리·민감정보 주의·기록 저장·이용 한도·결과 확인). **닫기**를 누르면 그 기기에서는 다시 뜨지 않습니다.
   - 공지 내용을 바꾸면 `www/js/app.js`의 `NOTICE_VERSION` 값을 바꿔야 모든 사용자에게 다시 뜹니다.
2. 키 입력 없이 바로 채팅으로 운동을 말하거나 기록 사진을 올리면 됩니다. 채팅 위에 오늘 남은 무료 횟수가 표시됩니다.
3. 설정 탭 → **샘플 데이터 넣기**로 대시보드를 미리 볼 수 있습니다 (나중에 "모든 기록 삭제"로 지우면 됩니다).

---

## 앱 기능

| 기능 | 구현 |
|---|---|
| 대화 → 정형화 | 채팅 문장을 AI가 `workout_logs` 테이블 형식으로 변환 |
| **기록 사진 분석** | 카메라 버튼 → 러닝·클라이밍·헬스 앱 기록 사진(최대 4장)에서 거리·시간·페이스·심박·완등 수 등을 읽어 기록. 갤럭시·아이폰 고효율(HEIC) 사진도 자동 변환 |
| 새 종목 확인 | 기존에 없는 종목이면 "새로 추가 / 기존 종목에 넣기 / 빼기"를 묻고, "오늘 운동은 다음과 같이 기록할까요?" 창에서 **확인**을 눌러야 저장 (이 단계는 AI 횟수 차감 없음) |
| 자동 대시보드 | 기간 / 운동종류 / 세부운동 슬라이서. `detail_json`의 숫자가 종목별 코딩 없이 자동으로 차트가 됨 |
| 대시보드 특수 표시 | 페이스는 축을 뒤집어 빠를수록 위 · 색깔별 완등 수 같은 묶음 값은 여러 선을 한 차트에 겹침 · 인터벌은 뛰기/걷기 구간 평균 페이스와 평균 구간 거리 |
| 이용 안내 공지 | 첫 실행 시 1회 표시, 닫기 버튼 |
| 백업 | 설정 → CSV로 내보내기 / 백업 CSV 불러오기 (엑셀·파워BI 호환) |
| AI 연결 방식 | 기본(무료, 서버 경유 Gemini) 또는 내 API 키(OpenAI·OpenRouter) |

### 테이블 `workout_logs` (기기 안)

| 컬럼 | 예시 |
|---|---|
| id | 103 |
| date | 2026-10-01 |
| exercise_type | 러닝 |
| exercise_name | 야외 러닝 |
| duration_min | 34 |
| result_summary | 5km 무휴식 |
| detail_json | `{"distance_km":5.0,"pace_sec_per_km":408,...}` |
| memo | 허벅지가 먼저 힘들었음 |
| created_at / updated_at | 입력·수정 시각 |

### 서버 테이블 (Supabase)

| 테이블 | 내용 |
|---|---|
| `ai_usage` | 사용자(익명 ID)별 날짜별 AI 사용 횟수. 한국 시간 0시 초기화 |
| `ai_logs` | 호출 기록: 모델, 상태 코드, 입력·출력 토큰. 대화 내용과 사진은 저장하지 않음 |

두 테이블 모두 RLS로 앱에서 직접 접근할 수 없고, 서버 함수만 읽고 씁니다.

---

## 문제 해결

**1. `.github` 폴더가 안 올라갔어요**
점(.)으로 시작하는 폴더는 업로드에서 빠질 때가 있습니다. 직접 만들면 됩니다.
1. 저장소에서 **Add file → Create new file**
2. 파일 이름 칸에 `.github/workflows/build.yml` 을 그대로 입력 (슬래시를 치면 폴더가 자동으로 생깁니다)
3. 내 PC의 `workout-app/.github/workflows/build.yml` 파일을 메모장으로 열어 내용을 전부 복사 → 붙여넣기
4. **Commit changes**

**2. Actions에서 빨간 X(실패)가 떴어요**
실패한 줄 → 빨간 작업 → 빨간 단계를 눌러 나오는 로그를 복사하거나 캡처해서 Claude에게 보여 주세요.
- `Homepage (GitHub Pages)`만 실패: 2단계(Settings → Pages → Source: GitHub Actions)를 하고, 실패한 실행 화면 오른쪽 위 **Re-run all jobs**를 누르세요.
- `Publish to Releases`에서 권한(403) 오류: Settings → Actions → General → Workflow permissions → **Read and write permissions** 선택 → Save 후 Re-run.

**2-1. 다운로드 버튼을 눌렀는데 404가 떠요**
`Android APK` 작업이 아직 안 끝났거나 실패한 것입니다. 저장소 오른쪽 **Releases**에 `workout-log.apk`가 있는지 확인하세요. 저장소가 Private이면 다른 사람은 받을 수 없습니다.

**3. "AI 서버가 응답하지 않아요" 오류**
괄호 안의 원문 메시지를 확인하세요. 차감된 1회는 자동으로 돌려줍니다.
- `GEMINI_API_KEY가 없어요`: 3단계 5번 Secrets 등록.
- `no longer available` / 모델 이름 안내: `GEMINI_MODEL` Secret을 안내된 새 모델로 변경.
- 429 / quota: 무료 한도 초과. 1분 한도면 잠시 후 다시, 하루 한도면 다음 날 초기화. 한도가 넉넉한 Flash-Lite로 바꾸는 것을 권장.

**4. AI 응답이 느려요**
- `GEMINI_REASONING`을 `minimal`(또는 `none`)로, `GEMINI_MODEL`을 Flash-Lite로 바꾸면 빨라집니다.
- 원인 확인: SQL Editor에서
  ```sql
  select created_at, model, status, prompt_tokens, completion_tokens
  from ai_logs order by created_at desc limit 10;
  ```
  `completion_tokens`가 수천이면 생각 시간 때문, 같은 시각에 줄이 여러 개면 재시도·다음 모델로 넘어간 것입니다.

**5. 내 API 키 모드에서 "API 키가 올바르지 않습니다" / 429 오류**
- 키를 다시 복사해 보세요 (앞뒤 공백 주의).
- OpenAI 429 / quota: platform.openai.com → Billing에서 크레딧 충전 (ChatGPT Plus 구독과 별개).
- OpenRouter 무료 모델 429: 사용량 제한입니다. 잠시 후 다시 보내거나 설정에서 다른 모델을 고르세요.

---

## 아이폰 (홈페이지에서 설치)

아이폰은 Apple 정책상 App Store·TestFlight 말고는 앱 파일을 홈페이지에서 받아 설치할 수 없습니다. 그래서 아이폰은 **홈 화면에 추가하는 웹앱(PWA)** 방식으로 설치합니다. 같은 앱 코드(`www`)가 홈페이지의 `app/` 주소로 자동 배포됩니다.

1. 아이폰 **Safari**에서 홈페이지(`https://내아이디.github.io/workout-app/`)를 엽니다.
2. **아이폰에 설치**를 누르면 앱 화면이 열립니다.
3. 아래쪽 **공유** 버튼 → **홈 화면에 추가** → **추가**.
   - 공유 버튼이 안 보이면 주소창 옆 **···** → 공유
   - 카카오톡 등 앱 안에서 열었으면 먼저 **Safari로 열기**
4. 홈 화면의 **Caldron** 아이콘으로 열면 전체 화면 앱처럼 동작합니다. 사진 올리기·AI 분석·대시보드 모두 안드로이드 앱과 같습니다.

**알아 둘 점**
- 기록은 그 아이폰 안(홈 화면 앱 저장소)에만 저장됩니다. 홈 화면 앱을 삭제하면 기록도 지워지니, 설정 → **CSV로 내보내기**로 가끔 백업하세요. 폰을 바꾸면 **백업 CSV 불러오기**로 옮길 수 있습니다.
- Safari 브라우저 탭과 홈 화면 아이콘은 저장소가 따로라서, **홈 화면 아이콘으로만** 쓰는 걸 권장합니다.
- 업데이트는 자동입니다.

**진짜 iOS 앱(App Store/TestFlight)으로 배포하려면**
Apple Developer Program(연 $99) 가입이 필요합니다. 가입하면 GitHub Actions에서 iOS 앱을 서명·빌드해 TestFlight에 올릴 수 있습니다. (Mac 없이도 가능)

---

## 앞으로 할 일

- **Google Play 출시**: PWABuilder로 TWA 패키지(`com.caldron.ai`) 생성 → assetlinks 연결 → 비공개 테스트(테스터 12명 · 14일) → 정식 출시.
- **Google 로그인 + 서버 백업**: 여러 기기에서 쓰기 위해 기록을 서버에 저장. 로그인 시 서버 저장 동의(건강 정보 별도 체크)·동의 철회 기능, 개인정보처리방침 갱신.

---

## 파일 구조

```
.github/workflows/build.yml   APK 빌드 → Releases 업로드 → 홈페이지·웹앱 배포 (HEIC 변환기 포함)
site/                         다운로드 홈페이지 (index.html, img/) — 배포 시 www가 site/app/(웹앱)으로 복사됨
www/                          앱 화면과 기능
  index.html, css/app.css
  privacy.html     개인정보 처리방침
  js/app.js        채팅, 사진 첨부, 새 종목 질문, 확인 창, 설정, 이용 안내 공지
  js/ai.js         AI 호출 (기본: 서버 경유 / 내 키: 기기에서 직접), 사진 변환
  js/server.js     Supabase 익명 로그인·서버 함수 호출·남은 횟수
  js/config.js     Supabase URL + publishable key (공개용)
  js/db.js         기기 내부 저장 (SQLite, 실패 시 브라우저 저장소)
  js/dashboard.js  슬라이서 + 자동 차트
  js/metrics.js    detail_json → 숫자 지표 (인터벌 구간 요약 포함)
  js/charts.js     차트
  manifest.webmanifest, sw.js, icons/   웹앱(PWA) 설정·오프라인 보관
supabase/schema.sql             서버 테이블·RLS·하루 한도 함수
supabase/functions/ai/index.ts  AI 중계 함수 (지시문·검증·Gemini 호출·모델 순차 시도)
resources/icon.png, apply_icon.py   앱 아이콘
resources/debug.keystore       앱 서명키 (업데이트 설치용, 지우지 말 것)
capacitor.config.json, package.json  앱 이름·패키지 설정
```
