# llmfit 전수조사 분석 및 활용·수익화 정리 (한국어)

> 이 문서는 `llmfit` 저장소를 파일 단위로 전수조사한 결과와, 설치·사용법,
> 정체(플러그인/스킬/MCP), API 토큰 필요 여부, AI 에이전트 활용성,
> React/PHP 구현 가능성, 유튜브 콘텐츠화, 수익화 아이디어를 정리한 문서입니다.

## 저장소 주소

| 구분 | 주소 |
|---|---|
| **이 저장소 (포크)** | https://github.com/bmshin94/llmfit |
| **원본 (업스트림)** | https://github.com/AlexsJones/llmfit |
| 릴리스 | https://github.com/AlexsJones/llmfit/releases |
| crates.io | https://crates.io/crates/llmfit |
| Docker 이미지 | `ghcr.io/alexsjones/llmfit` |
| 공식 설치 스크립트 | https://llmfit.axjns.dev/install.sh |

- 라이선스: **MIT** (Copyright (c) 2026 Alex Jones)
- 분석 시점 버전: **v1.1.15**
- 분석 기준 커밋: `d45b3f9`

### 자매 프로젝트

| 프로젝트 | 설명 |
|---|---|
| [llmserve](https://github.com/AlexsJones/llmserve) | 로컬 LLM 서빙용 TUI |
| [llama-panel](https://github.com/AlexsJones/llama-panel) | llama-server 관리 macOS 네이티브 앱 |
| [sympozium](https://github.com/sympozium-ai/sympozium/) | 쿠버네티스 에이전트 관리 |
| [llmfit-gui](https://github.com/raiyyan729-cloud/llmfit-gui) | Windows GUI (PowerShell + WinForms) |
| [llm-checker](https://github.com/Pavelevich/llm-checker) | 대안 도구 (Node.js, MoE 미지원) |

---

## 1. 한 줄 요약

> **"내 컴퓨터에서 어떤 오픈소스 LLM이 실제로 돌아갈까?"** 를 명령어 한 줄로
> 알려주는 **Rust 기반 터미널 도구**.

CPU / RAM / GPU / VRAM을 자동 스캔해 **모델 12,937개**와 대조하고,
품질·속도·적합도·컨텍스트 4축으로 점수를 내서 "실제로 잘 돌아갈 모델"을 추천한다.

llmfit은 **모델을 실행하지 않는다.** 진단·추천 도구이고, 실행은
Ollama / llama.cpp / LM Studio 등이 담당한다. (의사 vs 약국)

---

## 2. 저장소 구조 (파일 568개 / Rust 코드 약 5.7만 줄)

```
llmfit/
├── llmfit-core/    두뇌 — Rust 라이브러리 (모듈 18개)
│   ├── src/
│   │   ├── hardware.rs    5,883줄  하드웨어 탐지
│   │   ├── fit.rs         5,378줄  적합도/점수/양자화/속도
│   │   ├── providers.rs   7,682줄  런타임 연동 (Ollama 등 7종)
│   │   ├── models.rs      4,498줄  모델 카탈로그
│   │   ├── plan.rs        2,077줄  하드웨어 요구량 역산
│   │   ├── share.rs       1,675줄  벤치 결과 GitHub PR 자동 제출
│   │   ├── bench.rs       1,501줄  실측 벤치마크
│   │   ├── quality.rs     1,020줄  품질 점수
│   │   ├── update.rs      1,070줄  모델 DB 갱신
│   │   ├── benchmarks.rs  1,089줄  벤치 데이터 로딩/인덱싱
│   │   ├── hwprofile.rs     968줄  하드웨어 프로파일(schema v1)
│   │   ├── storage.rs       728줄  SSD 용량 계획
│   │   ├── concurrency.rs   496줄  동시성 분석
│   │   ├── analysis.rs      486줄  결과 조립 + 보정
│   │   ├── claim.rs         364줄  K8s DRA ResourceClaim 생성
│   │   ├── doctor.rs        278줄  진단 리포트
│   │   └── task_bench.rs / lib.rs
│   └── data/
│       ├── hf_models.json       11MB  모델 12,937개 (컴파일 시 바이너리에 내장)
│       ├── benchmark_cache.json 3.4MB
│       ├── benchmarks.yaml       78KB
│       ├── community/            GPU 43종 / 실측 JSON 354건
│       └── schema.json, baselines.json, use_case_benchmarks.json, docker_models.json
│
├── llmfit-tui/     실행 바이너리 `llmfit` (CLI + TUI + HTTP + MCP)
│   ├── tui_app.rs    6,081줄
│   ├── tui_ui.rs     5,961줄
│   ├── main.rs       4,410줄  (clap 플래그 파싱 + 인터페이스 선택)
│   ├── serve_api.rs  1,419줄  Axum REST API
│   ├── display.rs    1,341줄
│   ├── mcp_server.rs   630줄  ★ MCP 서버 (도구 6개)
│   └── theme.rs / serve_shared.rs / tui_events.rs / events.rs 등
│
├── llmfit-web/     React 18 + Vite 5 대시보드 (파일 28개, i18n: en/zh-CN)
├── llmfit-desktop/ Tauri 데스크톱 앱 (v0.4.8)
├── llmfit-python/  pip/uv 설치용 파이썬 래퍼 (바이너리 동봉)
├── skills/llmfit-advisor/SKILL.md   ★ OpenClaw 에이전트용 스킬
├── scripts/        HF 크롤러(134KB) + 검증 스크립트 10개
├── docs/           가이드 9개 (tui, cli, benchmarking, providers, how-it-works ...)
├── .github/workflows/   CI, 릴리스, 매주 월 02:00 UTC 모델 DB 자동 갱신
└── README.md / README.zh.md / README.ja.md   (영·중·일 — 한국어 없음)
```

---

## 3. 동작 원리 4단계

### ① 하드웨어 탐지 (`hardware.rs`)

| 대상 | 방법 |
|---|---|
| NVIDIA | `nvidia-smi` (멀티 GPU는 VRAM 합산, 실패 시 GPU명으로 추정) |
| AMD | `rocm-smi`, sysfs |
| Intel Arc | 외장 sysfs / 내장 `lspci` |
| Apple Silicon | `system_profiler` + Metal `recommendedMaxWorkingSetSize` |
| Ascend NPU | `npu-smi` |
| CPU / RAM | `sysinfo` 크레이트 |

백엔드(CUDA / Metal / ROCm / SYCL / CPU ARM / CPU x86 / Ascend)도 자동 판별.

### ② 모델 12,937개 대조

- 모델 DB는 `include_str!` 로 **바이너리에 컴파일 타임 내장** → **오프라인 동작**
- **동적 양자화**: Q8_0(고품질) → Q2_K(초압축) 순으로 내려가며 "내 메모리에 들어가는
  가장 고품질 설정"을 자동 선택. 안 들어가면 컨텍스트를 절반으로 줄여 재시도
- **MoE 지원**: Mixtral 8x7B = 총 46.7B지만 토큰당 ~12.9B만 활성 →
  VRAM 23.9GB → **6.6GB** 로 보정

### ③ 4축 채점 (각 0~100)

| 축 | 측정 |
|---|---|
| **Quality** | 파라미터 수, 패밀리 명성, 양자화 손실, 용도 적합도 |
| **Speed** | 예상 tok/s |
| **Fit** | 메모리 활용 효율 (스윗스팟 50~80%) |
| **Context** | 컨텍스트 창 크기 |

용도별 가중치 상이 — 추론(reasoning) Quality 0.55 / 채팅(chat) Speed 0.35.
용도 적합도는 `use_case_benchmarks.json` 큐레이션 표 기반(공개 리더보드 집계).

**속도 추정 공식 (디코드, 메모리 대역폭 바운드):**
```
tok/s ≈ (대역폭 GB/s ÷ 모델크기 GB) × 0.55
```
GPU 약 80종의 실제 대역폭 표 내장. 미등록 GPU는 백엔드 상수 fallback:

| 백엔드 | 상수 |
|---|---|
| CUDA | 220 |
| Metal | 160 |
| ROCm | 180 |
| SYCL | 100 |
| CPU (ARM) | 90 |
| CPU (x86) | 70 |
| NPU (Ascend) | 390 |

**프롬프트 처리(prefill / TTFT)** 는 연산 바운드 — 프롬프트 토큰당 약
`2 × 활성파라미터` FLOPs. GPU의 fp16 처리량을 알 때만 보고하고,
모르면 `0.0`이 아니라 `null` 을 반환한다(= "측정 안 함"과 "무한히 느림"을 구분).

### ④ 판정

| 메모리 사용률 | 판정 |
|---|---|
| ≤ 60% | **Perfect** (여유롭게 돌아감) |
| ≤ 85% | **Good** (잘 돌아감) |
| ≤ 98% | **Marginal** (간신히, 느릴 수 있음) |
| > 98% | **Too Tight** (불가 — 추천 안 함) |

- 98%에서 끊는 이유: 마지막 1%까지 채우면 할당자 여유/단편화 때문에 실제로 안 올라감
- 실행 모드: **GPU** / **MoE 오프로드** / **CPU+GPU** / **CPU**
- MoE 오프로드·CPU+GPU·CPU 는 Perfect 대신 **Good 으로 상한** (Perfect = "여유 + GPU 실행")

---

## 4. 가장 독특한 설계: 추정값 → 실측값 커뮤니티 루프

```
llmfit bench               내 PC에서 실제 측정
   → 로컬 저장 (~/.local/share/llmfit/benchmarks/pending/)
   → 내 화면의 추정치가 실측치로 교체
   → 1B 이상 dense 모델 측정값은 다른 모든 모델의 추정도 보정(calibrated)

llmfit bench --share       GitHub 자동 포크 + 커밋 + PR 생성
   → gh CLI 불필요, 제3자 계정 불필요
   → GitHub device flow (브라우저 1회 승인) 또는 GITHUB_TOKEN
   → 이미 열린 벤치 PR이 있으면 거기에 append
   → 멱등(idempotent): 재시도해도 중복 제출 안 됨

머지 → 다음 릴리스에 내장 → 같은 하드웨어 쓰는 전 세계 사용자가
       설치 즉시 "실측 ✓" 숫자를 봄
```

### 신뢰도 등급 (모든 수치에 라벨이 붙는다)

| 등급 | 의미 |
|---|---|
| `measured_local` | 내가 이 PC에서 직접 측정 (최우선) |
| `measured_community` | 동일 하드웨어 사용자가 측정 |
| `calibrated` | 공식 + 이 하드웨어 보정계수 |
| `estimated` | 순수 공식 추정 |
| `unsupported` | 추정 불가 (llmfit이 모델링 못 하는 런타임) |

신뢰 순서: **내 측정 > 동일 하드웨어 커뮤니티 > localmaxxing 중앙값 > 공식 추정**

현재 커뮤니티 데이터: **GPU 43종, 실측 354건**
(RTX 5090 Laptop, M5 Max, NVIDIA GB10(DGX Spark), Radeon AI PRO R9700,
Intel Iris Xe/Arc 내장, GTX 1050 Ti 등 구형·내장 GPU까지 포함)

> AI 도구에서 "이 숫자가 추측인지 실측인지"를 구분해 보여주는 것은 드문 정직함이다.

---

## 5. 인터페이스 5종 (같은 코어, 다른 껍데기)

| # | 인터페이스 | 실행 | 대상 |
|---|---|---|---|
| 1 | **TUI** (기본) | `llmfit` | 사람 |
| 2 | **CLI** | `llmfit recommend --json` | 스크립트·자동화 |
| 3 | **웹 대시보드** | `llmfit serve` → :8787 | 브라우저 (React 18) |
| 4 | **REST API** | `GET /api/v1/models` | 클러스터 스케줄러 |
| 5 | **MCP 서버** | `llmfit serve --mcp` | **AI 에이전트** |

추가: Tauri 데스크톱 앱, Docker 멀티아키 이미지, NATS 이벤트 발행,
Unix 도메인 소켓(`--unix-socket`, mode 0660, K8s hostNetwork 사이드카용)

### TUI 단축키

| 키 | 기능 |
|---|---|
| `↑` `↓` / `k` `j` | 목록 이동 |
| `/` | 검색 (이름/제작사/양자화) |
| `h` | 도움말 |
| `p` | 플랜 모드 |
| `S` | **하드웨어 시뮬레이션** ★ |
| `b` | 커뮤니티 리더보드 (설치+실행 중이면 벤치 먼저 제안) |
| `I` | 실시간 추론 벤치 (TTFT/TPS/총지연) |
| `D` | 다운로드 매니저 |
| `A` | 고급 설정 (효율계수 0.55, 대역폭 override 등) |
| `Esc` | 뒤로 / 검색 해제 (벤치 중에는 백그라운드 전환) |

### 연동 런타임 7종
Ollama · llama.cpp · MLX(Apple) · Docker Model Runner · LM Studio · vLLM · RamaLama
(원격 인스턴스 지원. vLLM/Ferrum은 `/v1/models` 의 `owned_by` 로 구분,
llama-server는 8080 포트 `/props` 로 자동 탐지)

---

## 6. 설치 및 사용법

### 설치

```sh
# Windows
scoop install llmfit

# macOS
brew install AlexsJones/llmfit/llmfit    # 권장 (프리빌트)
brew install llmfit                       # homebrew-core
port install llmfit                       # MacPorts

# Linux / macOS 스크립트
curl -fsSL https://llmfit.axjns.dev/install.sh | sh
curl -fsSL https://llmfit.axjns.dev/install.sh | sh -s -- --local   # sudo 없이

# Python
uv tool install -U llmfit
uvx llmfit          # 설치 없이 실행
pip install llmfit

# Docker
docker run -it --rm ghcr.io/alexsjones/llmfit --tui

# 소스 빌드
git clone https://github.com/bmshin94/llmfit.git
cd llmfit && cargo build --release      # → target/release/llmfit
```

### 주요 명령어

```sh
llmfit                                # TUI (기본)
llmfit --cli                          # 클래식 표 출력
llmfit system                         # 내 사양
llmfit doctor                         # 진단 리포트 (버그 신고용)
llmfit list                           # 전체 모델
llmfit search "llama 8b"              # 검색
llmfit info "Mistral-7B"              # 상세 + 추정 근거 + 검증 명령
llmfit fit --perfect -n 5             # Perfect 등급만 5개
llmfit recommend --json --limit 5     # JSON 상위 5개
llmfit recommend --use-case coding --limit 3
llmfit recommend --force-runtime llamacpp
llmfit plan "Qwen/Qwen3-4B" --context 8192 --target-tps 25 --json
llmfit storage --keep 3 --selection largest --json
llmfit bench                          # 실측
llmfit bench --all --share            # 측정 + PR 제출
llmfit bench --all --share --dry-run  # 네트워크 미접속 미리보기
llmfit bench --all --share --yes      # 확인 프롬프트 생략(자동화)
llmfit serve --host 0.0.0.0 --port 8787
llmfit serve --mcp                    # MCP 서버
```

### 하드웨어 오버라이드 (GPU 구매 검토용)

```sh
llmfit --memory 24G --ram 64G --cpu-cores 16 --max-context 8192 recommend
```

### Docker Compose

```yaml
services:
  llmfit:
    image: ghcr.io/alexsjones/llmfit:latest
    restart: unless-stopped
    command: ["serve", "--host", "0.0.0.0", "--port", "8787"]
    ports: ["8787:8787"]
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8787/health"]
      interval: 15s
```

> 기여 전 필수: `cargo fmt` — README에 "CI 실패 대부분이 포맷 문제"라고 명시됨.

---

## 7. 플러그인? 스킬? MCP?

> **본체는 "독립 실행 바이너리"이고, 스킬 껍데기와 MCP 껍데기가 둘 다 붙어 있다.**

| 질문 | 답 | 근거 |
|---|---|---|
| 독립 프로그램? | **네 — 이게 본질** | Rust 단일 바이너리 `llmfit` |
| 스킬? | **네 — 제공됨** | `skills/llmfit-advisor/SKILL.md` (OpenClaw 포맷) |
| MCP? | **네 — 내장** | `llmfit-tui/src/mcp_server.rs` 630줄 |
| 플러그인? | **아니요** | 특정 앱 플러그인 규격 없음 |

### 스킬 (OpenClaw)

```sh
./scripts/install-openclaw-skill.sh
# 또는
cp -r skills/llmfit-advisor ~/.openclaw/skills/
```

트리거 문구: "what local models can I run?", "which LLMs fit my hardware?",
"recommend a local model", "can I run Llama 70B locally?",
"help me pick a local model for coding" 등.

스킬 본문에는 에이전트 워크플로가 명시돼 있다:
하드웨어 감지 → 추천 → HuggingFace 이름을 Ollama 태그로 매핑(표 10개)
→ `openclaw.json` 의 `models.providers.ollama.models` 자동 설정.

> 주의: OpenClaw 포맷이다. Claude Code 스킬로 쓰려면 frontmatter의
> `metadata.openclaw` 를 바꿔야 한다. 본문(명령어, 출력 해석법)은 거의 그대로
> 재사용 가능 — **개조 난이도 낮음.**

### MCP 서버 (가장 표준적, 권장)

```json
{
  "mcpServers": {
    "llmfit": { "command": "llmfit", "args": ["serve", "--mcp"] }
  }
}
```

| 도구 | 설명 | 파라미터 |
|---|---|---|
| `get_system_specs` | 노드 하드웨어 (RAM/GPU/CPU) | 없음 |
| `recommend_models` | 이 하드웨어용 추천 | `limit?` `use_case?` `min_fit?` `runtime?` `license?` `sort?` |
| `search_models` | 자유 텍스트 검색 | `query` `limit?` |
| `plan_hardware` | 모델별 하드웨어 요구량 | `model` `context?` `quant?` `target_tps?` |
| `get_runtimes` | 설치된 런타임 확인 | 없음 |
| `get_installed_models` | 로컬 런타임의 모델 목록 | 없음 |

### REST API

| 엔드포인트 | 용도 |
|---|---|
| `GET /` | 웹 대시보드 |
| `GET /health` | 생존 확인 |
| `GET /api/v1/system` | 하드웨어 |
| `GET /api/v1/models` | 전체 모델 + 적합도 |
| `GET /api/v1/models/top` | 상위 추천 |
| `GET /api/v1/models/{name}` | 단일 모델 |
| `POST /api/v1/plan` | 플랜 계산 (`disk_size_gb` 포함) |

API 프리픽스는 `v1`. 장기 클라이언트는 `/api/v1/...` 로 고정하고
`scripts/test_api.py` 로 검증 권장.

---

## 8. API 토큰 필요한가?

### 결론: 기본 기능은 **토큰 0개. 인터넷도 불필요.**

```
llmfit / recommend / fit / plan / storage / doctor / serve / serve --mcp / bench
→ 전부 토큰 불필요
```

### 선택적으로만 필요한 4가지

| 토큰 | 언제 | 필수? |
|---|---|---|
| `GITHUB_TOKEN` / `GH_TOKEN` | `bench --share` 로 PR 제출 | ❌ device flow로 대체 가능 |
| `LMSTUDIO_API_KEY` | LM Studio가 "Require API Key" 켜짐 | ❌ 그 설정 쓸 때만 |
| `LOCALMAXXING_API_KEY` | localmaxxing.com 보조 데이터 완전 접근 | ❌ 없어도 작동 |
| crates.io 토큰 | 메인테이너 배포용 | ❌ 사용자 무관 |

### GitHub 토큰은 안 만들어도 된다

`llmfit bench --share` → 브라우저에서 코드 1회 승인(**GitHub device flow**,
`gh auth login` 과 동일 방식) → 토큰이 `~/.config/llmfit/` 에 캐시.

- OAuth App client id는 바이너리에 내장 (device flow는 client secret 불필요 → 설계상 안전)
- `LLMFIT_GH_CLIENT_ID` 로 override 가능. 빈 문자열이면 대화형 로그인 비활성
- `--share` 는 **벤치 시작 전에** 인증을 먼저 검증 → 10분 측정 후 토큰 에러 참사 방지
- `--dry-run` 은 네트워크를 전혀 건드리지 않음

### 중요

> llmfit은 **OpenAI / Anthropic API 토큰이 전혀 필요 없다.**
> AI를 호출하지 않고, "내 사양 숫자 + 모델 크기 숫자"로 수식 계산만 한다.
> **AI API 비용 0원.**

### 프라이버시

- 기본 동작은 네트워크 미사용 (모델 DB가 바이너리 내장)
- 외부 통신은 사용자가 명시적으로 해당 기능을 쓸 때만:
  모델 다운로드 / 런타임 조회 / 커뮤니티 리더보드 / 벤치 공유
- README에 "사용자가 요청하지 않는 한 어떤 정보도 외부로 전송하지 않음" 명시
- MIT 오픈소스이므로 코드로 직접 검증 가능

---

## 9. AI 에이전트 구축에 도움이 되는가 → 매우 큼

### 해결해주는 문제

```
로컬 LLM 에이전트의 첫 번째 난관: "어떤 모델을 쓰게 할까?"
  ├── 하드코딩 → 다른 PC에서 안 돌아감 (배포 불가)
  ├── 제일 큰 것 → 메모리 터짐
  └── 제일 작은 것 → 성능 부족
```

llmfit은 이것을 **런타임에 자동 해결**한다.
에이전트가 **자기가 올라간 기계를 스스로 파악해 모델을 고른다.**

### 활용법 4가지

**(a) MCP 연동 — 5분**

```
사용자: "내 컴퓨터에서 코딩 도와줄 AI 세팅해줘"
에이전트:
  → get_system_specs        (RTX 4070, VRAM 12GB)
  → get_runtimes            (Ollama 설치됨)
  → recommend_models(use_case="coding", min_fit="good")
  → get_installed_models    (이미 받은 모델 확인)
  → "Qwen2.5-Coder-7B Q5_K_M 추천. 예상 42 tok/s. ollama pull 할까요?"
```

**(b) JSON 파이프라인 — 가장 가볍다**

```sh
MODEL=$(llmfit recommend --json --use-case coding --limit 1 | jq -r '.models[0].name')
```
에이전트 부팅 스크립트 3줄로 모델 자동 선택 완료. MCP 서버도 불필요.

**(c) 멀티 노드 / 클러스터**

```sh
# 각 노드에 사이드카
llmfit serve --host 0.0.0.0 --port 8787

# 중앙 스케줄러
for node in worker-1 worker-2 worker-3; do
  curl -s http://$node:8787/api/v1/models/top
done
```

NATS 이벤트 발행 (`cargo build --features nats`):

```sh
llmfit serve --send-events --nats-url nats://localhost:4222
nats sub 'llmfit.>'
```

| 주제 | 발행 시점 |
|---|---|
| `llmfit.system.{host}` | 시작 + 60초마다 |
| `llmfit.fit.{host}` | 적합도 분석 후 |
| `llmfit.plan.{host}` | 플랜 계산 후 |
| `llmfit.runtimes.{host}` | 시작 + 조회 시 |
| `llmfit.installed.{host}` | 시작 + 조회 시 |

**(d) 쿠버네티스 DRA** — `claim.rs` 가 `ResourceClaim` /
`ResourceClaimTemplate` 매니페스트를 생성. K8s에서 LLM 워크로드를
돌릴 때 "이 모델에 필요한 GPU 리소스 클레임"을 YAML로 뽑아준다.

### 참고할 설계 패턴

| 배울 점 | 위치 |
|---|---|
| 같은 코어 → 5개 인터페이스 분리 | `llmfit-core` ↔ `llmfit-tui` |
| 신뢰도 등급을 출력에 명시 | `analysis.rs`, `benchmarks.rs` |
| **추정 근거(`estimate_basis`)를 함께 반환** → 에이전트가 검증·인용 가능 | `fit.rs` |
| 에이전트가 자기 환경을 자기가 파악 | `mcp_server.rs` |
| 데이터 바이너리 내장으로 오프라인 동작 | `build.rs` + `include_str!` |
| `null` 과 `0.0` 을 구분하는 정직한 API | prefill/TTFT 처리 |

### 한계 (솔직하게)

| 한계 | 설명 |
|---|---|
| 모델 **실행** 엔진이 아님 | Ollama/vLLM 등 필요 |
| 에이전트 프레임워크 아님 | LangChain/CrewAI 대체 아님 — **부품** |
| Python 네이티브 API 없음 | `llmfit-python` 은 바이너리 래퍼. `subprocess` + JSON 사용 |
| 모델 DB 갱신 = 재설치 | 바이너리 내장이라 `brew upgrade llmfit` 필요 |

→ 포지션은 **"에이전트의 하드웨어 인식 센서"**. 두뇌가 아니다.

---

## 10. React / PHP 로 만들 수 있는가

### React → 이미 만들어져 있다

`llmfit-web/` = React 18.2 + Vite 5.4 + Vitest 2.1

```
llmfit-web/src/
├── App.jsx
├── api.js                  REST 호출 레이어
├── components/             ModelTable, SystemPanel, DetailPanel,
│                           ComparePanel, FilterBar, Header
├── contexts/               ModelContext, FilterContext, I18nContext
├── hooks/                  useModels.js, useSystem.js
├── i18n/locales/           en.js, zh-CN.js    ← ko.js 추가 시 한국어판
├── themes.css / styles.css
└── *.test.jsx              테스트 포함
```

```sh
cd llmfit-web
npm ci && npm run dev      # 개발
npm run build              # → dist/ (llmfit-tui 빌드 시 바이너리에 임베드)
npm test                   # Vitest
```

아키텍처가 완전 분리:
```
[React 프론트] --HTTP--> [llmfit serve REST API] --> [Rust 코어]
```
→ 프론트를 Next.js / Vue / Svelte 로 바꿔도 **백엔드 수정 불필요**.

**즉시 가능한 작업 (난이도 ★☆☆☆☆):**
`src/i18n/locales/ko.js` 추가 → `I18nContext` 등록 → 한국어 llmfit.
en/zh-CN 구조가 이미 있으므로 **업스트림 PR 머지 가능성 매우 높음.**

### PHP → 가능 (코어 재작성 불필요, 감싸기만)

**방법 1 — CLI 호출**
```php
<?php
exec('llmfit recommend --json --use-case coding --limit 5', $out);
$data = json_decode(implode('', $out), true);
foreach ($data['models'] as $m) {
    printf("%-45s %5.1fB  %s  %s  %.1f tok/s\n",
        $m['name'], $m['params_b'], $m['best_quant'],
        $m['fit_level'], $m['estimated_tps']);
}
```

**방법 2 — REST 프록시 (서버 환경 권장)**
```php
<?php
$json = file_get_contents('http://127.0.0.1:8787/api/v1/models/top');
$models = json_decode($json, true);
```

유닉스 소켓(네트워크 미경유):
```php
$ch = curl_init('http://localhost/api/v1/system');
curl_setopt($ch, CURLOPT_UNIX_SOCKET_PATH, '/run/llmfit/llmfit.sock');
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
$res = curl_exec($ch);
```

**방법 3 — Laravel 패키지**
```php
LlmFit::system();
LlmFit::recommend(useCase: 'coding', limit: 5);
LlmFit::plan('Qwen/Qwen3-4B', context: 8192);
```
Packagist 등록 시 **PHP/Laravel 생태계 최초의 로컬 LLM 사양 진단 패키지**.

### PHP 의 중대한 제약

> **웹 브라우저에서는 "방문자의" PC 사양을 알 수 없다.** (브라우저 샌드박스)

```
❌ 불가: 방문자가 버튼 누르면 그 사람 PC 사양 자동 스캔
         (nvidia-smi 실행 불가 — 원천적으로 불가능)

✅ 가능 1: 사용자가 사양을 폼에 입력 (RAM/VRAM 드롭다운)
           → llmfit --memory 24G --ram 64G recommend    ★ 가장 현실적
✅ 가능 2: 서버 자기 사양 진단 (GPU 서버 관리 대시보드)
✅ 가능 3: 사용자가 `llmfit doctor` 출력을 붙여넣기 → 파싱 분석
```

> WebGPU로 GPU 정보를 일부 얻을 수 있으나 VRAM 총량은 안 나온다.
> 정확한 진단은 네이티브 실행만 가능 → **입력 폼 방식이 정답.**

---

## 11. 유튜브 강의 제작 가능성 → 매우 적합

### 왜 좋은 소재인가

| 이유 | 설명 |
|---|---|
| 검색 수요 큼 | "로컬 LLM", "내 PC AI", "Ollama 설치", "AI용 GPU 추천" |
| 즉각적 비주얼 | TUI 점수 표 + 4색 신호등 → 썸네일 잘 나옴 |
| 즉시 체험 | 시청자가 명령어 1줄 복붙으로 재현 → 이탈률 낮음 |
| 명확한 Pain Point | "40GB 받고 메모리 에러" = 모든 입문자의 실제 경험 |
| 에셋 제공 | `assets/demo.gif`, 벤치마크 가이드 스크린샷 4장 |

### 제작 리스크

| 항목 | 상태 |
|---|---|
| 라이선스 | **MIT** — 상업적 이용·영상화·수익화 전부 자유 |
| 저작자 표기 | 법적 필수 아니나 매너상 필수 (설명란에 GitHub 링크) |
| 에셋 사용 | demo.gif, 아이콘, 스크린샷 모두 MIT 범위 |
| 설치 난이도 | 낮음 (`scoop install` / `brew install` 한 줄) |

### 5부작 기획안

| EP | 제목 | 길이 | 타겟 |
|---|---|---|---|
| 1 | AI 모델 받기 전에 이거 먼저 돌려보세요 | 8분 | 입문 후킹 |
| 2 | **GPU 사기 전에 꼭 보세요 — 시뮬레이션으로 300만원 아끼기** | 12분 | 조회수·제휴 1등 후보 |
| 3 | 추정값 믿지 마세요 — 실측하고 오픈소스에 기여하기 | 15분 | 개발자 유입 |
| 4 | AI 에이전트가 스스로 모델 고르게 만들기 (MCP) | 18분 | 개발자, CPC 높음 |
| 5 | React로 나만의 AI 사양 진단 사이트 만들기 | 25분 | 실습, 완주율 높음 |

**EP1 구성 예시**
```
0:00  훅: "40GB 받고 메모리 부족 에러 나본 분?" (실제 에러 화면)
0:30  llmfit 소개 — 3초 해결
1:00  설치 (Windows/Mac)
2:00  실행 → 신호등 4색 설명
4:00  Perfect 모델 골라서 Ollama로 실제 실행
6:30  일부러 Too Tight 모델 시도 → 왜 안 되는지
7:30  정리
```

### 추가 아이디어 (데이터로 뒷받침 가능)

- "노트북 내장그래픽으로 AI 돌릴 수 있나?" → Intel Iris Xe, Radeon 780M 실측 존재
- "10년 전 GTX 1050 Ti로 AI 돌려봤습니다" → 1050 Ti 실측 존재
- "M1 vs M5, AI 성능 몇 배?" → M1 Max ~ M5 Max 실측 전부 존재
- "맥북 64GB vs 128GB, AI용으로 의미 있나?" → 시뮬레이션 검증
- "사양 진단 사이트 만들어 수익화하기" → 강의 상품화

### 핵심 승부처: 한국어 콘텐츠 공백

- README: 영어 / 중국어 / 일본어 — **한국어 없음**
- 웹 i18n: `en.js`, `zh-CN.js` — **ko.js 없음**
- 한국어 유튜브 강의 거의 없음

→ **선점 기회.** 영상 찍으면서 `ko.js` PR 제출 시
"이 도구 한국어판 만든 사람" 권위까지 확보.

---

## 12. 수익화 아이디어

> 전제: MIT 라이선스이므로 상업적 이용·수정·재배포·유료 판매 전부 합법
> (저작권 고지 유지 조건).
>
> 솔직한 전제: llmfit 자체 판매는 불가능에 가깝다(무료 오픈소스).
> 돈이 되는 지점은 **① 접근성 ② 한국어 ③ 구매 결정 지원 ④ 기업 서비스**.

### 아이디어 1. "AI 돌아가나요?" 사양 진단 사이트 (제휴 수익) ★ 최우선

```
[사용자]  사양 드롭다운 선택 (RAM / GPU / VRAM) 또는 GPU명 검색
   ↓
[백엔드]  llmfit --memory X --ram Y recommend --json
   ↓
[결과]    🟢 돌아가는 모델 23개 (한국어 설명 + 예상 속도)
          🔴 안 되는 모델 (이유 설명)
          💡 "VRAM 8GB 더 있으면 이 모델들도 가능"
   ↓
[수익]    ① GPU 제휴 링크 (쿠팡파트너스 / 다나와 / 아마존)
          ② 구글 애드센스
          ③ 전체 진단 리포트 PDF 유료 (3,900원)
```

| 요소 | 설명 |
|---|---|
| 구매 의도 | "AI용 GPU 찾는 사람"은 곧 GPU를 산다 → 전환율 최상위권 |
| 객단가 | GPU 50~300만 원. 수수료 3% = 건당 1.5~9만 원 |
| 재방문 | 신모델 출시마다 ("RTX 5080 추가됨") |
| SEO | "RTX 4070 LLM", "맥북 M4 AI 성능" 등 롱테일 수백 개 자동 생성 |

- 필요: PHP 또는 Next.js + llmfit 바이너리 1개. 서버 1대. **AI API 비용 0원**
- 예상: 월 1만 방문 × 전환 0.5% × 건당 3만 원 ≈ **월 150만 원** + 애드센스
- MVP **2주** 가능

### 아이디어 2. 한국어 유튜브 + 온라인 강의 (퍼널) ★ 최우선

```
[무료] 유튜브 5부작 → 애드센스 + 구독자 + 제휴 링크
   ↓
[중간] 유료 강의 "로컬 AI 완전정복" (인프런/클래스101) 49,000~99,000원
   ↓
[고가] 1:1 컨설팅 / 기업 교육 — 시간당 10~30만 원
```

| 채널 | 예상 |
|---|---|
| 애드센스 | 10만 뷰당 20~50만 원 (IT CPM 높음) |
| 쿠팡파트너스 (GPU) | 객단가 큼 |
| 유료 강의 | 100명 × 49,000원 = **490만 원** (1회 제작, 반복 판매) |
| 기업 교육 | 회당 100~300만 원 |
| 멤버십 | 월 9,900원 × 100명 = 월 99만 원 |

**핵심 레버리지:** 영상 제작과 동시에 `ko.js` 한국어 PR을 업스트림에 제출
→ 권위 확보 + 그 과정이 EP3 콘텐츠 + GitHub 컨트리뷰터 이력 = 강의 신뢰도

### 아이디어 3. 기업용 로컬 AI 도입 컨설팅 (최고 단가)

```
기업: "ChatGPT 쓰고 싶은데 사내 데이터 유출은 안 됨" → 로컬 LLM이 유일한 답
   → GPU 서버 구성법을 아는 사람이 없음 → 벤더 말만 듣고 A100 과잉 구매
```

| 서비스 | 가격 |
|---|---|
| 사양 진단 리포트 (`doctor` + `storage` + 시뮬레이션) | 100~300만 원 |
| PoC 구축 (모델 선정 → Ollama/vLLM → 사내 챗봇) | 500~2,000만 원 |
| 클러스터 설계 (`serve` 노드 + K8s DRA ResourceClaim) | 2,000만 원~ |
| 월 유지보수 | 100~500만 원/월 |

**세일즈 포인트:** "근거 있는 숫자"를 들고 간다. `estimate_basis` 와 검증
명령어, 커뮤니티 실측 354건이 뒷받침. 벤더가 "A100 8장" 할 때
**"4090 2장으로 충분합니다, 여기 실측 데이터입니다"** 가 가능.
수억 원이 걸린 신뢰.

진입 경로: 아이디어 2(유튜브) → 권위 → 문의 유입. **영업 비용 0원.**

### 아이디어 4. SaaS: GPU 서버 플릿 모니터링 (B2B 구독)

```
각 노드에 llmfit serve 사이드카 → NATS/REST 중앙 수집 → 대시보드
  - 노드별 돌릴 수 있는 모델
  - 유휴 VRAM / 과잉 프로비저닝 경고
  - "worker-3에 7B 하나 더 올릴 수 있음"
```

- 요금: 노드당 월 $20~50 / 엔터프라이즈 $500+
- 타겟: GPU 서버 5대 이상 운영하는 AI 스타트업·연구실·클라우드 재판매사
- 강점: 코어 로직 불필요(llmfit이 다 함). 수집 + 시각화 + 알림만 만들면 됨
- 난이도: ★★★★☆ (B2B 영업)

### 아이디어 5. 한국어 특화 Fork / 제품화

- **A. 데스크톱 앱** — `llmfit-desktop/`(Tauri)을 한국어화 + 쉽게 포장
  (버튼 클릭 → 진단 → 카드 UI → Ollama 자동 설치 → 바로 채팅)
  무료 + 프리미엄 5,900원, 또는 완전 무료 + 제휴 링크
- **B. 한국 하드웨어 DB** — 조립PC 사양, 갤럭시북/그램 프로파일 추가
  → 한국에서만 가능한 데이터 우위
- **C. 다나와/퀘이사존 연동** — "이 예산으로 AI용 PC 조립하면 뭐가 돌아가나"

### 아이디어 6. 콘텐츠 자산화 (최저비용)

| 콘텐츠 | 수익 |
|---|---|
| 월간 "GPU별 LLM 성능 리포트" 블로그/뉴스레터 | 애드센스 + 스폰서 |
| 전자책 "2026 로컬 LLM 하드웨어 가이드" | 9,900~29,000원 |
| GPU 성능 비교 데이터 시각화 사이트 | 애드센스 + 제휴 |
| 노션 템플릿 "로컬 AI 도입 체크리스트" | 5,000원 |

`llmfit fit --json` 을 GPU 사양별로 반복 실행 → SEO 페이지 자동 생성 가능.
단, 사람이 읽을 가치 있는 해설을 반드시 얹을 것 (순수 자동 생성은 구글 패널티 위험).

### 종합 비교

| # | 아이디어 | 초기비용 | 난이도 | 회수속도 | 상한 | 추천도 |
|---|---|---|---|---|---|---|
| 1 | 사양 진단 사이트 (제휴) | 낮음 | ★★☆☆☆ | **빠름(2주)** | 월 수백만 | ⭐⭐⭐⭐⭐ |
| 2 | 유튜브 + 강의 | 거의 0 | ★★☆☆☆ | 중간(2~3달) | 월 수백만 | ⭐⭐⭐⭐⭐ |
| 3 | 기업 컨설팅 | 거의 0 | ★★★★☆ | 느림(영업) | **건당 수천만** | ⭐⭐⭐⭐ |
| 4 | SaaS 모니터링 | 중간 | ★★★★☆ | 느림 | 높음(구독) | ⭐⭐⭐ |
| 5 | 한국어 앱/Fork | 중간 | ★★★☆☆ | 중간 | 중간 | ⭐⭐⭐ |
| 6 | 콘텐츠 자산화 | 거의 0 | ★☆☆☆☆ | 중간 | 낮음~중간 | ⭐⭐⭐ |

### 실행 순서

```
[1~2주]   ① ko.js 한국어 PR 업스트림 제출  (공짜로 권위 확보)
          ② 유튜브 EP1 촬영

[3~4주]   ③ 사양 진단 사이트 MVP (아이디어 1)
          ④ 유튜브 EP2 "GPU 사기 전에"  ← 조회수 1등 후보

[2~3개월] ⑤ EP3~5 완성 → 유료 강의 패키징
          ⑥ 사이트 SEO 콘텐츠 축적 (GPU별 페이지)

[3~6개월] ⑦ 유입된 기업 문의 → 컨설팅 (아이디어 3)
          ⑧ 데이터 쌓이면 SaaS 검토 (아이디어 4)
```

**핵심 전략 — 세 개가 서로를 먹여주는 구조**
```
아이디어 2 (유튜브)  = 유입 엔진 (비용 0, 권위 생성)
        ↓
아이디어 1 (사이트)  = 현금 흐름 (제휴 수익)
        ↓
아이디어 3 (컨설팅)  = 고수익 전환 (건당 수백~수천만)
```

### 법적 체크리스트

| 항목 | 확인 |
|---|---|
| MIT 상업적 이용 | 가능 |
| 저작권 고지 유지 | **필수** — `Copyright (c) 2026 Alex Jones` |
| "llmfit" 상표 | 제품명으로 쓰지 말고 **"llmfit 기반"** 으로 표기 |
| 포크 후 유료 판매 | 가능하나 원작 링크 + 기여 환원 권장 (커뮤니티 평판) |
| 쿠팡파트너스 | "파트너스 활동으로 수수료를 받습니다" 고지 **의무** |
| 사양 데이터 수집 | 개인정보 아니지만 수집 고지 권장 |

---

## 13. 주의사항 / 특이점

- **Windows 코드 서명** — SignPath로 서명되지만, `sign-windows` 잡이
  스킵/실패하면 **서명 없는 바이너리가 그대로 배포**될 수 있다고 README에 명시.
  서명이 중요하면 직접 검증 필요
- **MODELS.md 108개 vs DB 12,937개** — 전자는 수작업 큐레이션, 후자는 자동
  크롤링 전체 카탈로그. 자동 카탈로그는 노이즈를 순위에서 제외한다:
  투기적 디코딩 draft head(EAGLE/DFlash/DSpark), 이름이 암시하는 파라미터 수가
  선언값과 4배 이상 차이나는 항목, 물리적으로 불가능한 footprint.
  강등된 항목은 검색·조회에는 남지만 적합도 순위에 오르지 않음
- **모델 DB 자동 갱신** — GitHub Actions가 **매주 월요일 02:00 UTC** 에
  HuggingFace를 크롤링 (`scripts/scrape_hf_models.py`, stdlib만 사용, 134KB)
- **`recommended_ram_gb` 는 판정에 쓰이지 않는다** — 카탈로그 전역
  `model_size × 2.0` 휴리스틱이라 양방향으로 왜곡을 일으켰음.
  현재는 메모리 사용률만으로 판정
- **기여 전 `cargo fmt`** — CI 실패 대부분의 원인

---

## 14. 핵심 요약 10줄

1. llmfit = 내 PC 사양으로 **실행 가능한 LLM을 찾아주는 Rust 터미널 도구** (MIT)
2. 모델 **12,937개**가 바이너리에 내장 → **오프라인 동작, 토큰 불필요**
3. 판정은 **Perfect / Good / Marginal / Too Tight** 4색 신호등
4. 속도는 메모리 대역폭 기반 추정 + **실측 354건(GPU 43종)** 으로 보정
5. 모든 수치에 **신뢰도 등급**이 붙는다 (measured_local ~ estimated)
6. 인터페이스 5종: **TUI / CLI / 웹 / REST / MCP**
7. **MCP 서버 도구 6개** 제공 → AI 에이전트에 즉시 연동 가능
8. React 대시보드가 이미 있고, **`ko.js` 추가만으로 한국어판** 가능
9. **TUI `S` 키 하드웨어 시뮬레이션** = GPU 사기 전 검증 (숨은 킬러 기능)
10. 수익화 추천: **사양 진단 사이트(제휴) + 한국어 유튜브 → 기업 컨설팅**

---

*분석 대상: https://github.com/bmshin94/llmfit (업스트림: https://github.com/AlexsJones/llmfit)*
*버전 v1.1.15 / 커밋 d45b3f9 기준*
