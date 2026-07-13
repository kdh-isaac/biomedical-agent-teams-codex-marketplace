# BMAT 개선안 비평 및 정제된 실행 로드맵

- 작성일: 2026-07-13 KST
- 대상: `kdh-isaac/biomedical-agent-teams-claude-marketplace` 개선 플랜 패키지(마스터 로드맵 + 4개 세부계획 + backlog/acceptance/risk CSV)
- 기준 commit: `516aedfc7b3b0d10a566a0539c91284bdfc22192`, plugin `1.1.0`
- 이 문서의 목적: (1) 기존 플랜의 논리적 강점과 결함을 감사하고, (2) 그 결과를 반영한 착수 가능한 실행 로드맵을 제시한다. 원본 계획을 대체하지 않고 그 위에 실행 판단 레이어를 얹는다.

---

## 0. 한 줄 결론

기존 플랜은 **설계 품질과 내부 정합성이 매우 높다**(42 tasks가 acceptance/risk와 상호참조되고 DAG가 acyclic이며 self-attestation·empty-evidence·fabricated-ID 우회를 fail-closed로 막는 구조가 명확). 그러나 **하나의 연속된 180일 로드맵으로 제시된 것이 가장 큰 실행 리스크**다. 실제로는 성격이 전혀 다른 두 개의 프로젝트 — 유지보수자가 통제 가능한 *엔지니어링 하드닝*과, 외부 인력·예산에 종속된 *경험적 벤치마크 연구* — 가 하나로 묶여 있다. 아래 비평의 핵심은 이 둘을 분리하고, 유지보수자 단독으로도 사용자에게 가치를 출시할 수 있는 경로를 확보하는 것이다.

---

## 1. 플랜의 유지해야 할 강점

이 부분은 재설계하지 말고 그대로 계승한다.

- **두 축 분리(`workflow_label` vs `assurance_level`)**: "절차 완결성"과 "증거 검증 정도"를 분리한 것이 이 플랜의 개념적 핵심이고 정확하다. `Full protocol followed`가 과학적 진실을 함의하지 않게 만든 것은 감사 결함 #1을 정면으로 해결한다.
- **Deterministic validator가 assurance를 계산(model이 선택 못 함)**: `assurance_level_computed`를 receipt에서 규칙으로 산출하는 구조는 self-attestation 우회를 근본적으로 차단한다.
- **Receipt provenance 계층화**: `lifecycle_event_receipt`(hook 관찰) / `review_binding_receipt`(harness 결합) / `user-decision-receipt`(runtime 생성)를 분리해 "model이 쓴 JSON은 증거가 아니다"를 강제한 것이 옳다.
- **Same-model second pass ≠ independent review**: `same_model_corroboration`을 별도 surface로 둔 것은 대부분의 multi-agent 플러그인이 놓치는 부분이다.
- **YAGNI 예산(8장)**: router 재작성 금지, vector DB 금지, entailment 모델 validator 직접 삽입 금지 — 범위 확장을 선제적으로 막은 것이 현실적이다.
- **경험적 우월성 주장을 gate 뒤에 잠금**: primary endpoint의 CI lower bound > 0 전에는 "single-agent 대비 우월" 표현을 금지한 것이 과학적으로 정직하다.

이 강점들은 그대로 두고, 아래는 실행 관점의 결함이다.

---

## 2. 핵심 비평 (우선순위 순)

### C-1. 리소스 현실과 로드맵 프레이밍의 모순 — **가장 중요**
`context.md`/master §7은 이미 정직하게 인정한다: 총 **196–295 person-days**, solo **7–10개월**, Day-180은 "최소 2 engineers + adjudicator 사전 commitment가 있을 때만" 성립. 그런데 로드맵 본문(§5)은 30/90/180을 마치 기본 경로처럼 제시한다. 이 프레이밍이 실행 시 "일정 실패"로 오인될 소지가 크다.

**권고**: 로드맵을 **캘린더가 아니라 리소스 조건부 트랙**으로 재구성한다(§3). Day-180을 deadline이 아니라 "2인+reviewer가 확보됐을 때의 재평가 checkpoint"로 명시적으로 강등하고, solo 경로의 실제 산출물을 별도로 정의한다.

