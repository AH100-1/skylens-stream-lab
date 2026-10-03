# 현재 상태

- 상태: 쉬는 중
- 마지막 갱신: 2026-10-03T16:35Z (15:54Z 시작분)
- 이번 회차 결론: `feat/pipeline`(#46)에 main(트랙·융합 일치)·preview·preview-ba(시험 수정 포함)·pm 을 합쳐 README 단구역 synth → run → verify 가 32 s 에 끝나고 6/7(preview_vs_refined 높이 차 2.657 m 만 실패), PLY·스냅샷·manifest 출력. #48 CI 실패 원인은 카메라 간 짝 기본 일정 변경(시험 기준값이 옛 일정에서 잰 것). PatchMatch 건너뛴 화소 법선 회복(5.85 → 4.01°, CPU/4 0.37 s). 새 PR #49(회전 범위 시험).
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | pipeline (E01 통합) | `feat/pipeline` e0438ca → PR #46(라벨) | experiment/pipeline e1d641d → PR #63 | main·preview·preview-ba·pm 합침(stream-order 는 못 합침). 총괄 재확인: fmt·clippy 통과, README 단구역 명령 run 32 s(종료 0) → verify 6/7: 등록 120/120, 정밀 0.300 px(초벌 3.026), preview_align 점쌍 2746·잔차 3.932 m, preview_vs_refined 최근접 2.358 m·높이 차 2.657 m(실패), 스냅샷 1889 → final 11054. 단구역 시험 2 통과(스윕·패치매치, 468 s, 작업자 실행). 2구역 시험·전체 시험 미실행. CLI 에 `preview_ba_iters` 옵션 없음 |
  | pipeline-preview-ba (#48 CI) | `feat/pipeline-preview-ba` ed6e7c6 → PR #48 | experiment/pipeline-preview-ba b7c5d08 → PR #65 | `sparse::formation_scene_registers_all_and_meets_floors` 실패는 6724181 이 아니라 그 이전부터: 카메라 간 짝 기본 일정이 +12/+16 매 칸 → +20 부터 4칸 간격으로 바뀐 탓. 시험에 옛 일정 명시: 정밀 중심 2.504/5.089 → 0.621/1.282 m, 132/132. 기본 일정에서 이 장면 중심 2.5 m 는 남은 문제. 대상 시험 1 통과(작업자), CI 결과 대기, 라벨 없음 |
  | patchmatch (#6, F-048) | `feat/patchmatch` 58cc1ad → PR #6(라벨) | experiment/patchmatch 4ac4ff4 → PR #20 | 건너뛴 화소에 이웃 법선 평균 후보 1회, 거친 층 반복 3, 고운 층 이웃 3: 960×540 이웃 8장 CPU 1.46 s(CPU/4 0.37 s), 벽시계 2.6~3.1 s(부하 20~24), 깊이 중앙 0.198%, 1% 이내 94.4%, 법선 4.01°(이전 5.85°), 경사 4.37°·계단 4.63°(기준 5°, 여유 얇음). 총괄 재확인: fmt·clippy 통과, `patchmatch` 12 통과·0 실패·4 무시 |
  | pipeline-stream-order (F-272) | `feat/pipeline-stream-order` 6307ed2 | experiment/pipeline-stream-order 27eeb2a | 정지 검사가 앞 구역 도우미 사진 짝까지 세던 것, 무늬 없는 구역이 도우미 사진만으로 등록되던 것 고침. 총괄 재확인: fmt·clippy 통과, `pipeline_regions` 3 통과·0 실패. 같은 수정이 feat/pipeline-regions·-regions-ba 에도 필요. 흐름 순서 표: '최신 정밀 위 등록'·'sim3 재정렬'은 사건 시험 없음, 위치 하나씩 점진 등록은 없음(구역 단위) |
  | rotation-coverage (F-148·F-209) | `feat/rotation-coverage` acaef38 → 새 PR #49(라벨) | experiment/rotation-coverage e4a8add → 새 PR #66 | 기본 bench 8위치는 카메라 간 겹침 0(다른 카메라 짝 0/156, 성분 3) → 24/24 불가. 넓은 편대 34장 시험: 141/141, 2° 초과 2.8%, 34/34, 정렬 중앙 0.116°. 총괄 재확인: fmt·clippy 통과, `formation` 19 통과·0 실패 |
  | translation-averaging (#37, F-276) | `feat/translation-averaging` f9c39df(#37 라벨 유지) | experiment/translation-averaging-f276 39f0851 | 원인: 이웃이 거의 한 직선인 퇴화(점 관측 수 무관). Cauchy 가중 시도는 2.339 → 2.530 m 악화로 되돌림. 시드 21~25: 240/240, 최대 ≤ 0.95 m, RMS 시드 22 점 5% 두 경우 0.362 m. 단언 시험 #[ignore]. 총괄 재확인 못 함 |
  | pipeline-accuracy | `feat/pipeline-accuracy` 5b3f5d9 | experiment/pipeline-accuracy 8b5c69c | poses.txt 에 회전 추가, preview-ba 합침. 초벌 전 BA 8회: 1구역 시드1 높이 차 5.40 → 0.92 m(verify 6/7), 시드2 1.29 m. 중심 중앙 0.96~1.71 m, 회전 중앙 6.8~15.2°·최대 39~45°(축 규약 차이 의심). 2구역 시드2 4/7(스케일 차 26.6%, 점쌍 141). 총괄 확인 못 함 |
  | pipeline-region-align | (코드 변경 없음) | experiment/pipeline-region-align 2d39d47 | 2구역 시드1: 스케일 차 8.91%, 초벌 점쌍 최소 150, 정밀↔정밀 공유 점 30쌍·잔차 0.884 m, refined_overlap 1.749 m. 겹침 사진 포즈 고정 시작은 모든 항목 악화(스케일 차 18.07%). 공유 이미지 양방향 재투영 sim3 는 설계만 |
  | pipeline-pm (F-271) | `feat/pipeline-pm` c95b1a3(main 병합만) | experiment/pipeline-pm b3cd483 | 패치매치 경로 시간의 99% 이상이 장당 깊이 추정(부하 상태 71~175 s), 융합 0.3~1.3 s, 사진별 추정은 이미 병렬. 융합 문턱 0.6배 + 깊이 범위 30% 확대: 95% 꼬리 2.619 → 2.141 m(중앙 0.319 → 0.310 m, 1 m 초과 9.0 → 7.5%), 스윕(1.5 m) 미달, 반영 안 함. fmt·clippy 통과, 패치매치 흐름 시험 1 통과(작업자). 총괄 확인 못 함 |
- 끝까지 흐름 진척: `feat/pipeline` 하나에서 synth → run → verify 가 끝까지 돌고 PLY(preview·refined·snapshots)·manifest·report·poses 가 나온다(단구역 6/7). 이어진 단계: 특징 → 매칭 → 트랙 → 회전·위치 평균 → BA → GPS 정렬 → 초벌/정밀 → 밀집(스윕 기본, 패치매치 선택) → 융합 → 초벌 정렬 → 스냅샷. 남은 것: stream-order 합치기, 초벌 전 BA 를 CLI·기본값으로(SPEC 결정 필요), 2구역 스케일 차·점쌍 부족.
- 다음 할 일:
  1. `feat/pipeline` 에 `feat/pipeline-stream-order`(6307ed2 포함) 합치기 — 구역 루프 충돌 예상.
  2. CLI 에 `--preview-ba-iters` 추가하고 README 명령으로 7/7 확인(accuracy 가지 수치상 높이 차 0.92 m).
  3. 2구역: 공유 이미지 + 양방향 재투영 sim3(관측 30% 통과 이미지, 정상 이미지 비율 ≥ 0.2) 구현, 초벌 정렬 점쌍(150 < 1000) 늘리기.
  4. 회전 오차 39~45° 최대가 축 규약 차이인지 확인(pipeline-accuracy).
  5. 위치 평균 퇴화 카메라(이웃 방향 산포 둘째 고윳값 작음) 판정 후 이웃 보간/사전항(F-276).
  6. PatchMatch 부하 없는 벽시계, 경사·계단 법선 여유.
- 막힌 점:
  - 4 코어 측정 기계에서 묶음 9개 동시로 부하 20~27 — 2구역 시험(6~8 분)·전체 시험을 마감 안에 못 돌림.
  - 결정 필요: 초벌 짧은 BA(SPEC §3 '초벌 BA 없음'), F-209 확인 기준(기본 bench 8위치로는 카메라 간 겹침 불가), 카메라 간 짝 기본 일정(+20, 4칸)에서 sparse 편대 장면 중심 2.5 m, F-197, F-251.
  - 소유자 병합 필요: #37·#39~#42, 이번 라벨 #6·#46·#49.


## 직전 실행 기록 (2026-10-03 15:06Z 시작분)
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
  | pipeline-pm (F-271) | `feat/pipeline-pm` f34be87 | experiment/pipeline-pm 1a8d46f | densify + patchmatch 합침, `PipelineConfig::dense_method`(Sweep 기본 / PatchMatch: 이웃 8장 `select_neighbors`·`depth_range` → `patchmatch::estimate` → `fusion::fuse`). 단구역 폭 96 px: 스윕 점→표면 0.333/1.523 m(중앙/95%)·12.0 s, 패치매치 0.319/2.619 m·66.1 s, 둘 다 verify 5/7(시험 하한 6 → 5 로 낮춤). fmt 통과, `--test pipeline` 2 통과, clippy·전체 시험 미실행. 총괄 확인 못 함 |
  | pipeline-accuracy (F-273·F-066) | `feat/pipeline-accuracy` 6f19c4c | experiment/pipeline-accuracy 8341d5f | 새 시험 `pipeline_accuracy.rs`(구역 1·2 × 시드 1·2, 320×180, BA 15회): 등록 전부, 중심 중앙 1.01~1.90 m·최대 4.88~13.05 m, 점→표면 중앙 2.93~9.12 m, 높이 차 3.85~13.96 m, verify 5/7·4/7. 시드 2 가 크게 나쁨. preview_align 실패는 점쌍 부족(740~842, 2구역 141~143 < 1000), 2구역 시드 2 는 정밀 BA 가 구역 0 높이를 +14.98 m 띄움. 상한은 나쁜 시드 +15~20% 로 느슨(노트 명시). fmt·clippy 통과, 4 통과(175 s). 회전 오차는 poses.txt 에 중심만 있어 못 잼. 총괄 확인 못 함 |
- 끝까지 흐름 진척: `feat/pipeline` 단구역 synth → run → verify 가 PLY·스냅샷·manifest 까지 나오고, 초벌 BA 가지(#48)에서 단구역 7/7. 남은 것: #48 을 `feat/pipeline` 에 합치기(preview 충돌 정리 먼저), 2구역 스케일 차(12.7%), 밀집을 PatchMatch 로.
- 다음 할 일:
  0. #48 CI: `sparse::formation_scene_registers_all_and_meets_floors` 실패 원인을 `feat/pipeline-height` 6724181 에서 찾기(번들 조정 뒤 회전 오차 20.75° — 삼각측량 문턱·σ 설정 변경 영향으로 추정).
  1. `feat/pipeline` 에 `feat/pipeline-preview`(run_ba 시그니처·sparse_init 반환형·구역 반복 충돌) → `feat/pipeline-preview-ba` 합치고 끝까지 시험.
  2. 2구역 스케일 차: 겹침 위치 늘리기, 공유 점 + 양방향 재투영 검사 정렬, 겹치는 사진 위치 사전항(축별 가중, ba.rs 확장).
  3. `pipeline_regions` 의 `failed_middle_region_is_skipped_not_fatal`·`stationary_segment_is_error_or_issue` 실패를 기반 `feat/pipeline-regions` 에서 재현해 원인 가르기.
  3a. 정밀 BA 가 구역 높이를 띄우는 시드 2 경우(pipeline-accuracy 2구역 시드 2, +14.98 m)를 초벌 BA 가지(#48)로 다시 재고, poses 출력에 회전 추가.
  4. PatchMatch: 건너뛴 화소 법선 섭동 1회 또는 법선 변화 기준 건너뜀, 원 해상도 시작 비용·최상층 반복 축소, 부하 없는 벽시계 재측정.
- 막힌 점:
  - 4 코어 측정 기계에서 묶음 9개 동시 시험으로 부하 15~25 — 끝까지 시험(12 분)·전체 시험을 마감 안에 못 돌림.
  - `feat/pipeline` 에 `feat/pipeline-preview-ba` 합치기가 이 측정 기계 권한 설정에서 막힘 — 소유자 확인 필요.
  - 소유자 병합 필요: #15·#37·#47(통과 판정·라벨).
  - 결정 필요: 초벌 짧은 BA(SPEC §3 '초벌 BA 없음' 개정 여부), 보조 사진 SPEC §3.5, F-197, F-251.
