---
layout: post
title: "[Project] Super Position — Domain-wall QUBO로 회의시간 최적화 (SA/QA, D‑Wave)"
date: 2026-03-02 17:30:00 +0900
excerpt: "주간 시간표를 QUBO(BQM)로 정식화하고 Simulated Annealing / Quantum Annealing(D‑Wave)로 최적 회의 시작 시간을 찾는 웹 서비스"
categories: [projects, quantumcomputing]
tags: [QUBO, domain-wall, quantum-annealing, simulated-annealing, dwave, fastapi, javascript, optimization]
---

## 한 줄 요약

주간 시간표(가중치/선호/야간 페널티)를 입력하면, **Domain-wall QUBO(BQM)** 로 “회의 시작 시간”을 최적화해 **최적 후보(Top‑K)와 시각화**까지 제공하는 웹 앱.

- Live Demo: https://eyedicamp.github.io/SuperPosition/frontend/
- GitHub Repo: https://github.com/eyedicamp/SuperPosition

---

## 1) 문제 정의

현실적인 미팅 시간 선택은 “가능한 시간(any feasible)”이 아니라 **“가장 좋은 시간(best)”** 문제다.

고려 요소(본 프로젝트 구현 기준):

- 참석 가능 인원을 최대화하되, 사람마다 중요도가 다름(가중치)
- “선호 시작 시간대”를 반영(선호 보너스)
- 늦은 시간대는 피하고 싶음(야간 페널티)
- 회의 길이(분)가 슬롯 크기(분)로 나누어 떨어져야 함 (예: 180분 회의, 30분 슬롯)

---

## 2) 웹 앱 기능(Frontend)

프론트는 순수 정적 페이지(HTML/CSS/JS)로 구성되어 GitHub Pages에서 바로 동작한다.

### 2.1 Data 패널

1) Random Schedule 생성
- 사람 수 / 슬롯 크기 / 시드(seed)로 주간(월–일) 시간표 랜덤 생성
- 기본 활성 시간대(예: 09:00–21:00)를 기반으로 “추가 바쁨 블록”을 랜덤 삽입하여 현실적인 스케줄을 만든다.
- 생성된 스케줄은 브라우저 저장소(LocalStorage/SessionStorage)에 저장된다.

2) CSV Upload
- CSV 템플릿 다운로드(현재 people/slot 설정에 맞춰 생성)
- CSV 업로드 후 로드
- 로드된 스케줄은 “View Timetable” 페이지에서 요일 탭 기반으로 시각화 가능

### 2.2 Optimization 패널

- Backend API Base URL 입력(예: Render 배포 주소)
- Meeting length / Preference window length / pref_bonus / late_hour / late_penalty
- Important people(예: `0,3`) + Important weight로 가중치 조정
- Solver 선택
  - SA: Simulated Annealing (neal)
  - QA: Quantum Annealing (D‑Wave QPU) — Token 입력 + QPU 모델 목록 로딩
- 실행 결과:
  - 최적 시작/종료 시각, 점수, 참석자 목록, 선호 만족자 목록, solver meta 정보 출력
  - Top‑K 후보 리스트 출력(각 후보의 점수/참석수/가중 참석/가중 선호/야간 겹침)
  - Top‑K를 주간 그리드에 오버레이하여 “어떤 시간 슬롯이 얼마나 좋은 후보에 포함되는지” 시각화

---

## 3) 스코어(목적함수) 정의

시간을 일정한 슬롯으로 이산화한다.

- 슬롯 크기: `slot_minutes`
- 주간 슬롯 수: `T = 7 * slotsPerDay`
- 회의 길이(슬롯 수): `L = meetingMinutes / slot_minutes`
- 구현에서는 시작 후보를 `s = 0 .. (T - L - 1)`로 두어 후보 개수는 `n = T - L`이다.

사람 `i`에 대해:

- 가중치: `w_i`
- 가용(availability): `a_{i,t} ∈ {0,1}` (슬롯 t에서 참석 가능 여부)
- 시작 선호(preference): `p_{i,s} ∈ {0,1}` (시작 슬롯 s가 선호 윈도우에 포함되는지)

회의가 시작 슬롯 `s`에서 시작하여 길이 `L`로 진행될 때, 사람 i가 “회의 전체”에 참석 가능한지:

$$
A_{i,s} = \prod_{k=0}^{L-1} a_{i,s+k}
$$

야간 페널티를 위해, 하루 내 late 기준 슬롯 인덱스를 `t_late`로 두고(예: late_hour=20이면 20시 이후 슬롯), 회의가 늦은 시간과 겹치는 슬롯 수:

$$
\ell_s = \sum_{k=0}^{L-1} \mathbf{1}\{ (s+k)\bmod slotsPerDay \ge t_{late} \}
$$

이때 후보 시작 시간 s의 점수(가치)를 다음처럼 둔다(프론트/백엔드 동일한 형태):

$$
W_s =
\sum_{i=1}^{N} w_i A_{i,s}
+ \beta \sum_{i=1}^{N} w_i A_{i,s} p_{i,s}
- \rho \, \ell_s
$$

