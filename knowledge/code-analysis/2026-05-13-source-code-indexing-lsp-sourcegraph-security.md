# 소스코드 인덱싱 서버, LSP, Sourcegraph의 보안 분석 활용

Date: 2026-05-13
Category: Code Analysis / Source Code Indexing / Security Engineering
Status: 정리 노트
Source: windshock / ChatGPT conversation synthesis, "LSP and Sourcegraph for large-scale security analysis", 2026-05-13

## 1. 핵심 요약

대형 코드베이스 보안 분석에서 소스코드 인덱싱은 단순한 편의 기능이 아니라, **어디를 먼저 분석할지 줄여주는 전처리 계층**이다.

IDE에서는 보통 LSP(Language Server Protocol)를 통해 언어별 코드 이해 기능을 제공한다. LSP는 자동완성, 정의 이동, 참조 찾기, 심볼 검색 같은 IDE 기능을 제공하기 위한 표준 프로토콜이다.

Sourcegraph는 LSP보다 더 조직/전사 코드베이스 관점에 가깝다. 여러 저장소를 가로지르는 검색, 코드 탐색, 참조 추적, 관련 코드 context 수집에 강점이 있어 대형 코드베이스 보안 분석과 LLM 기반 코드 분석에 유용하다.

핵심 결론은 다음과 같다.

```text
LSP = 현재 프로젝트/IDE 중심의 언어-aware 코드 탐색 계층
Sourcegraph = 조직 전체 코드베이스 중심의 검색, 탐색, context 수집 플랫폼
Security engine = dataflow, control-flow, taint, sanitizer 판단을 수행하는 별도 분석 계층
```

## 2. 논의 흐름

처음에는 VS Code가 소스코드 인덱싱에 어떤 기술을 쓰는지 확인했다. 결론적으로 VS Code는 하나의 전역 인덱서를 쓰는 구조라기보다, 목적별로 여러 계층을 사용한다.

```text
문자열/파일 검색
→ ripgrep 기반 빠른 검색

언어별 코드 이해
→ LSP 또는 언어별 확장

C/C++ 정확한 코드 해석
→ compile_commands.json + clangd / C/C++ IntelliSense
```

C/C++에서는 빌드 과정에서 `compile_commands.json`을 생성해 실제 컴파일 옵션, include path, macro 정보를 언어 서버나 분석 도구에 전달할 수 있다.

예시:

```bash
cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=ON -S . -B build
```

이 파일은 인덱스 자체가 아니라, **인덱서와 정적 분석기가 코드를 정확히 해석하기 위한 빌드 지도**에 가깝다.

## 3. LSP의 역할

LSP는 자동완성, 정의 이동, 참조 찾기, 심볼 검색 같은 IDE 기능을 위한 프로토콜이다. 보안진단 관점에서 LSP 자체가 취약점 판정 엔진은 아니다.

하지만 본격적인 보안 분석 전에 다음 정보를 빠르게 수집하는 데 유용하다.

```text
- 함수/클래스/변수 정의 위치
- 참조 위치
- workspace symbol
- document symbol
- 구현체
- type definition
- call hierarchy 후보
- entrypoint 후보
- sink 후보
```

즉 LSP는 보안 분석에서 다음 역할을 한다.

```text
대형 코드베이스
→ LSP로 정의/참조/심볼/호출 후보 수집
→ 분석 대상 파일/함수 범위 축소
→ CodeQL/Semgrep/Joern/custom analyzer로 정밀 분석
```

## 4. LSP만으로 부족한 점

SQL Injection, Command Injection, Path Traversal, Deserialization 같은 취약점은 단순한 참조 찾기만으로 판단하기 어렵다.

필요한 질문은 다음과 같다.

```text
외부 입력 source가
검증/정제 없이
위험 함수 sink까지 도달하는가?
```

이를 판단하려면 보통 다음 정보가 필요하다.

```text
- AST
- CFG(Control Flow Graph)
- Call Graph
- Type Graph
- Dataflow Graph
- Taint Propagation
- Sanitizer / Validator 모델
- Framework-specific security semantics
```

LSP 서버 내부에 일부 정보가 있을 수는 있지만, LSP 표준 API가 이 정보를 보안 분석에 충분한 형태로 노출하는 것은 아니다.

따라서 LSP는 **보안 분석의 주 엔진**이라기보다, **정밀 분석 전 후보를 줄이는 인덱싱/탐색 계층**으로 보는 것이 적절하다.

## 5. 회사들이 별도 LSP 서버 또는 코드 인덱싱 서버를 만드는 이유

일부 회사들은 기존 언어 서버를 그대로 쓰지 않고, 사내 LSP 서버 또는 유사한 코드 인텔리전스 서버를 구축한다. 이유는 다음과 같다.

```text
1. 사내 빌드 시스템이 복잡함
2. monorepo / multi-repo / cross-repo 분석이 필요함
3. 사내 framework annotation, RPC, route, permission 모델을 이해해야 함
4. PR 리뷰, CI, IDE, Web UI, AI assistant가 같은 코드 지도를 공유해야 함
5. 보안팀이 source/sink 후보와 영향 범위를 빠르게 파악해야 함
```

이때 LSP는 단순 IDE 기능을 넘어서 다음 시스템의 입력 계층이 될 수 있다.

```text
- 코드 검색
- 코드 리뷰 보조
- 보안진단 후보 추출
- 변경 영향 분석
- 테스트 범위 추천
- LLM/RAG context provider
- 취약 패턴 클러스터링
```

## 6. Sourcegraph의 포지션

