# 현재 상태

- 상태: 쉬는 중
- 마지막 갱신: 2026-10-03T22:40Z (22:06Z 시작분)
- 이번 회차 결론: 첫 시점에 흐름 머리 `feat/pipeline-merge-2106`(e064aab) 전체 시험을 출력 파일로 돌림 — fmt·clippy 통과, core lib **322 통과·1 실패(F-294 `preview_default_pose_error_bounds` 하나)·25 무시**(760 s), dataset_synth 2·perf_structure 6·cli 단위 3 통과, cli `pipeline` 통합 시험은 회차 마감까지 끝나지 않음. 짝 맞춤 RANSAC 최소 반복 300 → 100·후보 평가 조기 중단으로 단구역 run 벽시계 54~60 → 30~38 s(짝 맞춤 40~42 → 17~21 s), verify 7/7 유지(pipeline-ransac, 총괄 재확인 통과). preview-tri·two-region-height 를 합친 merge-2206 은 2구역 높이 차 5.829 → 4.555 m 이나 e2e 두 시험 실패. F-294 는 정밀 BA 시작점을 초벌 롤과 분리해 2구역 정밀 표면 회복, 2구역 초벌 스케일 차 1.38% 로 아직 실패. 머리 전체 시험이 끝나지 않아 #46 갱신·새 PR 없음.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | 머리 전체 시험 (merge-2106) | `feat/pipeline-merge-2106` e064aab | — | fmt·clippy 통과. core lib 322 통과·1 실패(F-294)·25 무시. 통합 dataset_synth·perf_structure·cli 단위 통과. cli `pipeline` 이후 묶음 미완(부하 12~15) |
  | pipeline-ransac | `feat/pipeline-ransac` 77d9489 (stage-time 합침 b8fcf61 포함) | experiment/pipeline-ransac c8e4e31 | 원인: 적응 종료는 있었으나 최소 반복 300 이 바닥(내점 비율 중앙 0.993, 교과서 식으로 3회면 충분). 후보 평가 조기 중단(결과 동일)·최소 100·평면 판정 적응 종료(최소 40·상한 200)·정상 수 절반 미만 조기 포기. 전/후 번갈아 2회(부하 12~20): run 54.4/60.3 → 29.9/38.1 s, 짝 맞춤 40.4/42.4 → 16.9/20.8 s. 7/7·120/120 유지, 정밀 재투영 0.300 → 0.313 px, 높이 차 1.553 → 1.456 m. 총괄 재확인: fmt·clippy 통과, `two_view::`·`matching::` 66 통과·0 실패·7 무시, `pipeline_e2e single_region` 통과(45.9 s) |
  | pipeline-merge-2206 | `feat/pipeline-merge-2206` 845aa3e | experiment/pipeline-merge-2206 d9a9302 | merge-2106 + preview-tri + two-region-height(하한 50). 단구역 7/7·높이 차 1.114 m·최근접 1.109 m, 2구역 6/7·높이 차 4.555 m·최근접 3.515 m·정밀 표면 중앙 0.305 m. `pipeline_e2e` 2 실패: 단구역 정밀 표면 95% 1.896 > 1.758 m, 2구역 점쌍 최소 1170 < 1200·스케일 차 1.66% > 1.0%. 상한 유지 |
  | pipeline-f294e (F-294) | `feat/pipeline-f294e` f8d677a | experiment/pipeline-f294e 009ddaa | 원인: 정밀 BA 시작점이 초벌과 같은 희소 초기화(롤 포함). 정밀 시작점만 예전 롤 규칙으로 → 2구역 정밀 표면 중앙·95% 0.742/3.869 → 0.494/3.022 m. F-294 시험·단구역 e2e 통과, 2구역 e2e 는 초벌 구역 간 스케일 차 1.38%(상한 1.0%) 로 실패. 임시 처방(정밀 BA 가 롤을 스스로 풀게 하는 것이 근본) |
