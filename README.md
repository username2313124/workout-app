# 운동기록 앱 — 다운로드 홈페이지 만들기 가이드 (데스크탑 + 안드로이드(출시예정) + 아이폰(웹앱))

AI에게 오늘 한 운동을 말하거나 **러닝 앱·클라이밍·헬스 앱 기록 사진**을 올리면, 정해진 형식으로 폰 안의 DB에 기록하고 대시보드를 자동으로 만들어 주는 안드로이드 앱입니다.

이 가이드를 따라 하면 **`https://내아이디.github.io/workout-app/`** 주소의 홈페이지가 생기고, 안드로이드는 **APK 다운로드**, 아이폰은 **아이폰에 설치** 버튼으로 앱을 설치할 수 있습니다. PC에 아무것도 설치할 필요가 없고, 처음 한 번 20~30분 걸립니다. 이후에는 파일을 고쳐 올리기만 하면 APK와 홈페이지가 자동으로 새 버전이 됩니다.

---

## 준비물 체크리스트

| 준비물 | 어디서 | 비용 |
|---|---|---|
| 이 폴더 (zip 압축 풀기) | 받은 파일 | - |
| GitHub 계정 | github.com → Sign up | 무료 |
| AI API 키 (둘 중 하나) | **OpenRouter**: openrouter.ai/keys · **OpenAI**: platform.openai.com | 사용한 만큼 (기록 1건에 몇 원 수준). OpenRouter는 무료 모델로 시험 가능 |
| 안드로이드 폰 | - | - |

> OpenAI API는 ChatGPT 유료 구독(Plus)과 **별개**입니다. platform.openai.com → Billing 에서 크레딧을 먼저 충전해야 키가 동작합니다 (최소 $5).
>
> **OpenRouter로 시험하기**: openrouter.ai 에서 구글 계정으로 가입 → Keys → Create Key (sk-or- 로 시작). 무료 모델은 충전 없이 쓸 수 있지만 하루 사용량 제한이 있고, 유료 모델은 Credits 충전이 필요합니다 (충전 시 약 5.5% 수수료).

> **API 키는 앱 안에서 각자 입력**하므로 저장소에는 들어가지 않습니다. 그래서 저장소를 공개(Public)로 만들어도 안전합니다.

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

## 3단계. 파일 올리기

1. 저장소 첫 화면(**Code** 탭)에서 **uploading an existing file** 링크를 누릅니다.
   (또는 **Add file → Upload files**)
2. zip을 푼 `workout-app` 폴더를 열고, **안에 있는 것 전부**(`.github`, `www`, `site`, `resources` 폴더와 `package.json` 등)를 선택해서 브라우저 창으로 끌어다 놓습니다.
3. 아래 **Commit changes**를 누릅니다.
4. 올라간 목록에 **`.github`** 폴더가 보이는지 확인합니다.
   - 안 보이면 → 아래 "문제 해결 1"을 따라 하세요.

## 4단계. 자동 빌드·배포 기다리기

1. 저장소 위쪽 **Actions** 탭을 누릅니다.
2. `Build apps`가 노란색(진행 중)으로 돌고 있으면 기다립니다. **약 5~10분**.
   - 목록이 비어 있으면: 왼쪽 **Build apps** → 오른쪽 **Run workflow** → 초록 버튼 **Run workflow**.
3. 그 줄을 눌러 보면 3개 작업이 있습니다.
   - `Android APK` ✓ → APK가 만들어져 **Releases**에 올라감
   - `Homepage (GitHub Pages)` ✓ → 홈페이지가 열림. 여기 나오는 주소가 홈페이지 주소입니다.
   - `iOS build check` → 실패해도 상관없습니다.

## 5단계. 홈페이지에서 설치하기 (안드로이드 · 아이폰은 아래 "아이폰" 항목)

1. 폰 브라우저(Chrome 등)로 **`https://내아이디.github.io/workout-app/`** 을 엽니다.
   - PC에서 열면 오른쪽에 QR 코드가 나와요. 폰 카메라로 찍으면 바로 열립니다.
