# 현재 상태

- 상태: 진행 중
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

## 직전 실행 기록 (2026-10-03 15:54Z 시작분)
- 이번 회차 결론: `feat/pipeline`(#46)에 main(트랙·융합 일치)·preview·preview-ba(시험 수정 포함)·pm 을 합쳐 README 단구역 synth → run → verify 가 32 s 에 끝나고 6/7(preview_vs_refined 높이 차 2.657 m 만 실패), PLY·스냅샷·manifest 출력. #48 CI 실패 원인은 카메라 간 짝 기본 일정 변경(시험 기준값이 옛 일정에서 잰 것). PatchMatch 건너뛴 화소 법선 회복(5.85 → 4.01°, CPU/4 0.37 s). 새 PR #49(회전 범위 시험).
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | pipeline (E01 통합) | `feat/pipeline` e0438ca → PR #46(라벨) | experiment/pipeline e1d641d → PR #63 | main·preview·preview-ba·pm 합침(stream-order 는 못 합침). 총괄 재확인: fmt·clippy 통과, README 단구역 명령 run 32 s(종료 0) → verify 6/7: 등록 120/120, 정밀 0.300 px(초벌 3.026), preview_align 점쌍 2746·잔차 3.932 m, preview_vs_refined 최근접 2.358 m·높이 차 2.657 m(실패), 스냅샷 1889 → final 11054. 단구역 시험 2 통과(스윕·패치매치, 468 s, 작업자 실행). 2구역 시험·전체 시험 미실행. CLI 에 `preview_ba_iters` 옵션 없음 |
  | pipeline-preview-ba (#48 CI) | `feat/pipeline-preview-ba` ed6e7c6 → PR #48 | experiment/pipeline-preview-ba b7c5d08 → PR #65 | `sparse::formation_scene_registers_all_and_meets_floors` 실패는 6724181 이 아니라 그 이전부터: 카메라 간 짝 기본 일정이 +12/+16 매 칸 → +20 부터 4칸 간격으로 바뀐 탓. 시험에 옛 일정 명시: 정밀 중심 2.504/5.089 → 0.621/1.282 m, 132/132. 기본 일정에서 이 장면 중심 2.5 m 는 남은 문제. 대상 시험 1 통과(작업자), CI 결과 대기, 라벨 없음 |
  | patchmatch (#6, F-048) | `feat/patchmatch` 58cc1ad → PR #6(라벨) | experiment/patchmatch 4ac4ff4 → PR #20 | 건너뛴 화소에 이웃 법선 평균 후보 1회, 거친 층 반복 3, 고운 층 이웃 3: 960×540 이웃 8장 CPU 1.46 s(CPU/4 0.37 s), 벽시계 2.6~3.1 s(부하 20~24), 깊이 중앙 0.198%, 1% 이내 94.4%, 법선 4.01°(이전 5.85°), 경사 4.37°·계단 4.63°(기준 5°, 여유 얇음). 총괄 재확인: fmt·clippy 통과, `patchmatch` 12 통과·0 실패·4 무시 |
  | pipeline-stream-order (F-272) | `feat/pipeline-stream-order` 6307ed2 | experiment/pipeline-stream-order 27eeb2a | 정지 검사가 앞 구역 도우미 사진 짝까지 세던 것, 무늬 없는 구역이 도우미 사진만으로 등록되던 것 고침. 총괄 재확인: fmt·clippy 통과, `pipeline_regions` 3 통과·0 실패. 같은 수정이 feat/pipeline-regions·-regions-ba 에도 필요. 흐름 순서 표: '최신 정밀 위 등록'·'sim3 재정렬'은 사건 시험 없음, 위치 하나씩 점진 등록은 없음(구역 단위) |
  | rotation-coverage (F-148·F-209) | `feat/rotation-coverage` acaef38 → 새 PR #49(라벨) | experiment/rotation-coverage e4a8add → 새 PR #66 | 기본 bench 8위치는 카메라 간 겹침 0(다른 카메라 짝 0/156, 성분 3) → 24/24 불가. 넓은 편대 34장 시험: 141/141, 2° 초과 2.8%, 34/34, 정렬 중앙 0.116°. 총괄 재확인: fmt·clippy 통과, `formation` 19 통과·0 실패 |
  | translation-averaging (#37, F-276) | `feat/translation-averaging` f9c39df(#37 라벨 유지) | experiment/translation-averaging-f276 39f0851 | 원인: 이웃이 거의 한 직선인 퇴화(점 관측 수 무관). Cauchy 가중 시도는 2.339 → 2.530 m 악화로 되돌림. 시드 21~25: 240/240, 최대 ≤ 0.95 m, RMS 시드 22 점 5% 두 경우 0.362 m. 단언 시험 #[ignore]. 총괄 재확인 못 함 |
  | pipeline-accuracy | `feat/pipeline-accuracy` 5b3f5d9 | experiment/pipeline-accuracy 8b5c69c | poses.txt 에 회전 추가, preview-ba 합침. 초벌 전 BA 8회: 1구역 시드1 높이 차 5.40 → 0.92 m(verify 6/7), 시드2 1.29 m. 중심 중앙 0.96~1.71 m, 회전 중앙 6.8~15.2°·최대 39~45°(축 규약 차이 의심). 2구역 시드2 4/7(스케일 차 26.6%, 점쌍 141). 총괄 확인 못 함 |
  | pipeline-region-align | (코드 변경 없음) | experiment/pipeline-region-align 2d39d47 | 2구역 시드1: 스케일 차 8.91%, 초벌 점쌍 최소 150, 정밀↔정밀 공유 점 30쌍·잔차 0.884 m, refined_overlap 1.749 m. 겹침 사진 포즈 고정 시작은 모든 항목 악화(스케일 차 18.07%). 공유 이미지 양방향 재투영 sim3 는 설계만 |
  | pipeline-pm (F-271) | `feat/pipeline-pm` c95b1a3(main 병합만) | experiment/pipeline-pm b3cd483 | 패치매치 경로 시간의 99% 이상이 장당 깊이 추정(부하 상태 71~175 s), 융합 0.3~1.3 s, 사진별 추정은 이미 병렬. 융합 문턱 0.6배 + 깊이 범위 30% 확대: 95% 꼬리 2.619 → 2.141 m(중앙 0.319 → 0.310 m, 1 m 초과 9.0 → 7.5%), 스윕(1.5 m) 미달, 반영 안 함. fmt·clippy 통과, 패치매치 흐름 시험 1 통과(작업자). 총괄 확인 못 함 |
- 끝까지 흐름 진척: `feat/pipeline` 하나에서 synth → run → verify 가 끝까지 돌고 PLY(preview·refined·snapshots)·manifest·report·poses 가 나온다(단구역 6/7). 이어진 단계: 특징 → 매칭 → 트랙 → 회전·위치 평균 → BA → GPS 정렬 → 초벌/정밀 → 밀집(스윕 기본, 패치매치 선택) → 융합 → 초벌 정렬 → 스냅샷. 남은 것: stream-order 합치기, 초벌 전 BA 를 CLI·기본값으로(SPEC 결정 필요), 2구역 스케일 차·점쌍 부족.
- 다음 할 일:
  1. `feat/pipeline` 에 `feat/pipeline-stream-order`(6307ed2 포함) 합치기 — 구역 루프 충돌 예상.
  2. CLI 에 `--preview-ba-iters` 추가하고 README 명령으로 7/7 확인(accuracy 가지 수치상 높이 차 0.92 m).
  3. 2구역: 공유 이미지 + 양방향 재투영 sim3(관측 30% 통과 이미지, 정상 이미지 비율 ≥ 0.2) 구현, 초벌 정렬 점쌍(150 < 1000) 늘리기.
  4. 회전 오차 39~45° 최대가 축 규약 차이인지 확인(pipeline-accuracy).
  5. 위치 평균 퇴화 카메라(이웃 방향 산포 둘째 고윳값 작음) 판정 후 이웃 보간/사전항(F-276).
  6. PatchMatch 부하 없는 벽시계, 경사·계단 법선 여유.
- 막힌 점:
  - 4 코어 측정 기계에서 묶음 9개 동시로 부하 20~27 — 2구역 시험(6~8 분)·전체 시험을 마감 안에 못 돌림.
  - 결정 필요: 초벌 짧은 BA(SPEC §3 '초벌 BA 없음'), F-209 확인 기준(기본 bench 8위치로는 카메라 간 겹침 불가), 카메라 간 짝 기본 일정(+20, 4칸)에서 sparse 편대 장면 중심 2.5 m, F-197, F-251.
  - 소유자 병합 필요: #37·#39~#42, 이번 라벨 #6·#46·#49.
