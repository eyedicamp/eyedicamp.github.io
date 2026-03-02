---
layout: post
title: "[Project] Super Position — QUBO 기반 회의시간 최적화 (SA/QA, D‑Wave)"
date: 2026-03-02 17:30:00 +0900
excerpt: "QUBO로 회의 시작 시간을 정식화하고 Simulated Annealing / Quantum Annealing(D‑Wave)로 최적해를 찾는 웹 서비스"
categories: [Projects, QuantumComputing]
tags: [QUBO, quantum-annealing, simulated-annealing, dwave, fastapi, javascript, optimization]
---

> TL;DR — **주간 시간표(가중치/선호/야간 페널티)를 입력하면, QUBO로 회의 시작 시간을 최적화하여 최적 후보(Top‑K)와 함께 시각화까지 제공하는 웹 앱**입니다.  
> Demo: https://eyedicamp.github.io/SuperPosition/frontend/

---

## 1. 문제 정의: “모두가 되는 시간”이 아니라 “가장 좋은 시간”

실제 팀 미팅은 단순히 *가능한 시간*만 찾는 문제가 아니라, 아래 조건들을 같이 만족해야 합니다.

- 👥 참석자 수를 최대화 (가능하면 모두 참석)
- ⚖️ 사람마다 중요도가 다름 (의사결정권자/필수 인원 가중치 ↑)
- ⭐ “선호 시작 시간대”가 존재 (예: 10–12시 사이면 좋다)
- 🌙 너무 늦은 시간은 피하고 싶음 (야간 페널티)

즉, 목표는 “가능한 시간(any feasible)”이 아니라 **“목적함수를 최대화하는 최적 시간(best)”**입니다.

---

## 2. 시스템 개요 (Frontend ↔ Backend)

- **Frontend (GitHub Pages)**  
  - 랜덤 시간표 생성 / CSV 업로드  
  - 파라미터 입력(회의 길이, 선호 윈도우, 가중치, 야간 페널티 등)  
  - 결과(최적 시작 시간 + Top‑K 후보)를 그리드로 시각화
- **Backend (FastAPI, Render 등 배포 가능)**  
  - 입력을 QUBO/BQM(Binary Quadratic Model)로 구성  
  - **SA(dwave-neal)** 또는 **QA(D‑Wave QPU)**로 최적화  
  - best_start/best_end + 참석자 목록 등을 반환

(개념도)

```text
[Browser]
  ├─ Generate / Upload CSV
  ├─ Set params (L, pref, weights, late penalty)
  └─ POST /solve  ───────────────▶  [FastAPI Backend]
                                   ├─ Build QUBO/BQM
                                   ├─ Solve via SA or QA(QPU)
                                   └─ Return best + Top-K
```

---

## 3. 입력 데이터 모델

시간을 **슬롯(slot)** 단위로 쪼갭니다.

- 슬롯 크기: `slot_minutes` (예: 30분)
- 주간 전체 슬롯 수: $T$
- 회의 길이(슬롯 수): $L$  
  예) 180분 회의, 30분 슬롯이면 $L=6$

사람 $i \in \{1,\dots,N\}$에 대해:

- 가중치: $w_i \ge 0$
- 슬롯별 가능 여부: $a_{i,t}\in\{0,1\}$
- 시작 선호 여부: $p_{i,s}\in\{0,1\}$  (frontend의 `pref_start_ok`)

CSV는 다음 헤더를 사용합니다.

```csv
slot_minutes,person_id,weight,day,time,available,pref
```

---

## 4. 목적함수 설계 (Score)

회의가 시작 슬롯 $s$에서 시작해 $L$슬롯 동안 진행된다고 할 때,  
사람 $i$가 **회의 전체 구간에 참석 가능한지**는 다음으로 정의합니다.

$$
A_{i,s}=\prod_{k=0}^{L-1} a_{i,s+k} \in\{0,1\}
$$

즉, 연속 $L$개 슬롯이 모두 1이면 참석 가능.

또한 “야간 시작/야간 겹침”을 싫어한다고 하면, late 기준 슬롯 $t_{\text{late}}$ 이후에 겹치는 슬롯 수는

$$
\ell_s=\sum_{k=0}^{L-1}\mathbf{1}\{s+k\ge t_{\text{late}}\}.
$$

이를 바탕으로, 후보 시작 시간 $s$의 **유틸리티(가치)**를 아래처럼 둡니다.

$$
W_s =
\underbrace{\sum_{i=1}^{N} w_i A_{i,s}}_{\text{(1) 가중 참석}}
\;+\;
\underbrace{\beta \sum_{i=1}^{N} w_i A_{i,s}p_{i,s}}_{\text{(2) 선호 보너스}}
\;-\;
\underbrace{\rho\,\ell_s}_{\text{(3) 야간 페널티}}
$$

- $\beta$: `pref_bonus`
- $\rho$: `late_penalty_per_slot`

---

## 5. QUBO 정식화 (One-hot baseline)

### 5.1 결정변수

가능한 시작 슬롯 후보가 $K=T-L+1$개일 때,

$$
x_s\in\{0,1\},\quad s=0,\dots,K-1
$$

여기서 $x_s=1$은 “회의 시작은 $s$로 선택”을 의미합니다.

### 5.2 제약: 정확히 하나 선택

$$
\sum_{s=0}^{K-1} x_s = 1
$$

QUBO에서는 이를 **패널티**로 넣습니다.

