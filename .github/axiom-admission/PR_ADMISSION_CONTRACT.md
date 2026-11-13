# PR Admission Contract — Archright/nim-libp2p

## 0. 서두

이 문서는 Archright/nim-libp2p PR에 대해 Axiom 검증 결과를 병합 허가(merge admission)에 결속하는 시스템의 **보안·실행 신뢰(제1장) · 판정 의미(제2장) · 병합 권한과 복구(제3장)** 계약을 규정한다.

| 항목 | 값 |
|---|---|
| 문서 버전 | v1.0.0 (2026-11-13) |
| 정본 위치 (SSOT) | `Archright/nim-libp2p` `.github/axiom-admission/PR_ADMISSION_CONTRACT.md` — 이 문서 |
| 대상 저장소 | `Archright/nim-libp2p` |
| 대상 브랜치 | 실험 `refs/heads/axiom-admission-probe` → 운영 `refs/heads/master` (순차 적용, 제3장 §3.2) |
| 규정 대상 시스템 | org ruleset이 실행하는 신뢰 워크플로 기반 PR Admission 게이트 (제1장 §1.2) |
| 채택 명제 집합 | 인간 채택 명제 — 현재 1건 (제2장 §2.5) |
| 관측값 규칙 | 이 문서는 **확정된 실측값만** 기입한다. 런 ID·ruleset ID·체크 이름·응답 본문 등 실행 시점 관측값은 «실행 시 채워짐»으로 표기하고, 증거 문서에 관측값 그대로 기록한다 (제3장 §3.6) |

### 0.1 이 계약이 규정하지 않는 것

- **정식 캠페인 계약은 이 문서의 규정 대상이 아니다.** `.github/axiom-campaigns/campaign-001.json`과 `.github/workflows/axiom-verification.yml`은 이 계약과 무관하게 **불변**이며, 이 문서는 그 어느 것도 수정하지 않는다.
- PR Admission 게이트와 정식 캠페인(EXPERIMENT_ID 단위 실험)의 구분은 제2장 §2.7이 규정한다.

### 0.2 시스템 개요 (Fact)

| 요소 | 값 |
|---|---|
| 실행 주체 | org ruleset이 `Archright/nim-libp2p` PR에서 신뢰 워크플로를 실행 — 신뢰 워크플로는 `Archright/.github`(repository_id `1182157951`)의 `.github/workflows/nim-libp2p-pr-admission.yml`이며, ruleset이 repository_id + path + 40-hex commit SHA로 핀한다 (SHA·ruleset ID: «실행 시 채워짐») |
| 검증 내용 | PR 스모크는 **AI 추론 없이** 인간 채택 명제만 검증한다 — 현재 SPEC-LIBP2P-BV-SLICE (id `bytesview-toOpenArray-bv-slice`, target `libp2p/utils/bytesview.nim::toOpenArray`) |
| 어댑터 | `Archright/axiom-cli/.github/workflows/ax-verify.yml@4ca1892a69abcdb9036ec8a69d7bcedc940507d1` |
| 검증 엔진 | Axiom v0.6.3 릴리스 번들 — `bundle_sha256` `16180f3b1944c786d17de33db9f05f9989ec54e4d108dfcbce0eb67a1753ef2e` |
| 도구체인 핀 | Nim 2.2.10 / Z3 4.13.3 / Lean 4.33.0 (direct pinned downloads) |
| 잡 구조 | `propositions` → `verify-a` / `verify-b` → `admission-integrity` → `subject-verdict` (제1장 §1.5) |
| 채택 근거 | 라이브 캠페인 `external-verification-campaign-001` — 런 `43363278211` (2026-11-13, `workflow_dispatch`, `master`)에서 BYO 에이전트가 생성하고 Axiom이 A/B 재현 검증한 명제 중 **유일한 CAPABLE + 실측 판정 명제**. 판정 `DISPROVED_REPRODUCIBLE`, canonical digest `1603765c7df1c2f183e3f224d9b1fbb61ae3a1b77a1e5455cff7abea4953d14a` 양측(leg A·B) 일치. 채택은 인간의 행위이며 이 문서가 그 기록이다 (제2장 §2.5) |

---

## 제1장 Security & Execution Trust

### 1.1 검증 대상 신원 (Subject Identity)

**Authority**: 검증 대상 SHA의 단일 권위는 `github.sha`다.

