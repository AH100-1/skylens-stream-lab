# 현재 상태

- 상태: 진행 중
- 마지막 갱신: 2026-10-02T00:43Z (새 실행 시작)
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
  | ba-robust (P20 후속) | `feat/ba-robust` bca55eb (PR 없음) | experiment/ba-robust cb4c357 | F-036 편대 시험 추가했으나 무잡음 최소제곱도 100회 미수렴·중심 1.2 m(약한 방향) → 무시. F-034 main 에서 처리 확인 |
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
- 마지막 갱신: 2026-10-01T23:39Z
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