$$
\lambda\left(\sum_{s=0}^{K-1}x_s-1\right)^2
$$

### 5.3 최종 에너지(최소화 형태)

Quantum annealer는 에너지를 최소화하므로, 유틸리티를 최대화하려면 $-W_s$를 최소화하면 됩니다.

$$
E(\mathbf{x})
=
\lambda\left(\sum_{s=0}^{K-1}x_s-1\right)^2
\;-\;
\sum_{s=0}^{K-1} W_s x_s
$$

이를 전개하면 선형/이차항이 생기고, QUBO 형태로 떨어집니다.

> GitHub/KaTeX 호환성을 위해 체인부등식(subscript에 $0\le s<t\le \cdots$) 대신 **이중 합**으로 표기합니다.

$$
E(\mathbf{x})
=
2\lambda\sum_{s=0}^{K-1}\sum_{t=s+1}^{K-1} x_s x_t
\;+\;
\sum_{s=0}^{K-1}(-W_s-\lambda)x_s
\;+\;
\lambda
$$

따라서 QUBO 행렬 $Q$는 개념적으로 다음 구조를 가집니다.

- 대각(선형): $Q_{s,s}=-W_s-\lambda$
- 비대각(이차, $s<t$): $Q_{s,t}=2\lambda$

---

## 6. Domain-wall encoding (연속 정수 후보에 유리)

One-hot은 직관적이지만 변수 개수가 $K$로 커집니다.  
회의 시작 시간은 “연속 정수 인덱스”이므로, **domain-wall encoding**을 쓰면 더 구조적으로 다룰 수 있습니다.

### 6.1 아이디어

경계(boundary) 하나로 시작 인덱스를 표현합니다.

- 도메인 변수: $d_j \in \{0,1\},\; j=0,\dots,K-2$
- 형태: $d_0, d_1, \dots$가 **어느 지점부터 0으로 떨어지는 단조 시퀀스(1…1 0…0)**

시작 인덱스 $s$를 다음처럼 읽을 수 있습니다.

- $x_0 = 1 - d_0$
- $x_s = d_{s-1} - d_s \quad (1\le s \le K-2)$
- $x_{K-1} = d_{K-2}$

즉, 경계에서 차분(difference)을 취하면 one-hot과 동치가 됩니다.

### 6.2 단조성 제약을 QUBO로

단조성은 $d_j \ge d_{j+1}$이어야 하고, 이를 위반하는 패턴은 $(d_j, d_{j+1})=(0,1)$입니다.  
따라서 다음 패널티를 추가하면 됩니다.

$$
P_{\text{mono}}=\gamma\sum_{j=0}^{K-3} d_{j+1}(1-d_j)
$$

여기서 $d_{j+1}(1-d_j)$는 자연스럽게 **이차항**을 만들어 QUBO로 들어갑니다.

- 올바른 패턴(1→1, 1→0, 0→0)은 페널티 0
- 잘못된 패턴(0→1)만 페널티 1

> 실제 구현에서는 QPU 임베딩 난이도/계수 스케일/해 품질을 고려해 one-hot과 domain-wall 중 하나를 선택할 수 있습니다.

---

## 7. API 및 Payload 예시

Frontend → Backend로 `/solve`를 호출할 때 핵심 payload는 아래 구조입니다(요약).

```json
{
  "num_people": 1000,
  "slot_minutes": 30,
  "meeting_len_slots": 6,
  "availability": [[0,1,1,...], ...],
  "pref_start_ok": [[0,0,1,...], ...],
  "weights": [1.0, 2.0, ...],
  "pref_bonus": 0.7,
  "late_hour": 20,
  "late_penalty_per_slot": 1.0,
  "solver_mode": "qa",
  "dwave_token": "..."
}
```

---

## 8. SA vs QA: 왜 둘 다 지원했나?

- **SA (Simulated Annealing)**  
  - 빠르게 돌릴 수 있고 개발/디버깅에 유리  
  - QPU 없이도 실험 가능
- **QA (Quantum Annealing, D‑Wave)**  
  - 실제 QUBO를 하드웨어에서 샘플링  
  - 임베딩/계수 범위/샘플 수 같은 현실 제약을 경험할 수 있음  
  - “양자 최적화 서비스” 데모로 설득력이 큼

이 프로젝트는 **같은 목적함수(QUBO)를 서로 다른 샘플러에 태우는 구조**라서, SA/QA 비교 실험 및 시연이 용이합니다.

---

## 9. 로컬 실행 / 배포 메모

### Frontend
정적 파일이므로 아무 로컬 서버로도 실행 가능:

```bash
cd frontend
python -m http.server 5173
```

### Backend
FastAPI 기준(예시):

```bash
cd backend
pip install -r requirements.txt
uvicorn app:app --host 0.0.0.0 --port 8000 --reload
```

---

## 10. 회고 & 다음 개선 아이디어

- ✅ “회의시간 최적화”를 **QUBO로 완전하게 모델링**하고, 실제 QPU까지 연결해 본 점이 가장 큰 성과
- ✅ Frontend에서 CSV/시각화까지 제공해 **서비스 형태로 시연 가능한 완성도** 확보

향후 확장 아이디어:

- 여러 회의(다중 이벤트) 최적화(충돌 제약 포함)
- 캘린더 연동(Google Calendar) + 자동 참석자 availability 추출
- Top‑K 후보를 Pareto(참석/선호/야간) 기준으로 다목적 최적화 형태로 제공

---
