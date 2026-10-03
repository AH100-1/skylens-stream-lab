# 현재 상태

- 상태: 쉬는 중
- 마지막 갱신: 2026-10-03T15:50Z (15:06Z 시작분)
- 이번 회차 결론: 초벌 전 GPS 사전항 BA(`preview_ba_iters`)로 끝까지 단구역 verify 5/7 → 7/7(높이 차 4.62 → 0.107 m) — 새 PR #48(CI 실패로 라벨 없음). 융합 일치 표·경계 시험 PR #47. 트랙 #15 에 F-279 단언·거름 병렬화(라벨 다시). `feat/pipeline` 에 regions·tracks·ta·height 합침(preview·preview-ba 는 아직). PatchMatch 원 해상도 CPU 4.17 → 1.79 s 이나 법선 5.85° 로 나빠져 보류.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | pipeline-preview-ba (F-268·F-270 관련) | `feat/pipeline-preview-ba` e269734 → PR #48 | experiment/pipeline-preview-ba c9af9a3 → PR #65 | 초벌 BA 0/3/8회: verify 5/7·7/7·7/7, preview_align 6.97·0.515·0.046 m, 높이 차 4.62·0.324·0.107 m, 정밀 중심 0.337/1.26·0.323/3.29·0.334/1.27 m, 점→표면 0.342·0.404·0.441 m. 총괄 재확인: fmt·clippy 통과, `synthetic_single_region_end_to_end` 1 통과(82 s). 단 CI `test` 실패: `sparse::formation_scene_registers_all_and_meets_floors`(중심 2.50/5.09 m, 회전 20.75°, 132/132) — 기반 `feat/pipeline-height` 부터 있던 실패로 CI(부하 5)에서도 재현되어 부하 탓 아님. 라벨 뗌. 기본값 0 — SPEC '초벌 BA 없음'과 차이, 결정 필요 |
  | fusion-consistency (F-256·F-257) | `feat/fusion-consistency` b184820 → PR #47 | experiment/fusion-consistency 73a8930 → PR #64 | 알고리즘 변경 없음(일치 검사 이미 있음). σ 0.5%·이상치 10%: min_views 3 점 4486, 거리 중앙 0.0055 m·95% 0.0165 m, 이상치 잔존 0.20%. 경계 입력 패닉 없음. 총괄 재확인: fmt·clippy 통과, `fusion::` 19 통과·0 실패·2 무시 |
  | tracks (#15, F-279) | `feat/tracks` 969f14e | experiment/tracks 541f203 | 갈라진 점 비율 단언(30% ≤ 4%, 40·50% ≤ 1%), 거름 rayon 병렬. 수치 전/후 동일, 기준 규모 Split 23.9 → 17.9 s(부하 23). 총괄 재확인: fmt·clippy 통과, `tracks` 17 통과·0 실패·2 무시. 라벨 다시 |
  | pipeline (#46 합치기) | `feat/pipeline` b8c6c6a | experiment/pipeline 1194cd3 | regions·tracks·ta·height 합침, build·fmt·clippy 통과. CLI 실행: 단구역 verify 5/7(120/120), 2구역 4/7(240/240, refined_overlap 1.749 m ← 8.51 m). `synthetic_two_region_end_to_end` 실패(스케일 차 12.71%, 잔차 9.81 m). preview 합치기는 `pipeline.rs` 4곳 충돌로 중단, preview-ba 합치기는 이 측정 기계 권한 설정에서 막힘. 라벨 없음 |
  | patchmatch (#6, F-048) | `feat/patchmatch` 672f548 | experiment/patchmatch ee68e64 | 원 해상도 건너뜀 비용 0 → 0.08: 960×540 이웃 8장 CPU 4.17 → 1.79 s(CPU/4 0.45 s), 벽시계 2.72 s(부하 21~25). 깊이 기준 통과(중앙 0.187%, 1% 이내 94.8%)이나 960 시험 법선 중앙 2.85 → 5.85°. 총괄 재확인 못 함, 라벨 없음 |
  | pipeline-regions-ba (F-272·F-273) | `feat/pipeline-regions-ba` d7981e7 | experiment/pipeline-regions-ba a0d62d0 | `region_link` Off/Points/Centers(겹치는 사진 포즈 고정). 3구역 62/78: 겹침 4.20/4.48/4.20 m, 스케일 차 177.8/374.5/177.8% — 개선 없음, 기본 Off. `pipeline_regions` 2 실패(가운데 구역 건너뜀 기록·정지 구간 이슈) |
  | pipeline-stream-order (F-055 등 관련) | `feat/pipeline-stream-order` 6de27fc | experiment/pipeline-stream-order bd4fee2 | 위치 단위 도착 사건, 시점별 스냅샷 `snapshots/live/`, manifest `events`. 2구역 사건 순서 시험 통과, 스트림 41.6 s / 순차 51.5 s 수치 동일. `pipeline_regions` 2 실패(위와 같은 두 시험) |
  | pipeline-pm (F-271) | `feat/pipeline-pm` | experiment/pipeline-pm | 마감 시점 결과 미수신 — 브랜치 상태만, 총괄 확인 못 함 |
  | pipeline-accuracy (F-273·F-066) | `feat/pipeline-accuracy` | experiment/pipeline-accuracy | 마감 시점 결과 미수신 — 브랜치 상태만, 총괄 확인 못 함 |
- 끝까지 흐름 진척: `feat/pipeline` 단구역 synth → run → verify 가 PLY·스냅샷·manifest 까지 나오고, 초벌 BA 가지(#48)에서 단구역 7/7. 남은 것: #48 을 `feat/pipeline` 에 합치기(preview 충돌 정리 먼저), 2구역 스케일 차(12.7%), 밀집을 PatchMatch 로.
- 다음 할 일:
  0. #48 CI: `sparse::formation_scene_registers_all_and_meets_floors` 실패 원인을 `feat/pipeline-height` 6724181 에서 찾기(번들 조정 뒤 회전 오차 20.75° — 삼각측량 문턱·σ 설정 변경 영향으로 추정).
  1. `feat/pipeline` 에 `feat/pipeline-preview`(run_ba 시그니처·sparse_init 반환형·구역 반복 충돌) → `feat/pipeline-preview-ba` 합치고 끝까지 시험.
  2. 2구역 스케일 차: 겹침 위치 늘리기, 공유 점 + 양방향 재투영 검사 정렬, 겹치는 사진 위치 사전항(축별 가중, ba.rs 확장).
  3. `pipeline_regions` 의 `failed_middle_region_is_skipped_not_fatal`·`stationary_segment_is_error_or_issue` 실패를 기반 `feat/pipeline-regions` 에서 재현해 원인 가르기.
  4. PatchMatch: 건너뛴 화소 법선 섭동 1회 또는 법선 변화 기준 건너뜀, 원 해상도 시작 비용·최상층 반복 축소, 부하 없는 벽시계 재측정.
- 막힌 점:
  - 4 코어 측정 기계에서 묶음 9개 동시 시험으로 부하 15~25 — 끝까지 시험(12 분)·전체 시험을 마감 안에 못 돌림.
  - `feat/pipeline` 에 `feat/pipeline-preview-ba` 합치기가 이 측정 기계 권한 설정에서 막힘 — 소유자 확인 필요.
  - 소유자 병합 필요: #15·#37·#47(통과 판정·라벨).
  - 결정 필요: 초벌 짧은 BA(SPEC §3 '초벌 BA 없음' 개정 여부), 보조 사진 SPEC §3.5, F-197, F-251.


## 직전 실행 기록 (2026-10-03 14:06Z 시작분)
- 상태: 쉬는 중
- 마지막 갱신: 2026-10-03T14:50Z (14:06Z 시작분)
- 이번 회차 결론: 새 PR 없음. 위치 평균 #37 에 F-277·F-278 처리(기존 시드 11~13 출력 그대로), 트랙 #15 에 편대 재현율 표 시험(완전도 ≥0.9937, 9경우) — 둘 다 해당 모듈 시험 재확인 후 review-requested 다시 붙임. PatchMatch 경사·계단 법선 시험 통과로 복구, 속도는 CPU/4 1.19 s 로 목표 미달. `feat/pipeline` 합치기(regions·preview·tracks)는 커밋하지 못해 그대로.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | translation-averaging (#37, F-276~F-278) | `feat/translation-averaging` 209a6a6 (수정 3d65184) | experiment/translation-averaging 821d403 | F-277: 문턱 안 제약 < 4 또는 고윳값 비 < 1e-4 면 이전 중심 유지, 단위 시험 4경우. F-278: 카메라별 색인. F-276: 진단만(시드 15 카메라 41 은 점 관측 0 인 보충 카메라 2.339 m, 시드 7 카메라 5 원인 미확인). 총괄 재확인: fmt 통과, `translation_averaging` 10 통과·0 실패·4 무시(136 s). 전체 시험은 미실행 |
  | tracks (#15) | `feat/tracks` edb095a | experiment/tracks 06e61fb | 트랙 코드 변경 없음. 편대 기본 장면 재현율 30/40/50% × 오대응 0/5/10%: 완전도 0.9937~0.9996, 순도 0.9857~1.0. 시험 `formation_recall_table`(완전도 ≥0.95, 오대응 ≤5% 순도 ≥0.97). 총괄 재확인: fmt 통과, `tracks` 17 통과·0 실패·2 무시 |
  | patchmatch (#6, F-048) | `feat/patchmatch` 36aa847 | experiment/patchmatch 46f880c | 기본 경로를 체커보드 경로로, 120 px 시작 4층, 상위층 표본 간격 4, 상위 3 조기 중단, 세밀층 후보 비용 < 0.9. 경사·계단 법선 통과(이전 5.47°·5.36° 실패). 960×540 이웃 8장 CPU 10.28 → 4.77 s(4 스레드, 부하 15~20, CPU/4 1.19 s), 법선 중앙 2.43 → 2.85°. clippy 통과, 전체 시험 미완. 총괄 재확인 못 함 → 라벨 없음 |
  | pipeline-height (F-268·F-270) | `feat/pipeline-height` 6724181 | experiment/pipeline-height 763e15e | 삼각측량 문턱·GPS σ 수평/수직을 설정으로, synth 기체별 GPS 치우침 옵션. 끝까지 단구역 수치 변화 없음(verify 5/7, preview_align 6.97 m, 높이 차 4.62 m). 높이 차는 초벌 점 깊이 쪽으로 추정. 전체 시험 중 `sparse::formation_scene_registers_all_and_meets_floors` 실패(중심 2.50/5.09 m, 부하 20, 원인 미확인) |
  | pipeline-regions | `feat/pipeline-regions` 07c48d5 | experiment/pipeline-regions f75bc59 | 정밀 구역 재정렬을 이웃 따라 연쇄 합성: 3구역 겹침 차 3.04 → 2.40 m, 스케일 차 6.49%, verify 4/7. 남은 차이는 구역별 정밀 모델 자체(스케일·높이) — 구역 경계 묶는 BA 필요 |
  | rotation-coverage (F-209·F-148) | `feat/rotation-coverage` 4fa2bc6 | experiment/rotation-coverage 3ae6d09 | main 에서도 기본 bench 8/24: 8위치 bench 는 카메라 간 짝이 구조적으로 안 겹침(원인은 평면 순위 아님). 겹치는 편대 시험: 16/16, 2° 초과 3.9%, 정렬 중앙 0.141°. bench 에 짝 종류별 성공 수·성분 수 출력 |
  | pipeline-ta | `feat/pipeline-ta` 2e18bd8 | experiment/pipeline-ta 8786b03 | feat/pipeline + #37 합침(충돌 없음), `PipelineConfig::position`(기존 GPS 최소제곱 기본 / 위치 평균 선택, 3시점 이상 트랙 최대 3000개, 실패 시 기존 방식). 위치 평균 쪽 끝까지 수치는 마감 안에 못 냄(기존 쪽 120/120, verify 6/8 — preview_align 6.97 m·높이 차 4.62 m 실패, 119 s). fmt·clippy·전체 시험 미실행. 측정: `PIPE_POSITION=ta cargo test --release -p skylens-stream --test pipeline synthetic_single_region -- --nocapture` |
  | pipeline (#46, E01 합치기) | `feat/pipeline` ee9eb18(변경 없음) | | regions 합치기 충돌 해결·빌드까지 했으나 커밋 단계가 이 측정 기계 권한 설정에서 막혀 진행 못 함 |
  | fusion-consistency | 시작 못 함 | | 권한 설정에서 막힘 |
- 끝까지 흐름 진척: 지난 회차와 같음 — `feat/pipeline` 단구역 synth → run → verify 5/7, PLY·스냅샷·manifest 출력. 위치 평균 연결(pipeline-ta)·높이 설정(pipeline-height)·구역 연쇄 재정렬(pipeline-regions)이 각자 브랜치에 있고 아직 하나로 모이지 않음.
- 다음 할 일:
  0. #48 CI: `sparse::formation_scene_registers_all_and_meets_floors` 실패 원인을 `feat/pipeline-height` 6724181 에서 찾기(번들 조정 뒤 회전 오차 20.75° — 삼각측량 문턱·σ 설정 변경 영향으로 추정).
  1. `feat/pipeline` 에 regions 07c48d5 → preview 2406456 → tracks bb5ba24 → pipeline-ta 2e18bd8 → pipeline-height 6724181 순서로 합치고 끝까지 시험.
  2. 초벌 점 깊이(높이 차 4.6 m): 초벌 점군 전에 GPS 사전항 BA 몇 회.
  3. 구역 경계 스케일을 묶는 단계(겹치는 사진 포즈 고정 또는 합친 BA).
  4. PatchMatch 960 층 반복·평가 시간, 부하 없는 벽시계 재측정. 융합 일치 검사(재투영·상대 깊이·최소 시점 수).
  5. F-276 80경우 표·시드 21~25, F-209 확인 기준 개정(겹치는 위치 포함 bench).
- 막힌 점:
  - 4 코어 측정 기계에서 묶음 여럿이 동시에 시험하면 부하 15~20 으로 전체 시험이 마감 안에 끝나지 않고, 시간 단언 시험(`verify::nn_median_many_identical_points_is_fast_and_exact`)이 부하로 실패함.
  - `feat/pipeline` 합치기 커밋이 이번 회차에 권한 설정으로 막힘 — 소유자 확인 필요.
  - 소유자 병합 필요: #37·#15(통과 판정, 라벨 유지).
  - 결정 필요: 보조 사진 SPEC §3.5, F-197 카메라 간 일정, F-251 기준.