- 끝까지 흐름 진척: synth → run → verify 가 PLY·스냅샷·manifest·timing.json 까지. 머리 `feat/pipeline-merge-2106` 단구역 7/7·2구역 6/7, core 시험 실패는 F-294 하나. 단구역 실행 시간의 73% 이던 짝 맞춤이 절반 아래로(ransac 가지). 2구역 남은 실패는 초벌↔정밀 높이 차(최선 4.555 m, 기준 2 m)와 구역 간 초벌 스케일.
- 다음 할 일:
  0. 부하 없는 첫 시점에 merge-2106 의 cli 통합 시험만(`cargo test --release -p skylens-stream --tests`) 출력 파일로 — 나머지는 이번에 확인됨.
  1. 새 머리 = merge-2106 + ransac + f294e: 2구역 초벌 스케일 차(f294e 1.38%·merge-2206 1.66%, 합치기 전 0.05%)의 원인 — 초벌 롤이 구역 간 초벌 정렬에 닿는 경로.
  2. 정밀 BA 가 시작 롤에 의존하지 않게(반복 수·비행 축 둘레 회전 감쇠 확인) → f294e 임시 처방 제거.
  3. merge-2206 단구역 정밀 표면 95% 1.896 m 원인(정밀이 초벌에 닿는 경로가 또 있는지).
  4. F-292 노트(부하 낮은 `timing_960` 측정)·F-305 문서 정리 — 이번 회차 미착수.
- 막힌 점:
  - 4 코어 측정 기계: 전체 시험 + 묶음 3개로 부하 12~20, cli 통합 시험이 회차 안에 끝나지 않음.
  - 머리 계보의 옛 커밋 3개(63d15a9·634844d·5de63af) 메시지에 정리되지 않은 꼬리 줄이 남아 있음 — 여러 가지가 공유하는 이력이라 이번에 고치지 않음, 소유자 판단 필요.
  - 결정 필요: 초벌 정렬 닮음 + 보정장 허용 여부(SPEC §3.7), F-197, F-209 확인 기준, F-251, preview_align 점쌍 기준을 구역 길이에 비례시킬지.
  - 소유자 병합 필요: #6·#37·#39~#42·#49, #50(#6 위).

