# Project AIRI 분석 · 설치 · 활용 정리 (한국어)

> 이 저장소가 무엇인지, 어떻게 설치하고 쓰는지, 어떻게 수익화할 수 있는지,
> 그리고 PHP로 개발이 가능한지까지 정리한 문서입니다.

## 🔗 링크 모음

| 항목 | 주소 |
| --- | --- |
| **내 저장소 (fork)** | https://github.com/bmshin94/airi |
| **원본 저장소 (upstream)** | https://github.com/moeru-ai/airi |
| 웹에서 바로 써보기 | https://airi.moeru.ai |
| 설치 파일 (릴리즈) | https://github.com/moeru-ai/airi/releases/latest |
| 한국어 README | https://github.com/moeru-ai/airi/blob/main/docs/README.ko-KR.md |
| 기여 가이드 | https://github.com/moeru-ai/airi/blob/main/.github/CONTRIBUTING.md |
| 코드 자동 문서화 (DeepWiki) | https://deepwiki.com/moeru-ai/airi |
| 서브 프로젝트 조직 | https://github.com/proj-airi |
| Discord 커뮤니티 | https://discord.gg/TgQ3Cu2F7A |
| Live2D SDK 라이선스 (⚠️ 수익화 전 필독) | https://www.live2d.com/en/sdk/license/ |

---

## 1. 이 프로젝트는 뭔가요?

**한 줄 요약: "내 컴퓨터에서 사는 AI 캐릭터를 만드는 오픈소스 키트"**

- 공식 문구: *"Re-creating Neuro-sama, a soul container of AI waifu / virtual characters"*
- **Neuro-sama**라는 유명 AI 버튜버가 소스를 공개하지 않아서, 개발자들이 오픈소스로 다시 만든 프로젝트
- 현재 버전: `v0.12.0-beta.5` (아직 **베타** 단계)
- 라이선스: **MIT** (Copyright (c) 2024-PRESENT Neko Ayaka)

### 챗봇과 뭐가 다른가

ChatGPT는 텍스트 대화만 하지만, AIRI는 여기에 **몸**을 붙였습니다.

| 부위 | 하는 일 |
| --- | --- |
| 🧠 Brain | LLM으로 사고, 마인크래프트 · 팩토리오 · 체스 플레이, 디스코드 · 텔레그램 대화 |
| 👂 Ears | 마이크 음성인식(STT), 발화 감지 |
| 👄 Mouth | 음성합성(TTS) — ElevenLabs, Azure, OpenAI, 로컬 Kokoro 등 |
| 🕺 Body | Live2D / VRM 캐릭터, 자동 눈깜빡임 · 시선처리 · 립싱크 |
| 💾 Memory | 브라우저 내장 DB(DuckDB WASM), pgvector 기반 기억 |

---

## 2. 폴더 구조

**pnpm 모노레포** 구조이며, 로봇 조립에 비유하면 이해가 쉽습니다.

| 폴더 | 역할 |
| --- | --- |
| `apps/` | 🎬 **완성된 앱** (웹 / 데스크톱 / 모바일) |
| `packages/` | 🔩 **부품 창고** (약 51개 공용 패키지) |
| `server/` | 🏢 **백엔드** (API + 인증 + DB) |
| `plugins/` | 🧩 **기능 확장 플러그인** |
| `integrations/` | 🔌 **외부 플랫폼 연동** |
| `services/` | 🛠 독립 서비스 (computer-use MCP) |
| `engines/` | 🎮 Godot 게임엔진 실험 |
| `docs/` | 📖 문서 사이트 (한국어 README 포함) |

### `apps/` — 실제 제품

```
stage-web         브라우저 버전 (airi.moeru.ai 에서 도는 그것)
stage-tamagotchi  데스크톱 버전 (Electron, 바탕화면 캐릭터)
stage-pocket      모바일 버전 (Capacitor, iOS/Android)
ui-server-auth    로그인/인증 UI
component-calling 실시간 음성 통화 실험
```

