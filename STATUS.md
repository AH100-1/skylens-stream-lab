# 현재 상태

- 상태: 쉬는 중
- 마지막 갱신: 2026-10-04T05:50Z (05:06Z 시작분)
- 이번 회차 결론: **SPEC §6 기본 배치(위치 80곳·1 m 간격·240장·2구역)에서 synth → run → verify 가 끝까지 돈다** — 시드 1 은 7/7, 시드 2 는 6/7(이웃 정밀 구역 겹침 차 0.646 m > 0.3 m). 밀집 깊이 960px 기본 경로는 4 코어 부하 1.4 에서 장당 0.53~0.59 s 로 1 s 목표 충족(F-292 처리됨-검증대기). 초벌 정렬에 카메라 중심 짝을 더해 초벌 중심 오차 0.635 → 0.428 m. 편대 카메라 간 일정 시작 +28 로 120/120 반환·2° 초과 0.8%, 다만 정렬 중앙 1.287°.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | patchmatch-speed (F-048·F-292) | `feat/patchmatch-speed` 207d5ad → PR #52(base feat/patchmatch, review-requested) | experiment/patchmatch-speed 0581ed6, 연구 PR #79 | 총괄 재확인: fmt·clippy 통과, patchmatch 12 통과·4 무시(부하 약 23). 알고리즘 변경 없음 — 기본 경로는 이미 체커보드 병렬·창 통계 캐시·상위 k·단계식 해상도. 시험 통계에 95%·유효 비율 추가. 최하위 층 반복 2회로 줄이면 법선 4.88°, 1회 12.01° 라 기각 |
  | pipeline-e2e-spec (F-273) | `feat/pipeline-e2e-spec` 2cd505b (PR 없음) | experiment/pipeline-e2e-spec 539f918 | README 명령을 SPEC 기본 배치로(synth 64 s·run 244 s, 부하 약 20). 새 시험 2시드: 중심 오차 중앙/최대 0.279/3.152·0.518/3.306 m, 표면 거리 중앙/95% 0.447/2.261·0.479/4.588 m, 높이 차 1.048·1.239 m, 재투영 0.284·0.304 px. 시드 2 는 6/7 을 단언하는 형태 — 7/7 로 고쳐야 PR. 회전 오차는 출력에 회전이 없어 못 잼. 총괄 재확인 못 함(시험 하나 8~9분) |
  | pipeline-preview-center | `feat/pipeline-preview-center` c9e0330 (PR 없음) | experiment/pipeline-preview-center b04175c | 원인: 닮음 맞춤이 점 짝만 써서 0.487 → 0.635 m. 점 짝 + 중심 짝(3배) 으로 0.428 m, 높이 차 0.570 → 0.463 m, 최근접 0.436 → 0.357 m. 초벌 중심 정밀화 고윳값 비 하한 3e-3 + 경계 시험. cli 7/7 시험은 1/5 만 끝나 미확인. F-288·F-291 원래 위치(translation_averaging.rs)는 손대지 않음 |
  | formation-view-graph (F-148·F-197·F-209·F-251) | `feat/formation-view-graph` 80c565b (PR 없음) | experiment/formation-view-graph 418d0a5 | bench 기본 40위치(120장)·`scheduled_pairs` 사용. 시작 +20 → +28: 2° 초과 전체 5.8 → 0.8%, 카메라 간 39.2 → 8.3%, 반환 120/120, 성분 1, 정렬 중앙 0.504 → 1.287°(악화). 정상 수 문턱으로는 틀린 카메라 간 간선이 갈리지 않음. `cargo test` 미완 |
  | tracks-consistent (F-263·F-125·F-264·F-265) | `feat/tracks-consistent` 18958f9 (PR 없음) | experiment/tracks-consistent 4391081 | 에피폴라 근처 오대응 2%: 재현율 30/50% 순도 0.9949/0.9995·완전도 0.9976/0.9996. 일관 교환 1%: 순도 0.978/0.983(0.99 미달). 몰린 짝 변위 거름 21배 → 3.3배(분위 격자). 마지막 커밋 뒤 core lib 재실행 안 함 |
  | fusion-consistency (F-229·F-282~F-285) | `feat/fusion-consistency` e28ab33 → PR #53(base feat/pipeline, review-requested) | experiment/fusion-consistency bca8a23, 연구 PR #80 | 총괄 재확인: fmt·clippy 통과, fusion 20 통과·2 무시. 시험만 바뀜. 편대 42장 σ 0.1%·이상치 10% 기본 설정 391670점·0.3 m 초과 0·최대 0.0696 m(무리 설정 없으면 최대 11.538 m). 조건별 조임 점 수: 재투영 0.1 px 44, 상대 깊이 0.001 1367, 최소 시점 5 2531(기준 5931) |
  | ta-seeds (F-276) | 커밋 없음 | experiment/ta-seeds a487c11 | 시드 1~20 × 짝 이상치 10/20% × 점 이상치 0/5%: 모두 240/240, 16경우 최대 > 1.0 m(최악 1.746 m, 시드 7). 같은 시드가 이상치 비율과 무관하게 실패 → 이웃 기하 퇴화 의심(F-303 표 필요). F-215 는 triangulation.rs 범위라 손대지 않음 |