**Fact** (GitHub 공식 문서): `pull_request` 이벤트에서 `github.sha`는 PR merge branch(`refs/pull/<n>/merge`)의 merge 커밋이다.

| # | Invariant |
|---|---|
| I1 | 대상 신원은 `github.sha` 단일 권위로 결정된다. head SHA fallback은 금지한다. |
| I2 | PR의 `merge_commit_sha`가 존재하고 `github.sha`와 불일치하면 **거절**한다 (fail-closed). |
| I3 | merge conflict 상태의 PR은 워크플로 실행 자체가 불가능하다 → 체크 pending → 병합 불가. 이는 플랫폼 수준 fail-closed이며, 이 계약이 별도로 구현하지 않는다. |

### 1.2 신뢰 사슬 (Chain of Trust)

| 단계 | 결속 (Binding) | 관측값 |
|---|---|---|
| 1 | org ruleset — 워크플로 신원 = repository_id `1182157951` (Archright/.github) + path `.github/workflows/nim-libp2p-pr-admission.yml` + 40-hex commit SHA 핀 | ruleset ID·워크플로 SHA: «실행 시 채워짐» |
| 2 | 신뢰 워크플로 — 잡 구조 결정(§1.5), `propositions` 잡의 대상 신원 결정(§1.1) + 채택 명제 결합(제2장 §2.5) | «실행 시 채워짐» |
| 3 | 어댑터 — `Archright/axiom-cli/.github/workflows/ax-verify.yml@4ca1892a69abcdb9036ec8a69d7bcedc940507d1` | 확정 (40-hex 핀) |
| 4 | 모든 `uses:` 액션 — 40-hex commit SHA 핀만 허용 (태그 핀 금지) | 워크플로 원문 |
| 5 | Axiom v0.6.3 릴리스 번들 — SHA256SUMS + manifest 결속(bundleSha256/binarySha256/version/tag/platform) + attestation 서명 검증 | `bundle_sha256` `16180f3b1944c786d17de33db9f05f9989ec54e4d108dfcbce0eb67a1753ef2e` |
| 6 | 도구체인 — Nim 2.2.10 / Z3 4.13.3 / Lean 4.33.0 핀 (direct pinned downloads) | 확정 |
| 7 | 채택 명제 — 엔진 카탈로그(`ax capability --json`)의 `specId`가 정본이며, 채택 기록은 이 문서(제2장 §2.5) | 확정 |
| 8 | 대상 커밋 — §1.1의 단일 권위 | `github.sha` |

**Invariant**: PR 작성자는 이 사슬의 어떤 고리도 수정할 수 없다. PR이 변경할 수 있는 유일한 대상은 검증 대상 커밋의 내용이며, 그 변경은 곧 검증 대상이 된다.

### 1.3 권한 (Permissions & Zero-Footprint)

| 항목 | 계약 |
|---|---|
| 비밀키 | **0개** — 이 게이트는 어떤 secret도 소비하지 않는다. |
| 잡 권한 | `permissions: contents: read` 유일. |
| PR 코드의 지위 | 어댑터가 읽기 전용으로 checkout하는 **데이터**이며 실행되지 않는다 — Zero-Footprint 증명: `pre_tree_hash == post_tree_hash` + working tree 오염 없음 (제2장 §2.2 ZERO_FOOTPRINT). |
| 저장소 가드 | 모든 잡은 `github.repository == 'Archright/nim-libp2p'` 가드를 둔다 — 신뢰 워크플로가 상주하는 `Archright/.github` 자체의 PR 실행을 차단한다. |

### 1.4 실행 제약 (Ruleset Workflow Execution Constraints)

**Fact** (GitHub 문서 확인 완료 — org ruleset required workflows의 실행 의미):

