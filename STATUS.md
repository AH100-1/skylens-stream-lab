# 현재 상태

- 상태: 진행 중
- 마지막 갱신: 2026-10-04T20:59Z (20:58Z 시작분)
- 이번 회차 결론: **480x270 L 카메라 빠짐(54/81·6/7) 원인 두 갈래로 81/81·7/7 확인.** (a) 확대 채움 + 특징 3000 + 카메라 쌍 회전 투표(투표가 run 경로에 연결되지 않았던 점도 고침) → PR #67(선택 옵션, 기본 끔). (b) 카메라 간 짝 일정을 위치 차가 아니라 이동 거리로 환산 → 480·640·960 노트 설정·기본 경로 모두 7/7 이지만 `default_path` 점 표면 오차 중앙 0.523 m(한계 0.5 m)로 실패해 PR 없음. F-354 처리(PR #64 에 추가). 4 코어 기계라 묶음 3개.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | refit-anchor-sim (F-354) | `feat/region-cross-extend` 1d51876 → PR #64(review-requested) | experiment/refit-anchor-sim aeb049b → 연구 PR #99 | 다시 등록한 정밀 경로에서 고정 사진 중심 대응(3쌍 이상, 비퇴화)으로 닮음 변환을 다시 구함, 못 구하면 GPS 맞춤. 총괄 재확인: fmt·clippy 0, `--lib refit_anchor` 2, `helper_latency` 2 통과·1 무시(140 s), `pipeline_stream_order` 1(44 s). 앵커 켬 27위치 이음 잔차는 미측정 |
  | vote-upscale | `feat/vote-upscale` b32db2e → PR #67(20:57Z main 에 병합됨, #66 커밋 포함) | experiment/vote-upscale ce1ac87 → 연구 PR #100 | `--upscale-fill --pair-vote --max-features 3000`: 480x270 54/81·6/7 → 81/81·7/7(투표가 F–L 틀린 해 4개 78~91° 를 뺌), 640x360 81/81·7/7, 960x540 81/81·7/7 유지. 총괄 재확인: fmt·clippy 0, `--lib features` 17, `--lib cross_camera_vote` 1, 480x270 run+verify 7/7·81/81(94 s). 기본값 그대로(끔) |
  | cross-pair-overlap | `feat/cross-pair-overlap` ae117a8(PR 없음, 기준 439bf49) | experiment/cross-pair-overlap bfa6c72 | 정답 겹침 최대 F–L 33 m(33.0 %)·F–R 42 m(39.1 %)·R–L 0 % — 기존 일정(위치 차 20..40)은 1 m 간격 가정이라 stride 3 에서 60 m 이상. `CrossSchedule::scaled` 로 환산하자 480·640·960 노트 설정·기본 경로 모두 81/81·7/7. 그러나 `default_path` 시험이 점 표면 오차 중앙 0.523 m(한계 0.5) 로 실패 — 기준 브랜치에서는 통과, 후퇴. `pipeline_stream_order` 미확인 |
- 끝까지 흐름 진척: main 779edb7 에서 전부 연결(변화 없음). #64 가 들어가면 기본 경로 7/7, #64 + #66 이면 320×240 기본 경로도 7/7, #67 옵션이면 480x270 노트 설정 7/7.
- 다음 할 일:
  1. cross-pair-overlap 의 `default_path` 점 표면 오차 0.523 m 원인(새 일정이 고른 긴 기선 짝의 삼각측량 오차인지) 확인, 한계는 풀지 말 것. 통과하면 PR.
  2. `--pair-vote` 를 기본으로 켤 수 있는지 core lib·skylens-stream 전체로 확인.
  3. F-354 앵커 켬 + `--coarse-back off` 27위치 이음 잔차 측정.
- 막힌 점:
  - 소유자 병합 필요: #64, #66, #65, #63·#62·#60·#61·#59·#58·#57·#56 → #55 → #54, #40, #53, #51·#49·#42·#41·#39, 연구 PR 들(#89 → #90 → #91, #92~#100).
  - F-197(높음)·F-348·F-349 는 SPEC·지연 수용 결정이 소유자 몫.
  - 4 코어 기계 — 동시 묶음 3개.


## 직전 실행 기록 (2026-10-04 19:06Z 시작분)