### C-2. Workstream C(벤치마크)는 별도 연구 프로젝트다
C는 12–20명 domain-expert steerer + 2명 blinded adjudicator + statistician + 120–140 case가 필요하다. 이건 플러그인 개선이 아니라 **소규모 연구(사실상 grant/IRB급 인력 조달)**다. 플랜은 C를 A/B/D와 연속선상에 두지만, 통제 가능성·비용·외부 종속성이 근본적으로 다르다.

**권고**:
- C 전체를 **명시적 go/no-go gate**(reviewer commitment + expert-hour 예산 확보) 뒤에 둔다. commitment 없으면 C는 착수하지 않는다.
- 대신 **Task 6A(source-verifier challenge, valid 150 + invalid 150 identifier)를 "경험적 증거 v0"로 승격**한다. 이건 blinded human adjudication 없이 deterministic하게 sensitivity/specificity를 산출하므로, empirical evidence 1.0 → 실측 수치를 **유지보수자 단독·저비용**으로 확보하는 유일한 현실적 경로다. 120-case 연구는 그 다음이다.

### C-3. `empirical model evidence 1.0 → 6.0`은 유일하게 외부 종속적 gate
truth enforcement / provenance / ops 목표는 유지보수자가 코드로 통제한다. 반면 empirical 목표만 외부 인간에 종속된다. 그런데 로드맵은 v2.0.0을 이 gate에 사실상 커플링한다.

**권고**: **v2.0.0(validator hardening 안정판)과 "scientific-performance claim 출시"를 명시적으로 디커플**한다. §9의 go/no-go 마지막 행("Gate 미통과 시 v2.0.0은 claim 없이 hardening release로만 출하")이 이미 이 문을 열어뒀으나, 로드맵 상단 프레이밍이 이를 가리고 있다. 이 디커플을 **1급 원칙으로 격상**해야 solo 유지보수자가 empirical 연구에 막혀 하드닝 가치를 못 내는 상황을 피한다.

### C-4. `v1.1.1`은 hotfix가 아니라 실질적 minor release
`v1.1.1`에 B0, A2, B10, D1–D5, D9 = 약 10개 task, **25–35 person-days**가 묶여 있다. 이건 hotfix 규모가 아니다. 더 중요한 문제:

- **A2(missing jsonschema를 Full label에서 hard-fail)는 동작 변경(behavior change)이다.** jsonschema 없이 Full을 통과하던 기존 사용자의 exit code가 바뀐다. "옳은" 변경이지만 semver상 **non-breaking으로 분류하기 어렵다**(최소 minor, 사용자 스크립트 관점에선 breaking). §04 version table의 `v1.1.1 = non-breaking`은 A2에 대해 부정확하다.

**권고**: `v1.1.1`을 **순수 위생(D1–D5, B0, D9 = 설치문서/LICENSE/CI/path/registry sync/release process)** 로 좁힌다. **A2 + B10(domain-pack gate)** 는 동작을 바꾸므로 `v1.1.2` 또는 `v1.2.0-alpha`로 옮기고 CHANGELOG에 behavior change로 명시한다.

### C-5. `independent_review_attested`(assurance level 4)는 구조적으로 도달이 드물다
B6이 정직하게 인정하듯, Claude Code hook은 spawned subagent의 **model identity를 암호학적으로 증명하지 못한다**(`model_identity_status`가 대부분 `not_observed`). 따라서 `separate_model` 승격 조건을 만족하는 실행이 실무에서 거의 없다. 결과적으로 **assurance ladder는 대부분의 run에서 level 3(`claim_grounded`)에 실질 상한**을 갖는다.

**권고**:
- 이걸 **결함이 아니라 설계된 운영 현실로 문서화**한다("v1.2에서 assurance는 claim_grounded까지가 일반적 상한이며, level 4+는 external verifier/human이 있을 때만").
- **B6(5–7 person-days + security reviewer, Node.js .mjs hook이라는 가장 취약한 새 표면)을 v1.2-beta에서 v1.3으로 연기**하는 것을 검토한다. level 3까지의 ladder를 먼저 안정 출시하고, level 4 인프라는 실제 external-verifier 수요가 확인될 때 붙인다. 이는 §8 "reviewer 수를 늘리지 않는다"와도 일관된다.

