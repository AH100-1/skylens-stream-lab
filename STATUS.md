# 현재 상태

- 상태: 진행 중(마무리)
- 마지막 갱신: 2026-10-02T04:25Z (03:49Z 시작, 4 코어 측정 기계, 9 묶음 동시 — 부하 평균 13~25, 시간 수치는 부풀려짐)
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | E01 pipeline | `feat/pipeline` 69a6363 | experiment/pipeline 6ea812e | 편대 겹침에 맞춘 카메라 간 짝 일정(F(p)–R(p+12..40), F(p)–L(p+16..40), 간격 4)으로 합성 40위치 × 3대 320×180 에서 **등록 120/120**(이전 18/54). 중심 오차 중앙 3.82 m·최대 12.76 m, 점→정답 표면 중앙 4.95 m, 재투영 초벌 2.05→정밀 0.24 px, `run` 80 s. verify 7항목 중 5 통과(preview_align 잔차 중앙 6.16 m > 6, preview_vs_refined 높이 차 중앙 6.18 m 실패; refined_overlap 은 구역 1개라 해당 없음). 총괄 재검증 fmt·clippy 통과, 전체 시험 진행 중 |
  | pipeline-sparse | `feat/pipeline-sparse` 560c063 | experiment/pipeline-sparse b39aeda | 짝 일정 설정화(F-197 기본값), 44위치 132/132 등록·그래프 연결 1개. 단계별 오차 표(위치 초기 4.55 m → BA 후 닮음 1.22 m → GPS 정렬 중앙 1.28 m·최대 3.01 m), 점 중앙 0.95 m — 목표(1 m·3 m·0.5 m) 미달. 초벌/정밀 분리, `refine_positions` 한 함수로 |
  | pipeline-dense | `feat/pipeline-dense` 4d66fe1 | experiment/pipeline-dense d0cb6fa | 깊이 추정기 교체 지점(`DepthEstimator`), 평면 스윕 rayon·창 통계 캐시·상위 2 집계·거친→고운 2단. 48장 480×270 462442점·표면 중앙 0.027 m·90% 0.090 m·58~64 s(부하 15~21). 960 px 12장 5.2 s/장(시험 없음). 30 s 목표 미확인 |
  | tracks (PR #15) | `feat/tracks` c31c759, 라벨 | experiment/tracks fad7351 | F-123 처리: 짝 내 국소 변위 중앙값 거르기(40 px, 96 px 칸). 유지 50·30% × 오대응 0·1% 순도 ≥ 0.9992·완전도 ≥ 0.9976. 총괄 재검증 fmt·clippy·`tracks` 시험 10 통과. F-127 Split 16.75 s(부하) 열림 |
  | tracks-alt | `feat/tracks-alt` df3b1c3 | experiment/tracks-alt e02cf18 | 삼각 순환 지지도 순 합치기: 240장 4.17 s(F-127 기준 5 s 안, 부하). 30%·1% 순도 0.9673 로 미달 — tracks 쪽 방식이 정확도 우위 |
  | translation-averaging (PR #37) | `feat/translation-averaging` bd271ab | (변경 없음) | CI 실패 원인: clippy `needless_range_loop`(c5dae23 의 채우기 반복) — 수정, 총괄 재확인 fmt·clippy 통과. F-214 80경우 표는 부하로 시험이 끝나지 않아 미측정, 라벨 안 붙임 |
  | translation-averaging-formation | 진행 결과 아래 | | F-213 |
  | patchmatch (PR #6) | 커밋 없음 | | 거친 층 이웃 4장 축소 실험(960×540 이웃 8장 22.15→13.53 s, 부하 18~20) 작업 트리에만 있음, 시험을 다 돌리지 못해 커밋 안 함 |
  | patchmatch-fast | 커밋 없음 | | 고운 시점 재사용·고운 층 전파만·4단 피라미드로 표본 870M→367M, 1스레드 16.5→9.5 s(부하). 정확도 시험 통과했으나 커밋하지 못함 |
- 끝까지 흐름 진척: 합성 장면에서 synth → run → verify 가 돌고 PLY·스냅샷·manifest 가 나온다. 세 카메라 모두 등록(120/120). 남은 것: GPS 고정항 없는 BA 로 초벌/정밀 높이 차(6 m), `sparse::reconstruct`·`dense::region_cloud`·tracks·위치 평균을 pipeline 에 연결(현재 pipeline 은 자체 희소 초기화·보간 깊이 사용), 위치 단위 차례 처리(초벌 즉시·정밀 병렬·sim3 재정렬).
- 다음 할 일:
  1. pipeline 에 pipeline-sparse(짝 목록 인자 추가)·pipeline-dense(main 융합 필드 반영) 합치기, BA 에 GPS 사전항.
  2. F-214 80경우 표를 부하 없는 때 측정, F-215·F-218.
  3. patchmatch 두 갈래 커밋·측정, F-048.
  4. tracks Split 속도(F-127).
- 막힌 점:
  - 시간 기준(F-048·F-127·밀집 30 s)은 부하 없는 기계에서만 판정 가능.
  - patchmatch·patchmatch-fast 변경은 이번 실행에서 커밋하지 못하고 작업 트리에만 남음(다음 실행에서 다시 해야 함).
  - 결정 필요: SPEC §3.2 카메라 간 짝 일정 개정(F-197, 실측 겹침 기준 F(p)–R(p+12..40)·F(p)–L(p+16..40)).

## 직전 실행 기록 (00:25Z 시작분)
- 마지막 갱신: 2026-10-02T04:06Z (2026-10-02T04:06Z 시작)
- 이번 실행(00:25Z 시작, 4 코어 측정 기계에서 12 묶음 동시 진행 — 부하 평균 24~41, 시간 수치는 부풀려짐):
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | P04 io-followup | PR #35 (6f045af) | PR #52 (→ scaffold) | F-134·F-135·F-155(main 0afc818 align 기준)·F-168·F-163·F-170 처리. F-169 재측정 ff3a769 lib 140(피드백 142 와 다름), F-016 현황만. 전체 시험 core 190 통과·8 무시, 4+4+6+21+2 통과 |
  | P13 stream-followup | PR #36 (03fa95c) | PR #53 (→ progressive-stream) | F-066·F-176·F-177 처리, F-175 코드(반경/√3 칸·상자 가지치기, 무차별 대조 일치) — 시간 비 미측정. 직접 재검증 core lib 199 통과·9 무시·1 실패(`five_point_terminates_on_many_seeds` 시간 단언, 부하 40) |
  | P15 benchmarks-args | PR #33 (4b4e869) | PR #50 (→ benchmarks) | F-184 처리, F-132 인자 없는 bench 471.6 s(부하 중), F-149 24장 `회전 평균(검증 결과)` 정렬 오차 중앙 69.3°, F-185 정정. 직접 재검증 fmt·perf_structure 6 통과, 새 main 과 충돌 없음 |
  | P03 two-view-hardening | PR #13 갱신 (ce66257), 라벨 안 붙임 | experiment/two-view-hardening e429b66 | F-158(`verify_pair`, bench 단계 5b 연결 — 맡은 파일 밖 pipeline.rs 한 곳)·F-159·F-160 처리. bench 2° 초과 간선 1→0, 정렬 오차 중앙 0.265→0.031°, 시점 8/24 그대로(다른 카메라 짝 18개 모두 검증 실패, F-148). lib 193 통과·1 실패(`large_chain_graph_is_fast` 5.24 s, 부하)·11 무시, 통합 시험 미실행 |
  | P05 rotation-averaging-followup | `feat/rotation-averaging-followup` 2983a77 (PR 안 엶) | experiment/rotation-averaging-followup 1d27a32 | F-171·F-172·F-173·F-138 처리, F-047 일부. 전체 시험 미실행 |
  | P20 ba-robust | `feat/ba-robust` 995020e (PR 안 엶) | experiment/ba-robust 0078057 | F-165·F-164·F-162·F-167·F-174 처리, F-166 대부분, F-036 원인(편대 장면 600점 중 597점이 한 카메라 종류에서만 관측, 종류 간 공유 트랙 0~13) — 무시 유지. ba:: 21 통과·3 무시, 전체 시험 미실행 |
  | P12 dataset-io | PR #4 갱신 (3bbf74b), 라벨 안 붙임 | experiment/dataset-io fd97ea7 | main 합침(main.rs run·verify 둘 다), F-065 로더 경계 표·스트림 `split_regions` 와 800경우 일치 시험. 전체 시험 미확인(core 219 통과·1 실패 시간 단언) |
  | tracks | PR #15 갱신 (c8fa58d), 라벨 안 붙임 | experiment/tracks 23f49c5 | F-123 일부: 재현율 30%·오대응 1% Split 순도 0.9672→0.9720(기준 0.99) — `sparse_recall_keeps_tracks_whole` 실패 상태 |
  | P11 fusion-tests | `feat/fusion-tests` 4995b33(00:46 같은 계정의 다른 커밋), 이쪽 커밋은 `feat/fusion-ratio-alt` e9fe7ae | experiment/fusion-tests 61cab77 (새 노트 없음) | F-179·F-161·F-178 같은 범위를 두 갈래가 따로 고침 — 하나를 골라야 함. 검증 미완 |
  | P06 translation-averaging | `feat/translation-averaging` 5f8fe0b (main 병합만) | experiment/translation-averaging 05c9681 | 시험 편대가 실측 배치와 달랐음(실측 배치로 바꾸면 점 제약 없이 중심 RMS 7.5~9.1 m). 무너짐은 점 관측 이상치가 있을 때만(이상치 0% 면 240/240·0.11~0.14 m). 코드 변경 없음 |
  | P10 patchmatch | `feat/patchmatch` 663c7d2 (main 병합만) | experiment/patchmatch 8ace1c6 | F-151 합친 트리 컴파일 통과 확인. F-048·F-050·법선 시험 2개 미처리(부하로 전후 비교 불가, 960px 257.67 s 부하 중) |
  | P14 verify-followup | PR #38 (91608c8) | PR #55 (→ scaffold) | F-180(같은 좌표 9만 점 39.4 s → 0.73 s, 20만×20만 < 1 s·정답 0.5 일치)·F-181(단계 상한, 목록 10개, JSON 깊이 64) 처리, F-089 SPEC 결정 대기. 전체 시험 239 통과·0 실패·8 무시, 새 main 과 충돌 없음 |
- 막힌 점:
  - 같은 시각 00:28Z 시작 실행(아래 기록)과 묶음이 겹쳤다. 00:23 무렵까지 다른 실행이 PR #29~#31·#24 를 갱신했고, 00:46 에 `feat/fusion-tests` 에 같은 계정 커밋이 올라와 이쪽 푸시가 거부됨 — 실행이 겹친다. STATUS 를 10분마다 갱신하지 않는 실행이 있다.
  - 12 묶음 동시 빌드로 부하 평균 40 안팎, 릴리스 빌드 5~11분 — 대부분 묶음이 전체 시험을 못 돌림. 시간 단언 시험 `five_point_terminates_on_many_seeds`·`large_chain_graph_is_fast` 가 부하에서 실패(단독 통과).
  - F-148: 다른 카메라 짝이 하나도 검증을 통과하지 못함 → 카메라별 회전 묶음을 GPS 진행 방향으로 잇는 방안 또는 SPEC §3.2 ±4 위치 확대 결정 필요.
  - F-036: 합성 편대 장면에 카메라 종류 간 공유 트랙이 거의 없음 — 장면 생성 쪽(synth) 변경 필요.
- 결정 필요(이전 그대로 + 추가):
  1. F-148 다른 카메라 연결 방안(위).
  2. P11 두 갈래(4995b33 / e9fe7ae) 중 선택.
  3. 이전 기록의 SPEC §3.4·§2 report.json·step 형 결정.
- 다음 할 일:
  1. 미재검증 브랜치(#13·#4·rotation-averaging-followup·ba-robust) 부하 없는 상태에서 전체 시험 후 PR·라벨.
  2. 묶음 수를 4~6 개로 줄여 빌드 부하를 낮출 것.
  3. P06 실측 배치에서 깨끗한 점 관측도 RMS 0.8~4.4 m 인 원인(s ≥ 1 하한 + L1 축척 수축).
  4. tracks F-123 남은 0.028, patchmatch F-048 속도.

## 같은 시각 실행 기록 (00:28Z 시작분)
- 마지막 갱신: 2026-10-02T04:06Z (2026-10-02T04:06Z 시작)
- 이번 실행(00:28Z 시작, 4 코어 측정 기계, 11 묶음 동시 진행 — 부하 평균 7~25):
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | matching-followup | PR #34 (9292466, main 병합 0c3c79a), 라벨 | PR #51 (→ matching-refine, fc46dfe) | F-182·F-183·F-141·F-142 처리, F-150 matching 쪽 확인. F-003 미착수. 직접 재검증 fmt·clippy·전체 시험 통과(core 215 통과·12 무시) |
  | translation-averaging (P06) | PR #37 (0e5981b, main 병합 1c47b2e), 라벨 | PR #54 experiment/translation-averaging-staged (→ translation-averaging, c8e4c04) | 잡음+이상치 시드 1~5 짝 이상치 10%·20% 모두 240/240, RMS 0.115~0.133 m, 무시 시험 해제. 직접 재검증 통과(core 218 통과·13 무시) |
  | fusion-tests (P11) | PR #32 갱신 (3d9eb00), 라벨 | PR #49 갱신 (513feac) | F-179·F-178·F-161 처리, F-069 같은 드론 8장 동의 허용으로 348914점·0.3 m 초과 0·최대 0.057 m. F-113 7.95 s(목표 2 s). 직접 재검증 통과(core 202 통과·7 무시), main 과 충돌 없음 |
  | ba-robust | `feat/ba-formation` 3d2a409 (PR 없음, 미재검증) | 노트 없음 | F-165·F-164·F-162·F-174·F-166·F-167, F-036 원인 = 편대 장면에서 두 드론이 함께 보는 점 약 3/600(정보 행렬 영 고윳값 3). 드론 공유 점 150 + 공분산 기준으로 `least_squares_on_formation_scene` 시드 41~50 수렴 6~7회·약한 방향 |z| 최대 1.97. 강건 손실 편대 시험은 무시 유지. `feat/ba-robust` 에 같은 항목 커밋 9cd1b0a 가 따로 올라와 합치지 않음 |
  | patchmatch (P10) | `feat/patchmatch-profile` 175e0e5 (PR 없음, 미재검증) | 노트 없음 | 단계별 시간·세밀 단계 이웃 4장·같은 평면 전파 생략. 법선 전용 정밀화는 효과 없음(7.41°·6.99°). 960px 시간 미측정 |
  | two-view-hardening (P03) | 푸시 없음 | 없음 | F-158·F-159 를 같은 시각 다른 커밋(12c66be·ce66257)이 처리해 중단. 측정: 다른 카메라 짝 비율 검사 대응 2~9개뿐, 기본 방위(−3°·125°·−116°)에서 겹침이 거의 없어 F-148 을 두 시점 검증으로는 잇지 못함 |
  | tracks | 푸시 없음 | 없음 | 다른 커밋 c8fa58d 와 겹쳐 중단. 단일 간선 결합 확률 거부 시도: 순도 0.9672→0.9724, 완전도 30% 0.787 로 기각. 불순도는 대부분 단일 간선 결합이 아님 |
  | bench-args (P15) | 푸시 없음 | 없음 | F-184 가 `feat/benchmarks-args` cd6accf 와 겹쳐 중단. 인자 없는 bench 286 s(부하 24.6), 회전 평균(검증 결과) 간선 2° 초과 27/101·정렬 오차 중앙 69.26°(F-148 그대로) |
  | io-readme (P04) | 푸시 없음 | 없음 | README·F-168·F-163 이 `feat/io-followup` 과 겹쳐 버림. F-169: ff3a769 lib 시험 수 `--list` 140 |
  | stream-followup (P13) | 푸시 없음 | 없음 | 같은 이름 브랜치에 다른 커밋 03fa95c(stream.rs 외 4 파일 −1450줄, 옛 main 에서 딴 것으로 보임 — 병합 전 확인 필요) |
  | verify-followup (P14) | 푸시 없음 | 없음 | 같은 이름 브랜치에 다른 커밋 fa12a7b 가 F-180·F-181 처리 |
  | rotation-averaging-checks (P05), align-followup (P08) | — | — | 이번 실행에서 시작하지 못함 |
- 막힌 점:
  - 같은 시각 다른 실행(23:50Z 시작분)이 같은 브랜치들에 계속 푸시 중이라 7 묶음이 겹쳐 중단됐다. 실행 일정이 겹치지 않도록 정리가 필요하다.
  - 연구 PR #51 은 부모 노드와 노트 충돌을 풀었다(fc46dfe). #54 는 05c9681 과 결론이 엇갈려 정리 필요.
  - ba: `feat/ba-robust` 9cd1b0a 와 `feat/ba-formation` 3d2a409 중 하나로 합쳐야 한다(F-036 편대 시험은 3d2a409 에만 있음).
- 다음 할 일:
  1. P06 시드 6~10 짝 이상치 20% 붕괴 3건: 정밀화가 나빠지면 출발 해로 되돌리는 안전장치.
  2. F-003 σ 별 σ₈/σ₁ 분포, F-113 사진별 병렬, F-048 960px 시간 측정(부하 없는 상태).
  3. ba 두 갈래 합치기, patchmatch-profile 재검증 후 PR #6 에 반영 여부 결정.
  4. P05 F-171~F-173·F-138·F-047, P08 F-154·F-099·F-153 (이번에 시작 못 함).

## 이전 실행 기록 (23:50Z 시작분)
- 마지막 갱신: 2026-10-02T04:06Z (2026-10-02T04:06Z 시작)
- 이번 실행(23:50Z 시작, 4 코어 측정 기계에서 11 묶음 동시 진행 — 부하 평균 25~80, 시간 수치 없음):
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | P14 verify | PR #10 갱신 (f5a9ac2), 라벨 | PR #24 갱신 (37ffa10) | main 병합(main.rs 충돌 해소), F-156·F-157 처리, F-089 결정 대기. 직접 재검증 fmt·clippy·전체 시험 통과(core 170·cli verify 21) |
  | P08 align-robust | PR #24 갱신 (a307c0c), 라벨 | PR #41 갱신 (9572590) | F-152 처리(`gps_align_poses`, 기울기 최악 0.022°), F-154 대부분, F-153·F-099 방위 결정 대기. 직접 재검증 통과(core 171) |
  | P12 dataset-io | PR #4 갱신 (214d40a), 라벨 | PR #18 (71a84df) | main 병합, F-065 로더 쪽(80/12/2 7구역, 1800경우 성질 시험). 직접 재검증 통과(core 183) |
  | P17 dense-prep | PR #29 (bad7449), 라벨 | PR #48 (→ view-selection) | F-092·F-110·F-111 재검증, 편대 이웃 1위 7칸 원인(공유 점 감소) 설명·기준 숫자화. 직접 재검증 통과(core 169) |
  | two-view-twin (P03 후속) | PR #30 (7f80318, #13 위), 라벨 | PR #46 (→ two-view-hardening) | F-144 처리: 편대 160경우 쌍둥이 0(교체 없으면 63, 1.9~15.2°). 직접 재검증 통과(core 168) |
  | matching-followup | PR #31 (729c03a), 라벨 | PR #47 (→ matching-refine) | F-139 처리, F-148 매칭 쪽 표(확정 21 중 5 가 2° 초과, 같은 카메라 4·5·16칸), F-150 matching 쪽. 직접 재검증 통과(core 162) |
  | fusion-tests (P11 후속) | PR #32 (804b236), 라벨 | PR #49 (→ fusion-hardening) | F-069(다른 드론 동의, 0.3 m 초과 0·최대 0.057 m, 점 수 3% 로 감소), F-070, F-112 일부. 직접 재검증 통과(core 168) |
  | tracks | PR #15 갱신 (956213a), 라벨 없음 | PR #31 갱신 (cfdc1b8) | F-123 일부: 30% 완전도 1.000, 30%·이상치 1% 순도 0.9672 < 0.99 로 `sparse_recall_keeps_tracks_whole` 실패 |
  | P10 patchmatch | PR #6 갱신 (a1840a5), 라벨 없음 | PR #20 갱신 (3d02b13) | F-151 처리(main 과 컴파일됨). 전체 시험 부하로 미완, F-048·F-050·법선 시험 미착수 |
  | P06 translation-averaging | `feat/translation-averaging` 53b3c78 (PR 없음) | experiment/translation-averaging ce22315 | 퇴행 원인 분리: 점 방향 이상치(5%)가 첫 해를 끌어 6° 거르기에서 그래프가 갈라짐 — 점 제약 없음 237~240, 깨끗한 점 방향 240/240(RMS 0.11~0.14 m). 고침 미완 |
  | ba-robust (P20 후속) | `feat/ba-robust` 995020e (PR 없음, bca55eb 뒤 00:47~00:51 커밋 2개 — 트랙 서로 다른 카메라 2대 요구·관측 카메라 축척 게이지·옵션 검사 등, 재검증 전) | experiment/ba-robust cb4c357 | F-036 편대 시험 추가했으나 무잡음 최소제곱도 100회 미수렴·중심 1.2 m(약한 방향) → 무시. F-034 main 에서 처리 확인 |
- 검증: 라벨 붙인 PR 머리마다 전용 빌드 폴더에서 fmt·clippy(-D warnings)·`cargo test --release` 전체 통과를 직접 확인. 부하 상태에서 `two_view::tests::five_point_terminates_on_many_seeds` 시간 단언이 여러 묶음 실행에서 실패, 단독 0.5~0.6 s 통과(F-059).
- 막힌 점:
  - tracks F-123: 단일 간선 결합의 오대응이 순도를 0.967 로 낮춤(다음 안: 단일 간선 결합 마지막·경쟁 시 소수 쪽 버림).
  - P06: 첫 풀이에서 점 간선 가중치를 낮추거나 2단계로 넣는 안 미실행.
  - ba-robust: 1 m 기선 편대에서 약하게 정해지는 방향 — 기준을 공분산에서 정해야 함.
  - fusion: 다른 드론 동의 조건으로 점 수 3% — 이웃 선택이 같은 드론 8장만 고르는 것(dense-prep)과 함께 결정.
  - 부하 때문에 시간 측정(F-048·F-113·F-127·F-128·F-040) 모두 미측정.
- 결정 필요:
  1. F-089 report.json (SPEC §2 에 넣기 / manifest 키 / 검증 대상 제외).
  2. F-153 GPS 잔차 상한 고정 3 m vs 잡음 비례, F-099 방위 기준(σ_ψ 배수).
  3. F-069 다른 드론 동의 조건 채택(밀도 3%) 여부.
  4. F-111 유효 영역 가장자리 반 화소 띠 규칙.
- 다음 할 일:
  1. F-148 재측정: two-view-twin(#30) 위에서 매칭 짝 종류별 표 다시.
  2. P06 점 간선 가중치/2단계, tracks 단일 간선 정책, 이웃 선택에 다른 드론 사진 포함.
  3. 부하 없는 상태에서 시간 측정 묶음 따로.

## 이전 실행 기록 (23:39Z)
- 마지막 갱신: 2026-10-02T04:06Z (2026-10-02T04:06Z 시작)
- 이번 실행(23:07Z 시작, 4 코어 측정 기계에서 13 묶음 동시 진행 — 부하 평균 25~42, 시간 수치는 부풀려짐):
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | P04 io-cleanup | PR #23 (ff3a769) | PR #40 (→ scaffold) | F-020·F-133~F-136 README 한·영, F-061·F-081·F-062·F-063·F-082·F-058 재시험. core 136 통과·3 무시 |
  | P08 align-robust | PR #24 (08bd593) | PR #41 (→ similarity-align) | F-099 일부: 기울기 0.200°(기준 0.25°)·위치 1.013 m·유지 97.2%, 방위 최악 0.641° = 이론 σ_ψ 의 1.95배. 무시 시험 없앰 |
  | P11 fusion-hardening | PR #25 (de4556e) | PR #42 (→ depth-fusion) | F-069 원인: 같은 드론 이웃 3장의 같은 배율 이상치가 만드는 가짜 평면. `min_ratio` 0.5 로 0.3 m 초과 101 → 5점, 최대 11.5 m 그대로(편대 시험 무시 유지). F-112·F-113 미착수 |
  | P05 rotation-averaging-robust | PR #26 (06ac8d2) | PR #43 (→ rotation-averaging) | F-137·F-138 처리, F-047 일부(무시 시험 해제). 152 통과·0 실패 |
  | P13 stream-hardening | PR #27 (3d831e6) | PR #44 (→ progressive-stream) | 오대응 30·50% 정렬 복구(스케일 비 1.000, fit 0.10 m), F-065·F-056 일부. core 141 통과 |
  | P15 benchmarks-followup | PR #28 (a7ad157) | PR #45 (→ benchmarks) | F-131·F-130 처리, F-149·F-132 일부, F-128·F-040 부하로 미측정. 144 통과 |
  | P03 two-view-hardening | PR #13 갱신 (997437e) | PR #29 갱신 | F-146 처리, F-148 일부(정상 수 유의성 검사, 합성 2° 초과 0/40·겹침 없는 짝 0/20). F-033·F-144·F-145 미착수. 직접 재검증 137+13 통과·0 실패 |
  | P02 matching-refine | PR #16 갱신 (223f30d) | PR #32 갱신 | F-140·F-143 처리. F-141·F-142·F-148(매칭 쪽 표) 미달. 직접 재검증 144+13 통과·0 실패, main 과 충돌 없음 |
  | P20 ba-hardening | PR #17 갱신 (022e5cf) | PR #33 갱신 | F-121·F-122 처리, F-036 진전 없음 |
  | P14 verify | PR #10 갱신 (0475047) | PR #24 갱신 | F-089 처리(report.json 없으면 1~3 판정 불가, 종료 코드 2). 직접 재검증 CLI 20·verify 8 통과 |
  | P01 formation-scene-check | PR #22 병합됨(e9f7187), 브랜치 d56dfde | experiment/formation-scene-check 71029cd | 최신 main 과 합친 트리에서 다른 모듈 시험 깨짐 없음, F-115·F-116 수치 재확인 |
  | tracks | PR #15 갱신 (cbb8255), 라벨 안 붙임 | experiment/tracks cb2f609 | F-124·F-125·F-126 처리. F-123 미달 — `sparse_recall_keeps_tracks_whole` 실패 상태(재현율 50% 완전도 0.9756, 30% 0.8313) |
  | P10 patchmatch | `feat/patchmatch` da06217 (main 병합만), 라벨 안 붙임 | experiment/patchmatch c5b64f0 | 부하로 속도·법선 실험 못 함, clippy·전체 시험 미확인 |
  | P06 translation-averaging | — | — | 이번 실행에서 시작하지 못함 |
- 검증: 위 PR 머리마다 전용 빌드 폴더에서 fmt·clippy(-D warnings) 통과 확인, 시험은 해당 모듈 + 전체(two-view·matching-refine). 동시 부하에서 `two_view::tests::five_point_terminates_on_many_seeds` 최악 호출 100 ms 단언이 여러 브랜치에서 실패, 단독 재실행 0.3~0.8 s 통과(F-059).
- 막힌 점:
  - 13 묶음 동시 진행으로 부하 평균 40 안팎 — 시간 측정 항목(F-048·F-113·F-128·F-040·F-127·F-056 속도)은 모두 미측정. 다음 실행은 시간 측정 묶음을 따로 돌릴 것.
  - P10 `From<&features::GrayImage>` 는 P04(#23) 병합 뒤 `Self::new(g.width(), g.height(), g.data().to_vec())` 로 바꿔야 컴파일된다.
  - tracks: 재현율 30~50% 에서 참 조각을 잇는 간선이 1개뿐인 경우가 많아 2단계 문턱(간선 2개)에 막힘.
  - 구역 분할 함수가 dataset·stream 두 벌(F-065).
- 결정 필요:
  1. F-099 방위 기준: 고정 0.5° 대신 이론 σ_ψ 배수(최대 |z| < 3.5) 로 바꿀지.
  2. SPEC §3.4 GPS 잔차 상한: "3 m 하한 + 잡음 비례 확장(최대 9 m)" 반영 여부.
  3. SPEC §2 `report.json`(등록 사진 수·재투영 오차)을 출력에 넣을지 — 없으면 verify 1~3 판정 불가.
  4. 회전 평균 '놓침' 정의 변경(문턱 + 두 끝 정점 오차 합).
  5. SPEC §2 manifest `step` 형(정수/"final")·정렬 실패 null 규칙.
- 다음 할 일:
  1. P06 translation-averaging 잡음+이상치 퇴행 원인 분리(이번에 시작 못 함).
  2. F-148: bench `회전 평균(검증 결과)` 재측정, 짝 종류별 틀린 간선 표(두 시점·매칭).
  3. F-069 가짜 평면: 다른 드론 시선 1장 이상 요구 검토.
  4. tracks F-123, patchmatch F-048 와 법선 시험.
  5. 부하 없는 상태에서 시간 측정 묶음.

## 이전 실행 기록 (19:40Z)
- 이번 실행(18:54Z 시작, 4 코어 측정 기계에서 15 묶음 동시 빌드 — 부하 평균 40~60, 시간 수치는 부풀려짐):
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | P15 benchmarks-scale | PR #21 (805d474, 병합됨) | PR #38 (→ benchmarks, 병합됨) | F-040 BA 240대·트랙 10만 반복당 97.1 s(부하), F-031 검출 시간 기록, 측정 틀 하나로(예제 삭제). 재검증 fmt·clippy 통과, core 131 통과·1 실패(F-059 시간 단언)·3 무시 |
  | P01 formation-scene-check | PR #22 (b198689) | PR #39 (→ formation-scene) | F-115·F-116. 재검증 fmt·clippy 통과, core 134 통과·0 실패·3 무시, 통합 5+4+4 |
  | P04 io-cleanup | `feat/io-cleanup` f80463b (PR 안 엶) | experiment/io-cleanup 4dc2d91 | F-061·F-081·F-062·F-063·F-082·F-058·README. 재검증 fmt·clippy 통과, 시험 재검증 미완 |
  | P08 align-robust | `feat/align-robust` acb6000 (PR 안 엶) | experiment/align-robust 72d5b41 | F-094·F-100·F-119·F-095·F-079·F-096, F-099 일부(σ 2 m 회전 0.671° > 0.5°, 무시 시험). 기본 GPS 정렬 임계가 잡음 비례로 바뀜 — SPEC §3.4(고정 3 m) 결정 필요 |
  | P20 ba-hardening | PR #17 갱신 (4cbfa24) | experiment/ba-hardening 64d57a1 | F-085·F-086, F-036 일부(단계 Cauchy, 엄격 기준 시드 21~30 미달). 재검증 미완 |
  | P17 dense-prep | `feat/dense-prep` fd0c1e0 (PR 안 엶) | experiment/dense-prep bfd0850 | F-060·F-053·F-092·F-111·F-054·F-110·F-052. 편대 장면 이웃 1위가 7칸(SPEC 예 8칸). 재검증 미완 |
  | P05 rotation-averaging-robust | `feat/rotation-averaging-robust` a944908 (PR 안 엶) | experiment/rotation-averaging-robust ad8de02 | F-075·F-101·F-076·F-077·F-009·F-046·F-078, F-047 일부. 놓침 정의 변경 검토 필요. 재검증 미완 |
  | P12 dataset-io | PR #4 갱신 (ab1fcef) | experiment/dataset-io 8503216 | F-041·F-042·F-043·F-064·F-065(로더)·F-044. 재검증 미완 |
  | P13 stream-hardening | `feat/stream-hardening` 02c991e (PR 안 엶) | experiment/stream-hardening 1aea73f | F-066·F-055·F-065(스트림)·F-067·F-068·F-080·F-057, F-056 일부. 재검증 미완 |
  | P14 verify | PR #10 갱신 (a327c61) | experiment/verify 76564ea | F-087·F-097·F-066(읽기)·F-117·F-118·F-088·F-098·F-090, F-089 일부. 재검증 미완 |
  | P11 fusion-hardening | `feat/fusion-hardening` b87ae13 (PR 안 엶) | experiment/fusion-hardening d9e51b1 | F-071·F-072·F-073·F-074. F-069(최대 11.885 m 그대로)·F-112·F-070·F-113 미달 |
  | P06 translation-averaging | `feat/translation-averaging` a096498 (PR 안 엶) | experiment/translation-averaging cb12be6 | 점 방향 제약·발자국 겹침 짝. 초벌 모델 시험 무시 해제(240/240, 점 RMS 0.270 m). 잡음+이상치 시드 1 은 105/240(지난번 239/240 보다 나빠짐) |
  | P03 two-view-hardening | PR #13 갱신 (d96c0f5) | experiment/two-view-hardening a9f6c9f | F-114 처리(`assess_pair` 시차 < 1.0° 신뢰 불가, 간격 1 시험 복원 9경우 통과). F-033 미달(30% 이상치 최악 바닥 대비 +0.2448°, 무시 유지). 전체 `cargo test --release` 미확인 |
  | P02 matching-refine | `feat/matching-refine` 2a1f60e | — | 이 기록 시점까지 결과 미보고 |
  | P10 patchmatch | PR #6 갱신 (e7d0c74) | experiment/patchmatch e92838a | F-083·F-084·F-049·F-051, F-048 일부(960×540·이웃 8장 23.0 s/장, 목표 0.7 s 의 33배, 부하), F-050 일부. 정면 경사·계단 법선 7.38°·6.93° > 5° 로 시험 2개 무시. `From<&features::GrayImage>` 가 P04 필드 비공개와 합칠 때 확인 필요 |
- 막힌 점:
  - 4 코어 기계에서 15 묶음을 함께 빌드하니 검증이 밀려 PR 은 직접 재검증을 마친 2건(#21·#22)만 새로 열었다. 나머지는 브랜치 푸시만 했고 작업자 쪽 검사(fmt·clippy·`cargo test --release`, F-059 시간 단언 1건 외 실패 0)만 있다 — 다음 실행에서 브랜치마다 전용 빌드 폴더로 재검증 후 PR.
  - `two_view::tests::five_point_terminates_on_many_seeds`(F-059)는 부하에서 모든 브랜치가 실패, 단독 통과. P03 이 구조 단언으로 바꾸는 중.
  - 묶음 사이 충돌 예상: P04 README 가 `bench_stages` 예제를 언급하지만 P15 가 예제를 지움. P17 이 `UndistortMap::new`·`pinhole_for_long_side` 를 Result 로 바꿈 → P10 patchmatch 와 합칠 때 확인. F-066 은 P13(쓰기)·P14(읽기)를 합친 트리에서 확인 필요. F-065 구역 분할 함수가 dataset·stream 두 벌.
  - P13 정렬이 오대응 30% 이상에서 무너짐(첫 추정 최소제곱) — P08 LMedS 첫 추정과 합치면 풀릴 것으로 보임.
  - P12: `feat/dataset-io` 에 19:04 다른 커밋(9aae457)이 있어 병합으로 받음(누락 카메라 GPS 를 다른 카메라 값으로 메우던 부분은 뺌).
- 다음 할 일:
  1. 미재검증 브랜치 재검증(한 번에 하나씩) → PR(#4·#10·#17 은 라벨 확인).
  2. P06 잡음+이상치 퇴행 원인 분리(점 제약 없이 재실행, 행별 오차).
  3. P11 F-069 떠 있는 점 하나를 기준 사진·동의 사진까지 추적.
  4. 결정 필요: GPS 정렬 기본 임계(고정 3 m 대 잡음 비례), F-099 기준 분리, 회전 평균 놓침 정의, SPEC §3.6 이웃 1위 예(8칸) 재확인, SPEC §2 step 형·report.json.

## 이전 실행 기록 (14:18Z)
- 이번 묶음 결과(13:45Z 시작, 4 코어 측정 기계에서 동시 빌드):
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | P20 ba-hardening | PR #17 (3de2b47) | PR #33 (→ bundle-adjustment) | F-034·F-035·F-037·F-039 처리, F-040 시간 표(병렬 누적 남음) |
  | P18 camera-unify | PR #18 (032177a) | PR #34 (→ camera-model) | F-023 단일 정규화 경로, 최신 main 병합 + `undistort.rs` 에 `dist` 한 줄 |
  | P19 readme-sync | PR #19 (d4804c4) | PR #35 (→ scaffold) | F-020, main 6c0eea7 기준 |
  | P16 sampson-lm | PR #20 (0cfcd94) | PR #37 (→ ransac-robustness) | F-026 처리, F-027 해석적 야코비안(0.1 s 미달, 열림) |
  | N03 features-notes | — | PR #27 (→ features) | T03 노트 보충 |
  | N04 matching-notes | — | PR #36 (→ matching) | T04 노트 보충: 편대 배치 카메라 간 짝 정답 0 인데 53/54 검증 통과 |
  | N05 two-view-notes | — | PR #28 (→ two-view) | T05 노트 보충: 평면 첫 후보 쌍둥이 해 문제 |
  - 검증: PR #17·#18·#19·#20 각각 최신 origin/main 과 합친 트리에서 fmt·clippy 경고 0·`cargo test --release` 실패 0 (core 132/128/124/126 통과).
- 막힌 점:
  - P17 pixel-convention: 제품 `feat/pixel-convention` 336621f(F-016 화소 중심 규약·`Keypoint::pixel()`, F-031 검출 해시 시험) — 최신 main(1609550 편대 장면)과 two_view.rs 시험에서 충돌해 PR 안 엶. 충돌 해소 필요. 연구 `experiment/pixel-convention` 81b65d3.
  - P06 translation-averaging: 제품 `feat/translation-averaging` 1aae95a(무잡음 결함 = 초기 최소제곱의 1e-9 대각항, 고침 → 240/240·최대 7e-11 m; 중심·길이 동시 풀기). 잡음+이상치 시험 2개 `#[ignore]` 남음(F 열 끝 74~78 위치 오차, R·L 행 RMS 0.8~0.9 m). 같은 브랜치에 다른 쪽 커밋(e144e46)이 함께 있음. 최신 main 미병합, PR 안 엶.
  - P03 two-view-hardening: 같은 묶음이 PR #13(133ef34)으로 따로 올라옴. 비교 브랜치 `feat/two-view-hardening-msac` 337c13a(MSAC 순위·다중 시작, 비평면 이상치 0 에서 0.027° vs 0.062°, `refine_relative_pose` 6.9/57.2 ms vs 9.4/86.2 ms). 둘 다 F-033 비평면 이상치 30% 0.245° 미달. 하나를 골라야 함.
  - P15 benchmarks: 이쪽 커밋 e30537e 가 PR #14 브랜치에 합쳐져 있음(별도 PR 없음).
  - 시간 단언 시험 `five_point_terminates_on_many_seeds` 는 동시 부하에서 계속 실패(단독 통과) — PR #13 이 구조 단언으로 바꿈.
- 다음 할 일:
  1. 감독 검토 결과 반영.
  2. pixel-convention 충돌 해소 → PR. undistort.rs 주점 변환 `(cx+0.5)·s−0.5` 규약 확인.
  3. translation-averaging 잡음 경우(시야 겹침 기반 짝, 축척 수축) → `#[ignore]` 해제.
  4. 매칭 검증이 겹침 없는 짝을 통과시키는 문제(N04) — 최소 정상 수·유의성 검사 피드백 등록 제안.
  5. 평면 후보 선택(N05, F-033).
- 검토 요청(제품 PR, `review-requested`, 연구 PR 은 부모 노드 기준):
  | 묶음 | 제품 PR | 제품 커밋 | 연구 PR | 결과 |
  |---|---|---|---|---|
  | P01 formation-scene | #12 | 75fd5ee | #26 (→ synthetic-scene) | F-029·F-024 처리, 매칭·두 시점 시험 기준 재측정 |
  | P02 matching-hardening | #9 | 8ec6123 | #23 (→ matching) | F-030·F-021 처리, F-003 동일선상만·F-031 일부 |
  | P04 io-robustness | #2 | 5fd09ef | #16 (→ scaffold) | F-017·F-018·F-019·F-022 처리, F-016·F-020·F-023 일부 |
  | P05 rotation-averaging-hardening | #7 | 3ea0c13 | #21 (→ rotation-averaging) | F-008·F-009 처리, F-006·F-007 일부 |
  | P07 bundle-adjustment | #1 | 395beb1 | #15 (→ two-view) | T08 희소 LM + 슈어 보수, 최종 RMS/기댓값 0.986~1.000 |
  | P08 similarity-align | #11 | 408aea1 | #25 (→ rotation-averaging) | T09 Umeyama·트리밍·GPS 정렬 |
  | P09 view-selection | #3 | a947561 | #17 (→ camera-model) | T10a 이웃 8장·깊이 범위·960 px 왜곡 보정 |
  | P10 patchmatch | #6 | 2ae1f7a | #20 (→ camera-model) | T10b 정확도 통과, 960 px 시간 미측정 |
  | P11 depth-fusion | #8 | d7e6617 | #22 (→ camera-model) | T11 3장 동의 융합 |
  | P12 dataset-io | #4 | e577cbf | #18 (→ scaffold) | 입력 읽기·구역 분할·`run` 뼈대 |
  | P13 progressive-stream | #5 | 1ba6d6e | #19 (→ two-view) | T12 정렬·잔상 거르기·스냅샷·manifest |
  | P14 verify | #10 | f79ef29 | #24 (→ scaffold) | T13 `verify <폴더>` 일곱 항목 |
- 13:44Z 시작분 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | P03 two-view-hardening | PR #13, 133ef34 | PR #29 (→ two-view), dd920ad | F-032·F-028·F-025 처리, 5점 시험 구조 단언. F-033 비평면 이상치 30% 0.245° 미달(무시 시험) |
  | P15 benchmarks | PR #14, 69fb55e | PR #30 (→ fast-matching), 6d979ba | 구간 시간 틀(예제 + bench, 두 도구 합침 결정 필요), 구조 시험 5개. 320×180 240장 262 s 중 두 시점 검증 209 s |
  | P16 matching-refine | PR #16, 92db4e9 | PR #32 (→ matching), 5bf1fdb | F-003 평면 모델 선택(GRIC + 시차 짝). F-026·F-027 은 sampson-lm 에 맡기고 되돌림 |
  | tracks (새 묶음) | PR #15, 03b28dd | PR #31 (→ matching), ca0ca16 | 다시점 트랙(Split 규칙), 오대응 1% 순도 0.9992·완전도 0.9989 |
  | P05 rotation-averaging-hardening | PR #7 갱신 b3ca12a (라벨 다시 붙임) | experiment/rotation-averaging-hardening 06eadbc | F-045·F-006·F-007 처리 |
  | P12 dataset-io | PR #4 갱신 c0bf96f (라벨 안 붙임) | experiment/dataset-io db207b9 | F-038 처리. 불합격 사유 F-041(카메라별 GPS) 미처리 |
  | P06 translation-averaging | feat/translation-averaging e144e46 (PR 없음) | experiment/translation-averaging 44bfc36 | 무잡음 정확 복원(정규화 1e-9 항 제거, 최대 오차 < 1e-6 m). 잡음 1° 에서 중심 RMS 9.8 m → 시험 3개 무시 유지 |
  | camera-unify-keypoints | feat/camera-unify-keypoints 155d613 (PR 없음) | experiment/camera-unify-keypoints 94746fc | F-016 검출 화소 중심 좌표(치우침 0.001 px)·F-023 형 하나로. 같은 묶음의 feat/camera-unify 와 API 충돌 → 하나를 골라야 함. matching.rs:990·two_view.rs:1118 의 수동 +0.5 때문에 `ransac_on_cross_camera_views` 실패 |
- 검증: PR #13·#14·#15·#16 머리에서 따로 fmt·clippy(-D warnings)·`cargo test --release` 통과(실패 0) 확인.
- 이번에 하지 못함: P10 patchmatch(F-048 960 px 시간), ba-hardening·io-conventions 는 이 실행에서 착수 못 함(다른 묶음표의 P20 ba-hardening·P19 readme-sync 가 같은 범위).
- 검증: 위 12개 브랜치 각각 따로 `cargo fmt --all --check`·`cargo clippy --all-targets -- -D warnings`·`cargo test --release` 통과(실패 0).
  단, 여러 빌드가 동시에 도는 동안 기존 시간 시험 `two_view::tests::five_point_terminates_on_many_seeds`(최악 100 ms·전체 10 s 단언)가 부하로 실패해 이 확인 실행에서는 그 시험만 빼고 돌렸다. main 에서 그 시험만 따로 돌리면 0.31 s 통과 — 시간 단언 여유 재검토 필요.
- 막힌 점:
  - P03 two-view-hardening: 제품 `feat/two-view-hardening` 574fb6a(프로크루스테스 1회 계산만, F-028 일부), 연구 `experiment/two-view-hardening` 556e264. PR 안 엶. F-032 고침(정상 표시 재판정 + 평면 비율 0.95)은 잡음 없는 평면 시험 퇴행(쌍둥이 해 병합)으로 미반영, F-033 비평면 30% 는 다중 시작 + MSAC 순위로 0.298°(기준 +0.1° 미달). F-026·F-027 코드는 matching.rs 쪽이라 이 묶음 범위 밖 — 묶음 표 수정 필요.
  - P06 translation-averaging: 제품 `feat/translation-averaging` e6484ac, 연구 `experiment/translation-averaging` 48f84ef. PR 안 엶(목표 미달 시험 4개 `#[ignore]`). 159/240 원인은 짝 그래프 연결성(R 카메라 시야 중심이 F·L 과 약 20 m 떨어져 12 m 기준이면 R 80대가 분리) → 22 m 로 239/240. 그러나 무잡음에서도 중심이 정확히 나오지 않아 정밀화 단계 결함(부호 접힘·축척 하한·긴 사슬 수렴 의심).
  - P15 benchmarks: 이번에 시작하지 못함.
- 다음 할 일:
  0. 동시에 두 묶음표가 돌아 같은 브랜치(benchmarks·camera-unify·two-view-hardening·translation-averaging)를 함께 고쳤다. 한 번에 하나만 돌도록 일정 정리. camera-unify 두 갈래 중 하나 선택.
  1. 감독 검토 결과 반영(PR 12쌍).
  2. P06 정밀화 결함(무잡음 정확 복원부터) → `#[ignore]` 4개 해제.
  3. P03 평면 판정 분리 → F-032·F-033.
  4. P15 시간 측정 틀, F-026·F-027(matching.rs) 묶음 배정.
  5. 병합 순서 메모: P09·P10·P11 의 임시 형(View·DepthMap)과 P13 의 임시 닮음 변환은 P08·P09 병합 뒤 교체. P04 와 P12 는 CLI main.rs·lib.rs 한 줄씩 겹침. P12 입력 구조(`images/camF/`)와 합성 출력(`images/camF_0000.jpg`) 불일치.
- 감독 지시: (13:57) PR #3·#5 병합. 불합격 PR #4(F-041 카메라별 GPS), #6(F-048 PatchMatch 960px 100 s), #7(F-045 정상 표시 불일치) — 높음부터 처리 후 재요청.