- 끝까지 흐름 진척: SPEC 기본 배치 240장·2구역에서 synth → run → verify 가 PLY·스냅샷·manifest 까지(`feat/pipeline-e2e-spec`, #46 머리 e388346 위). 시드 1 7/7, 시드 2 6/7. 다음 등록이 최신 정밀 모델 위에서 하지 않음(구역 자기 사진만) — SPEC 스트림 순서 중 빠진 단계. main 은 여전히 #37·#6·#46 병합 대기.
- 다음 할 일:
  1. 시드 2 겹침 차 0.646 m 원인(구역 1 정밀 앵커가 '겹침 부족'으로 건너뜀) — 고친 뒤 e2e-spec 시험을 7/7 단언으로 바꾸고 PR.
  2. preview-center 의 cli 7/7·2구역 확인 후 #46 으로.
  3. 다음 구역 등록을 최신 정밀 포즈 위에서(앵커 입력).
  4. 편대 카메라 간 간선: 평균 결과와 어긋나는 간선 제거 후 재평균, 삼각측량각 검사 — 정렬 중앙 1° 이하.
  5. F-276 시드 4·7·9·15·19 카메라별 표(F-303).
  6. 정밀 회전을 출력(poses)에 넣어 회전 오차 시험.
- 막힌 점:
  - 소유자 병합 필요: #37·#6 → #46(흐름), #52, #53, #51, #50, #49·#39~#42.
  - 4 코어 기계에 동시 묶음 7개 — 부하 15~30, 긴 cli 시험(8~9분)은 회차 안에 재확인 못 함.
  - 결정 필요: SPEC §3.2 카메라 간 일정 개정안(experiments/formation-view-graph.md), F-288·F-215 수정 범위(translation_averaging.rs·triangulation.rs), 초벌 정렬 닮음 + 보정장 허용(SPEC §3.7), F-197, F-209 확인 기준, F-251.

## 직전 실행 기록 (2026-10-04 04:06Z 시작분)
- 마지막 갱신: 2026-10-04T04:54Z (04:06Z 시작분)
- 이번 회차 결론: **F-290 해결 — 위치 평균 규모 시험 306 s → 8.5 s**(기준 60 s). 남은 비용은 반복 수였다: 풀이 8번이 모두 바깥 반복 300회 상한까지 돌았음. 구성 병렬화 + 재풀이 축척 이어받기 + 수렴 판정으로 해결, PR #37 머리 b90a177. 이어서 흐름 PR #46 을 새 머리 e388346(= preview-pos + #37 b90a177 + #6)으로 갱신 — 2구역 synth → run → verify **7/7**, 위치 평균 경로 시험 23 s 통과.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | ta-sparse-iters (F-290) | `feat/ta-sparse-iters` b90a177 → `feat/translation-averaging`(PR #37, review-requested) | experiment/ta-sparse-iters f2f03aa | 총괄 재확인: fmt·clippy 통과, `translation_averaging` 10 통과·8 무시(40.7 s), 규모 시험 8.52/8.42 s·RMS 0.0002·202 MB(4 코어 부하 3). 번호 이동 10% 단언은 짧은 시간에서 부하에 흔들림 |
  | ta-sparse-assemble | `feat/ta-sparse-assemble` 1a62b46 (위 묶음의 바탕) | experiment/ta-sparse-assemble da320f9 | 점당 카메라 목록 고정·조각별 상삼각 병렬 누적: 299.6 → 120.1 s |
  | ta-sparse-pcg | `feat/ta-sparse-pcg` eab212e (채택 안 함) | experiment/ta-sparse-pcg d616872 | 행렬 없는 곱 + 대각 전처리 공액 기울기: 114.7/101.4 s, 반복당 CG 약 3.2회. 구성 가속 쪽이 단순해 그쪽을 채택 |
  | pipeline-head (F-296) | `feat/pipeline-head` e388346 → `feat/pipeline`(PR #46, review-requested) | experiment/pipeline-head acae371 | 합침 충돌 없음, 두 파일 PR 머리와 diff 0. 작업자 실행: core `pipeline` 8 통과, cli pipeline_arrival·pipeline_regions·pipeline_stream·pipeline_stream_order·`--test pipeline` 4개 통과. `pipeline_e2e::two_region_end_to_end` 는 6/7 실패를 기대하던 낡은 기대 → 7/7·높이 차 < 2 m·최근접 < 3 m·점쌍 ≥ 1000 으로 갱신, 총괄 재실행 통과(38 s: 최근접 0.943 m, 높이 차 1.048 m, 점쌍 1178, 겹침 0.285 m). 위치 평균 경로 시험 총괄 재실행 23 s 통과 |
  | pipeline-preview-offset | `feat/pipeline-preview-offset` f489429 (측정 시험 + 기본 꺼진 선택지, PR 없음) | experiment/pipeline-preview-offset 0b971c4 | 초벌 공통 회전 2.75° 는 좌표계 맞춤의 **롤 선택** 단계에서 생김(방향 맞춤 직후 0.80°). 공통 회전 뺀 오차는 모든 단계 0.16°. stride 2 에서는 롤 선택이 방향 맞춤의 8.1° 롤을 0.48° 로 고쳐 주므로 없앨 수 없음. 특이값 비 문턱으로 건너뛰면 stride 1 높이 차 0.570 → 0.498 m — 기준 이미 충족이라 기본 끔 |
- 끝까지 흐름 진척: PR #46 머리 e388346 에서 synth → run → verify 단구역·2구역 7/7, PLY·스냅샷·manifest 까지. 위치 평균 경로도 이 머리에서 끝까지 23 s. main 은 여전히 트랙·회전 평균·융합까지 — #37·#6·#46 병합 대기.
- 다음 할 일:
  1. 감독 판정 반영(#37 b90a177, #46 e388346). F-290 번호 이동 10% 단언을 반복 측정 최소값 비교로 바꿀지 결정.
  2. 초벌 정렬 뒤 초벌 중심 오차가 0.49 → 0.64 m 로 느는 원인 분해(preview-offset 남은 문제).
  3. legacy_roll 제거(공유 영상 닮음 + 공유 점 약한 관측).
  4. 밀집 깊이 속도(F-048) 부하 낮은 벽시계 재측정을 노트에(F-292).
- 막힌 점:
  - 소유자 병합 필요: #37·#6 → #46(흐름), #51, #50, #49·#39~#42.
  - 4 코어 기계라 동시 묶음을 5개로 제한(빌드·시간 측정이 서로 간섭, 부하 최대 16).
  - 결정 필요: 초벌 정렬 닮음 + 보정장 허용(SPEC §3.7), F-197, F-209 확인 기준, F-251.

## 직전 실행 기록 (2026-10-04 02:05Z 시작분)
- 마지막 갱신: 2026-10-04T04:07Z (04:06Z 시작분)
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