| 제약 | GitHub 동작 | 이 계약의 귀결 |
|---|---|---|
| 트리거 | ruleset 워크플로는 `pull_request` 기본 activity types(`opened`/`synchronize`/`reopened`)만·상시 실행 | 검증 트리거는 이 세 activity로 확정된다. |
| 필터 | 워크플로의 `branches`/`paths`/`types` 필터는 **무시됨** | 모든 대상 PR이 **전체 채택 명제 집합**에 대해 검증된다 — 범위 선별 없음(정직성 요건). |
| 기존 PR | ruleset 생성 시점에 이미 열린 PR은 자동 실행되지 않음 | 기존 PR은 새 push(`synchronize`)로 게이트에 진입한다. |
| `GITHUB_TOKEN` 이벤트 | `GITHUB_TOKEN`이 트리거한 이벤트는 ruleset 워크플로 실행을 유발하지 않음 | 재귀·자기 트리거가 없다. |
| 직접 push | 차단됨 | 의도된 속성 — `master` 변경은 PR 경유만 가능하다. |
| 외부(조직 밖) fork PR | 실행 보장 없음 (레거시 문서: same-organization 소스 요건) | **UNKNOWN**으로 기록한다. 신뢰 주장 금지. |

### 1.5 잡 구조와 A/B 독립성

| 잡 | 책임 |
|---|---|
| `propositions` | 대상 신원 결정(§1.1) + 채택 명제 결합(제2장 §2.5). AI 추론 없음. |
| `verify-a` / `verify-b` | 독립 **두 개의 별도 잡**으로 어댑터를 실행한다. matrix 대신 별도 잡인 것은 **matrix 출력 붕괴 방지**(출력 독립 보존) 때문이다. 양측 모두 동일 어댑터 SHA(단계 3)·동일 도구체인(단계 6)을 사용한다. |
| `admission-integrity` | 6 불변식 검사 (제2장 §2.2). |
| `subject-verdict` | 판정 매핑 (제2장 §2.3). |

**Invariant**: 동일 어댑터·도구체인에서 두 독립 실행의 `canonical_digest` 일치가 재현성 증명이다.

---

## 제2장 Subject Verdict Semantics

### 2.1 2축 분리 (Two-Axis Separation)

| 축 | 체크 | 질문 |
|---|---|---|
| 증거 무결성 | `admission-integrity` | 검증 파이프라인이 계약대로 실행됐는가 |
| 판정 | `subject-verdict` | 대상 소프트웨어가 채택 명제를 통과했는가 |

**Fact**: `DISPROVED_REPRODUCIBLE`은 무결성 관점에서 **PASS**(반례 재현 성공)이며, 판정 관점에서 **FAIL**이다. 이 분리가 "검증 시스템의 성공"과 "소프트웨어의 합격"의 혼동을 차단한다.

**Invariant**: `subject-verdict`는 `admission-integrity`가 생산한 증거만 소비한다 — 자체 재판정 금지.

### 2.2 6 무변식 (`admission-integrity`)

| # | 불변식 | 조건 (모두 충족 시 PASS) |
|---|---|---|
| 1 | `SUBJECT_IDENTITY` | 선언된 대상 신원(§1.1) == 양측(`verify-a`·`verify-b`) 기록의 subject identity |
| 2 | `INPUT_EQUALITY` | 양측이 소비한 명제 문서의 sha256 동일 |
| 3 | `DETERMINISM` | 양측 `canonical_digest`가 **실 64-hex**이고 서로 동일 — 부재/빈 digest의 자명한 "동등함"은 거부한다 |
| 4 | `EVIDENCE_NON_VACUITY` | 기록(verify-record) 존재 · `verdict_summary` 유효 · `prove_exit` ∈ {0, 1, 3} |
| 5 | `ZERO_FOOTPRINT` | `pre_tree_hash == post_tree_hash` · 오염(porcelain) 없음 — 양측 모두 |
| 6 | `ENVIRONMENT_EQUALITY` | 양측 runner image 동일 · 도구체인(Nim/Z3/Lean/libpcre3) 동일 |

**Failure**: 어느 하나라도 위반하면 `admission-integrity` = FAIL이며, 이는 `subject-verdict` FAIL의 전제가 된다 (§2.3).

**Note**: 이 6 불변식은 정식 캠페인의 5가드(`INPUT_MISMATCH`·`DETERMINISM_MISMATCH`·`VACUOUS_BLOCKED`·`ZERO_FOOTPRINT_VIOLATION`·`ENVIRONMENT_MISMATCH`)와 **동일하지 않고 PR Admission 전용 정의**다. 이름이 유사한 검사가 동일하다고 주장하지 않는다.

### 2.3 판정 매핑 (`subject-verdict`)

