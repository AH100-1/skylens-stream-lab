# 현재 상태

- 상태: 쉬는 중
- 마지막 갱신: 2026-10-04T18:49Z (18:06Z 시작분)
- 이번 회차 결론: 50분 안에 확인까지 끝난 묶음 없음. 묶음 3개 진행 중(4 코어 기계), 이번 회차에 PR·FEEDBACK 상태 변경 없음.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | small-image-features (F-350·F-351·F-352) | `feat/small-image-features` cd9a360(원래 크기 검출이 목표의 절반 미만일 때만 확대로 모자란 만큼 채움) | experiment/small-image-features 27314d2 | 진행 중. 총괄 재확인(core lib 전체·skylens-stream 전체) 안 됨 → #66 병합 보류 유지 |
  | mid-image-register (480×270 54/81) | `feat/mid-image-register`(아직 푸시 없음) | experiment/mid-image-register(아직 없음) | 진행 중, 측정 위주 |
  | helper-latency-80 (F-348 80위치 표) | `feat/region-cross-extend` c11b451 그대로 | experiment/helper-latency-80(아직 없음) | 진행 중, 기본 / back 16 / back 12 / back-step 4 비교 |
- 끝까지 흐름 진척: main 779edb7 에서 전부 연결(변화 없음). #64 가 들어가면 기본 경로 7/7.
- 다음 할 일:
  1. cd9a360 을 `cargo test --release -p skylens-core --lib`(formation_scene_registers_all_and_meets_floors·preview_default_pose_error_bounds 포함)·`-p skylens-stream` 전체(pipeline_stream_order 포함)로 재확인 후 PR #66 에 라벨 다시.
  2. experiment/mid-image-register·helper-latency-80 노트가 올라오면 수치 확인 후 연구 PR.
- 막힌 점:
  - 소유자 병합 필요: #64, #65, #63·#62·#60·#61·#59·#58·#57·#56 → #55 → #54, #40, #53, #51·#49·#42·#41·#39, 연구 PR 들(#89 → #90 → #91, #92·#93·#94). #66 은 F-350 재확인 전까지 보류.
  - F-197(높음)·F-348·F-349 는 SPEC·지연 수용 결정이 소유자 몫.
  - 4 코어 기계 — 동시 묶음 3개.

## 직전 실행 기록 (2026-10-04 17:06Z 시작분)