> "stage(무대)"라는 이름은 **캐릭터가 서는 무대**라는 의미입니다.

### `packages/` — 주요 부품

- **렌더링**: `stage-ui`, `stage-ui-live2d`, `stage-ui-mmd`, `stage-ui-spine`, `stage-ui-three`, `stage-ui-tachie`
- **캐릭터 구동**: `model-driver-lipsync`(입모양), `model-driver-mediapipe`(웹캠 얼굴인식), `motion-driver-magic`
- **AI 코어**: `core-agent`(에이전트 오케스트레이션), `core-character`(감정·딜레이·TTS 파이프라인), `memory-pgvector`
- **DB**: `duckdb-wasm`, `drizzle-duckdb-wasm` (브라우저 안에서 도는 DB)
- **오디오**: `audio`, `pipelines-audio`, `testing-audio`
- **확장**: `plugin-sdk`, `plugin-protocol`, `server-sdk`
- **기타**: 폰트 4종, `i18n`(다국어), 게임패드 · 듀얼센스 입력

### `server/` — 백엔드

```
apps/api   리소스 API + DB 마이그레이션
apps/auth  Better Auth + OIDC 인증 서버
```

`docker-compose.yaml` 하나로 **PostgreSQL(pgvector) + Redis + Caddy**가 함께 뜹니다.

### `plugins/` — 플러그인

```
airi-plugin-web-extension    브라우저에서 보고 있는 내용을 캐릭터가 함께 봄
airi-plugin-claude-code      Claude Code 연동
airi-plugin-bilibili-laplace 빌리빌리 라이브 채팅 연동
airi-plugin-homeassistant    스마트홈 제어
airi-plugin-game-chess       체스
```

### `integrations/` — 외부 연동

```
discord-bot       디스코드 음성채널 참여
telegram-bot      텔레그램 대화
satori-bot        QQ/텔레그램/디스코드/Lark 통합 (자율 사고 루프 포함)
minecraft         마인크래프트 봇 (Mineflayer) ※ deprecation 안내 있음
twitter-services  트위터
vscode            VSCode 확장
```

### 기술 스택

- `TypeScript` 약 2,317개 파일 + `Vue` 653개 + `Godot(.gd)` 27개 + `Swift` 8개
- **Vue 3 + TypeScript + Electron + Capacitor**
- Rust 없음 (`Cargo.toml` 0개) — 전부 웹 기술 기반
- WebGPU / WebAssembly / Web Worker / WebSocket 적극 활용
- 백엔드: **Hono + Drizzle ORM + Better Auth + PostgreSQL + Redis**
- AI 에이전트 가이드 문서 다수: `AGENTS.md`(31KB), `.agents/`, `.cursor/`, `.gemini/`, `CLAUDE.md`

---

## 3. 설치 및 사용법

### 🅰️ 그냥 쓰기 (설치만, 5분)

**0단계 · 설치 없이 맛보기**

브라우저에서 https://airi.moeru.ai 접속

**1단계 · 앱 설치**

```powershell
# Windows
winget install MoeruAI.AIRI

# Windows (Scoop)
scoop bucket add airi https://github.com/moeru-ai/airi
scoop install airi/airi
```

```bash
# macOS
brew install --cask airi
```

- Linux 및 직접 다운로드: https://github.com/moeru-ai/airi/releases/latest
- 모바일: https://airi.moeru.ai 접속 후 **"홈 화면에 추가"** (PWA)

### 🅱️ 소스로 실행하기

**준비물**

| 항목 | 요구사항 |
| --- | --- |
| Node.js | **23 이상 필수** (`.tool-versions`는 `26.7.0` 지정) |
| pnpm | `11.24.0` (`packageManager` 필드에 고정) |

```bash
# Node 버전 확인
node -v          # v23 이상이어야 함

# pnpm 설치
corepack enable
corepack prepare pnpm@latest --activate
pnpm -v          # 11.x 확인
```

Linux에서 데스크톱 버전을 빌드할 때 추가로 필요한 패키지:

