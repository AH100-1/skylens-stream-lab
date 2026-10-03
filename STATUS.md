# 현재 상태

- 상태: 쉬는 중
- 마지막 갱신: 2026-10-03T19:50Z (19:06Z 시작분)
- 이번 회차 결론: 지난 회차 교훈대로 동시 묶음을 4개로 줄였으나 부하는 여전히 14~25. 흐름 통합 가지 `feat/pipeline-merge-1906`(= merge-1717 + e2e + arrival, 중심 재정렬 되돌림) 을 만들었고 README 명령 단구역 6/7·2구역 5/7 그대로. 2구역 정렬 점쌍 324 → 1221(창 대신 구역 전체 공유 관측)로 2구역 verify 5/7 → 6/7(region-pairs 가지). F-294 초벌 회전 1.48 → 1.06°(롤 결정 방식 교체)이나 시험의 다른 단언 실패. F-289 처리. 총괄 재확인은 guard2 만 끝남(위치 평균 시험 12 통과) → 이번에도 PR 새로 열지 않음.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | pipeline-merge-1906 | `feat/pipeline-merge-1906` 6b27a59 | experiment/pipeline-merge-1906 0352aa5 | merge-1717 에 e2e(빨리감기)·arrival(fedd1cd) 충돌 없이 합침. 이미 내보낸 정밀 구역 중심의 sim3 적용 되돌림 → 3구역 중심 중앙 3.60 → 2.514 m, 점군 재정렬 5.9455 → 0.0135 m 유지. `pipeline_stream_order` 182 행은 옛 가정(최종 점 ≥ 초벌 점; 정밀 BA 가 이상치를 걸러 18 → 11 점)이라 스냅샷 구성 등식 단언으로 바꿈. 작업자 확인: fmt·clippy, pipeline_e2e 2·pipeline_arrival 2·pipeline_stream_order 1 통과. README 명령 단구역 run 172 s → 6/7(높이 차 2.657 m), 2구역 489 s → 5/7(부하 19~25). 총괄 재확인 못 함 |
  | pipeline-region-pairs2 | `feat/pipeline-region-pairs` a3fc623 | experiment/pipeline-region-pairs2 e8e98fc | 원인 확인: 2구역 정렬 창 [46,50) 12장 = 트랙 324 개가 점쌍 전부. 구역 [lo,hi) 전체 공유 관측으로 바꿈(`SKYLENS_PAIR_SCOPE=window` 로 옛 동작). 점쌍 최소 324 → 1221, 스케일 차 8.26 → 0.05%, 높이 차 6.278 → 5.829 m, 2구역 verify 5/7 → 6/7. 작업자 확인: fmt·clippy, `synthetic_two_region` 통과(415 s). 단구역 재실행·총괄 재확인 못 함. 점쌍 1500 은 구역 트랙 총수(1310·1442)가 상한 |
  | pipeline-f294c (F-294) | `feat/pipeline-f294c` fbe912f | experiment/pipeline-f294c f55900c | 원인: 초벌 Kabsch 특이값 [5385, 18.2, 0.38] — 직선 비행이라 롤 미정, 기존 '평균 광축 z 최소' 기준이 카메라 방위 비대칭(−3°·125°·−116°)으로 약 4° 편향, 2° 격자 0.82° 는 우연. 롤을 광축 높이 분산 최소 닫힌 식으로: placed 회전 중앙 1.48 → 1.06°, 중심 1.12 m 그대로. `preview_default_pose_error_bounds` 는 pipeline.rs:2314 다른 단언에서 실패(비교 가지 상대 비율 단언 의심, 값 미확인) |
  | translation-averaging-guard2 (F-289) | `feat/translation-averaging-guard2` b324b21 | experiment/translation-averaging-guard2 0433a20 | 보충 지지 하한 max(직선 절반 올림, 4). 재현 시험 수정 전 실패 → 후 통과(두 카메라 미등록, 238, RMS 0.161 m). 시드 11~13 12경우 기준 유지(RMS 최대 0.2900 m). F-288 비엄격 채택 조건은 12경우 모두 악화로 뺌. 총괄 재확인: fmt·clippy 통과, core lib `translation_averaging` 12 통과·0 실패·7 무시(237 s) |