### C-6. 문서 ↔ CSV 마일스톤 드리프트 (정합성 자부심 대비 실제 결함)
`verification_log`는 dependency acyclicity·ID resolution은 검증했으나 **마일스톤 명칭의 서사↔CSV 일치는 검증하지 않았다.** 실제 드리프트:

- **master §5.2**는 Day 31–90(release candidate `v1.2.0-beta.1`)에 "24-task pilot 실행"과 "hypothesis ranking permutation/Bradley–Terry 추가"를 넣는다. 그러나 backlog CSV는 **B9 → `v1.3.0`**, **C1–C7 → `v1.3.0-pilot`** 이다.
- `v1.3.0`은 **master 로드맵의 release-candidate 목록(v1.1.1 / v1.2.0-alpha / v1.2.0-beta / v2.0.0-rc / v2.0.0) 어디에도 등장하지 않는다.**

즉 서사 로드맵과 CSV가 pilot·ranking의 릴리스 귀속에서 어긋나 있다.

**권고**: 하나로 통일한다. 권장 방향은 **CSV가 맞다(B9·pilot을 `v1.3.0`로)** — pilot을 v1.2-beta에 욱여넣지 말고 별도 v1.3 라인으로 두는 편이 리소스 현실(C-1, C-2)과 일치한다. master §5.2 문구를 "pilot 인프라 준비(실행은 v1.3)"로 수정한다.

### C-7. Day-30 gate의 "adversarial 20개 fail-closed"와 A10 배치 충돌
Day-30 go gate는 "신규 adversarial fixtures 최소 20개 전부 fail-closed"를 요구한다. 그러나 adversarial 스위트인 **A10은 `v1.2.0-beta`** 이고, A10의 필수 케이스 중 #20(recipe required node removed)·#21(recipe digest mismatch)은 **B2(v1.2-alpha)**, #17–19(reviewer receipt 계열)는 **B6(v1.2-beta)** 에 의존한다. 따라서 Day-30 시점에 22개 전부는 불가능하다.

