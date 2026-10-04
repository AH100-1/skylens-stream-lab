# 현재 상태

- 상태: 진행 중
- 마지막 갱신: 2026-10-04T03:06Z (03:05Z 시작분)
- 이번 회차 결론: **2구역 synth → run → verify 가 처음으로 7/7**. 초벌 카메라 중심만 회전 고정으로 다듬는 단계(점–카메라 광선 제약, Huber, 5바퀴, 2° 관측·1° 삼각측량각 거르기)를 넣어 2구역 stride 1 초벌↔정밀 높이 차 중앙 최대 4.715 → 1.048 m, 최근접 4.118 → 0.943 m, 초벌 정렬 잔차 4.965 → 0.735 m. 다듬기 자체 0.06 s. 단구역도 7/7(높이 차 0.266 m). 가지 `feat/pipeline-preview-pos` b50b837(= feat/pipeline-scope + 다듬기 + 시험 기대 갱신). 위치 평균 F-290 은 점 번호 압축·점 소거 축소 계통까지(PR #37 5b21293), 규모 기준은 아직 미달.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | pipeline-preview-pos | `feat/pipeline-preview-pos` 6845ede → b50b837 (PR 없음) | experiment/pipeline-preview-pos, 연구 PR #73 | 위 결론. 총괄 재확인(4 코어, 부하 2~9): fmt·clippy 통과, core `pipeline` 8 통과·2 무시, cli `pipeline` 4 통과(단구역·PatchMatch·2구역·preview_ba, `…translation_averaging` 제외). 2구역 시험 기대를 '높이 차 < 2 m 통과'로, 점쌍 하한을 SPEC 1000 으로(실측 1178, 이전 1211) |
  | ta-sparse (F-290) | `feat/translation-averaging` 66c8246·5b21293 (PR #37 갱신) | experiment/ta-sparse dbedeb6, 연구 PR #78 | 점 번호 압축, 점 소거 n_cam×n_cam 촐레스키. 총괄 재확인: fmt·clippy(5b21293 에서 clippy 한 곳 고침) 통과, `translation_averaging` 10 통과·8 무시. 240장×4000 관측 무시 시험은 부하 17 에서 772 s(기준 60 s) — 축소 행렬 구성 비용 남음 |
  | pipeline-preview-rot | `feat/pipeline-preview-rot` ff759d5 (진단 옵션, 기본값 불변, PR 없음) | experiment/pipeline-preview-rot 8f1ce73, 연구 PR #74 | stride 1 회전 평균 직후 0.16°, 초벌 포즈 2.75°(95% 2.98°) = 좌표계 맞춤의 공통 오프셋. 간선 거르기·가중 10가지 모두 개선 없음. 간격 1 간선이 가장 정확(0.077°) |
  | pipeline-refined-link | `feat/pipeline-refined-link` 5da7f05+1 (기본 꺼짐, PR 없음) | 노트 없음 | BA 고정 점 사전항 + 이웃 정밀 구역 연결: 새 롤+연결 σ 1 px 겹침 차 0.379 m(연결 점 131), 새 롤 단독 0.300, 예전 롤 0.285 — 미달, legacy_roll 유지 |
  | pipeline-dense-width | `feat/pipeline-dense-width` 32110bb (구간 시간·측정 시험, PR 없음) | experiment/pipeline-dense-width 2b67514, 연구 PR #75 | 정답 자세 구역 하나(48장), 부하 12: 폭 96/240/480/960 합계 21/36/51/103 s, 표면 중앙 0.040/0.036/0.038/0.036 m, 점 2.8만/20만/87만/356만. 960 은 추정 86%. README 예시 폭 240 권장 |
  | patchmatch-f293c (F-293·F-292) | `feat/patchmatch-f293c` 0bbbc34 (측정 시험만) | experiment/patchmatch-f293c 50561fa(연구 PR #77), experiment/patchmatch-f292 a600af6 | 폭 480: skip 0 표면 중앙/95% 0.0114/0.0581 m·점 717k, skip 0.08 0.0331/0.1369 m·695k·시간 −20%. timing_960 은 부하 15 에서 2.44~2.55 s(낮은 부하 값 여전히 없음) |
  | pipeline-two-region-notes (F-295) | `feat/pipeline-two-region-notes` 439e3b2 | experiment/pipeline-two-region-notes f51b7f3, 연구 PR #76 | 주석 = 출력(다듬기 이전 기준). preview-pos 가지에서 다시 갱신됨 |
- 끝까지 흐름 진척: synth → run → verify 가 단구역·2구역 모두 **7/7** (`feat/pipeline-preview-pos`). 흐름 PR #46(03f6009)은 아직 이전 머리 — pipeline-scope 계열은 #37 판 위치 평균 경로 시험이 규모 문제(F-290)로 끝나지 않아 올리지 않음.
- 다음 할 일:
  1. F-290 마무리: 축소 행렬을 블록 단위·병렬로 구성(또는 블록 야코비 PCG), 반복마다 재분해 줄이기 → `synthetic_single_region_translation_averaging` 이 끝나게 한 뒤 `feat/pipeline-preview-pos` 를 #46 으로 올림.
  2. 초벌 공통 회전 오프셋(stride 1 2.75°) 분해: GPS 방향 맞춤과 롤 선택 중 어디인지. 다듬기 뒤 정답 대비 중심 중앙·공통 회전 뺀 높이 차 측정.
  3. legacy_roll: 공유 영상 기반 닮음(양방향 재투영 비율)으로 먼저 맞추고 공유 점 상수 관측으로 BA, 연결 점 수 늘리기.
  4. README 예시 `--dense-width 240` 반영(P04 몫), 정밀 경로 skip_cost 기본값 결정.
- 막힌 점:
  - 소유자 병합 필요: #46, #37·#6, #50, #49·#39~#42, #51.
  - 결정 필요: PatchMatch 정밀 경로 skip_cost 기본 0(F-293 표 근거), 초벌 정렬 닮음 + 보정장 허용(SPEC §3.7), F-197, F-209 확인 기준, F-251.
  - 이번 회차는 4 코어 기계 부하가 11~17 로 높아 시간 측정값(F-292 0.7 s, F-290 60 s, 구역 30 s)은 판정에 쓸 수 없음.
  - 연구 PR #73~#78 본문 끝에 저장소 쪽에서 붙는 꼬리말 한 줄이 남음(수정해도 다시 붙음).

## 직전 실행 기록 (2026-10-04 01:05Z 시작분)
- 마지막 갱신: 2026-10-04T01:37Z
- 이번 회차 결론: 흐름 머리(PR #46, 03f6009)는 그대로. 2구역 초벌↔정밀 높이 차(4.4~4.6 m, 기준 2 m)의 원인을 분리 — **초벌 카메라 위치 오차(중심 중앙 1.09 m)가 stride 1 의 짧은 기선(상위 3 이웃 기선 각 중앙 6.7°)에서 깊이 오차로 증폭**된 것이며 밀집 깊이·구역 간 정렬·롤 규칙 탓이 아님. F-296 확인 기준(흐름 가지의 위치 평균·PatchMatch 파일 = 각 PR 머리)을 `feat/pipeline-scope` 에서 충족했으나, #37 머리 판의 `gp_solve` 가 흐름 입력에서 14분 넘게 끝나지 않아 #46 은 갱신하지 않음. F-231·F-306 고침(PR #51, CI 통과).
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | pipeline-scope (F-296) | `feat/pipeline-scope` e1dc44c (PR 없음) | experiment/pipeline-scope 830da98, 연구 PR #71 | 03f6009 + #37 aaff35f + #6 61fa09a `--no-ff` 합침, 충돌은 lib.rs 모듈 목록 한 곳. 두 파일 PR 머리와 diff 0. fmt·clippy 통과, core `pipeline` 7/0/2, cli pipeline.rs 단구역·PatchMatch·2구역 e2e·preview_ba 통과, 총괄 재실행 pipeline_arrival 2·pipeline_e2e 2·pipeline_regions 3·pipeline_stream 1·pipeline_stream_order 1 통과. **`synthetic_single_region_translation_averaging` 은 #37 머리 `gp_solve` 에서 14분+ 미종료**(F-290 밀집 행렬 의심) — 이 상태로 #46 에 올리면 CI 가 끝나지 않음 |
  | pipeline-region-align | 변경 없음 | experiment/pipeline-region-align db12821 (예전 같은 이름 가지 위, PR 없음) | 겹침 띠 점 + 공유 사진 중심 결합 닮음: 예전 롤 표면 중앙 0.447 → 0.483 m(악화), 새 롤 0.630 → 0.650 m. 새 롤은 중심 가중·σ×3·×5 어느 조합도 겹침 차 0.300~0.329 m(> 0.3). 예전 롤 겹침 0.285 통과. legacy_roll 유지. 단구역 새 롤 7/7(높이 차 1.364 m) |
  | pipeline-preview-depth | 변경 없음 | experiment/pipeline-preview-depth a088d1a, 연구 PR #72 | 구역 0, stride 1 높이 오차 중앙: 초벌 포즈 5.96 m, 정답 포즈 0.12 m, 회전만 정답 5.27 m, 위치만 정답 0.52 m; stride 2 초벌 1.21 m·정답 0.11 m. 초벌 회전 중앙 stride 1 2.75° 대 stride 2 0.48°. 최소 기선 각 9°/12° 강제는 4.76/4.47 m 로 기준 미달 — 커밋 안 함 |
  | test-timing-bounds (F-231·F-306) | `feat/test-timing-bounds` 30fc1c5, **PR #51**(CI 통과, review-requested) | experiment/test-timing-bounds 113071f, 연구 PR #70 | nn 질의당 노드 ≤ 32·점 ≤ 64 단언(실측 15/12), `>` 변이는 노드 32767 로 실패. 융합 직렬 구간 1357 → 751 ms, 단언 2.0 s × 8 / 스레드, 4 코어 부하 2 안팎 2.58 s < 4.00 s. 총괄 재확인: fmt·clippy 통과, core verify·fusion 30 통과·2 무시 |
- 끝까지 흐름 진척: 변함없음 — synth → run → verify 가 PLY·스냅샷·manifest·timing.json 까지(PR #46). 2구역 남은 기준: 초벌↔정밀 높이 차(원인 = 초벌 위치), legacy_roll 없이 겹침 차 0.3 m.
- 다음 할 일:
  1. 초벌 위치 개선: 구역 초벌에 짧은 위치 전용 다듬기(이웃 상대 이동 제약 또는 BA 2~3회)를 넣어 stride 1 중심 중앙 1.09 m 를 줄이고 2구역 높이 차 < 2 m 확인. stride 1 초벌 회전 2.75° 원인(회전 평균 입력 짝) 분리.
  2. #37 `gp_solve` 를 띄엄띄엄한 연립(F-290)으로 바꿔 흐름 입력에서 끝나게 한 뒤 `feat/pipeline-scope` 를 #46 으로 올림.
  3. legacy_roll 제거: 구역 1 정밀 BA 에 구역 0 공유 점을 약한 관측으로.
- 막힌 점:
  - 소유자 병합 필요: #46, #37·#6, #50, #49·#39~#42, 새 #51.
  - #37 머리를 흐름에 합치면 위치 평균 경로 시험이 끝나지 않음 — #37 병합 전 F-290 처리 권장.
  - 결정 필요: 초벌 정렬 닮음 + 보정장 허용(SPEC §3.7), F-197, F-209 확인 기준, F-251.

## 직전 실행 기록 (2026-10-04 00:05Z 시작분)
- 마지막 갱신: 2026-10-04T00:40Z
- 이번 회차 결론: 흐름 머리 `feat/pipeline-merge-2306`(03f6009) 전체 시험을 부하 낮은 첫 시점에 출력 파일로 끝까지 돌림 — fmt·clippy 통과, core lib **325 통과·0 실패·25 무시**(566 s), dataset_synth 2·perf_structure 6·cli 단위 3, cli 통합 pipeline_e2e 2(단구역·**2구역**)·pipeline_arrival 2·pipeline_regions 3·pipeline_stream 1·pipeline_stream_order 1 통과, `nn_median` 단독 2 통과. 이 머리를 `feat/pipeline` 로 빨리 감기 푸시해 **PR #46 갱신·review-requested**. 2구역 초벌 스케일 차는 이 머리에서 0.36% — 지난 1.38% 는 재현되지 않음. legacy_roll 제거는 원인만 분리(구역 1 BA 수렴 + 구역 간 정렬 증폭), 수정 없음.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | 머리 전체 시험 (merge-2306) | `feat/pipeline` = `feat/pipeline-merge-2306` 03f6009, PR #46 | — | 위 결론 수치. cli `pipeline.rs` 등 나머지 통합 묶음은 회차 끝까지 진행 중(개별 실행으로 흐름 시험은 모두 확인) |
  | pipeline-preview-scale2 | `feat/pipeline-preview-scale2` 4946dee(측정 시험만) | experiment/pipeline-preview-scale2 0a945de, 연구 PR #68 | 구역 0/1 점쌍 1355/1211, 잔차 4.965/4.483 m, 스케일 0.9464/0.9430(0.36%). 정렬 전 롤 오차 비행 축 성분: 새 규칙 −2.04°/+2.86°, 예전 −1.23°/−1.44°. 점군 기울기 새 3.44°/4.81°, 예전 3.13°/6.93°. 코드 변경 불필요 |
  | pipeline-region-refined | 변경 없음 | experiment/pipeline-region-refined ddbca64, 연구 PR #69 | 새 롤 시작 시 구역 0 표면 정렬 전 0.428 → 정렬 후 0.595 m(예전 0.395 → 0.406). 구역 1 은 BA 50회면 두 시작 모두 0.65~0.66 m — 예전 롤은 15회에 덜 수렴해 0.501 m. GPS 사전항 σ×5: 표면 0.519 m·중심 0.343 m 통과, 겹침 차 0.347 m(> 0.3) 실패. legacy_roll 유지 |
- 끝까지 흐름 진척: synth → run → verify 가 PLY·스냅샷·manifest·timing.json 까지, 단구역·2구역 e2e 시험 모두 통과하는 머리가 PR #46 에 올라감. main 은 트랙·회전 평균·융합까지(위치 평균 #37·밀집 깊이 #6 병합 대기).
- 다음 할 일:
  1. PR #46 감독 판정 반영.
  2. legacy_roll 제거: 겹침 띠 + 구역 카메라 중심을 함께 쓰는 구역 간 닮음 추정, 또는 앞 구역 공유 사진 포즈를 약한 사전항으로(σ×5 와 결합).
  3. 2구역 초벌↔정밀 높이 차(최선 4.555 m, 기준 2 m).
  4. F-292 노트·F-305 문서.
- 막힌 점:
  - 소유자 병합 필요: #46(흐름), #37·#6, #50(→ feat/patchmatch), #49·#39~#42.
  - 결정 필요: 초벌 정렬 닮음 + 보정장 허용 여부(SPEC §3.7), F-197, F-209 확인 기준, F-251.
  - 머리 계보 옛 커밋 3개(63d15a9·634844d·5de63af) 메시지 꼬리 줄 — 소유자 판단 필요.