- 끝까지 흐름 진척: 이어진 단계 특징 → 매칭 → 트랙 → 회전·위치 평균 → BA → GPS 정렬 → 초벌/정밀 → 밀집 → 융합 → 잔상 제거 → 스냅샷·manifest. 통합 머리 `feat/pipeline-merge-1906`: 단구역 6/7(높이 차 2.657 m), 2구역 5/7. region-pairs 변경을 합치면 2구역 6/7 예상(가지 단독 실측). 남은 공통 실패는 초벌↔정밀 높이 차(초벌 자세 휨). merge-1717 전체 `cargo test --release --workspace` 는 이번에도 40 분 한도 안에 core lib 단계를 넘지 못함(부하 14~17).
- 다음 할 일:
  1. f294c: 2314 행 실패 단언 값 확인 → 상대 비율 단언이면 SPEC 기준 숫자로 대체 여부 결정, 회전 1.06° 를 지면 평면 법선 기준으로 더 낮추기.
  2. `feat/pipeline-merge-1906` 에 region-pairs(a3fc623)·f294c 합치고 단구역·2구역 verify 재측정 → 부하 낮은 회차에 전체 시험 → `feat/pipeline`(#46) 로 올리기.
  3. 초벌↔정밀 높이 차(단구역 2.657 m·2구역 5.829 m) — f294c 롤 개선 뒤 재측정.
  4. guard2 를 #37 머리에 올릴지(소유자 병합 순서) 결정.
- 막힌 점:
  - 4 코어 측정 기계: 묶음 4개 + 전체 시험 1개에도 부하 14~25. 전체 시험은 동시 묶음 없이 돌려야 끝남. 다음 회차는 첫 20 분을 전체 시험 전용으로 두는 게 필요.
  - 결정 필요: 초벌 정렬 닮음 + 보정장 허용 여부(SPEC §3.7), F-197, F-209 확인 기준, F-251.
  - 소유자 병합 필요: #6·#37·#39~#42·#49 (감독 통과 판정 유지분).

## 직전 실행 기록 (2026-10-03 18:06Z 시작분)
- 상태: 쉬는 중
- 마지막 갱신: 2026-10-03T19:07Z (19:06Z 시작분)
- 이번 회차 결론: 끝까지 흐름 통합 가지 `feat/pipeline-merge-1717` 을 부하 낮은 시작 시점에 전체 시험으로 재확인(결과 아래 '흐름 진척'). 합성 정답 대비 끝까지 시험 `pipeline_e2e`(단구역·2구역) 추가. F-294 회귀 지점을 첫 합침 ece7a81 로 좁힘(원인 후보: 비행 축 둘레 회전 선택 휴리스틱). 초벌 높이 차 2.657 → 2.424 m(수평 잔차 보정장) — 기준 2 m 미달. 동시 묶음 6개로 줄였지만 부하 15~22 로 대부분 묶음이 시간 안에 검증을 못 끝냄 → PR 새로 열지 않음.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | pipeline-e2e (F-273) | `feat/pipeline-e2e` 5de63af | experiment/pipeline-e2e b66ea65 | 새 `crates/cli/tests/pipeline_e2e.rs`: README 명령 그대로 synth → run → verify, 출력 폴더 확인, verify 항목별 기대 판정, 정답 대비 상한(실측×1.2). 단구역(stride 2, 120장): 6/7(높이 차 2.657 m), 중심 오차 중앙/최대 0.329/2.912 m, 점→표면 중앙/95% 0.475/1.465 m, 점 11054. 2구역(stride 1, 240장): 5/7(점쌍 324, 높이 차 6.278 m), refined_overlap 0.257 m 통과(실제 판정), 중심 0.290/3.115 m, 점 0.494/3.022 m, 점 19855. 시험 통과·fmt·clippy(작업자). 회전 오차는 poses.txt 에 회전이 없어 못 잼. README '합성 장면으로 한 번 돌려 보기' 절 |
  | pipeline-f294b (F-294) | 변경 없음 | experiment/pipeline-f294b 3f10bd1 | 초벌 가지 끝 2406456: placed 회전 중앙 0.82° → 첫 합침 ece7a81 부터 머리까지 1.48°(중심 1.11~1.12 m 불변). 회전 평균 자체는 1.23 → 0.63° 로 좋아짐 → 삼각측량·위치 평균·짝 일정·초벌 BA 는 원인 아님. 의심: 좌표계 맞춤의 비행 축 둘레 회전 2° 격자 선택(0.1° 격자로 하면 1.73° 로 악화 → 휴리스틱 편향). 다음: 방향 일치 잔차 최소화로 롤 결정 |
  | pipeline-preview-height (F-286 후속) | `feat/pipeline-preview-height` f94b8f9 | experiment/pipeline-preview-height 5f6f690 | 원인은 초벌 자세 휨(초벌 중심 1.12 m·회전 1.48°, 정답 자세 삼각측량은 표면 0.07~0.19 m). 전체 닮음 뒤 잔차를 수평 12×12 칸 가우스 보정장으로: 단구역 높이 차 2.657 → 2.424 m, 최근접 2.358 → 2.110 m, 6/7 그대로. 단위 시험·fmt·clippy 통과, 통합·2구역 미실행. 보정장은 SPEC §3.7 '닮음 변환' 규칙 밖 → 채택 전 결정 필요 |
  | pipeline-arrival (흐름 순서) | `feat/pipeline-arrival` 63d15a9 | experiment/pipeline-arrival 007e32b | 순서 표: 사진 도착은 위치 단위, 등록은 구역 단위(점진 등록 없음), 최신 정밀 위 등록은 꺼짐(`ANCHOR_NEXT_REGION=false`, 켜면 나빠짐). 연쇄 재정렬을 `progressive::chain_realign` 으로 분리, 사건 순서 시험 `pipeline_arrival` 2 통과(재정렬 전 5.95 → 후 0.0135 m). 이미 내보낸 정밀 중심에도 sim3 적용 → GPS 정답 대비 중심 중앙 2.51 → 3.60 m 악화(최신 구역 스케일 1.126) — 이 부분은 되돌릴 후보. 기준 가지에서 `pipeline_stream_order` 가 182 행에서 실패(변경과 무관, 기준에서 재현) |
  | pipeline-region-pairs (2구역 점쌍) | `feat/pipeline-region-pairs` e869179 | experiment/pipeline-region-pairs 52a8050 | region-sim3 합침(충돌 1곳 정리) + 진단 출력(`SKYLENS_PAIR_DEBUG`). 측정 못 함(빌드 6 분, 시험 10 분 초과). 가설: 2구역 정렬 창 [46,50) = 12장뿐 → 점쌍 324 |
  | translation-averaging-guard (F-288·F-289) | `feat/translation-averaging-guard` 1379587 | experiment/translation-averaging-guard 272dda0 | F-288: 고윳값 비 하한 1e-4 → 1e-3, 단위 시험(퍼짐 0.3~1°, 잡음 1°, 정답 밖 출발 → 1 m 미만) 통과. 갱신 채택 조건은 시드 12 점 5% RMS 0.296 → 0.333 m 악화로 뺌. 시드 12 4경우 기준과 같음, 11·13 재측정·전체 시험 미실행. F-289 미착수(되돌림) |
- 끝까지 흐름 진척: `feat/pipeline-merge-1717`(92429d8) 그대로 — synth → run → verify 단구역 6/7·2구역 5/7(pipeline-e2e 실측, 위 표), PLY·스냅샷·manifest 출력. 이번 회차 총괄 재확인: fmt·clippy 통과, 전체 `cargo test --release --workspace` 는 40 분 한도에 걸려 중단(흐름 시험 단계, 출력 버퍼링으로 결과 기록 없음) → `feat/pipeline`(#46) 로 올리지 않음. 알려진 실패: `pipeline_stream_order` 182 행(기준 가지에서 재현), F-294 `preview_default_pose_error_bounds`.
- 다음 할 일:
  1. F-294: 롤(비행 축 둘레 회전)을 방향 일치 잔차 최소화로 정하고 정답 카메라 기울기와 비교 → 초벌 회전 1.2° 이하. 초벌 자세가 좋아지면 높이 차도 같이 줄 것으로 기대.
  2. 2구역 점쌍: `SKYLENS_PAIR_DEBUG=1` 로 창 이미지·관측 맺힘 비율 측정, 트랙 관측 전체로 대응 만들기.
  3. 재정렬 중심 적용(arrival)은 정확도 악화라 되돌리고 점군만 재정렬 유지, 그다음 `feat/pipeline` 에 arrival·e2e 합치기.
  4. 위치 평균: 시드 11·13 확인, F-289 비율 지지 하한.
  5. poses.txt 에 회전 추가(정답 대비 회전 오차 측정용).
- 막힌 점:
  - 4 코어 측정 기계: 묶음 6개 동시에도 부하 15~22, 릴리스 빌드 6 분·흐름 시험 10 분 이상. 묶음당 28 분 안에 측정까지 끝나지 못함. 다음은 묶음 3~4개 이하 + 빌드 폴더 공유가 필요.
  - 결정 필요: 초벌 정렬에 닮음 + 보정장 허용 여부(SPEC §3.7), F-197, F-209 확인 기준, F-251.
  - 소유자 병합 필요: #6·#37·#39~#42·#49 (감독 통과 판정 유지분).