**권고**: gate를 **레벨별 achievable subset으로 명시**한다. Day-30에서는 source/claim/graph/contradiction 계열(#1–16)만 요구하고(≈16개), recipe/reviewer 계열(#17–22)은 v1.2-beta gate로 이동. "20개"라는 반올림 숫자 대신 케이스 ID로 gate를 정의한다.

### C-8. v2 스키마 마이그레이션이 **기존 179 테스트 자체**에 주는 부담이 과소평가됨
A3는 `schema_version=2.0` + canonical 객체에 `additionalProperties:false` + strict enum + conditional required를 도입한다. 이는 **기존 fixture 다수를 깨뜨린다.** 플랜은 "179 tests가 약화 없이 pass"를 요구하지만, 그러려면 fixture들을 v2로 migrate하거나 v1 compatibility route로 재분류해야 하고, 그 작업량이 별도 라인으로 잡혀 있지 않다(A3의 3–4일에 암묵 포함).

**권고**: **fixture 마이그레이션을 명시적 sub-task/effort로 분리**(추가 2–3 person-days 추정)하고, "기존 valid fixture는 `structure_only`로 cap된 v1 route로 통과, 신규 grounded 동작은 v2 fixture로 검증"을 테스트 전략으로 못박는다.

### C-9. 새 런타임 표면(Node.js hook)이 "jsonschema 외 신규 의존성 없음" 원칙과 긴장
§Global Constraints는 "신규 production dependency는 jsonschema 외 원칙적 금지"라 하지만, **B6은 `.mjs` hook(Node.js) + `hooks.json`** 이라는 새 실행 표면을 추가한다. Claude Code가 Node를 요구하므로 신규 *설치* 의존성은 아니지만, **유지보수 표면**은 늘어난다.

**권고**: 제약 문구를 "신규 Python 의존성은 jsonschema만, hook은 표준 라이브러리 Node만"으로 정밀화하고, B6 연기(C-5)와 묶어 판단한다.

### C-10. 최종 사용자 관점의 가시적 가치 증분 부재
작업의 90%가 내부 정합성·provenance다. CAR-T 연구자(=유지보수자 본인)의 **일상 사용 경험**은 v2(7–10개월) 전까지 무엇이 좋아지는가가 로드맵에 드러나지 않는다.

**권고**: **D6(doctor/status read-only UX) + run-card(B7) + 정직한 assurance 라벨 표시**를 "v1.2의 눈에 보이는 산출물"로 전면에 배치한다. "더 안전해졌다"가 아니라 "무엇이 검증됐고 무엇이 안 됐는지 한 번에 보인다"는 UX 이득을 명시적 deliverable로 삼는다.

---

## 3. 정제된 실행 로드맵 — 리소스 조건부 3-트랙

기존 42-task를 폐기하지 않고, **캘린더 대신 리소스 조건**으로 재편한다.

### 트랙 구분

| 트랙 | 성격 | 통제 가능성 | 선행 조건 |
| --- | --- | --- | --- |
| **T1. 위생·릴리스 기반** | 순수 엔지니어링 | 유지보수자 단독 100% | 없음 |
| **T2. 과학적-진실 하드닝** | 스키마/receipt/gate | 유지보수자 단독(+도메인 리뷰 소량) | T1 일부 |
| **T3. 경험적 벤치마크** | 연구 프로젝트 | 외부 인력·예산 종속 | **명시적 go/no-go(§3.4)** |

핵심 원칙: **T1+T2는 T3와 무관하게 출시 가능하다.** v2.0.0(hardening 안정판)은 T3 결과를 기다리지 않는다. Scientific-performance claim만 T3 gate 통과 시 별도 출시한다.

### 3.1 T1 — 위생·릴리스 기반 (solo, ≈2–4주, `v1.1.1`)

목표: "설치되고, 검증되고, 릴리스되는" 제품 기반. **동작 변경 없음, 순수 non-breaking.**

| 순서 | Task | 산출물 | 비고 |
| --- | --- | --- | --- |
| 1 | D1 | 설치/업데이트/제거 문서 교정 | stale 명령 0 |
| 2 | D4 | path/subprocess portability(공백·CJK·Windows) | 기존 2건 버그 재현→해결 |
| 3 | D3 | LICENSE/SECURITY/CHANGELOG/community | MIT 텍스트-메타 일치 |
| 4 | D2 | VERSION 단일 트랜잭션·count drift 0 | dry-run 기본 |
| 5 | B0 | tool-registry JSON/MD/count sync (동작 변경 없음) | B4와 분리 유지 |
| 6 | D5 | uv.lock + 3-OS CI (live 호출 금지) | support 주장 = CI matrix |
| 7 | D9 | tag/Release/release-manifest 워크플로 | 이후 재사용 |

**T1 exit gate**: stale 설치명령 0 · 실제 LICENSE/SECURITY · version/count drift 0 · path matrix green · deterministic CI green · fresh-install smoke pass · **"live model benchmark 미실행" 상태 명시**.

> **C-4 반영**: A2·B10은 여기서 제외(동작 변경이므로).

### 3.2 T2 — 과학적-진실 하드닝 (solo + 도메인 리뷰 소량, ≈2–4개월)

두 개의 알파/베타로 나눈다.

**`v1.1.2` 또는 `v1.2.0-alpha` 초입 — behavior-change 배치 (C-4 반영)**
- A2 (jsonschema hard-fail) — CHANGELOG에 behavior change 명시
- B10 (domain-pack global gate)

**`v1.2.0-alpha` — 스키마·정합성 코어**
- A1 → A2(이미 상단) → **A3(+fixture 마이그레이션 sub-task, C-8 반영)** → A4 → A5
- A6 (PMID/DOI/NCT source receipts)
- A11 (additive v1→2.0-alpha 마이그레이터, source 불변)
- B1 (preflight schema 통합, offline `$ref` registry)
- B2 (canonical workflow recipe·mode matrix) → B4 (logical capability registry)
- D7 (privacy·egress threat model + gate)

**`v1.2.0-beta` — grounding·런타임 진실·가시적 UX**
- A7 (claim–passage grounding + final-claim map) → A8 (contradiction adjudication) → A9 (deterministic assurance ceiling, **level 3 상한으로 안정 출시**)
- A10 (adversarial 스위트 — **레벨별 subset gate, C-7 반영**)
- B3 (mode-specific DAG + writer/validator 분리)
- B5 (reviewer evidence packet)
- B7 (C0/C1/C2 user checkpoint + run card) · B8 (local minimal telemetry)
- **D6 (doctor/status UX) — "눈에 보이는 v1.2 이득"으로 전면 배치 (C-10)**

**연기 검토 (C-5 반영)**: **B6(hook attestation)** 와 그에 의존하는 `independent_review_attested`(level 4)를 **v1.3으로 연기**. v1.2는 assurance ladder를 level 3까지로 정직하게 상한 설정하고 출시.

**`v1.3.0` — level 4 인프라 + 랭킹 안정성 + pilot 준비 (C-6 반영)**
- B6 (lifecycle/review-binding attestation, external-verifier 수요 확인 시)
- B9 (ranking permutation/Bradley–Terry sensitivity)
- C1–C7 pilot 인프라 (실행은 T3 go 조건 하에서)

**`v2.0.0-rc → v2.0.0` — strict-v2 안정판 (T3와 무관하게 출시 가능)**
- A12 (strict-v2 non-promoting migration) · D8 (compatibility governance) · D10 (stable/rollback docs)
- **이 릴리스는 validator-hardening 안정판이며, scientific-performance claim을 포함하지 않는다.**

### 3.3 T3 — 경험적 벤치마크 (go/no-go 종속, ≈7–10개월 별도)

**저비용 선행(유지보수자 단독)**: **Task 6A source-verifier challenge (valid 150 + invalid 150)** 를 먼저 실행 → empirical evidence 1.0 → 실측 sensitivity/specificity 확보. 이것이 "값싼 경험적 증거 v0"이며 human adjudication 불요.

**본 연구(C8/C9, go 조건 충족 시에만)**: 120–140 case × 4 arm, blinded double adjudication, 12–20 steerer. 사전등록·frozen corpus·locked analysis 후 실행. 결과가 §9 conjunctive gate 통과 시에만 "single-agent 대비" 표현 허용.

### 3.4 T3 착수 go/no-go 체크리스트 (신규 — 기존 플랜엔 암묵적)

아래가 **모두 yes**일 때만 C8(full run)에 자원을 투입한다. 하나라도 no면 T3는 6A 수준에서 멈추고 T1/T2 가치로 출시한다.

- [ ] blinded adjudicator 2명 + third adjudicator 1명이 일정에 사전 commit했는가
- [ ] D-arm steerer 12–20명(도메인/난이도 균형 가능)이 확보됐는가
- [ ] pilot 실측 `minutes/output` 기반 expert-hour 예산(≈46–68 expert-days)이 확보됐는가
- [ ] statistician(0.1 FTE)이 locked-analysis-plan을 검토·서명했는가
- [ ] private gold를 repo 밖(`BMAT_BENCH_GOLD_DIR`)에 격리하는 인프라가 준비됐는가
- [ ] pilot inter-rater agreement ≥0.70(1회 rubric 개정 후)을 통과했는가

---

## 4. 첫 4주 착수 계획 (solo, 즉시 실행 가능)

T1과 T2 코어의 최전단만. 모두 test-first, 원본 repo는 실행 승인 전까지 미변경.

| 주차 | 작업 | 검증 명령 | 완료 기준 |
| --- | --- | --- | --- |
| W1 | D4 path portability + 기존 2건 버그 재현 | `pytest -q biomedical-agent-teams/tests/test_path_portability.py` | 공백·CJK·Windows 경로 동일 결과, 179 tests green |
| W1–2 | D1·D3·D2 문서/라이선스/버전 트랜잭션 | `python scripts/bmat_release_check.py`(dry-run) | version/count drift 0, MIT 일치 |
| W2 | B0 registry sync | `pytest -q ...test_bmat_package_check.py && python scripts/bmat_package_check.py` | ID/count/MD drift 0 |
| W2–3 | D5 CI matrix + uv.lock | CI green (Ubuntu 3.10–3.13, macOS, Windows) | PR CI에 live 호출 0 |
| W3 | D9 release 워크플로 → `v1.1.1` 후보 | fresh-install smoke (disposable HOME) | tag↔version↔SHA 일치, benchmark=`not_run` 명시 |
| W3–4 | A1 label 분리 (behavior-change 배치 시작) | `pytest -q ...test_bmat_validate.py -k 'assurance or full_protocol'` | `Full + structure_only` 표현 가능, `workflow_label != final_label` block |
| W4 | A2 jsonschema hard-fail (CHANGELOG에 behavior change) | `python -S` negative fixture | metadata_verified↑ 요청 시 exit non-zero |

---

## 5. 리스크 레지스터 보강 (기존 26개에 추가 권장)

기존 R001–R026은 유효하다. 실행 관점에서 **누락된 4개**를 추가 권장한다.

| 신규 ID | 리스크 | 발생 신호 | 완화 | 잔여 |
| --- | --- | --- | --- | --- |
| R027 | T3 reviewer/예산 미확보로 empirical gate 무기한 지연 → v2가 hardening만으로 출시 못 함(잘못된 커플링) | go/no-go 체크리스트 미충족 | v2.0.0을 T3와 디커플(C-3); 6A로 empirical v0 확보 | low |
| R028 | v2 스키마 전환이 기존 fixture/사용자 번들을 대량 파손 | A3 후 179 tests red | fixture 마이그레이션 sub-task(C-8); v1 compat route `structure_only` cap | medium |
| R029 | `independent_review_attested`가 실무 미도달 → 사용자가 level 4를 기대하나 항상 level 3 | hook `model_identity_status=not_observed` 상시 | level 3 상한을 문서화(C-5); B6 v1.3 연기 | low |
| R030 | 문서↔CSV 마일스톤 드리프트가 릴리스 계획 혼선 유발 | v1.3.0이 로드맵에 부재(C-6) | 서사/CSV 마일스톤 단일화, CI에 문서-CSV 정합 체크 추가 | low |

---

## 6. 정합성 수정 액션 (기존 산출물에 반영할 편집 목록)

착수 전 **문서 수정만으로** 해소 가능한 항목:

1. **master §04 version table**: `v1.1.1 = non-breaking` → A2를 여기서 제거하거나 "A2는 behavior change"로 각주 (C-4).
2. **master §5.2**: "24-task pilot 실행"·B9를 v1.3으로 이동, "pilot 인프라 준비(실행 v1.3)"로 문구 변경 (C-6).
3. **master §5.1 Day-30 gate**: "adversarial 20개" → "source/claim/graph/contradiction 계열 케이스 #1–16 fail-closed"로 ID 기반 재정의 (C-7).
4. **backlog A3 effort**: fixture-migration sub-task를 명시(+2–3 pd) (C-8).
5. **§Global Constraints 의존성 문구**: "신규 Python 의존성 jsonschema만; hook은 표준 라이브러리 Node만" (C-9).
6. **verification 스크립트**: 문서 서사 마일스톤 ↔ CSV `release_milestone` 일치 체크를 추가(현재 미검증) (C-6).

---

## 7. 무엇을 하지 말아야 하는가 (기존 §8 YAGNI 재확인 + 추가)

- 벤치마크(T3)를 릴리스 크리티컬 패스로 두지 말 것 — 유지보수자 단독 가치를 막는다.
- `independent_review_attested`를 달성 가능한 기본값처럼 마케팅하지 말 것.
- v1.1.1에 동작 변경(A2/B10)을 hotfix로 포장하지 말 것.
- hook(B6) 취약 표면을 assurance ladder 안정 출시의 선행조건으로 두지 말 것.
- pilot 결과로 scientific threshold를 사후 조정하지 말 것(기존 플랜도 금지 — 재확인).

---

## 8. 요약 우선순위

1. **디커플**: v2.0.0(하드닝) ⟂ scientific-performance claim(T3). — 가장 중요.
2. **T1 위생 릴리스**를 4주 내 `v1.1.1`로 (동작 변경 제외).
3. **A2/B10 재배치**(behavior change로 명시)와 **문서-CSV 드리프트 6건 수정**.
4. **B6/level 4를 v1.3으로 연기**, v1.2는 assurance를 level 3 상한으로 정직 출시 + **D6 UX를 가시적 이득으로 전면화**.
5. **Task 6A(source-verifier 150+150)를 empirical v0로 승격** → 저비용으로 evidence 1.0 탈출.
6. **T3 go/no-go 체크리스트** 통과 시에만 120-case full run 착수.

이 로드맵은 실행 권한을 부여하지 않는다. 원본 repository 수정·commit·push·release·external DB 호출은 별도 명시적 승인 후 시작한다.