| `admission-integrity` | 관측 | `subject-verdict` | 기록 의무 |
|---|---|---|---|
| PASS | exit 0 — `PROVED_REPRODUCIBLE` | **PASS** | 명제별 증명 기록 |
| PASS | exit 1 — `DISPROVED_REPRODUCIBLE` | **FAIL** | **반례 + 대상 SHA 명시** |
| PASS | exit 3 — `REFUSAL_REPRODUCIBLE` | **FAIL (NOT READY)** | 엔진이 판정 불가임을 기록 — 파이프라인 성공과 별개 축 |
| FAIL 전제 | `INPUT_REJECTED`·사전 거부 등 무결성 위반 | **FAIL** | 위반된 불변식 명시 |
| FAIL 전제 | A≠B (exit·digest 불일치)·`CONTRADICTION` | **FAIL** | 불일치 관측값 명시 |
| FAIL 전제 | 증거 누락 | **FAIL** | 누락 필드 명시 |

**Invariant**: `exit 0`은 "모든 명제 증명됨"이 아니다 — 판정은 명제 단위 어휘로 기록된다.

### 2.4 CAPABLE 최소 요건 (Minimum Capability Requirement)

| # | Invariant |
|---|---|
| C1 | 필수 채택 집합은 **≥1개의 CAPABLE 명제**를 포함해야 한다. |
| C2 | 채택 집합이 비었거나 채택 명제 전원이 INCAPABLE이면 **PASS 금지** (공허한 통과 차단). |
| C3 | INCAPABLE 명제는 **채택 대상에서 제외**된다 (채택 정책 — INCAPABLE 명제가 병합을 영구 블록하는 것을 방지). 채택된 명제가 INCAPABLE로 판정되면 해당 채택을 재검토한다. |
| C4 | INCAPABLE은 증거에 **INCAPABLE로 정직 기록**된다 — 판정을 지어내지 않는다. |

**Fact (선례)**: 라이브 캠페인 `external-verification-campaign-001`(런 `43363278211`)에서 claim-form 명제가 `INCAPABLE{UNSUPPORTED_LOGIC}`로 분류되었고, 그 상태가 A·B 양측 증거에 그대로 기록되었다 — 판정 없는 정직 열화(INCAPABLE×2)의 선례다.

### 2.5 채택 절차와 현재 채택 기록 (Adoption)

**Authority**: 새 명제의 채택은 **인간의 행위**다. 이 문서의 채택 기록이 그 기록이다.

**절차**: 채택 시 이 문서에 `specId`·명제 id·target·명제 digest·근거 캠페인(런 ID·분류·canonical digest)을 기록하고 문서 버전을 상향한다. AI가 생성한 후보는 채택 없이는 게이트에 들어올 수 없다 — **발견 ≠ 권한**.

현재 채택 기록 (v1.0.0):

| 항목 | 값 |
|---|---|
| specId (정본: 엔진 카탈로그) | `SPEC-LIBP2P-BV-SLICE` |
| 명제 id | `bytesview-toOpenArray-bv-slice` |
| target | `libp2p/utils/bytesview.nim::toOpenArray` |
| 근거 캠페인 | `external-verification-campaign-001` — 런 `43363278211` (2026-11-13, `workflow_dispatch`, `master`) |
| 근거 분류 | `DISPROVED_REPRODUCIBLE` — BYO 에이전트 생성·Axiom A/B 재현 검증 명제 중 유일한 CAPABLE + 실측 판정 명제 |
| canonical digest (양측 일치) | `1603765c7df1c2f183e3f224d9b1fbb61ae3a1b77a1e5455cff7abea4953d14a` |
| PR Admission 소비 명제 문서 sha256 | «실행 시 채워짐» |
| 채택 시점 | 이 계약 v1.0.0 (2026-11-13), 인간의 행위 |

### 2.6 보증 범위 (Warranty Scope)

**Invariant**: 판정은 **채택 명제별 보증**이다. "master가 PROVED 기선"은 채택 집합에 한정된 표현이지 저장소 전체 동작의 증명이 아니다.

**Failure**: 채택 집합 밖 결함의 부재는 이 게이트의 주장이 아니다.

### 2.7 PR 스모크 ≠ 정식 캠페인

