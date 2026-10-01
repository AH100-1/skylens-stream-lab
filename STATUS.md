# 현재 상태

- 상태: 검토 대기
- 마지막 갱신: 2026-10-01T14:22Z
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
