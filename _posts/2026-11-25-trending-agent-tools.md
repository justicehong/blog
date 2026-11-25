---
layout: post
title: "11월 GitHub에서 급부상한 AI 코딩 에이전트 도구 4가지"
description: "rust-anything, fast-anything, Modern Web Guidance, Chrome DevTools MCP. 최근 스타가 빠르게 늘어난 에이전트용 도구가 각각 무엇이고 언제 쓰는 것인지 정리했습니다."
date: 2026-11-25 08:00:00 +0900
tags: [AI 에이전트, Rust, 웹, 오픈소스]
---

11월 중순 GitHub 트렌딩에는 Claude Code, Codex, Cursor 같은 코딩 에이전트에 붙여 쓰는 도구가 여럿 올라왔습니다. 그중 스타가 특히 빠르게 늘어난 네 가지를 골라, 각각 무엇을 하는 도구이고 어떤 상황에 맞는지 정리했습니다.

> 직접 써 본 후기가 아니라, 각 저장소의 README와 공식 문서 같은 공개 자료를 2026년 11월 25일 기준으로 정리한 글입니다. 스타 수는 집계 시점과 사이트에 따라 차이가 있으니 대략적인 규모로만 봐 주세요.

## 한눈에 보기

| 도구 | 한 줄 요약 | 만든 곳 | 스타 규모 |
|---|---|---|---|
| rust-anything | 앱 전체를 검증하면서 Rust로 이전 | 개인 개발자 | 약 9천 (11월 16일 공개) |
| fast-anything | 느린 함수만 골라 Rust로 교체 | 개인 개발자 | 이틀간 약 1,900 증가로 보도 |
| Modern Web Guidance | 에이전트가 최신 웹 표준으로 코딩하게 유도 | Google Chrome 팀 외 | 출시 48시간 만에 트렌딩 1위 |
| Chrome DevTools MCP | 에이전트가 실제 Chrome으로 디버깅 | Chrome DevTools 팀 | 약 5만 9천 |

## 1. rust-anything: 동작을 증명하면서 Rust로 옮기기

