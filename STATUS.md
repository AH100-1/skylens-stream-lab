# 현재 상태

- 상태: 쉬는 중
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