- 상태: 쉬는 중
- 마지막 갱신: 2026-10-04T19:44Z (19:06Z 시작분)
- 이번 회차 결론: **F-350 확인 끝 — #66(cd9a360, 머리 b97a3ea) core lib 328 통과·skylens-stream 전부 통과로 라벨 다시.** 인자 없는 기본 경로 320×240 은 #64 + #66 이면 7/7·81/81(#64 단독 3/7). F-353 처리분을 #64 에 추가(439bf49, 라벨 다시). 480×270 다른 카메라 짝 회전 투표는 효과 없음(54/81 그대로) — PR 없이 브랜치만. 4 코어 기계라 묶음 3개.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | small-image-followup (F-351·F-352) | `feat/small-image-features` b97a3ea → PR #66(review-requested) | experiment/small-image-followup 43f8639 → 연구 PR #98 | 시험 이름 정리 + 합성 장면 320×240 확대 검출 수 단언(끔 393, 확대 2679). 기본 경로 320×240: #64+#66 7/7·81/81(210 s), #64 단독 3/7·31/81. 총괄 재확인: cd9a360 core lib 328 통과·0 실패·30 무시(745 s), skylens-stream 전부 통과(pipeline 5 565 s·arrival 2·e2e 2·regions 3·stream 1·stream_order 1·ply_info 4·run 6·synth_args 6·verify 24); b97a3ea fmt·clippy·`--lib features` 16 |
  | coarse-back-anchor (F-353) | `feat/region-cross-extend` 439bf49 → PR #64(review-requested) | experiment/coarse-back-anchor 7fc7218 → 연구 PR #97 | 다시 등록 목록에서 앵커 고정 사진을 gid 로 다시 찾음(3장 이상), 못 쓰면 GPS 맞춤. 앵커가 지금 꺼져 있어 off 경로는 이미 GPS 맞춤을 돌았음 — 13.48 % 는 초벌 preview_align 값. off 27위치 5/7(초벌 항목만), 정밀 이음 잔차 0.043·0.030·0.043 m. 총괄 재확인: fmt·clippy, `helper_latency` 2 통과·1 무시(91 s), `pipeline_stream_order` 1. 남은 점: 앵커 켠 다시 등록에서 `an.sim` 이 초벌 좌표계 기준(F-353 이력) |
  | pair-rotation-vote (480×270) | `feat/pair-rotation-vote` 4d28805(PR 없음) | experiment/pair-rotation-vote 1d954d0 | 카메라별 같은 카메라 간선 회전 평균 뒤 다른 카메라 짝의 장착 회전 D 를 5° 무리 투표로 거름. 단위 시험(틀린 해 4 + 맞는 해 3 → 맞는 3만 남음) 통과, sparse 9·rotation_averaging 13·stream_order 1. 그러나 480×270 54/81·6/7, 640×360 54/81, 960×540 81/81 그대로 — 기본 경로에서 F–L 짝이 RANSAC 을 하나도 통과하지 못해 투표할 간선이 없음. 효과 없어 PR 보류 |
- 끝까지 흐름 진척: main 779edb7 에서 전부 연결(변화 없음). #64 가 들어가면 기본 경로 7/7, #64 + #66 이면 320×240 기본 경로도 7/7.
- 다음 할 일:
  1. 480×270·640×360 F–L 짝 대응 부족: 다른 카메라 짝을 겹침이 실제로 있는 위치 차로 고르기(예약 위치 차 20..26 이 맞는지 정답 겹침으로 확인) 또는 확대 검출 조건을 '긴 변 800 미만'으로 넓힌 위에서 회전 투표 재측정.
  2. 앵커 켠 경로의 다시 등록 닮음 변환을 고정 사진 대응으로 다시 구하기(F-353 남은 점).
- 막힌 점:
  - 소유자 병합 필요: #64, #66, #65, #63·#62·#60·#61·#59·#58·#57·#56 → #55 → #54, #40, #53, #51·#49·#42·#41·#39, 연구 PR 들(#89 → #90 → #91, #92·#93·#94·#95·#96·#97·#98).
  - F-197(높음)·F-348·F-349 는 SPEC·지연 수용 결정이 소유자 몫.
  - 4 코어 기계 — 동시 묶음 3개.