2. **APK 다운로드**를 누릅니다.
3. 다운로드가 끝나면 알림창(또는 다운로드 폴더)에서 `workout-log.apk`를 누릅니다.
   - "출처를 알 수 없는 앱" 안내 → **설정** → 이 출처 허용 → 뒤로 → **설치**
   - Play 프로텍트 경고 → **세부정보 더보기** → **무시하고 설치**
4. 홈 화면에 **운동기록** 아이콘이 생깁니다.

> 안드로이드 보안 정책상 "다운로드 누르면 자동 설치"는 어떤 앱도 할 수 없고, 마지막 **설치** 버튼은 사용자가 직접 눌러야 합니다. Play 스토어에 올리면 이 과정이 "설치" 버튼 하나로 줄어듭니다 (Google 개발자 계정 $25 일회성).

> 홈페이지 주소를 친구에게 보내면 친구도 똑같이 설치할 수 있습니다. 기록은 각자 폰에만 저장됩니다.

### 업데이트할 때
`www` 폴더의 파일을 고쳐서 같은 방법으로 다시 올리면(같은 이름은 덮어쓰기) 자동으로 새 버전(1.0.2, 1.0.3 …)이 만들어집니다. 홈페이지에서 다시 다운로드해 설치하면 **기록은 그대로** 두고 앱만 바뀝니다. (고정 서명키 `resources/debug.keystore`를 쓰기 때문에 가능하니, 이 파일은 지우지 마세요.)

## 6단계. 처음 실행

1. 앱을 열면 AI 서비스 선택 화면이 나옵니다. **OpenRouter** 또는 **OpenAI (ChatGPT)**를 누릅니다.
2. 고른 서비스의 API 키를 붙여넣고 **시작하기**를 누릅니다.
   - OpenRouter: https://openrouter.ai/keys
   - OpenAI: https://platform.openai.com/api-keys
3. OpenRouter를 고르면 사진을 읽을 수 있는 모델 중 저렴한 모델이 자동으로 선택됩니다. **설정 → 모델**에서 무료/유료 모델 목록(가격 포함)을 보고 바꿀 수 있습니다.
4. 설정 탭 → **샘플 데이터 넣기**로 대시보드를 미리 볼 수 있습니다 (나중에 "모든 기록 삭제"로 지우면 됩니다).

---

## 앱 기능

| 요구사항 | 구현 |
|---|---|
| APK 다운로드 | 홈페이지(GitHub Pages)의 다운로드 버튼 → Releases의 최신 APK |
| ChatGPT 로그인 | OpenAI 또는 OpenRouter API 키 1회 입력 → 확인 후 폰에 저장. 설정에서 언제든 바꾸기 |
| 사용자별 폰 DB | 기기 내부 SQLite `workout_logs` 테이블 |
| 대화 → 정형화 | 채팅 문장을 ChatGPT가 테이블 형식으로 변환 |
| **기록 사진 분석** | 카메라 버튼 → 나이키런·스트라바·가민·클라이밍 기록 사진(최대 4장)에서 거리·시간·페이스·심박·완등 수 등을 읽어 기록 |
| 자동 대시보드 | 기간 / 운동종류 / 세부운동 슬라이서. 종류를 고르면 detail_json 숫자가 차트로 자동 생성 |
| 새 종목 확인 | 기존에 없는 종목이면 "새로 추가 / 기존 종목에 넣기 / 빼기"를 묻고, "오늘 운동은 다음과 같이 기록할까요?" 창에서 **확인**을 눌러야 저장 |
| 내보내기 | 설정 → CSV 내보내기 (엑셀·파워BI) |

### 테이블 `workout_logs`

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

**3. 앱에서 "API 키가 올바르지 않습니다" / 429 오류**
- 키를 다시 복사해 보세요 (앞뒤 공백 주의).
- 429 / quota 오류는 OpenAI 크레딧이 없는 경우입니다 → platform.openai.com → Billing에서 충전.

