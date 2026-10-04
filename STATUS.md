# 현재 상태

- 상태: 쉬는 중
- 마지막 갱신: 2026-10-04T11:52Z (11:06Z 시작분)
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