## 직전 실행 기록 (2026-10-03 21:05Z 시작분)
- 이번 회차 결론: 흐름 머리 후보 `feat/pipeline-merge-2106`(= merge-2006 + height2 + 시험 폴더 분리) 에서 단구역 synth → run → verify **7/7**(높이 차 1.553 m, 최근접 1.416 m), 2구역 6/7(높이 차 5.829 m 그대로). 머리 시험 실패 2개 중 `preview_ba_option_does_not_change_refined` 는 원인 확인·수정(시험들이 같은 인자일 때 같은 작업 폴더를 병렬로 지우고 덮어씀 — 이름에 시험 이름 추가, 단언 그대로). F-294 는 합치면 시험 통과·단구역 1.538 m 이나 2구역 정밀 점 표면 거리 중앙 0.494 → 0.742 m 로 e2e 절대 상한(0.60 m) 초과 → 별도 가지에만. 단계별 시간: 단구역 99 s 중 짝 맞춤 73%(그중 RANSAC 이 대부분), 밀집 20%. 머리 전체 시험을 돌리지 못해 #46 갱신·새 흐름 PR 은 열지 않음. 패치매치 비용 기반 섭동 폭 옵션만 PR #50(기준 feat/patchmatch, 기본 끔).
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | pipeline-merge-2106 | `feat/pipeline-merge-2106` e064aab | experiment/pipeline-merge-2106 0818638 | merge-2006 + height2 + preview-ba-fix. 단구역 7/7(1.553 m), 2구역 6/7(5.829 m). README 단구역 열·`pipeline_e2e` 단구역 단언을 7/7 로. 작업자 확인: fmt·clippy, height2 상태 `pipeline` end_to_end 3·`pipeline_e2e` 2 통과. preview-ba-fix 합친 뒤 시험 재실행 못 함 |
  | pipeline-merge-2106-f294 (F-294) | `feat/pipeline-merge-2106-f294` df7a0c7 | 같은 노트 | `preview_default_pose_error_bounds` 통과(회전 1.06°·중심 1.12 m), 단구역 7/7·1.538 m, 2구역 6/7·높이 차 6.763 m, `pipeline_e2e` 2구역 표면 거리 단언 실패(중앙 0.742 > 0.60, 95% 3.869 > 3.63). 상한 유지 |
  | pipeline-preview-ba-fix | `feat/pipeline-preview-ba-fix` 90e72c0 | experiment/pipeline-preview-ba-fix 5c3f4e8 | 원인: `run_case_with` 작업 폴더가 프로세스 번호+인자뿐 → 단구역 시험과 같은 인자. 단독 실행 통과(초벌 BA 0/8 정밀 결과 동일, 중심 중앙 0.3284 m). 수정 뒤 병렬 실행에서 해당 3 시험 통과, 2구역 2 시험은 미완 |
  | pipeline-two-region-height | `feat/pipeline-two-region-height` bec30fb | experiment/pipeline-two-region-height f4b2161 | 원인: 초벌 광선 각 거름 하한 '20° 이상 200점 미만이면 거르지 않음' — 2구역 구역 0 은 84/1310 점이라 거름 꺼짐. 하한 50: 2구역 높이 차 6.278 → 3.469 m(이 가지 2구역은 점쌍 324 측정, merge-2006 점쌍 수정 미포함), verify 5/7 그대로, 단구역 7/7 유지. 시험 미실행 |
  | pipeline-preview-tri | `feat/pipeline-preview-tri` 66fce68 | experiment/pipeline-preview-tri 05eddc9 | 초벌 점: 최대 사잇각 쌍 초기값 + 모든 관측 각도 잔차 가우스–뉴턴 4회(카메라 고정). 단구역 높이 차 1.553 → **1.114 m**, 최근접 1.416 → 1.109 m, 7/7. 문턱 10° 1.711 m, 5° 3.581 m(6/7) → 20° 유지. 단위 합성에서는 이득 재현 못 함(나빠지지 않음만 단언). `tri_tests` 7·`run::` 3 통과 |
  | pipeline-stage-time | `feat/pipeline-stage-time` 92b70e4 | experiment/pipeline-stage-time 8ddb533 | 출력 폴더에 timing.json. 단구역 99.4 s(부하 약 20): 짝 맞춤 73.0 s(73%), 초벌 밀집 10.6, 정밀 밀집 9.3, 특징 5.4, 정밀 BA 0.9. 기준 가지의 `pipeline_e2e` 단구역 기대값이 낡아 실패(merge-2106 에서 고쳐짐) |
  | patchmatch-f293b (F-293) | `feat/patchmatch-f293b` 042d9fc, **PR #50** | experiment/patchmatch-f293b f28dcc6, 연구 PR #67 | 192 px 은 한 층뿐이라 건너뜀 0%, 960 px 고운 층 85~92%. 비용 기반 섭동 폭+후보 축소: CPU 2.02 s·98.5%·2.90° (skip 0.08 1.42 s·94.4%·4.01°). 기본 끔. 총괄 재확인: fmt·clippy 통과, core `patchmatch` 13 통과·6 무시, CI 초록 |
  | 알고리즘 정리 4편(밀집·자세·트랙·BA) | — | — | 요점: 초벌은 최대 사잇각 쌍 + 각도 잔차 정제, 좁은 각 점은 깊이 불확실도로 가중. 거의 연직 촬영에선 초점거리와 높이가 함께 움직이므로 초벌도 정밀 BA 의 내부 파라미터를 쓰는 것이 높이 차를 줄이는 싼 방법. 전역 위치는 점–카메라 결합 목적식 + GPS 사전항(수평 σ 1.5 m·수직 3 m). 짝 맞춤 RANSAC 반복 상한·적응 종료 점검. 패치매치는 건너뛰지 말고 비용 기반 섭동 폭 |