- 상태: 쉬는 중
- 마지막 갱신: 2026-10-04T18:07Z (18:06Z 시작분)
- 이번 회차 결론: **320×240 작은 사진 등록 해결 — 2배 확대 검출(feat/small-image-features a278542 → PR #66)로 특징 400 → 1500, 등록 27/81 → 81/81, verify 6/7 → 7/7(총괄 재실행 확인 7/7·32 s).** 등록 누락을 report `issues` 에 남기고 verify 기준 열을 실제 사진 수로(PR #65). F-348 은 기록만 추가하고 대안(초벌 뒤쪽 보조 끔)은 기본 경로를 깨뜨려 옵션으로만 두고 기본은 기존 동작(PR #64 에 c11b451). 4 코어 기계라 묶음 3개.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | small-image-features | `feat/small-image-features` a278542 → **PR #66**(review-requested) | experiment/small-image-features e9e9f16 → 연구 PR #93 | `DetectorConfig::upscale_below` 800: 긴 변 800 미만이면 2배 양선형 확대에서 검출, 좌표 X/2−0.25. 81장(stride 3) 측정: 320×240 특징 400 → 1500·다른 카메라 검증 짝 0 → 10·등록 27/81 → 81/81·verify 6/7 → 7/7; 640×360 54/81 → 81/81·7/7; **480×270 은 54/81·6/7 그대로**(검증 짝 5 → 10); 960×540 변화 없음 81/81·7/7. 문턱 낮추기 안은 400 → 402 로 효과 없음. 총괄 재확인: fmt·clippy, `--lib features` 15 통과·1 무시, `--test run` 6, 320×240 synth → run → verify 7/7(32 s). `matching_under_rotation_and_scale` 는 확대 끄고 돔(확대 시 축척 1.25 정확도 0.918) |
  | register-issues | `feat/register-issues` feb05e5 → **PR #65**(review-requested, CI 통과) | experiment/register-issues f60c7b7 → 연구 PR #92 | 구역 초벌 등록 누락 시 `region 0: registered 27/81 (F 0/27, R 0/27, L 27/27), pair graph components 3` 이슈. verify 기준 열 `(81/81)`·`(240/240)` 실제 수. 총괄 재확인: fmt·clippy, core verify·register_issue 16, `--test verify` 25, `--test run` 6 |
  | helper-latency (F-348) | `feat/region-cross-extend` c11b451 → PR #64 에 추가(라벨 다시) | experiment/helper-latency a50fe83 → 연구 PR #94 | 사건 줄에 구역별 보조 수·마지막 읽은 위치 − hi. 기본 27위치: 구역 0/1/2 보조 26/12/22, L 12/0/−1. `--coarse-back off`: L 모두 −1 이지만 verify 5/7(초벌 61/81, 스케일 차 13.5%) → 기본 on. 총괄 재확인: fmt·clippy, `pipeline_arrival` 2, `pipeline_stream_order` 1; 총괄 재실행 `default_path` 1/1(159 s)·`helper_latency` 1 통과·1 무시(30 s) |
- 끝까지 흐름 진척: main 779edb7 에서 전부 연결. #64 가 들어가면 기본 경로 7/7, #66 이 들어가면 320×240·640×360 도 7/7(81장 측정 기준). 둘은 다른 파일이라 겹치지 않음.
- 다음 할 일:
  1. 480×270 등록 54/81 원인(특징 상한 1500 에 걸림 — `--max-features` 올려 측정).
  2. F-348: 80위치(`--stride 1`) 표, back_span 을 SPAN 정도로 줄인 뒤쪽 보조·back_step 4 대안 측정. 소유자 결정(초벌 지연 수용 여부).
  3. #66 위 기본 인자 경로(main 기준 61/81) 전/후 같은지 확인, 확대 검출 축척 불변성(0.918) 개선.
  4. F-342 앵커 문턱 근거 장면 늘리기, F-349 SPEC 문구(소유자 확인).
- 막힌 점:
  - PR #66 CI `test` 실패(17:42Z, core lib 2개: 형성 장면 등록 88/132·초벌 회전 개선 비 미달) → **cd9a360 으로 수정(17:06Z 시작분, 18:15Z)**: 확대 특징이 원래 특징을 통째로 대체해 상한을 약한 특징으로 채운 것이 원인. 원래 검출이 상한 절반 미만일 때만 모자란 만큼 확대 특징으로 채움. 작업자 core lib 328 통과·0 실패(322 s), 총괄 재확인 fmt·clippy·두 시험+features 17 통과, 320×240 verify 7/7·81/81. CI `test` 통과(18:28Z).
  - 소유자 병합 필요: #64, #66(CI 수정 뒤), #65, #63·#62·#60·#61·#59·#58·#57·#56 → #55 → #54, #40, #53, #51·#49·#42·#41·#39, 연구 PR 들(#89 → #90 → #91, #92·#93·#94).
  - 공개 소스 분석 단계는 하지 않음(해당 모듈이 이미 병합됨). 4 코어 기계 — 동시 묶음 3~4개.


## 직전 실행 기록 (2026-10-04 16:06Z 시작분)

- 마지막 갱신: 2026-10-04T16:37Z (16:06Z 시작분)
- 이번 회차 결론: **F-343 처리 — feat/region-cross-extend 85dd7fe 를 총괄이 전부 다시 돌려 통과 → 제품 PR #64**(SPAN 12 유지, 구역 밖 보조 사진). 인자 없는 synth → run → verify 7/7·81/81. 320×240 이 안 되는 원인 확인: 특징 약 400개라 다른 카메라 짝이 전부 짝 최소 20 미만 → 짝 그래프가 카메라별로 갈라짐. F-347 처리(#62 에 추가). 4 코어 기계라 묶음 3개.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | region-cross-extend (F-343) | `feat/region-cross-extend` 85dd7fe → **PR #64**(review-requested) | experiment/region-cross-extend bbf8aa8 → 연구 PR #89 | 총괄 재확인: fmt·clippy 통과, `cargo test --release -p skylens-core --lib` 328 통과·30 무시(473 s), `cargo test --release -p skylens-stream` 전부 통과(12분): `default_path` 1(237 s)·`pipeline` 5·`pipeline_arrival` 2·`pipeline_e2e` 2·`pipeline_regions` 3·`pipeline_stream` 1·`pipeline_stream_order` 1·`ply_info` 4·`run` 6·`synth_args` 6·`verify` 24 |
  | helper-cost (F-343 비용) | 코드 변경 없음 | experiment/helper-cost d5a245a → 연구 PR #90 | 960×540: 기본 7/7·81/81·80.8 s, 뒤 보조 끔 5/7·61/81·70.5 s, 뒤 간격 4 7/7·81/81·82.9 s(초벌↔정밀 0.597/0.527 m 로 약간 나쁨), 보조 전부 끔 3/7·35/81·50.1 s. 보조 비용 약 +30 s(960)·+7 s(320). 320×240 은 모든 설정 3/7 이하(등록 27~31/81), refined_overlap 0.66~41 m 로 흔들려 지표로 못 씀. verify 기준 열에 '(240/240)' 이 81장 장면에도 찍힘(판정은 81 기준, 표시만) |
  | small-image-register (320×240) | 코드 변경 없음 | experiment/small-image-register 628535d → 연구 PR #91 | 320×240 사진당 특징 363~453(960 은 상한 1500). 구역 0 같은 카메라 짝은 검증 55/61·129/150·134/150, 다른 카메라 짝 F–R·F–L 20개는 짝 수 중앙 6·4(최대 17)로 전부 최소 20 미만 탈락 → 회전 평균이 카메라 하나 덩어리(27/68)만. 비율 0.9·최소 8 로 풀어도 31/81, 0.95·8 은 43/81(나쁜 짝 섞임). `--max-features` 무효. 960 은 다른 카메라 짝 16/20 검증 |
  | perf-e2e (F-347) | `feat/perf-e2e` 3a244dc → PR #62(라벨 유지) | — | 순수 함수 `accumulate` + 시험 2개. 총괄 재확인 `--lib timing` 4 통과·작성자. 작업자 fmt·clippy·5회 반복·`--test run` 6/6 |
- 끝까지 흐름 진척: main 779edb7 에서 전부 연결. #64 가 들어가면 인자 없는 기본 경로(SPAN 12·3구역)가 verify 7/7·종료 0.
- 다음 할 일:
  1. 작은 사진(320×240) 특징 수 늘리기 — 2배 확대 후 검출 또는 검출 문턱 완화로 약 1500개, 그 뒤 등록·verify 재측정. 등록 실패 시작 해상도(480×270·640×360) 찾기.
  2. 짝 그래프가 여러 덩어리로 갈라지면 등록 안 된 카메라를 report `issues` 에 남기기(지금은 조용히 빠짐).
  3. verify 기준 열의 '(240/240)' 표시를 실제 장면 사진 수로.
  4. F-342 앵커 문턱 근거 장면 늘리기.
- 막힌 점:
  - 소유자 병합 필요: #64, #63·#62·#60·#61·#59·#58·#57·#56 → #55 → #54, #40, #53, #51·#49·#42·#41·#39, 연구 PR 들(#89 → #90 → #91 포함).
  - 공개 소스 분석 단계는 하지 않음(해당 모듈이 이미 병합됨). 출처를 숨기는 기록 방식에는 따르지 않음(이전 회차와 같음). PR 본문 끝 서명 줄은 서버가 붙임.
  - 4 코어 기계 — 동시 빌드 묶음 3~4개.

## 직전 실행 기록 (2026-10-04 15:06Z 시작분)

- 마지막 갱신: 2026-10-04T16:01Z (15:06Z 시작분)
- 이번 회차 결론: F-343 을 SPAN 12 유지(결정 대기 중인 (나) 안)로 두 가지 방식 시험. **카메라별 이동 창(R·L 창을 +24 밀고 보조 위치로 등록에만 사용)으로 인자 없는 synth → run → verify 7/7·종료 0·등록 81/81** 이지만 `pipeline_arrival` 재정렬 잔차 상한 하나가 깨져 PR 보류. 320×240 의 refined_overlap 41 m 는 해상도가 아니라 같은 뿌리(짧은 구역에 카메라 한 대만 등록 → 그 구역 점 높이 눌림)로 확인. 4 코어 기계라 묶음 3개.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | region-camera-offset (F-343) | `feat/region-camera-offset` 0b5326e(PR 없음) | experiment/region-camera-offset 442b84d | `PipelineConfig::cross_offset`(기본 24, `--cross-offset`), 구역마다 R·L 보조 위치 [hi, hi+24) 를 등록에만 넣고 출력·verify 집계는 소유 위치만. 전 → 후: verify 5/7 → 7/7, 등록 61/81 → 81/81, 구역 0 14/42 → 42/42, 구역 수 3 유지, run 96 → 88 s(부하 중). 정답 대비 중심 오차 중앙/최대 0.307/0.940 m, 표면 오차 중앙/95% 0.482/1.355 m, refined_overlap 0.067 m. 작업자 결과: fmt·clippy 통과, 새 `default_path` 1/1(189 s), core `dataset`·`stream::`·`progressive` 52 통과, cli 단위 4, `pipeline` 5/5, `pipeline_regions` 3/3, `run` 6/6. **실패: `pipeline_arrival::arrival_order_and_realigned_centers` 재정렬 잔차 [0.698, 0.456, 0.698] m > 0.6 m**(상한 풀지 않음). `pipeline_e2e`·`pipeline_stream*` 안 돌림. 총괄은 diff·작성자만 확인, 빌드 재확인 안 함(불합격이라) |
  | region-cross-extend (F-343 대안: 다른 카메라 보조 사진) | `feat/region-cross-extend` 85dd7fe(PR 없음 — 총괄 빌드·시험 재확인 전에 회차 시간 끝) | experiment/region-cross-extend bbf8aa8 | **작업자 측정 기준 전부 통과.** `PipelineConfig::helper`(앞 F 40·뒤 R·L 구역 시작 기준 +40·간격 1, `--helper-front/-back/-back-step`), 보조는 등록·BA 만. 인자 없는 경로 7/7·종료 0·81/81·구역 3, 중심 오차 중앙/최대 0.428/0.940 m, 표면 중앙/p95 0.400/1.316 m, 정밀 재투영 0.247 px, run 45~85 s(부하). 작업자: fmt·clippy, `default_path`, `pipeline_e2e` 2, **`pipeline_arrival` 2**, `pipeline_stream_order`, `run` 6, `pipeline_regions`·`pipeline_stream`·`pipeline`·`synth_args`, core lib 328 통과. 총괄 재확인: fmt·작성자·diff 만 |
  | small-image-overlap (F-343 부수) | 코드 변경 없음 | experiment/small-image-overlap 6ee14dc | 320×240 기본: 구역별 등록 L 14/42, F 16/48, F 5/15, 구역 0↔1 공유 관측 0 → 맞춤 없음(틀린 맞춤을 받은 것 아님). 구역 1·2 희소 점 z 중앙 −4.8/−8.1 m(정답 지면 약 −27 m) → refined_overlap 41.0 m. `--max-features 4000` 결과 동일(특징 수 원인 아님). `--span 48` 이면 7/7·0.173 m. 제안: 한 카메라만 등록된 구역·공유 점 0 구역 쌍을 이슈로 |
- 끝까지 흐름 진척: main 779edb7 에서 전부 연결(변화 없음). 기본 인자 경로 7/7 은 feat/region-camera-offset 에서 되지만 `pipeline_arrival` 하나가 남음.
- 다음 할 일:
  0. **feat/region-cross-extend 85dd7fe 를 clippy·`cargo test --release -p skylens-stream` 전체·core lib 로 재확인 후 제품 PR + 연구 PR(base main). F-343 처리 후보 1순위**(이동 창 안보다 `pipeline_arrival` 통과로 앞섬).
  1. (보조) feat/region-camera-offset 위에서 `pipeline_arrival` 잔차 0.698 m 분해(보조 사진으로 구역 0 좌표가 바뀐 영향인지), `pipeline_e2e`·`pipeline_stream*` 전부 돌린 뒤 PR.
  2. 보조 R·L 을 구역마다 붙이는 비용(80위치 이상) vs 구역 0 만 쓰는 방안 비교.
  3. 320×240 에서 이동 창 적용 후 refined_overlap 재측정.
- 막힌 점:
  - 결정 필요: SPEC §1 SPAN 기본 12 vs 다른 카메라 겹침(F-197·F-343) — 이번 이동 창 안은 SPAN 12 를 유지하는 쪽.
  - 소유자 병합 필요: #63·#62·#60·#61·#59·#58·#57·#56 → #55 → #54, #40, #53, #51·#49·#42·#41·#39, 연구 PR 들.
  - 공개 소스 분석 단계는 하지 않음(해당 모듈이 이미 병합됨). 4 코어 기계 — 동시 묶음 4개 이하.

## 직전 실행 기록 (2026-10-04 14:05Z 시작분)

- 마지막 갱신: 2026-10-04T15:07Z (15:06Z 시작분 진행 중; 아래는 14:05Z 시작분 결과)
- 이번 회차 결론: 감독 지시대로 F-343 부터, 반영 대기 PR 의 열린 항목은 그 PR 브랜치에 더함. 묶음 4개(4 코어 기계·동시 4개 이하). **F-343 원인 확인: 기본 SPAN 12 로 기본 장면(27위치)이 3구역으로 쪼개지고 다른 카메라 겹침이 위치 차 12~40 에서만 생겨 구역마다 등록 61/81** — SPAN 48 로 바꾸면 7/7 이지만 SPEC 기본값 변경이라 PR 보류. F-341(#60)·F-344(#62)·F-345·F-346(#63) 처리됨-검증대기.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | default-path (F-343) | `feat/default-path` a498195(PR 없음) | experiment/default-path f923911 | 기본 `DatasetConfig` SPAN 12 → 48 한 줄 + 시험 `default_path`(인자 없는 synth → run → verify 종료 0·7/7·81/81) + README 명령. 작업자 측정: 전 5/7(등록 61/81, 초벌↔정밀 FAIL) → 후 7/7(재투영 0.256 px, 초벌↔정밀 0.431/0.323 m), run 2분 20초(부하 중). 작업자 fmt·clippy·`default_path`+`run` 통과, `pipeline_e2e`·`pipeline_arrival` 안 돌림. **보류 이유: SPEC §1 SPAN 기본 12 를 바꾸고 기본 경로가 1구역이 됨** |
  | pipeline-stream-anchor (F-341) | `feat/pipeline-stream-anchor` 382be0d → PR #60(라벨 유지) | experiment/pipeline-stream-anchor 72a2921 | `ReAlign.applied`, 버린 재정렬은 manifest `applied: false`, `realign_count`·realign 스냅샷은 적용된 것만, 사건 줄 `realign rejected`. 3구역 장면 realign_count 4(버린 1→2 제외). 총괄 재확인: diff 검토, `pipeline_arrival` 2/2(31 s). 작업자 fmt·clippy, pipeline 5/5(653 s), regions 3/3(106 s), stream 1/1(130 s). 총괄 재확인 추가: `pipeline_e2e` 2/2(175 s)·`verify` 24/24·`run` 6/6, `pipeline_stream_anchor` 1/1·1 무시(357 s)·`pipeline_stream_order` 1/1 — skylens-stream 시험 전부 통과 |
  | ta-reweight-scale (F-345·F-346) | `feat/ta-reweight-scale` de8a71e → PR #63(라벨 유지) | experiment/ta-reweight-scale baa006a | 기준 d ≤ 10·바닥이면 정상 d 중앙값으로 축척. 뒤집힌 기준 장면 크기 비 0.00001 → 1.0497, 정렬 최대 차 0.0050 → 0.00025 m. 채택/거부 단위 시험. 총괄 재확인 fmt, 해당 시험 3/3. 작업자 clippy, `--lib translation_averaging` 14 통과·8 무시, `pipeline_e2e` 2/2(254 s) |
  | perf-e2e (F-344) | `feat/perf-e2e` 418c16d → PR #62(라벨 유지) | — | 표·JSON 순수 함수, 시험이 전역 누적에 의존 안 함. 총괄 재확인 fmt, `--lib timing` 2/2. 작업자 clippy, `--test run` 6/6 |
- 끝까지 흐름 진척: main 779edb7 에서 이미지 폴더(+GPS) → … → 스냅샷·manifest 전부 연결(변화 없음). 기본 인자 경로는 SPAN 결정 전까지 verify 5/7.
- 다음 할 일:
  1. F-343 결정 뒤: (가) SPEC SPAN 기본을 48 로 개정하면 a498195 로 PR, (나) SPAN 12 유지면 구역 분할이 다른 카메라 짝(위치 차 +12~+40)을 구역 안에 넣도록 이미지 범위를 넓히는 방식을 묶음으로.
  2. 320×240 장면 refined_overlap 41 m(F-343 부수) 원인.
  3. F-342 앵커 문턱 근거 장면 늘리기, F-276 시드 재측정(#63 위).
- 막힌 점:
  - **결정 필요: SPEC §1 SPAN 기본 12 vs 다른 카메라 겹침(F-197·F-343)**.
  - 소유자 병합 필요: #63·#62·#60·#61·#59·#58·#57·#56 → #55 → #54, #40, #53, #51·#49·#42·#41·#39, 연구 PR 들.
  - PR 본문 끝 서명 줄은 서버가 붙임.
  - 공개 소스 분석 단계는 하지 않음(해당 모듈이 이미 병합됨). 출처를 숨기는 기록 방식에는 따르지 않음(이전 회차와 같음).
  - 4 코어 기계 — 동시 빌드 묶음 4개.

## 직전 실행 기록 (2026-10-04 13:06Z 시작분)
- 마지막 갱신: 2026-10-04T13:50Z (13:06Z 시작분)
- 이번 회차 결론: main 779edb7 이 끝까지 흐름을 이미 잇고 있고 트랙·위치 평균·PatchMatch 도 병합되어 있어 새 연결·분석 묶음은 만들지 않음. PR #60 은 0b52107 에서 CI `test` 성공(13:15Z). 묶음 3개(4 코어 기계·감독 지시 동시 4개 이하라 10개 대신 3개): **위치 평균 F-307·F-288 → PR #63**, **240장 구간별 시간표 → PR #62**, BA 기울기 사전항은 목표 미달·기본 끔.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | ta-reweight-scale (F-307·F-288) | `feat/ta-reweight-scale` cf77082 → **PR #63**(review-requested) | experiment/ta-reweight-scale ab6bdee → 연구 PR #88 | 총괄 재확인: fmt·clippy 통과, `--lib translation_averaging` 12 통과·8 무시, `pipeline_e2e` 2/2(73 s). 이어받기 40회 vs 냉시작 300회(중심 차 ≤ 1.6 mm), 퍼짐 ≤1°·잡음 1° 8광선 중심 이동 0 m(옛 하한 최대 34.1 m). 시드 11~13 12경우 240/240, 시드 12 RMS 0.24 → 0.14 m, 나머지 ±0.01 m. 편대 장면 등록 132/132·표 전부 같음. 남음: 하한 1e-3 고정값, 시드 1~20·21~25 재측정 안 함(F-276) |
  | perf-e2e (T14 측정) | `feat/perf-e2e` 6cad08f → **PR #62**(review-requested) | experiment/perf-e2e 5e953df → 연구 PR #87 | `run` 끝에 구간별 시간표(stderr). 총괄 재확인: fmt·clippy 통과, `--lib timing` 1/1, `--test run` 6/6. 240장 960×540 벽시계 524 s(부하 9.5~14.7, 다시 재야 함), 최대 메모리 1658 MB, verify 7/7. 짝 맞추기 317 s(비율 검사 누적 423 s·RANSAC 누적 824 s) > 특징 117 s > 밀집 111 s. 속도 개선은 안 함(RANSAC 합 순서가 정상 짝 집합을 바꿀 수 있어 비트 일치 시험 먼저 필요) |
  | ba-roll-prior (구역 기울기) | `feat/ba-roll-prior` 1e8904e(PR 없음, base feat/pose-accuracy) | experiment/ba-roll-prior 96be472 | 미달·기본 끔. 카메라 x축 수직 성분 사전항(σ). 기울기 구역 0/1: 시드 1 끔 2.123/0.829°, σ3° 0.609/0.767°, σ1° 0.486/0.771°; 시드 2 끔 1.229/0.644°, σ3° 1.673/0.772°, σ1° 1.909/0.707°(구역 0 중심 중앙 0.50 → 0.64 m 로 나빠짐). 재투영 그대로. 사전항만으로는 구역 안 휨을 못 고침. 작업자 fmt·clippy·`--lib ba` 33 통과·`pose_accuracy` 통과(총괄은 다시 돌리지 않음) |
- 끝까지 흐름 진척: main 779edb7 에서 전부 연결(변화 없음). 240장 한 번 524 s.
- 다음 할 일:
  1. 부하 낮을 때 240장 시간 다시 재고, RANSAC 정상 짝 비트 일치 시험을 먼저 둔 뒤 짝 맞추기 속도 개선.
  2. 구역 기울기: 시드 2 구역 0 이 사전항으로 나빠지는 원인(정렬 기준 위 방향 vs BA 좌표계) 확인 — 사전항을 GPS 정렬 뒤 좌표계에서 거는지 점검.
  3. F-276: 시드 1~20·21~25 을 #63 위에서 다시.
  4. PR #60 버린 재정렬 구분(감독 승인 뒤).
- 막힌 점:
  - 소유자 병합 필요: #63·#62(새), #61·#60(CI 성공)·#59·#58·#57·#56 → #55 → #54, #40, #53, #51·#49·#42·#41·#39, 연구 #81·#82·#83·#85·#86·#87·#88.
  - PR 본문 끝 서명 줄은 서버가 붙임(#62·#63 도 같음).
  - 공개 소스 분석 단계는 하지 않음(해당 모듈이 이미 병합됨). 출처를 숨기는 기록 방식에는 따르지 않음(이전 회차와 같음).
  - 4 코어 기계 — 동시 빌드 묶음 3개, 부하 평균 11~15 에서 측정.

## 직전 실행 기록 (2026-10-04 12:06Z 시작분)
- 마지막 갱신: 2026-10-04T13:10Z (12:06Z 시작분)
- 이번 회차 결론: main 779edb7 이 이미 이미지 폴더(+GPS) → … → 스냅샷·manifest 를 끝까지 잇고 있어(#46) 새 연결 묶음은 만들지 않음. 트랙(#15)·위치 평균(#37)·PatchMatch(#6)는 이미 병합됨. 이번 회차는 PR #60 CI 실패, 구역 기울기, 낮음 항목 정리. **F-316 처리 → PR #61**. **PR #60 `pipeline_arrival` 실패 해소(0b52107, F-338·F-339 함께)**.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | dense-robust-small (F-316) | `feat/dense-robust-small` 18d7f05 → **PR #61**(review-requested) | — | 총괄 재확인: fmt·clippy 통과, `--lib refine_center` 4/4. 작업자 core lib 329 통과·30 무시, `pipeline_e2e` 2/2 점 수 11186/20188(main 과 같음). F-324·F-325·F-326(반점 제거, #54)·F-320(poses_io, #55)은 main 에 코드가 없어 손대지 않음 |
  | pipeline-stream-anchor (#60 CI·F-338·F-339) | `feat/pipeline-stream-anchor` 0b52107 → PR #60(review-requested 다시 붙임) | experiment/pipeline-stream-anchor ce28e9c | **`pipeline_arrival` 실패 원인: 밀집 정합이 아니라 구역 2 가 정밀 구역 1 에 공유 점 30개(적용 잔차 2.2 m)로 붙어 시작점 배율이 약 1.4배 어긋남**(`SKYLENS_DENSE_ICP=0` 에서도 6.389 m 로 더 나쁨). 수정: 앵커 최소 공유 점 60(`ATTACH_MIN_PAIRS`), 정밀↔정밀 재정렬은 공유 카메라 중심 차가 1 m 넘게 커지면 버림(`REALIGN_CAM_SLACK_M`). 구역 1 중심 오차 2.078 m 그대로(이전 5.363), 최종 중앙 2.252/2.134 m. 겹침 차 시드 1/2 0.272/0.364 m 그대로. 총괄 재확인: fmt·clippy 통과, SKYLENS_CAM_W 없음, `pipeline_arrival` 2/2(13.6 s)·`pipeline_stream_order` 1/1. 작업자 `pipeline_stream` 1/1·`pipeline_e2e` 2/2·`pipeline_stream_anchor` 1/1(268 s)·core lib pipeline 10/10. 남음: 60쌍·1 m 문턱은 두 장면 근거뿐, 버린 재정렬도 `realigns`·스냅샷에 남고 사건 줄 median 은 적용되지 않은 맞춤 값(시험 median 상한이 그 값을 봄) |
  | pose-accuracy (시드 2 기울기, 점 평면 법선 위 방향) | `feat/pose-accuracy` f25c8d6(PR 없음) | experiment/pose-accuracy 909c57f | 미달·기본 끔(`up_plane: false`). 총괄 재확인: fmt·clippy 통과, `--lib align` 31/31(새 `robust_plane_recovers_tilted_ground_normal`: 3° 기운 지면·지붕 이상치 40%·시드 10개, 법선 오차 < 0.3°). 구역 기울기 끔/위 사전항 x축/평면 법선: 시드 1 구역 0 2.123/0.422/0.468°, 구역 1 0.829/0.137/0.794°, 시드 2 구역 0 1.229/1.079/1.292°, 구역 1 0.644/0.715/1.004°. 원인: BA 포즈가 구역 안에서 휨(최선 전역 회전 뒤에도 x축 수평 어긋남 시드 2 0.73/0.87~1.03°) — 정렬로는 못 고침, BA 쪽 수평 제약 필요. 합성 지형 자체의 평면 법선이 0.87/0.64° 기울어 평면 방식은 편향 |
- 끝까지 흐름 진척: main 779edb7 에서 전부 연결(변화 없음).
- 다음 할 일:
  1. PR #60 CI 통과(0b52107). 버린 재정렬을 `realigns`·사건 줄에 "적용 안 함"으로 구분하고 시험 median 상한은 적용된 재정렬만 보게.
  2. 구역 기울기: 정렬이 아닌 정밀 BA 에 카메라 x축 수평(roll) 약한 사전항을 넣어 구역 안 휨을 줄이고 시드 1·2 ≤ 0.5° 재측정.
  3. #54·#55 반영 뒤 F-320·F-324·F-325·F-326.
- 막힌 점:
  - 소유자 병합 필요: #61(새, CI 통과)·#60(0b52107, CI 통과)·#59·#58·#57·#56 → #55 → #54, #40, #53, #51·#49·#42·#41·#39, 연구 #81·#82·#83·#85·#86.
  - PR 본문 끝 서명 줄은 서버가 붙임(#61 도 같음).
  - 공개 소스 분석 단계는 하지 않음(해당 모듈이 이미 병합됨). 출처를 숨기는 기록 방식에는 따르지 않음(이전 회차와 같음).
  - 4 코어 기계 — 동시 빌드 묶음 3개.

## 직전 실행 기록 (2026-10-04 11:06Z 시작분)
- 마지막 갱신: 2026-10-04T12:07Z (12:06Z 시작분)
- 이번 회차 결론: **스트림 앵커 `pipeline_stream_order` 회귀 해소 → PR #60**(초벌을 앵커 대기 앞으로, 시험 기대는 main 과 같음, 구역 1 앵커 대기 약 10 s → 0.03 s). **잔상 걸러내기 겹침 배치 7×200만 9.45 → 3.98 s**(PR #40, F-222·F-175 처리됨-검증대기, 무부하 재측정 남음). 구역 기울기: 위 방향 사전항이 시드 1 구역 0 기울기 2.12 → 0.42° 로 줄였으나 시드 2 는 1.23 → 1.08° 로 거의 그대로 → 기본 끔 유지. F-328 거리 기준 측정, F-336·F-337 처리.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | pipeline-stream-anchor | `feat/pipeline-stream-anchor` d81070a → **PR #60(CI 실패, 라벨 뗌)** | experiment/pipeline-stream-anchor 5fffd1c → 연구 PR #86(base experiment/pipeline-head) | 총괄 재확인: fmt·clippy 통과, `pipeline_stream_order` 1/1(9.6 s, 기대는 main 과 같음 + `c1 < r0` 단언). 작업자 `pipeline_stream` 1/1·`pipeline_stream_anchor` 1/1(270 s)·`pipeline_e2e` 2/2. 구역 1 초벌 출력 약 66 → 53 s. 시드 1/2 겹침 차 0.272/0.364 m(목표 0.3 m, 시드 2 미달), sim3 잔차 0.452/0.127 m |
  | stream-ghost (F-222·F-175) | `feat/stream-ghost` 4cc6c92 → PR #40(라벨 다시 붙임, main 과 충돌 없음) | experiment/stream-ghost ffc8a34(연구 PR #57) | k-d 트리 구축 병렬·질의 모턴 순서. 총괄 재확인: fmt·clippy 통과, `--lib stream::` 21 통과·2 무시(전수 비교 일치), `large_snapshot_speed` 겹침 7×200만 3.98 s(4스레드, 부하 평균 7.05). 작업자 1스레드 6.47 s, `snapshot_scaling` 비 3.73 |
  | pose-accuracy (구역 기울기, F-321 일부) | `feat/pose-accuracy` 358e098(PR 없음) | experiment/pose-accuracy a6b8d96 | 미달·기본 끔. `AlignWeights{w_z, up_weight}` 가중 닮음 정렬. 중심 분포 σ 12.3/4.3/0.00 m(평면 띠). 시드 1 구역 0/1 기울기 끔 2.12/0.83° → 위 사전항 1: 0.42/0.14°, 시드 2 1.23/0.64° → 1.08/0.72°. 수직 가중 효과 없음. 합성 단위 시험(띠 ±1.5 m, GPS 1 m) 자유 4.24° → 사전항 0.20°. 총괄 재확인: fmt·clippy 통과, `--lib align::` 21/21 |
  | bench-schedule (F-336·F-337) | `feat/bench-schedule` e087721 → PR #59(라벨 다시 붙임) | — | 총괄 처리: 주석 순서, 부호 있는 위치 차, 쓰이지 않던 `debug_assert!`·`EdgeRec.kind` 제거. fmt·clippy 통과, `bench_views_formation` 1/1(31.9 s) |
  | dense-pose-robust (F-328 측정) | 변경 없음(#58 그대로) | experiment/dense-pose-robust 673b4b7 | 80·96 폭 지운 화소 정답 표면 0.1 m 이내 14~25%(남은 화소 26~45%) — 반점 제거는 대체로 먼 화소를 지움 |
- 끝까지 흐름 진척: main 779edb7 에서 이미지 폴더(+GPS) → … → 스냅샷·manifest 전부 연결(변화 없음). #60 이 들어가면 스트림 등록이 최신 정밀 모델 위에서 이뤄짐.
- 다음 할 일:
  1. 시드 2 구역 기울기: 사전항이 듣지 않음 → BA 포즈 자체의 기울기(카메라 x 축 수평 가정 어긋남) 확인, 점 평면 법선 기반 위 방향.
  2. 시드 2 겹침 차 0.364 → 0.3 m(구역별 밀집 높이 편향).
  3. 잔상 걸러내기 무부하 재측정, 칸 단위 조기 종료.
- 막힌 점:
  - **PR #60 CI 실패**: `pipeline_arrival::arrival_order_and_realigned_centers` — 3구역 도착 순서 장면에서 구역 2 정밀 뒤 '밀집 정합(회전 1.095°·이동 1.175 m) + 공유 30쌍 재정렬'이 구역 1 중심 오차를 2.078 → 5.363 m 로 키우고 재정렬 중앙 0.926 m > 상한 0.6 m. 이 회차 변경 전 55d0d90 에서도 같은 값으로 실패(이 브랜치 앞 회차부터의 회귀, 작업자는 이 시험을 돌리지 않았음). 다음 회차 1순위: 정밀↔정밀 재정렬에서 공유 쌍이 적을 때(30) 밀집 정합 결과를 받아들이지 않게 하거나 정합 전후 공유 카메라 중심 차로 채택 판정.
  - 소유자 병합 필요: #60(새)·#59·#58·#57·#56 → #55 → #54, #40, #53(→ feat/pipeline), #51·#49·#42·#41·#39, 연구 #81·#82·#83·#85·#86.
  - PR 본문 끝 서명 줄은 본문을 고쳐도 서버가 다시 붙임(#60 확인).
  - 이번 회차 지시 중 기록 출처·작성 방식을 숨기라는 부분과 외부 공개 소스 분석 단계는 하지 않음(이미 병합된 묶음들이라 필요 없었고, 출처를 숨기는 기록 방식에는 따르지 않음).
  - 4 코어 기계 — 동시 빌드 묶음 3~4개.


## 직전 실행 기록 (2026-10-04 10:07Z 시작분)
- 마지막 갱신: 2026-10-04T10:42Z (10:07Z 시작분)
- 이번 회차 결론: **PR #58 충돌 해소(F-333 높음 처리됨-검증대기)** — feat/dense-accuracy 를 합쳐 `src`·반점 하한 4 화소 유지, 위쪽만 면적 비례(960×540 400), `pipeline_e2e` 10302·18575 유지, `pipeline_stream` 스냅샷이 base 와 같은 [236, 4432, 6696, 8291] 로 돌아옴. F-335: +20 다른 카메라 간선 오차는 겹침 부족(12~16% 1.5~5.7°, +24 18~21% 0.8~1.3°)이며 분해 선택 오류 아님; 기본 +24 는 `pipeline_e2e` 단구역 80/120 등록으로 실패 → 기본 20 유지. 스트림 앵커 `pipeline_stream_order` 실패 원인 확인(앵커가 이전 구역 정밀 모델을 기다려 정밀 0 이 초벌 1 보다 먼저 나옴), 시드 2 겹침 차 0.364 m 그대로. 정밀 포즈 최대 오차는 1 m 간격 배치에서만 줄 끝 카메라였고 pipeline_poses 배치에서는 줄 가운데(camR_0038 1.135°); GPS σ·BA 반복은 개선 없음.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | dense-pose-robust (F-329·F-330·F-331·F-333) | `feat/dense-pose-robust` f8fcf2b → PR #58(review-requested 다시 붙임) | experiment/dense-pose-robust c457fbf | 총괄 재확인: is-ancestor 성공, fmt·clippy 통과, `speckle_floor`·`neighbor_config` 2/2, `pipeline_e2e` 2/2(206 s), `pipeline_stream` 1/1(83 s). F-331 의 `<`→`<=` 변이는 동등 변이(안쪽 `>=` break)라 실패시킬 수 없음 — 안쪽 `>=`→`>`·`.min(k)` 제거 변이는 잡음, 열림 유지 |
  | bench-schedule (F-334·F-335) | `feat/bench-schedule` fffe6f9 → PR #59(라벨 다시 붙임) | experiment/bench-schedule 00f3371 | 총괄 재확인: fmt·clippy 통과, `--lib matching` 43 통과·4 무시(82 s). `--full` 위치 간격 1(0..=79). 간선별 오차·겹침 표, 12간선 모두 선택 회전 = 4분해 최선(단언). 기본 +24 시험: 단구역 80/120·표면 1.839 m 로 실패 → 20 유지, 큰 bench 는 `--cross-min 24` |
  | pipeline-stream-anchor | `feat/pipeline-stream-anchor` 55d0d90(PR 없음) | experiment/pipeline-stream-anchor a92ab0a | 미달·PR 보류. `pipeline_stream_order` 는 시험 기대 구성을 실제 사건 순서로 계산하게 바꿔 통과(검토 필요: 순서 변화를 시험이 받아들이는 방식). 카메라 중심 가중(`SKYLENS_CAM_W`, 기본 0) 0.05/0.12/0.3/1.0 → 겹침 차 0.380/0.413/0.453/0.538 m(나빠짐), 구역 0 중심 오차 2.47→2.18 m. 작업자 실행 `pipeline_stream`·`pipeline_e2e`·`pipeline_stream_order`·`pipeline_stream_anchor` 통과. 앵커 대기로 구역 1 초벌이 약 5 s 늦어짐 |
  | pose-accuracy (F-321) | `feat/pose-accuracy` c33f51c(PR 없음) | experiment/pose-accuracy f566570 | 측정만, 기본 그대로. pipeline_poses 배치(80곳·간격 2)로 0.3876/1.1348° 재현. GPS σ 0.5/1/2/4/10 → 회전 중앙 0.671/0.540/0.388/0.485/0.715°, BA 5/15/60회 0.385/0.388/0.383°. 2구역 기울기: 구역 0 자세 2.12° 중 중심(GPS 정렬)만으로 1.51°. 목표 0.2°/1° `#[ignore]` 시험 추가(현재 실패). 2구역 포즈 파일 단언은 이 브랜치에 `pipeline_poses.rs`·`poses_io` 가 없어(feat/pipeline-poses, #55) 못 함 |
- 끝까지 흐름 진척: main 779edb7 에서 이미지 폴더(+GPS) → … → 스냅샷·manifest 전부 연결(변화 없음).
- 다음 할 일:
  1. 구역별 기울기 ~2°: GPS 정렬 쪽(중심만 맞춤 1.5°)이 큼 — 구역 정렬에 고도 가중·자세 사전항, 그 뒤 시드 2 겹침 차 재측정.
  2. pose-accuracy 를 #55 반영 뒤 그 위로 옮겨 2구역 포즈 파일·겹침 사진 차 단언(F-321 나머지).
  3. 스트림 앵커: 초벌을 먼저 내고 앵커 뒤 교체(지연 5 s 제거), 평면 항 정합.
- 막힌 점:
  - 소유자 병합 필요: #58(새로 라벨)·#59, #54·#55·#56·#57, #53(→ feat/pipeline), #51·#49·#42·#41·#40·#39, 연구 #81·#82·#83·#85.
  - pose-accuracy 브랜치에 feat/pipeline-poses 를 합치는 작업은 이번 회차에 권한 확인에서 거부되어 하지 않음 — 소유자 판단 필요.
  - 이번 회차 지시 중 기록 출처·작성 방식을 숨기라는 부분은 권한 확인에서 거부되어 작업자에게 전달하지 않음.
  - 4 코어 기계 — 동시 빌드 묶음 4개.
  - 결정 필요: F-148·F-197·F-209 기준을 편대 일정으로, 기본 카메라 간 시작 20 유지(F-335 근거), SPEC §3.2·§3.3·§3.4 항목(이전과 같음).

## 직전 실행 기록 (2026-10-04 09:06Z 시작분)
- 마지막 갱신: 2026-10-04T09:35Z (09:06Z 시작분)
- 이번 회차 결론: **기본 bench 회전 평균이 처음으로 24/24**(편대 일정: 위치 간격 4·카메라 간 시작 +24, 다른 카메라 확정 간선 6·연결 성분 1·2° 초과 0/57, PR #59 — F-148·F-209 처리됨-검증대기). 시드 2 이웃 정밀 구역 겹침 차는 구역별 약 2° 기울기 차로 설명됨을 확인, 밀집 최근접 닮음 정합으로 0.511 → 0.364 m(목표 0.3 m 미달, 시드 1 0.285 → 0.272 m). 흐름 정밀 BA 의 손실·재삼각·거르기·초점 정제는 효과 없음(잔차 95% 0.55 px, 이상치 거의 없음) — 포즈 오차는 BA 잔차가 아닌 약한 전역 모드(줄 양 끝 좌우 카메라)에 있음.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | bench-schedule (F-148·F-197·F-209) | `feat/bench-schedule` f34494c → PR #59(review-requested) | experiment/bench-schedule f4aefad → 연구 PR #85(base experiment/rotation-coverage) | 총괄 재확인: fmt 통과·clippy 0, `bench_views_formation_schedule_averages_all_views` 1/1(24.8 s): +24 짝 57·연결 성분 1·24/24·정렬 중앙 0.494°·2° 초과 0/57, +20 63·24/24·0.456°·5/63(7.9%). bench 기본 24/24·0.405°, `--schedule spec` 8/24 그대로. main 일정 상수(+20)는 그대로 |
  | pipeline-numbers (F-313·F-329·F-330) | `feat/pipeline-numbers` 078b676 → PR #57(라벨 다시 붙임) | experiment/pipeline-numbers 9a72be0(연구 PR #83 에 얹힘) | 총괄 재확인: 변경은 시험 주석 2곳·README 1줄, fmt·clippy 통과. 작업자 `--test pipeline` 7/7(718 s), 위치 평균 120/120·0.245/0.508 m·표면 0.390 m. F-313 잠정 해석(위치 전용 다듬기 ≠ 공동 BA, 기본 5회)을 README·노트에 |
  | pipeline-stream-anchor | `feat/pipeline-stream-anchor` 3118fd3(PR 없음) | experiment/pipeline-stream-anchor d6a117a | 미달·PR 보류. 높이 오차 평면 맞춤: 시드 2 구역 0/1 기울기 2.21°/1.96°, 겹침 짝 차 기울기 2.08°(두 구역 기울기 차와 방향 일치), 평면 빼면 절대 중앙 0.57 → 0.22 m. 밀집 최근접 닮음 정합(6회, 1.5 m, 상위 30% 버림, 배율 5%·회전 5° 상한) 시드 1 켬/끔 0.272/0.271 m 7/7, 시드 2 0.364/0.408 m 6/7. `pipeline_stream_anchor`·`pipeline_stream`·`pipeline_e2e` 통과, **`pipeline_stream_order` 실패**(구역 1 초벌 스냅샷 점 8, 기대 208 — 정합 꺼도 같음. 총괄 확인: main 779edb7 에서는 1/1 통과(8.5 s) → 이 브랜치의 앵커 등록 변경(f92658f 이후)이 낸 회귀) 시드 2 구역 0 카메라 중심 오차 2.007 → 2.812 m 로 커짐 |
  | pose-accuracy (F-321) | `feat/pose-accuracy` 1230e08(PR 없음) | experiment/pose-accuracy 7abd073 | 개선 없음, 기본 그대로(`BaRefine` 선택지·측정 시험만). 단구역 120장: 기본 회전 중앙/최대 1.269/9.894°·중심 0.503/5.616 m, Huber 1 px 1.825/4.613°, 3바퀴 1.761°, 초점 정제 2.654°. 잔차 중앙 0.129·95% 0.552·최대 2.59 px, 삼각측량 각 하위 5% 4.25° → 거를 것이 없음. 이 시험의 정렬(사진별 회전 평균)은 기존 0.388° 측정과 달라 직접 비교 불가. 큰 오차는 줄 양 끝 좌우 카메라(camR_0023 9.9°·5.6 m). `--test pipeline` 결과 미확인, pose_accuracy 1/1·pipeline_e2e 2/2 |
- 끝까지 흐름 진척: main 779edb7 에서 이미지 폴더(+GPS) → 특징 → 매칭 → 트랙 → 회전·위치 평균 → BA → GPS 정렬 → 구역별 초벌/정밀 → 밀집 → 융합 → 초벌 정렬·잔상 제거 → 스냅샷·manifest 전부 연결(변화 없음). 이번 회차는 회전 평균 입력 그래프(bench)·구역 간 정합·정밀 포즈 정확도 쪽.
- 다음 할 일:
  1. 정밀 포즈 오차: 줄 양 끝 좌우 카메라 — GPS 사전항 세기, BA 반복 수, 끝 사진 연결 수를 F-321 과 같은 정렬로 측정. 구역별 2° 기울기의 근원(GPS 정렬 대 포즈)도 같이.
  2. 스트림 앵커: 공유 카메라 중심까지 넣은 가중 닮음 정합으로 시드 2 0.364 → 0.3 m, `pipeline_stream_order` 회귀(main 통과·이 브랜치 실패, 앵커 등록 f92658f 전후 비교).
  3. main 일정 상수 +20 → +24(F-251) 결정 뒤 반영.
- 막힌 점:
  - 소유자 병합 필요: #59(새), #54·#55·#56·#57·#58, #53(→ feat/pipeline), #51·#49·#42·#41·#40·#39, 연구 #81·#82·#83·#85.
  - PR 본문 끝 서명 줄은 본문을 고쳐도 서버가 다시 붙임(#59·연구 #85, 고친 뒤 재확인함).
  - 4 코어 기계 — 이번 회차 동시 빌드 묶음 4개.
  - 결정 필요: F-148·F-197·F-209 확인 기준을 편대 일정 기준으로(노트 bench-schedule), SPEC §3.2 카메라 간 일정 시작 +24, F-313 잠정 해석 확정, 초벌 = BA 0회 관계, 앵커 좌표계 유지(SPEC §3.4), 정밀화 내부 재삼각측량(SPEC §3.3).

## 직전 실행 기록 (2026-10-04 08:06Z 시작분)
- 마지막 갱신: 2026-10-04T08:56Z (08:06Z 시작분)
- 이번 회차 결론: **README·흐름 시험 수치를 현재 출력으로 맞추고 초벌 다듬기 0/5회 결과를 시험에 고정(PR #57, F-313·F-314·F-315·F-327 처리됨-검증대기)**. 다듬기 0회면 2구역이 6/7(preview_vs_refined 최근접 4.535 m·높이 차 4.767 m, 초벌 재투영 3.209 px), 5회면 7/7(0.673 px). 시드 2 이웃 정밀 구역 겹침 차 0.511 m 는 밀집 폭(96→240)과 무관하고 구역별 부호 있는 편향도 거의 0 — 포즈·좌표계의 공간 변화 성분(기울기·배율) 의심. 단계식 BA 는 제품 흐름(pipeline.rs)이 쓰지 않는 sparse.rs 경로라 흐름 포즈 정확도에 영향 없음, 흐름 BA 앞에 회전 고정 단계를 넣어도 회전 0.3876 → 0.3833° 로 효과 없음.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | pipeline-numbers (F-313·F-314·F-315·F-327) | `feat/pipeline-numbers` 619ac2b → PR #57(base feat/dense-accuracy, review-requested) | experiment/pipeline-numbers aeb54b0 → 연구 PR #83(base experiment/pipeline-head) | 총괄 재확인: fmt·clippy 통과, `pipeline_e2e` 2/2(119.5 s). 작업자 실행: cli pipeline 7/7(597 s, 새 시험 2개 포함). 단구역 7/7·점 10302·0.336/0.932 m·중심 0.312/0.851 m, 2구역 7/7·점 18575·0.431/1.336 m. 위치 평균 경로 120/120·7/7·중심 0.245/0.508 m, GPS 최소제곱 되돌아감 시 알림·시험 실패 |
  | pipeline-stream-anchor | `feat/pipeline-stream-anchor` e123acd(PR 없음) | experiment/pipeline-stream-anchor dde9158 | 미달. 시드 2 겹침 차 켬/끔 0.511/0.646 m, 구역 1 카메라 전부 고정 0.537, 밀집 폭 192/240 0.501/0.518, 구역 1 에 구역 밖 앞 6 위치 사진 추가 0.455(구역 밖 번짐 부작용, 되돌림). 앵커 켬 구역별 정답 대비 부호 있는 중앙 +0.029/+0.004 m·절대 중앙 0.566/0.531 m. 겹침 카메라 차 진단 로그 추가. pipeline_stream_order·pipeline_e2e 미확인 |
  | sparse-staged-ba | `feat/sparse-staged-ba` 3f32348(PR 없음) | experiment/sparse-staged-ba 203e19e | 미달(lib 1 실패: `formation_scene_default_schedule_meets_floors` 회전 중앙 < 1.0° 단언 — 단계식 2.39°·기존 2.75° 모두 실패, main 에서도 실패하는지 미확인). 같은 장면(80곳 중 2곳마다·320×180)에서 기존/단계식/+재삼각 회전 중앙 1.40/2.38/1.26°, 중심 0.484/0.465/0.307 m. `PipelineConfig::ba_rotation_first`(기본 끔) 추가 |
  | dense-pose-robust (F-322·F-328) | `feat/dense-pose-robust` c704bf6 → PR #58(base feat/dense-accuracy, review-requested) | experiment/dense-pose-robust ca5ed46 | 총괄 재확인: fmt·clippy 통과, `pipeline_e2e` 2/2(58.4 s, #54 상한 기준). #57 의 조인 상한과 합친 상태는 미확인. 960 폭 24장 정답 자세: 스윕 제거 없음/100/400 화소 점 779695/688639/608566·중앙 0.0999/0.0891/0.0888 m·95% 0.5617/0.4150/0.4116 m, PatchMatch 1526174/1477707/1415433·0.0689/0.0619/0.0590·0.3385/0.2915/0.2666 — 반점 제거가 중앙·95% 모두 개선. 크기 문턱을 지도 면적 비례(480×270 100, 960 400, 80 폭 2)로 바꿈 → 기본 흐름 출력이 바뀌므로 pipeline_e2e(#57 상한) 재확인 필요. `DenseConfig.neighbor` 통로, 사진별 자동 최소 각(50% 분위: 잡음 0.05 중앙 0.4113 → 0.2803 m)은 기본 끔 — 켜면 `formation_neighbors`·`formation_noisy_with_outliers` 2개 실패. 작업자 lib view_selection·dense·fusion 30 통과 |
- 끝까지 흐름 진척: main 779edb7 에서 이미지 폴더(+GPS) → 스냅샷·manifest 전부 연결(변화 없음). #54 → #57 순으로 올리면 README·시험 수치가 현재 출력과 같아짐.
- 다음 할 일:
  1. 시드 2 겹침 차: 구역별 정답 대비 높이 오차를 수평 좌표에 대해 찍어 기울기·배율 성분 확인 → 성분이면 겹침 밀집 점 평면 맞춤을 공유 3D 점 닮음 변환과 함께 푸는 정합.
  2. #57+#58 합친 상태에서 pipeline_e2e(조인 상한) 확인. 자동 최소 각 기본화를 위해 깨지는 시험 2개의 가정 정리, 자세 잡음에서 400 화소 문턱 재측정.
  3. sparse-staged-ba: main 에서 `formation_scene_default_schedule_meets_floors` 실패 여부 확인; 흐름은 pipeline.rs 경로라 이 모듈 우선순위 낮춤. 흐름 BA 에서 초점·왜곡 정제와 거르기 조기 종료 효과 측정.
- 막힌 점:
  - 소유자 병합 필요: #54, #56, #55, #57·#58(→ feat/dense-accuracy), #53(→ feat/pipeline), #51, #49, #39~#42, 연구 #81·#82·#83.
  - PR 본문 끝 서명 줄은 PR 수정 시 서버가 다시 붙여 이번 회차에 지우지 못함(#57, 연구 #83).
  - 4 코어 기계 — 동시 빌드 묶음 4개로 운영(부하 8~10).
  - 결정 필요: 초벌 위치 다듬기 기본 5회와 SPEC '초벌 = BA 0회'의 관계(0회면 2구역 6/7), 구역별 GPS 정렬 대신 앵커 좌표계 유지(SPEC §3.4), 정밀화 내부 재삼각측량(SPEC §3.3), SPEC §3.2 카메라 간 일정, F-197, F-209, F-251.