```bash
sudo apt install libssl-dev libglib2.0-dev libgtk-3-dev \
  libjavascriptcoregtk-4.1-dev libwebkit2gtk-4.1-dev
```

Windows는 Visual Studio 설치 시 **C++ 빌드 도구**와 **Windows SDK**를 체크해야 합니다.

**설치 & 실행**

```bash
pnpm i     # 부품 다운로드 (5~10분, 패키지가 51개라 오래 걸리는 게 정상)
pnpm dev   # 실행 → http://localhost:5173
```

**실행 스크립트 목록**

```bash
pnpm dev              # 🌐 브라우저 버전 (기본, 가장 가벼움)
pnpm dev:tamagotchi   # 💻 데스크톱 앱 (바탕화면 캐릭터)
pnpm dev:docs         # 📖 문서 사이트
pnpm dev:web:https    # 🔒 HTTPS 모드 (마이크 권한이 필요할 때)
pnpm dev:backend      # 🏢 백엔드 풀스택 (Docker 필요)
```

### ⚙️ 실행 후 설정하기

앱을 켜면 캐릭터는 보이지만 **AI 두뇌를 연결하기 전까지는 말하지 않습니다.**

**1단계 · Providers 연결 (필수)** — `설정 → Providers`

| 항목 | 역할 | 필수 여부 |
| --- | --- | --- |
| 💬 Chat | 생각하고 문장 생성 | ✅ **필수** |
| 🔊 Speech | 목소리 출력 (TTS) | 선택 |
| 🎤 Transcription | 음성 인식 (STT) | 선택 |
| 👁️ Vision | 이미지 인식 | 선택 |
| 🎨 Artistry | 이미지 생성 | 선택 |

**지원 서비스 (60종 이상)**

- 유료: OpenAI, Anthropic(Claude), Google Gemini, DeepSeek, xAI(Grok), Mistral, Groq, Moonshot, OpenRouter, Together AI, Perplexity, Azure, Amazon Bedrock 등
- 무료 · 로컬: **Ollama**, **LM Studio**
- 음성(TTS): ElevenLabs, OpenAI, Azure, **Kokoro(로컬 무료)**, **브라우저 기본 음성(무료)**, VOICEVOX

**추천 조합**

```
🥇 무료로 맛보기   Chat: Ollama(로컬)        + Speech: 브라우저 기본 음성
🥈 가성비          Chat: DeepSeek / Gemini   + Speech: Kokoro(로컬)
🥉 최고 품질       Chat: Claude / GPT        + Speech: ElevenLabs
```

API 키는 **브라우저 · 로컬에만 저장**되며 외부로 전송되지 않습니다.

**2~5단계**

| 단계 | 위치 | 내용 |
| --- | --- | --- |
| 2 | `설정 → Models` | 사용할 모델 선택 (예: `gpt-4o`, `claude-sonnet-4`, `deepseek-chat`) |
| 3 | `설정 → Scene` | 캐릭터 모델 지정 (Live2D `.zip` / VRM) |
| 4 | `설정 → AIRI Card` | 캐릭터 이름 · 성격 · 말투 설정 |
| 5 | `설정 → Memory` | 대화 기억 기능 활성화 |