| 구분 | PR Admission | 정식 캠페인 (`campaign-001.json` + `axiom-verification.yml`, 불변) |
|---|---|---|
| run_mode | `pr_admission` | 캠페인 (EXPERIMENT_ID 단위 실험) |
| EXPERIMENT_ID | 없음 | `external-verification-campaign-001` |
| 명제 출처 | 인간 채택 명제만 (AI 추론 없음) | BYO AI 에이전트 발견(live) 또는 pinned fixture |
| 검사 어휘 | 6 불변식 + 판정 매핑 (이 장) | 5가드 + 캠페인 분류 (캠페인 SSOT) |
| SSOT | 이 문서 | `campaign-001.json`·`axiom-verification.yml` |

**Invariant**: 라이브 AI 발견(Discovery)은 `workflow_dispatch` 전용으로 유지된다 — PR Admission 경로에서 AI 추론은 수행되지 않는다.

---

## 제3장 GitHub Merge Authority + 복구

### 3.1 권한 구조 (Authority Structure)

**Authority**: 병합 허용/거부의 최종 권한은 **GitHub**이다 — org ruleset의 required workflows 규칙.

| # | Fact |
|---|---|
| A1 | 워크플로 신원은 repository + path + 40-hex SHA 결속이며 **체크 이름 기반이 아니다** → 동명 체크 스푸핑 불가. |
| A2 | 별도 GitHub App은 불필요하다. |
| A3 | Axiom은 **판정만** 소유한다 — 병합 권한은 없다. |

### 3.2 적용 대상 (Targets)

| 단계 | 대상 ref | 상태 |
|---|---|---|
| 실험 (probe) | `refs/heads/axiom-admission-probe` | 선적용 |
| 운영 (production) | `refs/heads/master` | probe 통과 후 순차 전환 |

**Invariant**: enforcement는 **active**다 (`evaluate`는 Enterprise 전용 기능이므로 의존하지 않는다).

| 관측값 | 값 |
|---|---|
| ruleset ID | «실행 시 채워짐» |
| ruleset 이전 상태 | «실행 시 채워짐» |

### 3.3 병합 거부 수용 기준 (Merge-Rejection Acceptance)

거부의 수용(완료 증명)은 다음 **4요건 모두** 충족이다:

| # | 요건 |
|---|---|
| R1 | 거부 사유가 required workflow/ruleset임 (응답·UI 메시지로 확인) |
| R2 | PR `merged == false` |
| R3 | master SHA 불변 (거부 전후 동일) |
| R4 | 동일 계정·동일 게이트에서 **통과 PR의 병합 성공** (양성 대조 — 권한 오류와 정책 거부의 구분) |

**Fact**: HTTP 코드(405/403/422) 자체는 증거가 아니다.

### 3.4 복구 계약 (Recovery)

| 항목 | 계약 |
|---|---|
| 사전 기록 | ruleset ID·이전 상태(enforcement·조건·워크플로 핀)를 §3.2 표에 기록한다. |
| 비활성화 경로 (org-admin) | `gh api -X PATCH /orgs/Archright/rulesets/<id> -f enforcement=disabled` |
| 비상 시 | 사유·audit 기록을 보존한다 (사후 재구성 불가능한 관측값은 선기록). |
| probe 단계 오작동 | master 적용 없이 **즉시 `disabled`**. |

### 3.5 미래 확장 (Future)

| 항목 | 계약 |
|---|---|
| merge queue 도입 시 | 워크플로에 `merge_group` 트리거 추가 — **계약 개정(이 문서 버전 상향) 선행**. |
| 관리자 bypass 정책 | `bypass_mode` = **null** (우회 없음). |

### 3.6 운영 부록 (Operations Appendix)

| 항목 | 규칙 |
|---|---|
| 관측 값 | 런 ID·ruleset ID·체크 이름·응답 본문은 증거 문서에 **관측값 그대로** 기록한다 (요약·변형 금지). |
| 신뢰 워크플로 갱신 | 새 커밋 SHA로 ruleset 워크플로 핀을 업데이트한다 — **probe에서 선실험 후 master 전환**. 갱신 시 이 문서의 해당 핀 값과 버전을 상향한다. |
| 이 문서의 변경 | 이 문서 변경 = 계약 개정 — 버전 상향 + 아래 이력 기록. |

### 변경 이력

| 버전 | 일자 | 내용 |
|---|---|---|
| v1.0.0 | 2026-11-13 | 최초 착지 — 3장 구성 (Security & Execution Trust / Subject Verdict Semantics / GitHub Merge Authority + 복구), 첫 채택 명제 기록 |