- 저장소: [erkamyaman/rust-anything](https://github.com/erkamyaman/rust-anything)
- 형태: CLI, MCP 서버, 에이전트 스킬
- 라이선스: MIT

기존 앱을 Rust로 다시 쓸 때 가장 큰 걱정은 "옮긴 뒤에도 똑같이 동작하는가"입니다. rust-anything은 이 부분을 맡는 도구입니다. 코드를 자동으로 번역해 주지는 않습니다. Rust 코드는 코딩 에이전트나 개발자가 직접 쓰고, 도구는 원본의 동작을 기록해 두었다가 새 코드가 그 기록과 일치하는지 확인합니다.

### 작업 흐름

1. **기록(record)**: 원본 프로그램을 여러 입력으로 실행해 결과를 저장합니다.
2. **포팅(bridge)**: Rust 크레이트와 래퍼를 만들어, 기존 호출부가 같은 API로 Rust 구현을 부르게 합니다.
3. **대조(diff, gate)**: 기록해 둔 케이스를 Rust 구현에 다시 돌립니다. `gate`를 통과해야 그 조각이 끝난 것으로 봅니다.
4. **전환(app lift)**: 앱 전체가 통과하면 원본 없이 도는 순수 Rust 앱으로 바꿉니다.

조각 단위로 진행하기 때문에, 이전하는 동안에도 앱은 계속 동작하는 상태를 유지합니다.

### 설치

```bash
cargo install rust-anything
rust-anything setup        # Claude Code, Codex, Cursor 등 에이전트에 등록

# 또는 npm으로
npm install -g rust-anything
```

### 지원 범위

- 대상: CLI, HTTP 서버, 라이브러리
- 언어: TypeScript/JavaScript, Python, C, Go, Ruby, Java
- PHP와 C#은 일부 기능만 지원하는 미리보기 단계

### 스타 추이

11월 16일에 공개된 저장소인데 18일 약 3,300개에서 22일 약 8,000개로 나흘 만에 4,700개 가까이 늘었고, 25일 현재 약 9,200개입니다. 공개 첫 주에 메이저 버전이 3.0까지 올라갈 만큼 변화가 빠른 프로젝트라서, 실제로 쓴다면 버전을 고정해 두는 편이 안전해 보입니다.

### 주의할 점

- 기록에 담긴 동작만 검증됩니다. 기록하지 않은 입력에서 달라지는 부분은 잡아내지 못합니다.
- 기록과 대조 과정에서 원본 코드가 반복 실행됩니다. 파일을 쓰거나 DB, 외부 API를 호출하는 코드라면 그 동작도 매번 일어나므로, README는 운영 자격증명 없이 테스트 DB를 붙인 복사본에서 돌리라고 안내합니다.
- 작은 함수 하나를 호출할 때마다 Rust로 넘어가는 구조는 JIT보다 오히려 느릴 수 있습니다.

## 2. fast-anything: 느린 5%만 Rust로

- 저장소: [EL4CTEO/fast-anything](https://github.com/EL4CTEO/fast-anything)
- 형태: CLI, MCP 서버(도구 8개), Claude Code 플러그인, 스킬
- 라이선스: MIT

rust-anything과 비슷한 시기에 나왔지만 방향은 반대입니다. 전체를 다시 쓰는 대신, 실행 시간 대부분을 차지하는 함수 몇 개만 Rust로 바꾸고 나머지는 원래 언어 그대로 둡니다. 저장소 소개 문구도 "Don't rewrite it in Rust. Rewrite the 5% that's slow."입니다.

### 작업 흐름

| 명령 | 하는 일 |
|---|---|
| `scan` | 앱을 프로파일링해서 함수별 소요 시간과 이식 가능성으로 순위를 매김 |
| `port` | 실제 호출을 최대 500건 기록하고, 함수 하나짜리 Rust 크레이트와 에이전트용 작업 지시서를 생성 |
| `verify` | 릴리스 빌드 후 기록된 호출을 전부 재생해 결과 비교 |
| `enable` / `disable` | Rust 구현을 켜거나 끔 |
| `bench` | 앱 전체 실행 시간을 원본과 비교 |
| `go` | 위 과정을 한 번에 실행 |

검증 케이스가 하나라도 원본과 다르면 Rust 구현은 활성화되지 않습니다.

### 설치와 실행

```bash
npm i -g @el4cteo/fast-anything
# 또는
cargo install fast-anything

# 프로파일링부터 교체까지 한 번에
fast-anything go --agent claude -- python app.py
```

포팅한 코드는 로컬의 Rust 툴체인(rustup)으로 컴파일하므로 Rust가 설치되어 있어야 합니다.

### 지원 언어

Python 3.10+, Node 22+ 기반 JavaScript/TypeScript, Ruby 3.3+, Java 22+, Go 1.22+, PHP 8.1+, C/C++

### 벤치마크 (README 기준)

퍼지 중복 제거 예제에서 Python 116배, PHP 55배, Go 21배, TypeScript 20배, Ruby 14배 빨라졌다고 합니다. 작성자가 측정한 수치이므로 자기 환경에서 `bench`로 직접 확인하는 것이 좋습니다.

### 주의할 점

- 숫자, 문자열, 리스트, 딕셔너리 같은 단순 데이터를 받는 최상위 결정적 함수만 옮길 수 있습니다.
- 작은 내부 함수보다 바깥쪽 루프를 통째로 옮겨야 효과가 큽니다.
- JVM 시작 시간처럼 함수 밖에서 드는 비용이 크면 전체 이득은 줄어듭니다. README의 Java 예제는 함수 자체는 11배 빨라졌지만 전체로는 3배였습니다.

### rust-anything과 비교

| | rust-anything | fast-anything |
|---|---|---|
| 목표 | 앱 전체를 단계적으로 Rust로 이전 | 느린 함수만 Rust로 교체 |
| 최종 결과 | 순수 Rust 앱 | 원래 앱 + Rust 모듈 몇 개 |
| 검증 방식 | 기록한 동작 전체를 조각마다 대조 | 실제 호출 최대 500건 재생 |
| 맞는 상황 | Rust로 완전히 넘어가기로 결정한 경우 | 기존 스택은 유지하고 병목만 풀고 싶은 경우 |

rust-anything에도 `profile`로 뜨거운 구간을 찾아 그 루프만 Rust로 보내는 기능이 있어서, 두 도구의 영역이 일부 겹칩니다.

## 3. Modern Web Guidance: 에이전트가 2019년식 웹 코드를 쓰지 않게

- 저장소: [GoogleChrome/modern-web-guidance](https://github.com/GoogleChrome/modern-web-guidance)
- 공식 문서: [developer.chrome.com/docs/modern-web-guidance](https://developer.chrome.com/docs/modern-web-guidance)
- 형태: 에이전트 스킬 묶음 + CLI

코딩 에이전트는 학습 데이터에 많이 들어 있던 예전 방식으로 웹 코드를 쓰는 경향이 있습니다. Modern Web Guidance는 에이전트가 웹 UI 코드를 쓰기 전에 최신 모범 사례와 브라우저 호환성 데이터를 먼저 확인하게 만드는 스킬 묶음입니다. Chrome 팀이 주도하고 Microsoft Edge와 Mozilla 쪽이 함께 참여했습니다.

### 들어 있는 것

- Chrome 전문가가 검수한 가이드 200개 이상
- 다루는 영역: CSS, Web UI, HTML, JavaScript, Core Web Vitals, 접근성, 보안, Web AI
- 기존 코드베이스에서 낡은 패턴을 찾아 주는 `legacy-scan`

### 설치

```bash
# 스킬 설치 (권장)
npx modern-web-guidance@latest install

# 기존 코드 점검
npx modern-web-guidance@latest legacy-scan .
```

Claude Code, Copilot CLI, Antigravity CLI용 플러그인으로도 설치할 수 있습니다.

### 효과

공식 페이지는 공개 벤치마크 통과율이 가이드 없는 모델은 약 51%, 가이드를 적용하면 91%라고 밝히고 있습니다. Google이 자체적으로 수행한 평가라는 점은 감안해야 합니다.

### 스타 추이

11월 둘째 주에 1.0이 나왔고, 출시 48시간 만에 GitHub 트렌딩 1위에 올랐습니다. 11월 16일에는 소스 저장소(modern-web-guidance-src)가 하루에 1,220개를 얻어 일간 증가 1위였습니다. 스킬 배포용 저장소와 소스 저장소가 나뉘어 있어서 집계 사이트마다 스타 수가 다르게 나옵니다.

## 4. Chrome DevTools MCP: 에이전트에게 브라우저를 보여 주기

- 저장소: [ChromeDevTools/chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp)
- 형태: MCP 서버 + CLI + 스킬 7개

에이전트가 프런트엔드 코드를 고쳐도, 실제 브라우저에서 어떻게 동작하는지는 볼 수 없는 경우가 많습니다. Chrome DevTools MCP는 에이전트가 실제 Chrome을 띄워 조작하고 DevTools의 정보를 읽을 수 있게 해 줍니다. 앞의 세 가지와 달리 새 프로젝트는 아니고, 이미 규모가 큰 저장소가 이번 달에 다시 빠르게 오른 경우입니다.

### 할 수 있는 일

- **성능 분석**: 트레이스를 기록하고 성능 문제를 뽑아냄
- **디버깅**: 네트워크 요청, 콘솔 메시지(소스맵 적용된 스택 트레이스), 스크린샷 확인
- **자동화**: Puppeteer 기반으로 브라우저 조작

함께 들어 있는 스킬은 접근성 디버깅, 쿠키 디버깅, LCP 최적화, 메모리 누수 디버깅 등 7개입니다.

### 설치

MCP 클라이언트 설정에 아래 항목을 추가합니다.

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest"]
    }
  }
}
```

기본적인 브라우저 작업만 필요하면 `--slim` 옵션으로 도구 수를 줄일 수 있습니다.

### 스타 추이

11월 16일에 하루 718개를 얻어 일간 증가 4위(누적 약 56,700개)였고, 11월 23일 주간 트렌딩 집계에서는 약 58,900개였습니다.

### 주의할 점

- 사용 통계 수집이 기본으로 켜져 있습니다. 끄려면 `--no-usage-statistics`를 붙입니다.
- 성능 도구가 트레이스 URL을 Google CrUX API로 보낼 수 있습니다. `--no-performance-crux`로 끌 수 있습니다.
- 공식 지원 브라우저는 Google Chrome과 Chrome for Testing입니다.

## 네 도구의 공통점

성격은 다르지만 구조는 닮았습니다. 코드를 쓰는 일은 에이전트에게 맡기고, 도구는 에이전트가 스스로 확인할 수 없는 **근거**를 제공합니다.

| 도구 | 에이전트에게 주는 근거 |
|---|---|
| rust-anything, fast-anything | 원본 프로그램이 실제로 어떻게 동작했는지에 대한 기록 |
| Modern Web Guidance | 지금 시점의 웹 표준과 브라우저 호환성 데이터 |
| Chrome DevTools MCP | 실제 브라우저에서 벌어지는 일 |

에이전트가 "그럴듯한 코드"를 쓰는 단계를 지나, 그 코드가 맞는지 확인할 수단을 붙이는 쪽으로 관심이 옮겨 가고 있다고 볼 수 있습니다.

## 어떤 상황에 맞을까

- **Python·Node 서비스에 CPU 병목이 있다**: fast-anything으로 병목 함수만 옮겨 보는 것이 부담이 가장 적습니다.
- **Rust로 완전히 넘어가기로 했다**: rust-anything으로 조각 단위로 검증하며 옮깁니다.
- **에이전트에게 웹 UI 작업을 자주 맡긴다**: Modern Web Guidance를 설치해 두면 됩니다. 설치가 명령 한 줄이라 시도 비용이 낮습니다.
- **프런트엔드 버그나 성능 문제를 에이전트와 함께 본다**: Chrome DevTools MCP를 연결합니다.

## 참고 링크

- [erkamyaman/rust-anything](https://github.com/erkamyaman/rust-anything)
- [EL4CTEO/fast-anything](https://github.com/EL4CTEO/fast-anything)
- [GoogleChrome/modern-web-guidance](https://github.com/GoogleChrome/modern-web-guidance) · [소스 저장소](https://github.com/GoogleChrome/modern-web-guidance-src) · [공식 문서](https://developer.chrome.com/docs/modern-web-guidance)
- [ChromeDevTools/chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp)
