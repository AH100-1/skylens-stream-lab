# 현재 상태

- 상태: 쉬는 중
- 마지막 갱신: 2026-10-03T18:47Z (18:06Z 시작분)
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

## 직전 실행 기록 (2026-10-03 17:06Z 시작분)
- 상태: 쉬는 중
- 마지막 갱신: 2026-10-03T18:06Z (18:06Z 시작분)
- 이번 회차 결론: 흐름 가지 CI 막힘 4개 중 F-287(희소 BA 회전 20.75 → 0.68°)·F-272(stream-order 합침, `pipeline_regions` 3/3)·F-286/F-295(초벌 BA 를 사본에만, 시험 기본 설정)를 각 가지에서 고쳤고, F-294(초벌 회전 1.485°)는 원인 못 찾음. 네 가지(sparse-ba·so-merge·preview-split·cli)를 `feat/pipeline-merge-1717` 하나로 합침 — fmt·clippy 통과, README 단구역 명령 run 38 s(종료 0) → verify 6/7(등록 120/120, 정밀 0.300 px·초벌 3.026 px, preview_align 점쌍 2746·잔차 3.932 m, preview_vs_refined 높이 차 2.657 m 만 실패, final 11054) — e0438ca 와 같은 수치. 측정 기계 부하 30~45 로 전체 시험은 어느 가지에서도 못 돌림 → PR 새로 열지 않음, 라벨 변경 없음.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | pipeline-sparse-ba (F-287) | `feat/pipeline-sparse-ba` 8cfb182 | experiment/pipeline-sparse-ba fbf21f5 | 원인: 위치 단계 중심 오차 4.36 m 상태로 카메라 0 만 고정한 사전항 없는 BA 가 회전 이탈. GPS 위치 사전항(σ 2 m, Huber 3σ) 넣음. 기본 일정: BA 뒤 회전 20.754 → 0.683°, 중심 2.504/5.089 → 0.266/0.457 m, 점 2.401 → 0.443 m, 132/132, 새 시험 통과. 옛 일정 시험·전체 시험 미확인 |
  | pipeline-so-merge (F-272) | `feat/pipeline-so-merge` 5d3e3c4·b0987a1 | experiment/pipeline-so-merge 74d9c1a | stream-order 합침(충돌 1곳), `pipeline_regions` 3 통과·0 실패(436 s, 5d3e3c4). Kabsch 짝 수·특이값 하한 검사 추가(b0987a1, 재실행 미확인). 합치기 전후 CLI 수치 미측정 |
  | pipeline-preview-split (F-286·F-295·F-297) | `feat/pipeline-preview-split` d1b3590 | experiment/pipeline-preview-split 357a823 | SPEC 초벌(BA 0회) 유지, 초벌 BA 는 사본에만. 기본 설정 CLI: 단구역 6/7(높이 차 2.657 m), 2구역 5/7(점쌍 324, 높이 차 6.278 m, 겹침 0.257 m 통과). README·시험·노트 같은 수치. 바뀐 시험·0 vs 8 비교 시험 결과 미확인 |
  | pipeline-cli (F-304) | `feat/pipeline-cli` aa923aa | experiment/pipeline-cli 916eaa5 | `run` 옵션 --dense-method·--position·--preview-ba-iters·--gps-sigma-h/v·--tri-*, `stand_in::build_tracks` 삭제, 스냅샷 단조 규칙 흐름·verify 공유. fmt·clippy 통과, stream 20·옵션 3 통과. 위치 평균 경로 수치 미측정 |
  | pipeline-f294 (F-294) | 변경 없음 | experiment/pipeline-f294 b5d45a4 | 1.4848° 재현, 옛 짝 일정으로 바꾸면 2.0547° — 짝 일정 가설 기각. 합침 커밋별(ece7a81·75486fc·86361e9·e0438ca) 측정이 다음 |
  | patchmatch (#6, F-292·F-293·F-302) | `feat/patchmatch` 61fa09a | experiment/patchmatch-f292 b6df199 | `timing_960` 이 기본 경로를 잼, 문서 정정, `fast_960` 법선 < 5° 단언(PM_SK=0.08 PM_SP=0 에서 6.26° 실패 확인). 빠른 경로 최대 메모리 +49.5 MB(기준 50 MB, 여유 0.5 MB). 부하 20 이상이라 0.7 s 벽시계 확인 못 함. fmt·clippy 통과, patchmatch 12 통과. 총괄 재확인 못 함 → 라벨 그대로 |
  | pipeline-pm-skip (F-293) | `feat/pipeline-pm-skip` e89414d | experiment/pipeline-pm-skip 4bed84b | 흐름이 패치매치 설정을 넘겨받음. 폭 96 단구역 건너뜀 0·0.08: 융합 12674 점, 점→표면 0.462/2.870 m 로 같음 — 한 층뿐이라 건너뜀이 작동 안 하는 것으로 보임. 192 px 이상 재측정 필요. clippy 미실행 |
  | pipeline-region-sim3 | `feat/pipeline-region-sim3` 21b2496 | experiment/pipeline-region-sim3 b17a5a2 | 구역 간 sim3 를 이미지 단위 양방향 합의(3점 가설 512, 정상 이미지 ≥ 20%)로, 환경 변수로만 켬(기본 끔). 단위 시험 2 통과, fmt·clippy 통과. 2구역 전/후 미측정 |
  | translation-averaging (#37, F-288·F-289·F-301) | `feat/translation-averaging` 46e3412·aaff35f | experiment/translation-averaging-f288 | main 합침(46e3412, lib.rs 충돌 정리), F-301 무시 사유 정정(aaff35f). F-288·F-289 첫 시도(고윳값 비 1e-3, 갱신 채택 조건, 보충 지지 과반)는 기존 시험 2개 실패·시드 11~13 RMS 악화(시드 12 점 5% 0.296 → 0.328 m)로 커밋 안 함 — 다음엔 점 다듬기는 그대로 두고 중심에만, 채택 조건은 비엄격. 총괄 재확인: fmt·clippy 통과, core lib 289 통과·0 실패·23 무시(298 s) → #37 라벨 다시 |
  | translation-averaging-sparse (F-290) | `feat/translation-averaging-sparse` fdfb39f | experiment/translation-averaging-sparse 199bde7 | 점 번호 압축 + 점 블록 슈어 보수 풀이(rayon). 기존 시험 소수 4자리 같음. 24장 5.81 → 0.18 s, 번호 1e6 밀기 7.23 → 1.32 s, 240장·점 1000 67.6 → 37.5 s. 점 2만·8만·메모리 미측정 |
- 끝까지 흐름 진척: `feat/pipeline-merge-1717`(= feat/pipeline + sparse-ba + so-merge + cli + preview-split) 에서 synth → run → verify: README 단구역 명령 run 38 s(종료 0) → verify 6/7(등록 120/120, 정밀 0.300 px·초벌 3.026 px, preview_align 점쌍 2746·잔차 3.932 m, preview_vs_refined 높이 차 2.657 m 만 실패, final 11054) — e0438ca 와 같은 수치. 위치 단위 도착 사건·구역 건너뜀이 흐름 가지에 들어감.
- 다음 할 일:
  1. 부하 없는 상태에서 `feat/pipeline-merge-1717` 전체 `cargo test --release` → 통과하면 `feat/pipeline`(#46) 으로 올리고 라벨.
  2. F-294: 합침 커밋별 초벌 회전 중앙 측정으로 회귀 지점 찾기.
  3. 2구역 초벌 높이 차 6.278 m·점쌍 324: region-sim3 모드 1·2 측정, 초벌 점 깊이 원인.
  4. 위치 평균: 슈어 풀이 점 2만·8만 측정, `global_positioning` 나머지 번호 비례 배열 정리.
  5. PatchMatch: 192 px 이상에서 건너뜀 비교, 부하 없는 벽시계 0.7 s 확인.
- 막힌 점:
  - 4 코어 측정 기계에 묶음 10개 동시 → 부하 30~45, 시험 하나 4~7 분. 이번 회차 어느 가지도 전체 시험을 못 돌림. 다음 회차는 동시 묶음을 줄이거나 검증 전용 시간 필요.
  - 결정 필요: F-197, F-209 확인 기준, F-251.
  - 소유자 병합 필요: #6·#37·#39~#42·#49 (감독 통과 판정 유지분).