- β = `pref_bonus`
- ρ = `late_penalty_per_slot`

---

## 4) Domain-wall QUBO(BQM) 정식화 (구현 기반)

이 프로젝트의 백엔드는 “시작 슬롯 s” 자체를 one-hot로 두기보다,
**Domain-wall 변수 z로 시작 인덱스 k를 표현**한다. (코드 구현이 이 방식)

### 4.1 변수 정의

가능한 시작 슬롯 후보 개수를 `n`이라 하자.

- `n = T - L`
- 후보 인덱스: `k ∈ {0, 1, ..., n-1}`
- Domain-wall 이진 변수: `z_0, z_1, ..., z_{n-2} ∈ {0,1}`

해석:
- `z_j = 1`이면 “k가 j보다 큰 쪽”을 선택했다고 볼 수 있고,
- 최종 선택 인덱스는 `k = sum(z_j)`로 복원한다.
- 올바른 해는 `111...1100...00` 형태(단조 감소)여야 한다.

### 4.2 점수 테이블을 선형항으로 바꾸기

모든 후보 k에 대해 `scores[k] = W_k`를 미리 계산한 뒤, Domain-wall 형태에서 다음이 성립한다.

$$
Score(k) = scores[0] + \sum_{j=0}^{n-2} (scores[j+1]-scores[j])\, z_j
$$

따라서 “점수 최대화”는 “에너지 최소화”로 바꾸면

$$
E_{obj} = -Score(k)
$$

즉 선형항(Linear)로 들어간다.

### 4.3 단조성(0→1 금지) 제약의 QUBO 패널티

Domain-wall에서는 `0 → 1` 패턴이 나오면 안 된다.
이를 막는 패널티는 다음과 같이 줄 수 있다.

$$
P_{mono} = A \sum_{j=0}^{n-3} z_{j+1}(1 - z_j)
$$

- `z_{j+1}(1 - z_j)`는 (z_j, z_{j+1}) = (0,1)일 때만 1이므로 위반에만 페널티
- 전개하면 선형(+A z_{j+1})과 이차항(−A z_j z_{j+1})로 QUBO/BQM에 자연스럽게 들어간다.

최종적으로 백엔드가 푸는 BQM은 개념적으로

$$
E(z) = E_{obj}(z) + P_{mono}(z)
$$

형태다.

### 4.4 패널티 스케일 A 선택(구현)

구현에서는 후보 점수 배열 `scores[k]`의 **인접 차이**를 이용해 A를 자동 설정한다.

- `max_delta = max_k |scores[k+1] - scores[k]|`
- `A = max(5.0, 5.0 * max_delta + 1.0)`

의도는 “점수 차이(목적함수 변화)”보다 단조성 위반 페널티가 충분히 커지도록 스케일을 맞추는 것이다.


### 4.5 실전 처리: repair + decode

샘플러가 완벽히 단조 해만 내주지 않을 수 있으므로,
백엔드는 샘플을 받은 뒤 다음과 같이 “수리(repair)” 후 k를 복원한다.

- 왼쪽에서 오른쪽으로 보며 0이 나오면 그 뒤는 모두 0으로 강제
- 그 다음 `k = sum(z_j)`로 start index를 결정
- 여러 샘플 중 `scores[k]`가 최대인 k를 최종 해로 선택

---

## 5) SA vs QA 동작 방식

### SA (neal)
- BQM을 Simulated Annealing으로 샘플링
- 구현 기본값:
  - `num_reads = 400`
  - `num_sweeps = 6000`
  - `beta_range = (0.01, 6.0)`

### QA (D‑Wave QPU)
- D‑WaveSampler + EmbeddingComposite로 QPU에서 샘플링
- 구현 기본값:
  - `num_reads = 100`
  - solver는 사용자가 선택하거나(모델명 지정), 미지정 시 “온라인 QPU 자동 선택”
- 프론트에서 D‑Wave 토큰을 넣으면 `/dwave/solvers`로 QPU 모델 목록을 로딩해 선택 가능

---

## 6) API 요약

### POST `/solve`
입력(요약):

- `availability` : [P][T] boolean
- `pref_start_ok`: [P][T] boolean
- `weights`: length P
- `meeting_len_slots`, `slot_minutes`
- `pref_bonus`, `late_hour`, `late_penalty_per_slot`
- `solver_mode`: "sa" or "qa"
- (qa) `dwave_token`, `dwave_solver`

출력(요약):

- `best_start`, `best_end`, `score`
- `attendees` (회의 전체 참석 가능한 사람 인덱스)
- `pref_hit_people` (best_start가 선호 시작인 사람 인덱스)
- `meta` (solver 정보/파라미터 등)

---

## 7) 배포 구조

- Frontend: GitHub Pages (정적 파일)
- Backend: Render 등 FastAPI 배포
- 프론트의 “Backend API Base URL”에 배포된 백엔드 주소를 넣어 호출

---

## 8) 개선 아이디어

- 단일 회의가 아니라 다중 회의(충돌 제약 포함)로 확장
- 캘린더 연동(구글 캘린더)로 availability 자동 생성
- 다목적 최적화(참석/선호/야간을 Pareto로 제공)

---