소스코드 인덱싱 서버로는 Sourcegraph가 대표적이다. 유료 제품이지만, 대형 코드베이스를 대상으로 한 다음 기능에 강점이 있다.

```text
- 다중 프로젝트 검색
- 전사 코드 검색
- 코드 탐색
- 참조 추적
- 관련 코드 context 수집
- LLM/보안 분석용 context retrieval
- 대규모 변경 관리
```

LSP가 보통 현재 workspace 또는 IDE 중심이라면, Sourcegraph는 여러 저장소와 조직 전체 코드베이스를 대상으로 한다.

```text
LSP
→ 현재 프로젝트를 언어-aware하게 이해하는 계층

Sourcegraph
→ 조직 전체 코드베이스에서 관련 코드 context를 찾아주는 계층
```

## 7. Sourcegraph의 context 장점

이번 논의에서 가장 중요한 결론은 Sourcegraph의 장점이 단순 검색이 아니라 **context 수집**에 있다는 점이다.

보안 분석이나 LLM 기반 코드 분석에서는 단일 파일만으로 판단하기 어렵다. 필요한 것은 관련 코드의 주변 맥락이다.

예를 들면 다음 질문이 중요하다.

```text
이 sink는 어디에서 호출되는가?
비슷한 안전한 사용 예시는 어디에 있는가?
이 wrapper 함수는 전사적으로 어디에서 쓰이는가?
이 패턴이 다른 repo에도 반복되는가?
공통 라이브러리 변경이 어떤 서비스에 영향을 주는가?
```

Sourcegraph는 이런 질문에 필요한 context를 여러 저장소에서 빠르게 가져오는 데 유리하다.

보안 분석 관점에서는 다음 흐름이 가능하다.

```text
Sourcegraph
→ 전사 코드 검색 / symbol reference / 관련 context 수집

LSP
→ 특정 프로젝트 내부의 정의/참조/심볼/호출 후보 수집

CodeQL / Semgrep / Joern / custom analyzer
→ dataflow / control-flow / taint / sanitizer 분석

LLM
→ 후보 경로 설명, false positive triage, PoC 가능성 검토, remediation 제안
```

## 8. 보안 분석용 권장 아키텍처

대형 코드베이스 보안 분석 자동화를 목표로 한다면 다음과 같이 나누는 것이 현실적이다.

```text
[Build Metadata]
compile_commands.json / Maven / Gradle / package.json / pyproject.toml

        ↓

[Language-aware Indexing]
LSP / clangd / jdtls / pyright / tsserver

        ↓

[Candidate Extraction]
entrypoint, sink, references, implementation, call hierarchy 후보

        ↓

[Code Context Retrieval]
Sourcegraph / code search / cross-repo references / similar patterns

        ↓

[Security Analysis Engine]
CodeQL / Semgrep / Joern / Clang Tooling / custom analyzer

        ↓

[Grouping / ML]
유사 entry-sink 구조 클러스터링
대표 payload 선정
테스트 우선순위화

        ↓

[Validation]
unit test / integration test / fuzzing / dynamic trace
```

이 구조에서 LSP와 Sourcegraph는 취약점 판정 그 자체보다, **정밀 분석 전에 분석 대상을 줄이고 관련 context를 모으는 역할**에 가깝다.

## 9. 보안진단 관점의 실용적 결론

LSP의 장점은 다음과 같이 요약할 수 있다.

```text
- 언어별 파서/타입 분석기를 직접 만들 필요가 없음
- 정의/참조/심볼/구현체 정보를 빠르게 수집할 수 있음
- IDE와 바로 연결 가능함
- 보안 분석 전 후보 파일/함수/경로를 줄일 수 있음
```

Sourcegraph의 장점은 다음과 같이 요약할 수 있다.

```text
- 여러 repo를 가로지르는 검색에 강함
- 조직 전체가 같은 코드 지도를 공유할 수 있음
- 관련 코드 context 수집에 강함
- LLM/RAG 기반 코드 분석에 유리함
- 보안 영향 범위 파악에 유리함
```

최종 결론:

```text
LSP는 대형 코드베이스에서 어디를 봐야 하는지 빠르게 찾게 해주는 언어-aware 인덱싱 계층이다.

Sourcegraph는 조직 전체 코드베이스에서 관련 context를 찾아주고, 분석 범위를 전사적으로 확장해주는 코드 인텔리전스 계층이다.

정확한 취약점 판정은 별도의 보안 분석 엔진이 담당해야 한다.
```

## 10. TODO / 실험 계획

- [ ] LSP 기반으로 Java/Spring 프로젝트에서 controller, service, repository, sink 후보를 수집하는 PoC 작성
- [ ] LSP reference 결과와 Semgrep/CodeQL 결과를 비교해 후보 축소 효과 측정
- [ ] Sourcegraph 또는 유사 code search API가 있을 때 cross-repo sink 사용처 수집 흐름 설계
- [ ] LLM context 구성 단위를 `entrypoint → service → repository/sink → sanitizer 후보` 형태로 표준화
- [ ] `fortify_ml`의 entry-sink 구조 그룹핑과 LSP/Sourcegraph context 수집을 연결하는 설계 작성

## 11. 참고 자료

- Language Server Protocol: https://microsoft.github.io/language-server-protocol/
- Sourcegraph: https://sourcegraph.com/
- CMake `CMAKE_EXPORT_COMPILE_COMMANDS`: https://cmake.org/cmake/help/latest/variable/CMAKE_EXPORT_COMPILE_COMMANDS.html
- clangd indexing design: https://clangd.llvm.org/design/indexing
- SCIP Code Intelligence Protocol: https://github.com/sourcegraph/scip