- 끝까지 흐름 진척: synth → run → verify 가 PLY·스냅샷·manifest·timing.json 까지. 머리 후보 `feat/pipeline-merge-2106`: 단구역 7/7, 2구역 6/7. 더 붙일 수 있는 것: preview-tri(단구역 1.114 m), two-region-height 하한 50(2구역 높이 차 개선 후보).
- 다음 할 일:
  0. 부하 없는 첫 20 분에 `feat/pipeline-merge-2106` 전체 `cargo test --release --workspace` 를 출력 파일로. F-294 시험만 실패하면 다음 단계 1 뒤 #46 갱신.
  1. merge-2106 에 preview-tri·two-region-height(하한 50) 합쳐 2구역 높이 차 재측정(점쌍 수정과 함께) → 2 m 기준 근접 여부.
  2. F-294: 롤 수정이 2구역 정밀 표면 거리를 나쁘게 하는 원인 분리(정밀 BA 시작점 롤 편향 가설).
  3. 초벌에 정밀 BA 의 내부 파라미터 쓰기(초점–높이 결합) 시험.
  4. 짝 맞춤 RANSAC 반복 분포 측정 → 상한·적응 종료 조정(단구역 시간 73%).
- 막힌 점:
  - 4 코어 측정 기계: 묶음 6개 동시에 부하 13~22, 전체 시험 못 돌림. 다음 회차는 동시 구현 묶음을 3개 이하로.
  - 결정 필요: 초벌 정렬 닮음 + 보정장 허용 여부(SPEC §3.7), F-197, F-209 확인 기준, F-251, preview_align 점쌍 ≥1000 기준을 구역 길이에 비례시킬지.
  - 소유자 병합 필요: #6·#37·#39~#42·#49, #50(#6 위).