**4. "model not found" 오류**
- OpenAI: 설정 탭 → 모델 칸에 사용 가능한 모델 이름(예: `gpt-4.1-mini`)을 입력하세요. 사진 분석이 되는 모델이어야 합니다.
- OpenRouter: 설정 탭 → **모델 목록 새로고침** 후 목록에서 다시 고르세요.

**5. OpenRouter 무료 모델에서 429 오류 / 응답을 해석하지 못함**
무료 모델은 사용량 제한이 있고 성능이 들쭉날쭉합니다. 몇 분 뒤 다시 보내거나, 설정에서 다른 모델을 고르세요. 사진 인식은 유료의 저렴한 모델(Gemini Flash 계열, GPT mini 계열 등)이 훨씬 정확합니다.

---

## 아이폰 (홈페이지에서 설치)

아이폰은 Apple 정책상 App Store·TestFlight 말고는 앱 파일(APK 같은 것)을 홈페이지에서 받아 설치할 수 없습니다. 그래서 아이폰은 **홈 화면에 추가하는 웹앱(PWA)** 방식으로 설치합니다. 같은 앱 코드(`www`)가 홈페이지의 `app/` 주소로 자동 배포됩니다.

1. 아이폰 **Safari**에서 홈페이지(`https://내아이디.github.io/workout-app/`)를 엽니다.
2. **아이폰에 설치**를 누르면 앱 화면이 열립니다.
3. 아래쪽 **공유** 버튼 → **홈 화면에 추가** → **추가**.
   - 공유 버튼이 안 보이면 주소창 옆 **···** → 공유
   - 목록에 없으면 아래로 내려 **더 보기** / **동작 편집**에서 찾기
   - 카카오톡 등 앱 안에서 열었으면 먼저 **Safari로 열기**
4. 홈 화면의 **운동기록** 아이콘으로 열면 전체 화면 앱처럼 동작합니다. 사진 올리기·AI 분석·대시보드 모두 안드로이드 앱과 같습니다.

**알아 둘 점**
- 기록은 그 아이폰 안(홈 화면 앱 저장소)에만 저장됩니다. 홈 화면 앱을 삭제하면 기록도 지워지니, 설정 → **CSV로 내보내기**로 가끔 백업하세요. 폰을 바꾸면 **백업 CSV 불러오기**로 옮길 수 있습니다.
- Safari 브라우저 탭에서 쓰는 것과 홈 화면 아이콘으로 쓰는 것은 저장소가 따로라서, **홈 화면 아이콘으로만** 쓰는 걸 권장합니다.
- 업데이트는 자동입니다. 파일을 고쳐 올리면 다음에 앱을 열 때 새 버전이 적용됩니다.

**진짜 iOS 앱(App Store/TestFlight)으로 배포하려면**
Apple Developer Program(연 $99) 가입이 필요합니다. 가입하면 GitHub Actions에서 iOS 앱을 서명·빌드해 TestFlight에 올리고, 홈페이지 버튼을 TestFlight 초대 링크로 바꿀 수 있습니다. (Mac 없이도 가능)

## 파일 구조

```
.github/workflows/build.yml   APK 빌드 → Releases 업로드 → 홈페이지 배포
site/                         다운로드 홈페이지 (index.html, img/) — 배포 시 www가 site/app/(아이폰 웹앱)으로 복사됨
www/                          앱 화면과 기능
  index.html, css/app.css
  js/app.js        채팅, 사진 첨부, 새 종목 질문, 확인 창, 설정
  js/ai.js         OpenAI/OpenRouter 호출 + 정형화/사진 분석 지시문
  js/db.js         폰 내부 SQLite
  js/dashboard.js  슬라이서 + 자동 차트
  js/metrics.js    detail_json → 숫자 지표
  js/charts.js     차트
  manifest.webmanifest, sw.js, icons/   아이폰 홈 화면 앱(PWA) 설정·오프라인 보관
resources/icon.png, apply_icon.py   앱 아이콘
resources/debug.keystore       앱 서명키 (업데이트 설치용, 지우지 말 것)
capacitor.config.json, package.json  앱 이름·패키지 설정
```
