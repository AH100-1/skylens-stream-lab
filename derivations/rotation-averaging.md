# 회전 평균 유도

참고: Hartley·Trumpf·Dai·Li, "Rotation Averaging"(IJCV 2013); Chatterjee·Govindu, "Efficient and Robust Large-Scale Rotation Averaging"(ICCV 2013). 아래는 직접 유도.

## 규약
- R_v: 세계→카메라 v. 간선 (i, j) 관측 R_ij ≈ R_j R_iᵀ (x_j = R_ij x_i + t).
- 간선 잔차 각 d_ij = ∠(R_ijᵀ R_j R_iᵀ). 해는 오른쪽 곱 R_v ← R_v G (세계 회전) 하나만큼 정해지지 않으므로 기준 정점 R_root = I.

## 정점별 현(chordal) 평균
정점 v 에 닿는 간선마다 예측 P_k = R_ij R_i (v = j) 또는 R_ijᵀ R_j (v = i).
min_{R∈SO(3)} Σ w_k ‖R − P_k‖_F² = max tr(Rᵀ M), M = Σ w_k P_k.
M = U Σ Vᵀ 이면 R = U diag(1, 1, det(UVᵀ)) Vᵀ (직교 프로크루스테스).
가우스–자이델로 정점을 차례로 갱신하면 비용이 줄지만, 사슬 길이 L 의 저주파 오차는 반복마다 O(1/L²) 비율로만 줄어 느리다.

## 강건 가중치
IRLS 코시 가중치 w_k ← w_k / (1 + (d_k/σ)²), σ = 2°. 이상치 문턱 5° 넘는 간선은 마지막 단계에서 뺀다.

## 리 대수 선형 최소제곱
왼쪽 섭동 R_v ← exp([ω_v]×) R_v. Q = R_j R_iᵀ 라 두면
exp(ω_j) Q exp(−ω_i) = exp(ω_j) exp(−Q ω_i) Q   (Q exp(a) Qᵀ = exp(Q a) 이용).
잔차 벡터 r_ij = log(Q R_ijᵀ) ∈ ℝ³ 에 대해 BCH 1차 근사:
r_ij(ω) ≈ r_ij + ω_j − Q ω_i.
야코비안 블록 ∂r/∂ω_j = I, ∂r/∂ω_i = −Q. 정규 방정식 H δ = −g,
H = Σ w Jᵀ J, g = Σ w Jᵀ r, 기준 정점 변수는 뺀다(게이지 고정). H 는 대칭 양정치(연결 그래프) → 촐레스키.
갱신 후 Q 와 r 을 다시 계산해 몇 번 반복(가우스–뉴턴). 잡음 없는 그래프에서 7e-14° 까지 수렴.

## 다수결 탐욕 초기화
무작위 신장 트리는 간선 10% 가 이상치일 때 트리(간선 N−1 = 239개) 전체가 정상일 확률이 0.9²³⁹ ≈ 1e-11 이라 쓸모없다.
대신 놓인 정점과 이어진 간선이 가장 많은 정점부터 놓고, 그 정점에 대한 예측들 가운데 서로 5° 안에서 가장 많이 일치하는
무리의 현 평균을 쓴다. 이웃 예측이 2개 이상이면 이상치 하나는 다수결로 걸러진다. 시작 정점 8개로 반복해 문턱 안 간선이
가장 많은 초기값을 고른다.

## 정답 비교
추정과 정답의 세계 회전 차이 G = argmin Σ ‖R_est,v G − R_gt,v‖ = proj_SO3(Σ R_est,vᵀ R_gt,v) 로 맞춘 뒤 정점별 각 오차.