## 직전 실행 기록 (2026-10-03 20:06Z 시작분)
- 이번 회차 결론: 첫 시점에 `feat/pipeline-merge-1906` 전체 `cargo test --release --workspace` 를 출력 파일로 돌림 — core lib 320 통과·1 실패(F-294 `preview_default_pose_error_bounds`, 2274 행)·25 무시(1150 s), dataset_synth 2·perf_structure 6·cli 단위 3 통과, cli `pipeline` 시험에서 `preview_ba_option_does_not_change_refined` 실패(사유 기록 전 회차 마감). region-pairs 를 합친 `feat/pipeline-merge-2006` 으로 2구역 verify 5/7 → 6/7. F-294 시험은 f294d 가지에서 통과하나 높이 차가 2.948 m 로 나빠져 흐름 머리에 합치지 않음. 이번에도 PR 새로 열지 않음(머리 전체 시험 미통과).
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | pipeline-merge-2006 | `feat/pipeline-merge-2006` 4659a32 | experiment/pipeline-merge-2006 37831a3 | merge-1906 + region-pairs(merge 커밋). 충돌은 `progressive::chain_realign_with` 로 정리. 2구역 6/7(점쌍 최소 1221, 스케일 차 0.05%, 높이 차 5.829 m, 최근접 4.923 m), 단구역 6/7(높이 차 2.657 m). 중심·점 오차 이전과 같음. 2구역 시험 기대 판정 PASS 로, 정답 대비 상한은 조임(높이 7.6 → 7.0, 최근접 7.2 → 5.9 m). 총괄 재확인: fmt·clippy 통과, 단구역 e2e 통과(60.7 s), 2구역 재실행은 시간 부족 |
  | pipeline-f294d (F-294) | `feat/pipeline-f294d` 74d863a | experiment/pipeline-f294d d00eea7 | 2314 행 실패는 끈 가지 대비 상대 비율 단언(중심 1.118/1.454 = 0.77, 기준 0.6) — 롤 수정이 끈 가지도 개선. 비율 0.85 로, 절대 상한 유지. core pipeline 7 통과. placed 회전 1.06°·중심 1.12 m. 단구역 verify 6/7, 높이 차 2.948 m(악화) |
  | pipeline-height2 | `feat/pipeline-height2` 94c6b50 | experiment/pipeline-height2 5e7cec9 | 원인: 초벌 카메라 높이 오차 중앙 0.15 m 뿐, 초벌 점 표면 오차 중앙 5.01 m — 광선 사잇각 좁은 점의 깊이 편향. 굽음(비행 축 6구간 −0.05~−0.27 m)·측면 경사 모두 작음. 초벌 점군 최소 광선 각 2 → 20°: 단구역 verify 6/7 → **7/7**, 높이 차 2.657 → 1.553 m, 최근접 2.358 → 1.416 m, 정밀 수치 불변. 닮음 정렬 변형은 효과 없음. 미확인: 2구역 시험(단언 2~8 m 충돌 가능), clippy, 단언 수정 뒤 재실행. 총괄 재확인 못 함 |
  | patchmatch-f293 (F-292·F-293) | 변경 없음 | experiment/patchmatch-f292 2232f16, experiment/patchmatch-f293 4256bf5 | F-292: `timing_960` 2.39/1.74/2.14 s(부하 16) — 0.7 s 확인 못 함. F-293: 192 px 흐름에서 skip 0·0.08 결과 동일(건너뜀 작동 미확인), 패치매치 단독 0.08 은 CPU −44%·1% 이내 −4.2%p |
  | 자세 단계·밀집 단계 분석 | — | — | 알고리즘 정리 2편. 요점: 방향 정합만 한 초벌은 비행 축 따라 굽음(2차 곡선 고도 잔차)이 생겨 닮음 변환으로 못 지움 → 위치 평균 목적식에 카메라별 GPS 사전항(수직 σ 크게)이 후보. 롤은 공선 판정(GPS 공분산 λ2) 뒤 연직 광축 닫힌 해/지면 평면으로. 밀집: 건너뜀 대신 좋은 화소는 섭동 폭만 줄이는 방식이 정확도 손실 없이 빠를 가능성 |
- 끝까지 흐름 진척: synth → run → verify 가 PLY·스냅샷·manifest 까지 나옴. 통합 머리 후보 `feat/pipeline-merge-2006`: 단구역 6/7·2구역 6/7. height2(초벌 점 광선 각 20°)로 단구역 7/7 첫 달성(작업자 실측, 원인은 자세 굽음이 아니라 좁은 광선 각 점의 깊이 편향) — 합치면 남는 것은 2구역 높이 차 확인과 머리 시험 실패 2개.
- 다음 할 일:
  0. height2 를 merge-2006 에 합치고 clippy·2구역 e2e 확인(2구역 높이 단언 갱신).
  1. 머리 전체 시험의 두 실패 처리: F-294(f294d 를 merge-2006 에 합치되 높이 차 악화 원인 먼저), `preview_ba_option_does_not_change_refined` 실패 사유 확인.
  2. 초벌 고도 잔차를 비행 축/측면 위치에 대해 분해 → 굽음이면 위치 평균에 GPS 사전항, 롤이면 공선 판정 + 연직 광축 닫힌 해.
  3. 부하 낮은 시점에 F-292 벽시계 재측정, F-293 건너뛴 화소 수 세기.
- 막힌 점:
  - 4 코어 측정 기계: 묶음 4개 + 전체 시험에 부하 13~17. 전체 시험 하나가 25 분.
  - 결정 필요: 초벌 정렬 닮음 + 보정장 허용 여부(SPEC §3.7), F-197, F-209 확인 기준, F-251.
  - 소유자 병합 필요: #6·#37·#39~#42·#49.

## 직전 실행 기록 (2026-10-03 19:06Z 시작분)
- 상태: 진행 중
- 마지막 갱신: 2026-10-03T22:06Z (22:06Z 시작분)
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