무료 VRM 모델은 [VRoid Hub](https://hub.vroid.com/) 등에서 구할 수 있습니다.

### 🚨 자주 나오는 문제

| 증상 | 해결 방법 |
| --- | --- |
| `pnpm: command not found` | `corepack enable` 다시 실행 |
| 설치 중 에러 | Node 버전 확인 (**23 이상**) |
| `pnpm i` 실패 | `rm -rf node_modules && pnpm i` |
| 캐릭터가 말을 안 함 | Providers에 **Chat**이 연결되지 않음 |
| 소리가 안 남 | Providers **Speech** 연결 + 브라우저 음소거 확인 |
| 마이크 안 됨 | 권한 허용 + `pnpm dev:web:https`로 실행 |
| 화면 버벅임 | 브라우저 하드웨어 가속 켜기 (WebGPU 사용) |
| 포트 충돌 | 5173 포트를 쓰는 다른 프로그램 종료 |

### ✅ 체크리스트

```
□ 1. Node.js 23+ 설치 확인 (node -v)
□ 2. corepack enable 로 pnpm 준비
□ 3. pnpm i
□ 4. pnpm dev
□ 5. 브라우저에서 localhost 열기
□ 6. 설정 → Providers → Chat 에 API 키 입력
□ 7. 설정 → Models 에서 모델 선택
□ 8. 채팅창에 인사해보기
```

---

## 4. 수익화 아이디어

### ✅ 라이선스: MIT — 상업적 이용 가능

| 가능 여부 | 항목 |
| --- | --- |
| ✅ | 돈 받고 판매 |
| ✅ | 자유로운 수정 |
| ✅ | 수정본을 비공개 소스로 유지 (GPL과 다른 점) |
| ✅ | 회사에서 상업적 사용 |

**유일한 의무**: 저작권 표시 + 라이선스 원문 포함

### ⚠️ 수익화 전 반드시 확인할 3가지

**1. Live2D SDK는 별도 라이선스 (가장 중요)**

이 프로젝트는 `@proj-airi/unplugin-live2d-sdk`를 통해 **Live2D Cubism SDK**를 사용합니다.
이 SDK는 MIT가 아니라 **Live2D 사의 독자 라이선스**이며, 매출 규모에 따라 별도 계약이 필요할 수 있습니다.

- 반드시 확인: https://www.live2d.com/en/sdk/license/
- **회피 방법**: Live2D 대신 **VRM(3D)** 만 사용하면 이 문제가 없습니다.

**2. 캐릭터 IP는 별도 확보 필요**

- 타인의 애니 캐릭터, 실존 인물(연예인 포함)의 얼굴 · 목소리 무단 사용 금지
- 상업용은 직접 제작하거나 상업 라이선스를 구매한 모델을 사용

**3. AI API 비용은 변동비**

사용자가 많이 쓸수록 비용이 늘어납니다. 정액제 설계 시 헤비 유저 대응 필요.

> 🚫 참고: README에 **"공식 코인/토큰 없음"** 경고가 명시되어 있습니다. 코인 관련 수익화는 금지.

### 💰 아이디어 (현실성 순)

| 순위 | 아이디어 | 특징 |
| --- | --- | --- |
| 🥇 1 | **AI 버튜버 직접 운영** | 치지직/SOOP/유튜브 방송. 초기비용 ≈ 0, 국내 선점 여지 있음 |
| 🥈 2 | **설치 · 세팅 대행** | 건당 5~20만원. 원가 0, 가장 빠르게 현금화 가능 |
| 🥉 3 | **호스팅형 SaaS** | 수익 규모 최대. `server/`에 인증·API·DB 기반 이미 존재 |
| 4 | **에셋 · 콘텐츠 판매** | Live2D/VRM 모델, 캐릭터 카드 프리셋, 한국어 목소리 팩, UI 테마 |
| 5 | **유료 플러그인** | 게임 전적 연동, 공부 파트너, 업무 비서, **영어회화 파트너** |
| 6 | **B2B** | 매장 키오스크, 학원 회화 선생님, 사내 헬프데스크. 마진 최고 |
| 7 | **정보 · 커뮤니티** | 강의, 유료 커뮤니티, GitHub Sponsors |

> 참고: 원본 팀도 `ko-fi`, `Patreon`, `Open Collective`, `GitHub Sponsors`로 후원을 받고 있습니다.

### 🎯 핵심 전략

> **"소프트웨어를 팔지 말고, 소프트웨어로 만든 결과물이나 서비스를 팔 것."**

원본 프로젝트가 계속 무료로 배포되기 때문에 포크를 유료로 파는 전략은 통하지 않습니다.
반면 **시간, 노하우, 콘텐츠**는 복제되지 않습니다.

**단계별 로드맵**

```
1개월차   직접 세팅 → 캐릭터 완성 → 과정을 블로그/유튜브로 기록
2~3개월   ① 방송 시작  또는  ② 세팅 대행 시작 (크몽/숨고/커뮤니티)
4~6개월   반응 좋은 쪽 집중 → 플러그인 판매 또는 SaaS 준비
6개월+    B2B 영업 또는 SaaS 정식 런칭
```

**가장 빠른 시작**: 크몽/숨고에 "AI 버튜버 세팅 대행" 등록 — 원가 0, 리스크 0, 수요 검증까지 동시에.

### ✅ 수익화 전 체크리스트

```
□ Live2D 미사용, 또는 사용 시 라이선스 조건 확인
□ 캐릭터 IP 직접 제작 또는 상업 라이선스 확보
□ MIT 저작권 표시 유지 (LICENSE 파일 포함)
□ AI API 원가 계산 (1인당 월 예상 비용)
□ 개인정보 / 청소년 이용 정책 정리
□ 코인 · 토큰 관련 금지
```

---

## 5. PHP로 개발할 수 있나요?

### 결론

> **PHP로 100%는 불가능. 하지만 PHP로 백엔드를 만드는 것은 100% 가능.**

| 구역 | 실행 위치 | PHP 가능 여부 |
| --- | --- | --- |
| 🎨 캐릭터 화면 | 사용자 **브라우저** | ❌ 불가능 |
| 🏢 서버 로직 | **서버** | ✅ 완전 가능 |

### ❌ PHP로 불가능한 부분

PHP의 성능 문제가 아니라 **실행 위치**의 문제입니다. Python이든 Java든 동일합니다.

```
pixi-live2d-display        Live2D 캐릭터 렌더링 (브라우저 GPU)
three, @tresjs/core        3D VRM 캐릭터 (WebGL)
@huggingface/transformers  브라우저 내 AI 추론
onnxruntime-web            브라우저 내 모델 실행
WebAudio                   실시간 오디오 + 립싱크
```

- 캐릭터는 **초당 60회** 다시 그려져야 하므로 매번 서버 왕복이 불가능
- 마이크 · 스피커는 사용자 하드웨어이므로 서버 언어가 접근 불가
- 립싱크는 오디오 파형에 **실시간** 반응해야 함

### ✅ PHP로 완전히 대체 가능한 부분

현재 백엔드는 `Hono(TypeScript) + Drizzle ORM + Better Auth + PostgreSQL + Redis` 구성이며, 전부 PHP로 교체할 수 있습니다.

| 현재 (TypeScript) | PHP 대체 | 난이도 |
| --- | --- | --- |
| Hono (API 서버) | **Laravel** / Slim | ⭐ |
| Better Auth (인증) | **Laravel Breeze / Sanctum** | ⭐ |
| Drizzle (ORM) | **Eloquent ORM** | ⭐ |
| PostgreSQL + Redis | 그대로 사용 ✅ | ⭐ |
| LLM API 호출 | **Guzzle** | ⭐ |
| ws (웹소켓) | **Laravel Reverb** (PHP 구현체) | ⭐⭐ |

**PHP가 오히려 유리한 영역**

- 💳 결제 연동 — 국내 PG(토스, 아임포트) PHP 자료가 풍부
- 👨‍💼 관리자 페이지 — Laravel Filament
- 📊 회원 · 구독 관리 — Laravel Cashier
- 🖥️ 호스팅 — PHP 호스팅이 저렴하고 널리 보급

### 🏗️ 추천 아키텍처

```
┌─────────────────────────────────────┐
│  🌐 브라우저 (사용자 화면)             │
│  캐릭터 렌더링 · 음성 · 마이크          │
│  → JavaScript (기존 코드 그대로 사용)  │
└──────────────┬──────────────────────┘
               │  API 통신 (JSON)
┌──────────────▼──────────────────────┐
│  🐘 PHP / Laravel 서버               │
│  ✅ 회원가입 · 로그인                  │
│  ✅ 결제 · 구독 관리                   │
│  ✅ 대화 기록 저장                     │
│  ✅ LLM API 중계 (API 키 은닉)         │
│  ✅ 사용량 제한 · 과금 계산             │
│  ✅ 관리자 페이지                      │
│  ✅ 디스코드/텔레그램 봇 웹훅            │
└──────────────┬──────────────────────┘
               │
        PostgreSQL + Redis
```

프론트엔드는 기존 코드를 그대로 쓰고, **PHP 백엔드만 새로 작성**하면 됩니다.

### 💰 이 구조가 유리한 이유

SaaS에서 **실제로 매출이 발생하는 로직은 전부 백엔드**에 있습니다.

```
사용자 화면 (JS)  →  보이는 부분
PHP 백엔드       →  돈 받는 부분 💰
```

특히 API 키는 반드시 서버에 숨겨야 하므로 백엔드가 필수입니다.

### 🚀 시작 방법

**🅰️ 추천 — PHP 백엔드만 새로 만들기**

```
1. 기존 프론트엔드는 그대로 실행 (pnpm dev)
2. Laravel로 API 서버를 새로 구성
3. 프론트의 API 주소를 내 서버로 변경
```

첫 목표는 **LLM 중계 API 하나**만 만들어 보는 것:

```
POST /api/chat
  → 로그인 확인
  → 사용량 체크 (오늘 20회 초과 여부)
  → OpenAI API 호출
  → SSE로 스트리밍 응답
  → 대화 기록 DB 저장
```

이것만 완성해도 유료 서비스의 뼈대가 갖춰집니다.

**🅱️ PHP로 처음부터 만들기 (규모 축소 버전)**

```
Laravel + Blade + 최소한의 JS
→ 캐릭터를 표정별 정적 이미지로 전환
→ 브라우저 기본 TTS 사용 (JS 몇 줄)
→ 대화 처리는 전부 PHP
```

Live2D를 쓰지 않으므로 라이선스 문제도 없습니다.

### ⚠️ PHP 선택 시 주의점

| 항목 | 문제 | 해결 |
| --- | --- | --- |
| 실시간 스트리밍 | 응답이 한 번에 전달됨 | **SSE** 사용 (PHP 지원 양호) |
| 웹소켓 | 기본 PHP로는 불가 | **Laravel Reverb** |
| 음성인식(STT) | PHP로 직접 처리 불가 | API 호출 또는 브라우저에서 처리 |
| 벡터 검색(기억) | 별도 기능 필요 | **pgvector** 그대로 사용 (SQL) |
| 장시간 작업 | 타임아웃 | **Queue + Worker** |

### 🎯 정리

```
❌ PHP로 AIRI 전체를 다시 만들기       → 비추천 (불가능한 영역 존재)
✅ PHP로 AIRI의 백엔드·수익 로직 만들기 → 추천
```

---

## 6. 요약

| 질문 | 답 |
| --- | --- |
| 이게 뭔가요? | 내 컴퓨터에서 사는 AI 캐릭터를 만드는 오픈소스 (Neuro-sama 오픈소스판) |
| 언제 쓰나요? | AI 버튜버 방송, 데스크톱 마스코트, 디스코드/텔레그램 AI 멤버, 게임 동반자, 개인 비서 |
| 설치는? | `winget` / `brew` / 릴리즈 파일, 또는 `pnpm i && pnpm dev` |
| 꼭 필요한 설정은? | `설정 → Providers → Chat`에 LLM API 키 등록 |
| 수익화 가능한가요? | MIT라 가능. 단 Live2D SDK 라이선스와 캐릭터 IP 확인 필수 |
| 가장 빠른 수익화는? | 설치 · 세팅 대행 (원가 0) |
| PHP로 되나요? | 화면은 JS 필수, **백엔드는 PHP(Laravel)로 완전 대체 가능** |

---

*작성일: 2026-08-31 · 대상 저장소: https://github.com/bmshin94/airi (upstream: https://github.com/moeru-ai/airi)*
