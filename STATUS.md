# 현재 상태

- 상태: 진행 중
- 마지막 갱신: 2026-10-07T05:08Z (05:03Z 시작분)
- 진행 묶음(10): cam2-cross-verify, cam-component-attach, tilt-combo-default, seed3-zone0-rot, speckle-cap, verify-report-total, dataset-gap-ranges, ba-schur-memory, json-escapes, ta-test-bounds

## 앞 회차 기록 (2026-10-07 04:06Z 시작분)

- 상태: 끝남
- 마지막 갱신: 2026-10-07T04:48Z (04:06Z 시작분)
- 이번 회차 결론: **두 기울기 옵션(SKYLENS_REALIGN_REF=1 + SKYLENS_ALIGN_LINE_FIX=0.1)을 합친 `feat/zone-tilt-combo` 가 시드 1 에서 verify 8/8·등록 81/81·표면 중앙 0.3356 m(기본 0.4042 m)로 악화 없음, 시드 5 7/8(기본 6/8, 남은 실패 registered), 시드 3 5/8(기본과 같은 수) → 제품 PR #107(기본 끔). 시드 4 구역 2 '회전 평균 약 4°' 는 이전 진단의 행·열 우선 읽기 차이로, 바른 규약에서 롤 선택 뒤 1.408°. 종류별 문턱은 시드 2 에서도 표면 중앙 1.1094 → 1.2533 m 로 나빠져 기본 끔 유지. F-451 교차 짝 단계별 진단은 만들었으나 실행 출력 유실로 짝별 표 미측정.** 4 코어 측정 기계에 동시 묶음 6개라 시드 하나 약 20~30분.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | zone-tilt-combo (두 옵션 합침, 시드 1) | `feat/zone-tilt-combo` 49d6bd9 (병합 5b12298 + 무시 시험 zone_tilt_combo.rs), PR #107 | experiment/zone-tilt-combo fbcf63a, PR #210 | 둘 다 켬: verify 8/8, 등록 81/81, 표면 중앙 0.3356 m, p95 1.103 m, 중심 중앙 0.4237 m, 구역 기울기 0.755/1.533/2.187°. 단독 (a)(b) 미측정. 총괄: fmt 통과, clippy 0 |
  | zone-tilt-seed3 | 측정만(로컬 병합, 푸시 없음) | experiment/zone-tilt-seed3 acb8398, PR #211 | 둘 다/기준만: verify 5/8 둘 다(preview_align·preview_vs_refined·up_cross 실패, 구역 0 붕괴 69.5°), 높이 차 중앙 최대 3.637/3.446 m. 끔 같은 부하 미측정 |
  | zone-tilt-seed5 | 측정만 | experiment/zone-tilt-seed5 afb5663, PR #212 | 둘 다: verify 7/8(registered 만 실패, 54/81), 높이 차 중앙 최대 0.776 m 통과, 구역 2 기울기 0.461°, 구역 0·1 18.7/12.4°. 끔 미측정 |
  | cam2-cross-verify (F-451) | `feat/cam2-cross-verify` 0628a3a (`SKYLENS_CROSS_DEBUG`, 무시 시험 cam2_cross_verify.rs, two_view.rs RansacStats 필드 추가) | experiment/cam2-cross-verify 0d979f8, PR #213 | 시드 5 1회 실행(1180 s) 후 해석기 열 오류로 출력 유실 → 짝별 표 미측정. 열 고침·stderr 저장 추가 후 미실행. 총괄: fmt 통과 |
  | zone2-rot-tilt | `feat/zone2-rot-tilt` 34228af (`SKYLENS_DIAG_ROT`, 무시 시험 zone2_rot_tilt.rs) | experiment/zone2-rot-tilt edc0fca, PR #214 | 시드 4 바른 규약: 롤 선택 뒤 기울기 구역 0/1/2 = 2.791/1.730/1.408°, 구역 2 간선 169·잔차 중앙 0.026°·가지치기 0. 이전 4° 는 진단 규약 오류(위치 평균·BA 단계 재측정 필요) |
  | rot-class-seed2 | 변경 없음 | experiment/rot-class-seed2 e186fca, PR #215 | 끔/켬: verify 8/8 둘 다, 표면 중앙 1.1094/1.2533 m, 중심 중앙 0.3456/0.3819 m → 기본 끔 유지(시드 1·2 모두 표면 악화, 이득은 시드 3 뿐) |
- 끝까지 흐름 진척: main c770c80 에서 전부 연결, 변화 없음. 기본 경로 verify: 시드 1 8/8, 시드 3 5/8, 시드 4 7/8, 시드 5 6/8. 두 기울기 옵션 함께 켜면 시드 1 8/8, 시드 5 7/8, 시드 3 5/8.
- 다음 할 일:
  1. F-451: feat/cam2-cross-verify 에서 `UPX_SEED=5 KEEP_OUT=1 SKYLENS_CROSS_MIN_MATCHES=10 cargo test --release -p skylens-stream --test cam2_cross_verify -- --ignored --nocapture` 1회로 짝별 탈락 단계·정답 겹침 표.
  2. PR #107 기본 켬 판단: 시드 4 합친 설정, 시드 3·5 끔 같은 부하 비교(동시 묶음 수를 줄여서).
  3. short-zone-tilt 의 위치 평균·BA 단계 기울기를 바른 회전 규약으로 다시 측정(DIAGPOSE 읽기 수정).
  4. 시드 2 표면 중앙 1.11 m(끔에서도 기준 0.5 m 초과) 원인.
- 막힌 점:
  - 소유자 병합 필요: #107(신규), #106, #105, #104, #93, #99(#101 포함), #97, #55, #54 → #58 → #87, #90, #76(#77 포함), #89.
  - 동시 묶음 6개에서 시드 하나가 20~30분 — 다음 회차는 4개 이하로.

## 앞 회차 기록 (2026-10-07 03:06Z 시작분)

- 상태: 끝남
- 마지막 갱신: 2026-10-07T03:42Z (03:06Z 시작분)
- 이번 회차 결론: **짧은 마지막 구역 연직 기울기(preview_vs_refined 실패)를 잡는 두 안이 시드 4 에서 모두 verify 7/8 → 8/8: 재정렬 기준을 기울기가 가장 작은 정밀 구역으로 고르기(SKYLENS_REALIGN_REF=1, 높이 차 중앙 최대 2.918 → 0.901 m), 위치가 일직선에 가까울 때 GPS 정렬 연직 고정(SKYLENS_ALIGN_LINE_FIX=0.1, 구역 2 기울기 13.458 → 1.230°). 시드 4 구역 2 기울기는 회전 평균에서 약 4°, GPS 정렬에서 약 6° 더해짐. 시드 5 카메라 2 미등록(F-451)은 가지치기가 아니라 카메라 2 교차 회전 간선이 애초에 0개(앞-왼 교차 짝 비율 대응 8~20 개)이고, 하한 10 완화로는 기하 검증에서 탈락해 54/81 그대로. 종류별 회전 문턱은 시드 1 표면 중앙 0.5059 m(기준 0.5 m)로 기본 끔 유지.** 4 코어 측정 기계라 동시 묶음 4개 + 뒤이은 1개.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | rot-class-default (시드 1 종류별 문턱) | `feat/rot-class-default` 914cb01 (시험 default_path_class.rs 만, PR 없음) | experiment/rot-class-default 098940d, PR #205 | 끔/켬: 등록 81/81, 중심 중앙 0.4265/0.4279 m, 표면 중앙 0.4042/0.5059 m, p95 1.3237/1.5687 m, verify 8/8 둘 다 → 기본 끔 유지. 시드 2 미측정. 총괄: fmt 통과 |
  | seed5-cam2-prune (F-451 원인) | `feat/seed5-cam2-prune` cfcca1f (#106 위 + rot-thresh-pipeline 병합, `SKYLENS_PRUNE_CAM` 진단, PR 없음) | experiment/seed5-cam2-prune fd7853b, PR #206 | 카메라 2 낀 간선 278·158·20 개 전부 카메라 2 끼리, 교차 0. 앞-왼 교차 짝 18 중 대응 20 이상 2(앞-오른 14). 가지치기 전 첫 평균부터 rot_none. 총괄: fmt 통과 |
  | seed5-cross-min (F-451 대안) | `feat/seed5-cross-min` fb5802b (`SKYLENS_CROSS_MIN_MATCHES`, 기본 20, PR 없음) | experiment/seed5-cross-min 145f743, PR #209 | 켬 10: 등록 54/81 그대로, 카메라 2 0/27, 하한 넘은 교차 짝 9개 전부 기하 검증 탈락, verify 6/8. 시드 1 미측정. 총괄: fmt 통과 |
  | realign-ref-zone (재정렬 기준 구역) | `feat/realign-ref-zone` a5cec08 (preview-refined-gap 위, `SKYLENS_REALIGN_REF` 기본 끔, PR 없음) | experiment/realign-ref-zone 92ed2fa, PR #207 | 시드 4 끔/켬: verify 7/8 → 8/8, 높이 차 중앙 최대 2.918 → 0.901 m, 최근접 중앙 최대 2.635 → 1.022 m, 구역 0·1 정밀 5 m 초과 32~34 % → 약 2.8 %. 새 시험 realign_ref_zone.rs 미실행. 시드 3·5 미측정. 총괄: fmt 통과, clippy 0 |
  | short-zone-tilt (짧은 구역 기울기 원인) | `feat/short-zone-tilt` 063ee9f (preview-refined-gap 위, `SKYLENS_ALIGN_LINE_FIX` 기본 끔, DIAGPOSE 진단, PR 없음) | experiment/short-zone-tilt 8191dc7, PR #208 | 구역 2 단계별(구역 0 대비): 회전 평균 +4.0°, 위치 평균 +4.0°, BA +4.0°, GPS 정렬 +9.9°. 켬 0.1: 구역 2 자기 정렬 13.458 → 1.230°, 정밀 1→2 9.630 → 0.449°, verify 7/8 → 8/8. 단언 고친 뒤 재실행 못 함. 시드 1·3·5 미측정. 총괄: fmt 통과, clippy 0 |
- 끝까지 흐름 진척: main c770c80 에서 전부 연결, 변화 없음. 기본 경로 verify: 시드 1 8/8, 시드 3 5/8, 시드 4 7/8(두 새 옵션 각각 켜면 8/8), 시드 5 6/8.
- 다음 할 일:
  1. 시드 1·3·5 에서 SKYLENS_REALIGN_REF=1 과 SKYLENS_ALIGN_LINE_FIX=0.1 (따로·함께) verify·표면 오차 — 시드 1 악화 없으면 기본 켬 PR.
  2. F-451: 카메라 2 교차 짝이 기하 검증 어느 단계(RANSAC 정상 수·우연 확률·자세 복원)에서 떨어지는지 계측, 앞-왼 교차 일정 오프셋이 실제 시야 겹침과 맞는지 확인(feat/seed5-cross-min 위).
  3. 시드 4 구역 2 회전 평균 단계 약 4°(짧은 구역 롤 선택) 원인.
  4. 시드 2 종류별 문턱 끔/켬(`CLASS_SEEDS=2`, feat/rot-class-default).
- 막힌 점:
  - 소유자 병합 필요: #106, #105, #104, #93, #99(#101 포함), #97, #55, #54 → #58 → #87, #90, #76(#77 포함), #89.
  - F-197(높음) 은 SPEC §3.2 개정 결정, F-447 의 SPEC §4 표는 소유자 결정. 종류별 문턱 기본 켬은 표면 기준 0.5 m 를 0.006 m 넘어 기준 재검토 여부도 소유자 결정.
  - 동시 묶음 수는 4 코어라 4~5개로 제한. 시드 하나 기본 경로가 부하 아래 약 9~10분.

## 앞 회차 기록 (2026-10-07 01:48Z 시작분)

- 상태: 끝남
- 마지막 갱신: 2026-10-07T02:30Z (01:48Z 시작분)
- 이번 회차 결론: **시드 1 표면 오차 0.585 m 는 덩어리 잇기 옵션 자체의 결과(같은 설정 반복 차 0, 끔 0.4042 / 켬 0.5849 m). 시드 3 에서는 종류별 문턱(SKYLENS_ROT_CLASS_THRESH)이 다리와 같은 정확도(정밀 BA 뒤 F/R/L 0.90/0.50/0.13°, verify 5/8 → 7/8)라 기본 켬 후보 — 시드 1 표면 오차 미측정. preview_vs_refined 높이 차의 뿌리는 초벌 점군이 아니라 짧은 마지막 구역(시드 4 구역 2) 정밀 모델의 연직 기울기 13.5° 이고, 그 모델 기준 재정렬이 구역 0·1 정밀을 9.6~10.2° 같이 기울임. 시드 5 미등록 27장은 모두 카메라 2 로 회전 평균 가지치기 단계에서 빠짐.** 4 코어 측정 기계라 동시 묶음 5개.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | bridge-onoff-load (시드 1 끔/켬) | `feat/bridge-onoff-load` 171ce39 (zone0-bridge-ba + main, 시험 default_path_onoff.rs, PR 없음) | experiment/bridge-onoff-load 2c30e5d, PR #202 | 끔/켬 2회씩: 등록 81, 중심 중앙 0.4265/0.3430 m, 표면 중앙 0.4042/0.5849 m, p95 1.3237/1.5751 m, verify 8/8 둘 다. 반복 차 0 → 옵션 탓 확정. 시드 2 미측정. 총괄: fmt 통과, clippy 0 |
  | rot-thresh-pipeline (시드 3 네 설정) | `feat/rot-thresh-pipeline` 09b7214 (rot-bridge-threshold + main; pipeline.rs `SKYLENS_ROT_CLASS_THRESH`·`SKYLENS_ROT_GM` 기본 끔, 시험 rot_thresh_pipeline.rs, PR 없음) | experiment/rot-thresh-pipeline 4917efd, PR #203 | 회전 평균 직후 L: 끔 84.72 / 다리 0.78 / 종류별 0.78 / GM 2.81°. 정밀 BA 뒤 F/R/L: 끔 3.46/1.10/74.30, 다리 0.96/0.55/0.14, 종류별 0.90/0.50/0.13, GM 1.03/0.46/0.12°. verify 5/8 → 7/8(셋 다 preview_vs_refined 만 실패). 총괄: fmt 통과, clippy 0, `--lib rotation_averaging` 13 통과 |
  | preview-refined-gap (시드 4 높이 차 뿌리) | `feat/preview-refined-gap` b328646 (main 위; `SKYLENS_DIAG_PREVIEW`, `SKYLENS_ALIGN_MAX_TILT` 기본 0, 시험 preview_refined_gap.rs, PR 없음) | experiment/preview-refined-gap 3527903, PR #204 | 정렬 전 초벌 높이 p50 0.05~0.44 m 정상. 정밀 구역 0·1 p05/p95 약 −6.7/+8.3 m(5 m 초과 32~34%). 구역 2 정밀 연직 13.458° 기울기 → 정밀 1→2 9.630°, 0→2 10.245° 같이 기울임. sim3 축척 0.97~1.01, 이동 z 작음. 6° 초과 거부 켬: 구역 0·1 정상화, 구역 2 정렬 없음으로 verify 5/8 → 기본 끔. 총괄: fmt 통과, clippy 0 |
  | seed5-reg-diag (시드 5 등록, F-446·F-447) | `feat/seed5-reg-diag` 11ee2a5, PR #106, 라벨 | experiment/seed5-reg-diag b9de258, PR #201 | 미등록 27장 전부 카메라 2(시점마다), 짝 4~12·내부 대응 ~1000 으로 충분, 회전 평균 가지치기에서 빠짐(가지친 간선 구역 0 139/346). up_cross 전부 null → '경고: 측정값 없음'(통과 유지). F-447 처리. 총괄: fmt 통과, clippy 0, `--lib verify::` 13·`--test verify` 25 통과 |
  | rig-tilt-gauge (F-449) | `feat/rig-tilt-gauge` 25873f0, PR #105, 라벨 | experiment/rig-tilt-gauge ea916a7, PR #200 | 고정 정답에서도 끔 R 1.449~1.947° 로 시작 0.703° 보다 나쁨 → 기체 간 결합 부족이 주원인. 감독 검토로 F-449 닫힘. 총괄: fmt 통과, clippy 0, `--test rig_tilt_ba` 2 통과·1 무시 |
- 끝까지 흐름 진척: main c770c80(#100 병합)에서 전부 연결, 변화 없음. 기본 경로 verify: 시드 1 8/8, 시드 3 5/8(종류별 문턱 켜면 7/8), 시드 4 7/8, 시드 5 6/8.
- 다음 할 일:
  1. 시드 1 기본 경로에서 `SKYLENS_ROT_CLASS_THRESH=1` 표면 오차 중앙(feat/rot-thresh-pipeline, `ROT_SETTINGS`) — 0.5 m 안이면 종류별 문턱 기본 켬 PR.
  2. 짧은 마지막 구역 정밀 BA 의 연직 기울기 13.5° 원인(시드 4 구역 2, 위치 22~27). 대안: 재정렬 기준을 기울기가 작은 구역으로 고르기.
  3. 시드 5 카메라 2 가 회전 평균 가지치기로 통째 빠지는 기준 — `SKYLENS_REG_DEBUG` 에 잘린 간선과 기준 출력 추가, 장착 자세 확인. 고친 뒤 시드 1·5 등록과 시드 5 up_cross(F-446).
  4. 덩어리 잇기 켬에서 중심은 좋아지는데 표면이 나빠지는 이유.
- 막힌 점:
  - 소유자 병합 필요: #106, #105, #104, #93, #99(#101 포함), #97, #55, #54 → #58 → #87, #90, #76(#77 포함), #89.
  - F-197(높음) 은 SPEC §3.2 개정 결정, F-447 의 SPEC §4 표는 소유자 결정.
  - 동시 묶음 수는 4 코어라 5개로 제한(10개 이상 지시와 다름). 시드 하나 기본 경로가 부하 아래 약 5~10분.

## 앞 회차 기록 (2026-10-07 01:05Z 시작분)

- 상태: 끝남
- 마지막 갱신: 2026-10-07T01:40Z (01:05Z 시작분)
- 이번 회차 결론: **덩어리 잇기(SKYLENS_ROT_BRIDGE=1)는 시드 3 구역 0 L 묶음을 정밀 BA 뒤에도 고친다(L 회전 오차 중앙 0.269°, verify 5/8 → 7/8). 시드 1 은 포즈 9묶음 모두 기준 안이지만, 기본 켬 상태 기본 경로 시드 1 시험에서 표면 오차 중앙 0.585 m(기준 0.5 m) 로 실패해 기본 켬은 되돌림.** 끔 상태 같은 부하 재측정이 없어 옵션 탓인지 미확정. 회전 평균 문턱을 종류별로 나누는 안·GM IRLS(σ 5°, 10° 거르기) 안은 합성 그래프에서 덩어리 잇기와 같은 수준(1.9~2.0°). 시드 4·5 기본 경로는 up_cross 거짓 실패 없음, 다만 시드 4·5 모두 preview_vs_refined 실패, 시드 5 는 등록 54/81. 4 코어 측정 기계라 동시 묶음 4개 + 분석 1개.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | zone0-bridge-ba (시드 3 정밀 BA 뒤) | `feat/zone0-bridge-ba` 7edac69 (zone0-rot 위 + main 병합; 기본 켬 4d0f406 → 되돌림 7edac69, PR 없음) | experiment/zone0-bridge-ba f2effef, PR #198 | 정밀 BA 뒤 구역 0 회전 중앙 F/R/L 0.862/0.523/0.269°, 위치 중앙 0.251 m. verify 켬 7/8(preview_align 0.26%, up_cross 0.894/0.258/1.700°, preview_vs_refined 높이 차 3.289 m 실패). 기본 켬 default_path 시드 1: 등록 81, 중심 중앙 0.343 m, 표면 중앙 0.585 m 실패. 총괄: 기본 동작 불변이라 제품 PR 안 엶 |
  | zone0-bridge-seeds (시드 1·2 끔/켬) | `feat/zone0-bridge-seeds` f946591 (zone0-rot 위; 시험 bridge_seeds.rs, PR 없음) | experiment/zone0-bridge-seeds bb655bc, PR #197 | 시드 1 세 구역 9묶음 모두 기준 통과(최대 악화 구역 1 L +0.045°, 구역 0 L 1.060 → 0.471°). 시드 2 미측정. 총괄: fmt 통과, clippy 0 |
  | rot-bridge-threshold (회전 평균 문턱) | `feat/rot-bridge-threshold` f0c8557 (zone0-rot 위; rotation_averaging.rs `class_thresholds`·`gm_irls`·`weak_bridge`, 기본 끔, PR 없음) | experiment/rot-bridge-threshold b671fdc, PR #196 | 맞는3·틀린2: 기본 3.489°, 하한 10° 1.928°, 잇기·종류별·GM 1.93~1.98°. 초기화가 틀린 다리: 기본·하한 10° 83°, 잇기·종류별·GM 1.9~2.0°. 틀린 쪽 가중합이 크면 새 방법도 83°. 정상 장면 기본 0.930 → 0.631°. 총괄: fmt 통과, clippy 0, `--test rotation_bridge --test rotation_bridge_threshold` 2+3 통과, `--lib rotation_averaging` 13 통과 |
  | verify-up-cross-seeds (F-446) | `feat/verify-up-cross-seeds` 5f072f0 (main 위; 시험 up_cross_seeds.rs, PR 없음) | experiment/verify-up-cross-seeds 27fcb59, PR #199 | 시드 4 0.559/1.477/2.135° 문턱 안, verify 7/8. 시드 5 등록 54/81, diff_deg 전부 null → 건너뜀, verify 6/8. F-446 열림 유지. 총괄: fmt 통과, clippy 0 |
  | (확인) rig-tilt-ba F-442 | `feat/rig-tilt-ba` 15d8622, PR #100 라벨 유지 | experiment/rig-tilt-fleet 3a949f3, PR #192 | 앞 회차 끝무렵 처리분 확인: fmt 통과, clippy 0, `--test rig_tilt_ba` 2 통과(29.7 s). F-442 처리됨-검증대기 |
- 끝까지 흐름 진척: main 8282283 에서 전부 연결, 변화 없음. 기본 경로 verify: 시드 1 8/8, 시드 3 5/8, 시드 4 7/8, 시드 5 6/8 — preview_vs_refined(높이 차 2.9~3.3 m)가 시드 3·4·5 공통 실패.
- 다음 할 일:
  1. 같은 부하에서 기본 경로 시드 1 끔/켬(SKYLENS_ROT_BRIDGE) 표면 오차 나란히 측정 — 0.585 m 가 옵션 탓인지 가르기. 시드 2 끔/켬 포즈 표(`ZONE0_SEEDS=2 cargo test --release -p skylens-stream --test bridge_seeds -- --ignored --nocapture`, feat/zone0-bridge-seeds).
  2. preview_vs_refined 높이 차 2.9~3.3 m 가 시드 3·4·5 공통 — 원인 조사(초벌·정밀 높이 기준 차이).
  3. 시드 5 등록 54/81 원인, up_cross 가 등록 부족 시 '건너뜀' 으로 통과하는 문제(F-446).
  4. rot-bridge-threshold 를 파이프라인 시드 3 에서 측정(종류별 문턱·GM 을 환경 변수로 켜는 연결은 pipeline.rs 쪽 묶음).
- 막힌 점:
  - 소유자 병합 필요: #104, #100, #93, #99(#101 포함), #97, #55, #54 → #58 → #87, #90, #76(#77 포함), #89.
  - F-197(높음) 은 SPEC §3.2 개정 결정 필요.
  - 동시 묶음 수는 4 코어라 4개로 제한(10개 이상 지시와 다름). 무거운 측정(시드당 8~10분)이 겹쳐 시드 2·끔 재측정이 마감 안에 못 들어감.

## 앞 회차 기록 (2026-10-07 00:45Z 시작분)

- 상태: 끝남
- 마지막 갱신: 2026-10-07T01:22Z (00:45Z 시작분)
- 이번 회차 결론: **회전 다리 옵션(`SKYLENS_ROT_BRIDGE`)을 켜면 시드 3 기본 경로 verify 가 5/8 → 7/8 — 위 방향 일치(69.5 → 0.89°)와 preview_align(스케일 차 25.41 → 0.26%)은 구역 0 L 묶음 회전 오류와 같은 뿌리. preview_vs_refined(높이 차 3.289 m)는 다른 뿌리로 남음. 정밀 BA 뒤 시드 1·2 는 18칸 중 17칸 +0.2° 이내, 시드 2 구역 0 L +0.222° 초과라 기본 켬 제안은 아직 불가.** 뿌리 쪽 대안(묶음 사이 간선 따로 문턱)은 합성 그래프에서 맞는 간선 3개 유지·틀린 2개 제외 확인, 시드 3 실측 남음. F-442(높음)·F-443 처리: 편대 장면에서 장착 상대 회전 공유 항은 기준 카메라 F 를 10.5% 나쁘게 해 기본 끔 유지. #103 main 병합됨. 4 코어 측정 기계라 동시 묶음 4개.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | rot-bridge-verify (시드 3 verify 끔/켬) | `feat/rot-bridge-verify` 1b499ac (zone0-rot + main 병합, 무시 시험 rot_bridge_verify.rs, PR 없음) | experiment/rot-bridge-verify e149e0a, PR #193 | 시드 3 끔 5/8 → 켬 7/8; preview_vs_refined 최근접 3.151 → 2.973 m, 높이 차 4.360 → 3.289 m(실패 유지). 시드 1 8/8 → 8/8, 최근접 0.473 → 0.737 m·높이 차 0.310 → 0.727 m(문턱 안이나 악화). 총괄: fmt 통과, clippy 0, 시험 컴파일·무시 1 |
  | rot-bridge-ba (정밀 BA 뒤 시드 1·2) | 변경 없음(feat/zone0-rot 6f369fa 로 측정) | experiment/rot-bridge-ba a2bc89a, PR #194 | 묶음별 회전 중앙 끔→켬: 시드 1 구역 0 L 1.060→0.471°, 시드 2 구역 0 L 0.994→1.217°(+0.222° 초과), 시드 2 구역 2 L 1.697→1.365°, 나머지 ±0.1° 안팎. 위치 중앙 시드 2 구역 0 0.304→0.380 m. 반복 분산 미측정 |
  | rot-cross-thresh (묶음 사이 간선 문턱) | `feat/rot-cross-thresh` 79c16af (zone0-rot 위; rotation_averaging.rs `cross_thresh` 기본 끔, pipeline.rs `SKYLENS_ROT_CROSS_THRESH`, 시험 rotation_cross_thresh.rs, PR 없음) | experiment/rot-cross-thresh b35373a, PR #195 | 합성(묶음 3개, 사이 맞는 3~4° 3개 + 83° 2개): 켬 사이 문턱 12.69°, B/C 오차 2.03/2.78°, 틀린 2개 제외; 끔 문턱 1.00°, B 3.59°. 유지 간선 ≤2 묶음 쌍 경고. 시드 3 실측 못 함. 총괄: fmt 통과, `--test rotation_cross_thresh` 3·`rotation_bridge` 2 통과 |
  | rig-tilt-fleet (F-442·F-443) | `feat/rig-tilt-ba` 15d8622, PR #100, 라벨 | experiment/rig-tilt-fleet 3a949f3, PR #192 | 흔들림 1° σ0.1 켬/끔 F 0.185/0.168, R 1.082/2.486, L 0.352/1.238° → 기본 끔 단언. 기울기 평균 +0.679/+0.716/+0.658°. 총괄: fmt 통과, clippy 0, `--test rig_tilt_ba` 2 통과 |
- 끝까지 흐름 진척: main e6be913(#103 포함)에서 전부 연결. verify 8개 항목으로 시드 3 회전 고장을 잡음. 다리 옵션 켜면 시드 3 이 7/8.
- 다음 할 일:
  1. 시드 3 preview_vs_refined 높이 차 3.3 m 의 뿌리 — 구역 1(위 방향 정상)도 초벌-정밀 높이 차 끔 2.30·켬 2.70 m(근사). 초벌 점군 자체인지 구역 1 정렬인지 가르기.
  2. `SKYLENS_ROT_CROSS_THRESH=1 ZONE0_SEEDS=3` 로 회전 평균 직후·정밀 BA 뒤 F/R/L, 이어서 시드 1·2 회귀. 다리 옵션보다 낫다면 그것을 기본 후보로.
  3. 다리 옵션 시드 2 구역 0 L +0.222° 가 반복 분산 안인지(같은 설정 2회), 시드 4·5 추가.
  4. 구역 2 축척(시드 2 구역 2 0.84), 간격 비 NaN — 앞 회차 그대로.
- 막힌 점:
  - 소유자 병합 필요: #104, #100(F-442 처리), #93, #99(#101 포함), #97, #55, #54 → #58 → #87, #90, #76(#77 포함), #89.
  - F-197(높음) 은 SPEC §3.2 개정 결정 필요.
  - feat/preview-realign 결정 필요.
  - 동시 묶음 수는 4 코어라 4개로 제한(10개 이상 지시와 다름).

## 앞 회차 기록 (2026-10-07 00:06Z 시작분)

- 상태: 진행 중
- 마지막 갱신: 2026-10-07T01:06Z (01:05Z 시작분)
- 이번 회차 결론: **시드 3 구역 0 의 카메라 L 묶음 회전 오류(74°) 원인을 찾음 — L 은 L–F 간선 5개로만 이어지고(L–R 0개) 그중 2개가 서로 일관되게 83° 틀림. 회전 평균 이상치 문턱이 묶음 내부의 정확한 간선 잔차에 맞춰져 하한 1° 로 떨어지므로, 맞는 묶음 사이 간선 3개(3~4° 오차)가 버려지고 틀린 2개만 남음. 좌표계 맞춤·방향 뒤바뀜 문제 아님, 삼각형 순환 검사로는 못 잡음(묶음 사이 간선에 삼각형 없음).** 덩어리 사이 보정 회전 다수결 옵션(기본 끔)으로 회전 평균 직후 L 84.72 → 0.78°. verify 에 위 방향 일치 항목 추가(시드 3 이 이제 실패로 잡힘), F-445 처리. 4 코어 측정 기계라 동시 묶음 4개 + 분석 1개.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | zone0-rot (구역 0 L 묶음 회전) | `feat/zone0-rot` 6f369fa (zone0-up 위; rotation_averaging.rs `bridge_components` 기본 끔, pipeline.rs `SKYLENS_ROT_BRIDGE`·`SKYLENS_DUMP_ROTAVG`, 시험 rotation_bridge.rs, PR 없음) | experiment/zone0-rot a2e9a96, PR #191 | 회전 평균 직후 묶음 오차 중앙 F/R/L 1.27/0.33/84.72° → 켬 0.81/0.20/0.78°. L–F 간선 5개 중 3개 3.2~4.0°, 2개 83.5/82.7°(측정 133.6/130.2° 대 정답 113°). 묶음 사이 유지 간선 F 8·R 3·L 5. 정밀 BA 뒤·시드 1·2 비교 미측정 → 기본값 아님. 총괄: 다시 돌리지 않음(PR 안 엶) |
  | verify-up-cross (verify 8번째 항목) | `feat/verify-up-cross` 102d5f0, PR #103, 라벨 | experiment/verify-up-cross 678bf79, PR #188 | 구역별 최대 어긋남 > 10° 실패, 0.3~10° 경고(통과), report 없음 건너뜀. 시드 1/2/3: 경고/경고/실패(구역 0 69.509°), 시드 3 verify 5/8 종료 코드 1. 총괄: fmt 통과, clippy 0, `--lib verify` 14 통과, `--test verify` 25 통과 |
  | up-cross-alert (F-445) | `feat/up-cross-alert` 360805f, PR #104, 라벨 | experiment/up-cross-alert 5b00627, PR #190 | 구름 2° 장면 camF 지목·다음 값과 차 0.267°, 구름 없는 장면 issues 줄 없음. 띠 배치 시드 0 은 구름 없는 camL 이 최대(지목은 후보). 총괄: fmt 통과, clippy 0, `up_cross` 2 통과, `--test pipeline_up_cross` 2 통과(57 s) |
  | zone2-gps-prior-seeds (구역 2 축척 시드 1·2) | `feat/zone2-gps-prior-seeds` 8a4386a (zone2-gps-prior 위; 시험만, PR 없음) | experiment/zone2-gps-prior-seeds 30e1e0c, PR #189 | 구역 간 축척 차 시드 1: 기본 0.175·w0.25 0.104·w0.5 0.164·cauchy2 0.141, 시드 2: 0.230·0.226·0.233·0.232 → w0.25 기본값 근거 부족. 시드 2 구역 2 축척 0.84(반대 방향). 간격 비 NaN 일부 남음. 총괄: fmt 통과, clippy 0 |
- 끝까지 흐름 진척: main f25e0a8 에서 전부 연결, 변화 없음. #103 이 들어가면 verify 가 구역 묶음 회전 고장(시드 3)을 실패로 잡음.
- 다음 할 일:
  1. `SKYLENS_ROT_BRIDGE=1` 의 정밀 BA 뒤 L 묶음 회전 오차(기준 ≤ 3°)와 시드 1·2 세 구역 변화(각 +0.2° 이내) 측정 → 통과하면 기본 켬 제안. 명령: `SKYLENS_ROT_BRIDGE=1 ZONE0_SEEDS=1,2,3 cargo test --release -p skylens-stream --test zone0_up -- --ignored --nocapture`(시드당 8~9분).
  2. 이상치 문턱 하한이 묶음 내부 잔차로 정해지는 문제 자체 — 묶음 사이 간선에 따로 문턱을 두는 안, 묶음 사이 간선 수·위치 수 하한 경고(L–R 간선 0개).
  3. verify-up-cross 보고: 시드 3 기본 경로는 preview_align(스케일 차 25.41%)·preview_vs_refined(최근접 3.151 m, 높이 차 4.360 m)도 실패였음 — 앞 회차 '81/81 통과' 는 등록 항목만의 결과. 구역 0 회전 고장과 같은 뿌리인지 확인.
  4. 구역 2 축척: 시드 2 구역 2 축척 0.84 원인, 간격 비 NaN 나머지. zone2-gps-prior-seeds 측정에서 등록 수가 54·31 로 나옴 — 시험 설정 차이인지 확인 필요.
- 막힌 점:
  - 소유자 병합 필요: #104, #103, #93, #99(#101 포함), #97, #55, #54 → #58 → #87, #90, #76(#77 포함), #89.
  - F-197(높음) 은 SPEC §3.2 개정 결정 필요. #100 은 F-442 로 보류.
  - feat/preview-realign 결정 필요.
  - 동시 묶음 수는 4 코어라 4개로 제한(10개 이상 지시와 다름).

## 앞 회차 기록 (2026-10-06 23:06Z 시작분)

- 상태: 끝남
- 마지막 갱신: 2026-10-06T23:50Z (23:06Z 시작분)
- 이번 회차 결론: **기본 경로 시드 3 구역 0 의 위 방향 어긋남 47~70° 는 교차 검사 퇴화가 아니라 정밀 포즈 자체 오류다 — 카메라 L 묶음 27대 회전 오차 중앙 74.3°(F 3.5°, R 1.1°), 위치 오차 최대 25 m. 회전 평균 단계에서 이미 생기고(L 123.6°) 정밀 BA 가 L 을 고치지 못함. verify 7개 항목은 회전 오차를 보지 않아 81/81 통과.** 초벌 정렬에 카메라 중심 쌍 추가는 효과 없음, 정밀 BA 시작점 다듬기 스위치는 시드 3·4 에서 악화(기본값 후보 아님), 구역 2 축척은 GPS 사전항 가중 0.25 배가 시드 3 에서만 기준 안(시드 1·2 미측정). 4코어 측정 기계라 동시 묶음 4개, 모두 측정·선택 옵션(기본 동작 불변).
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | zone0-up (시드 3 구역 0 위 방향) | `feat/zone0-up` 0a79d60 (up-cross-span12 위; 시험 zone0_up.rs + pipeline.rs 환경 변수 덤프, PR 없음) | experiment/zone0-up 425c230, PR #184 | 묶음별 회전 오차 중앙/최대 F 3.456/3.514·R 1.098/2.924·L 74.30/78.83°, 위치 중앙 6.21/8.88/12.52 m. 단계별 L: 회전·위치 평균 직후 123.6 → 초벌 BA 123.6 → 정밀 BA 74.3°. 묶음 방위 퇴화로 `up_own` 모두 None, 정답 회전 넣으면 같은 갈래 0.000°. 구역 1·2 정상(중앙 ≤1.45°). 총괄: fmt 통과, clippy 0, `--test zone0_up --ignored` 통과(189 s) |
  | preview-align-centers (초벌 정렬 중심 쌍) | `feat/preview-align-centers` f659238 (preview-error-stages 위; align.rs `umeyama_weighted`, stream.rs, pipeline.rs `SKYLENS_ALIGN_CENTERS` 기본 끔, PR 없음) | experiment/preview-align-centers 3ad1037, PR #185 | 구역 1 점 오차 중앙/p90 점만 2.115/4.168, 중심만 2.512/3.934, 점+중심 w1 2.122/4.028 m. 중심만 높이 차 0.609 m(점만 0.310). 모든 조건 SPEC §4 안, 개선 없음 → 제안 없음. 앞 회차 수치(0.64 대 5.24 m) 재현 안 됨(설정 차이 추정). 총괄: fmt 통과, clippy 0, `--test preview_align_centers` 1 통과·1 무시 |
  | zone2-gps-prior (구역 2 축척) | `feat/zone2-gps-prior` 1694de0 (zone2-scale 위; pipeline.rs `SKYLENS_GPS_PRIOR` 기본 끔, PR 없음) | experiment/zone2-gps-prior df13542, PR #186 | 시드 3, 구역 간 축척 차: 기본 0.139, w0.25 0.077(유일 통과), w0.5 0.114, w2 0.169, huber1/2 0.110, cauchy2 0.108, 풀이 뒤 정렬만 1.812. 시드 1·2 미측정, 카메라 간격 비 NaN(번호 짝짓기 의심). 총괄: fmt 통과, clippy 0 |
  | refine-start-seeds (BA 시작점 스위치) | `feat/refine-start-seeds` 8531add (tilt-stages 위; 시험만, PR 없음) | experiment/refine-start-seeds 82682fb, PR #187 | 단구역 위 방향 끔→켬 시드 1 0.434→0.280, 2 0.549→0.495, 3 2.259→2.691, 4 0.236→0.340, 5 1.575→1.140°; 평균 −0.021, 최악 +0.432°. 기본 경로 시드 1 1.715→1.221° 이나 표면 중앙 0.393→0.467 m. 기본값 후보 아님. 총괄: fmt 통과, clippy 0 |
- 끝까지 흐름 진척: main f25e0a8 에서 전부 연결, 변화 없음. 단, 기본 경로가 verify 를 통과하면서도 구역 하나의 카메라 묶음 회전이 74° 틀릴 수 있음이 드러남.
- 다음 할 일:
  1. 시드 3 구역 0 회전 평균: L 묶음이 따로 떨어진 연결 성분인지(간선 가지치기), 좌표계 맞춤(비행 축 둘레 회전 선택)이 틀린 것인지 — 회전 평균 직후·좌표계 맞춤 전 덤프로 가르기. 단구역 시드 3 의 회전 오차 4~5.5° 도 같은 뿌리인지.
  2. verify 에 정답 없이 볼 수 있는 회전 이상 검사(예: 위 방향 교차 검사 초과를 실패로) 넣을지 결정 — F-445 와 함께.
  3. zone2-gps-prior 시드 1·2 측정, 카메라 간격 비 NaN 원인(zone2_scale.rs 정답 번호 매김) 확인.
- 막힌 점:
  - 소유자 병합 필요: #93, #99(#101 포함), #97, #55, #54 → #58 → #87, #90, #76(#77 포함), #89.
  - F-197(높음) 은 SPEC §3.2 개정 결정 필요. #100 은 F-442 로 보류.
  - feat/preview-realign 결정 필요.

## 앞 회차 기록 (2026-10-06 22:06Z 시작분)

- 상태: 끝남
- 마지막 갱신: 2026-10-06T22:40Z (22:06Z 시작분)
- 이번 회차 결론: **단구역 출력 좌표계 기울기 0.81° 는 파이프라인이 아니라 GPS 위치 자체의 기울기(0.786°, 동/북/위 +0.721/−0.278/−0.142)를 중심이 따라간 것이다 — 미해명 0.3° 해소, 최종 단계 기여 0.0000°. 구역 2 초벌 축척 1.178 은 GPS 잡음이 같은 위치 카메라 3대 간격을 누른 것(간격 비 0.848, 정답 GPS 에서 1.006). 구역 1 초벌 점 오차는 카메라 위치가 지배하고 정렬 단계 손실이 크다(최적 닮음 0.64 m 대 현재 5.24 m). 기본 경로 위 방향 교차 검사는 시드 3개 모두 거짓 초과(9구역 중 5, 시드 3 구역 0 은 47~70°).** 4코어 측정 기계라 동시 묶음 4개, 모두 측정만(기본 동작 불변).
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | tilt-stages (단구역 기울기 단계별) | `feat/tilt-stages` e709deb (frame-tilt 위; tilt_stages.rs + pipeline.rs 진단 스냅숏, 기본 꺼짐, PR 없음) | experiment/tilt-stages 64d1bea, PR #182 | 위 방향 오차 회전 평균 직후 0.450 → BA 뒤 0.473°(BA 기여 0.023°). 회전 집합 자체 좌표계 기준 0.136/0.248°, 회전·중심 좌표계 어긋남 0.381/0.425°. 중심 기울기 0.810° = GPS 0.786°. BA 반복 3/8/15 회 위 방향 0.254/0.440/0.473°(단조 아님). 정밀 BA 시작점 위치 다듬기 스위치: 위 방향 0.473 → 0.166°, 중심 기울기 0.810 → 0.780°(한 실행). 총괄: fmt 통과, clippy 0, `--test tilt_stages --ignored` 통과(90 s), 표 일치 |
  | preview-error-stages (구역 1 초벌 점 오차) | `feat/preview-error-stages` 4d1d39c (align-pair-diag 위; preview_error_stages.rs + pipeline.rs 환경 변수 덤프, PR 없음) | experiment/preview-error-stages 0da6a9b, PR #180 | 공통 트랙 3748 점 오차 중앙/p90: 초벌 1.31/5.92, 정답 회전+추정 위치 1.40/5.93, 추정 회전+정답 위치 0.27/0.71, 정답 포즈 0.10/0.37 m. 정렬 쌍 거리 한정: ≤30 m 중심 1.47 m 이나 축척 1.098·점 중앙 4.82 m(현재 4.24) → 기본값 제안 없음. 최적 닮음 중심 0.64 m 대 현재 정렬 5.24 m. 총괄: fmt 통과, clippy 0, `--test preview_error_stages --ignored` 통과(143 s), 수치 일치 |
  | zone2-scale (구역 2 축척 18%) | `feat/zone2-scale` d911c11 (align-pair-diag 위; zone2_scale.rs + pipeline.rs 진단 필드·덤프, PR 없음) | experiment/zone2-scale 7bc8bcc, PR #181 | 중심 축척 구역 2 위치 풀이 1.018·최종 1.019(맞음), 카메라 3대 간격 비 GPS 0.903·풀이 0.792·최종 0.848(1/0.848 ≈ 1.18). 정답 GPS: 점 정렬 축척 0.9984/1.0060. 기본 경로 축척은 위치 평균이 아니라 방향 제약 + GPS 사전항 풀이가 정함. 구역 간 스케일 검사 0.175 > 0.10 으로 잡음. verify preview_align 은 구역 0 null 값으로 비교 전에 실패. 총괄: fmt 통과, clippy 0, `--test zone2_scale --ignored` 통과(211 s), 수치 일치 |
  | up-cross-span12 (기본 경로 교차 검사, F-445 기록) | `feat/up-cross-span12` 2d3e658 (main 위; up_cross_span12.rs 만, PR 없음) | experiment/up-cross-span12 86d0d0c, PR #183 | 시드 1/2/3 구역 0·1·2 최대 0.577·0.031·0.023 / 0.617·0.026·1.386 / 69.509·0.250·1.449°, 정답 회전 9구역 0.000°. 위치 수 하한 16 이면 거짓 초과 0 이나 검사 3구역. 총괄: fmt 통과, clippy 0, `SPAN12_SEEDS=3 --test up_cross_span12 --ignored` 통과(132 s), 시드 3 행 일치 |
- 끝까지 흐름 진척: main f25e0a8 에서 전부 연결, 변화 없음(이번 변경은 측정 시험·진단뿐).
- 다음 할 일:
  1. 기본 경로 시드 3 구역 0 정밀 모델이 재투영 0.254 px 인데 위 방향이 47~70° 어긋나는 원인 — 구역별 정밀 회전을 내보내 정답과 직접 비교. verify 가 81/81 로 통과시키는 이유도.
  2. 교차 검사 적용 조건: 구역 기울기 오차와 구름을 가르는 묶음 사이 비율 판정을 구름 장면과 함께 숫자로(F-445).
  3. 구역 1 초벌 정렬 손실(0.64 → 5.24 m): 카메라 중심 쌍을 정렬 항으로 넣는 안 비교.
  4. 구역 2 축척: 같은 위치 카메라 3대 간격이 눌리는 것 — SPEC §1 은 리그로 묶지 않으므로(F-442) 간격 고정이 아닌 방안(GPS 사전항 가중·강건화)으로 다른 시드 확인. verify preview_align 의 null 처리도.
  5. 단구역 위 방향: 정밀 BA 시작점 위치 다듬기 스위치를 시드 여러 개·점 정확도와 함께 재기.
- 막힌 점:
  - 소유자 병합 필요: #102, #99(#101 포함), #97, #93, #55, #54 → #58 → #87, #90, #76(#77 포함), #89.
  - F-197(높음) 은 SPEC §3.2 개정 결정 필요. #100 은 F-442 로 보류.
  - feat/preview-realign 결정 필요(좌표계 일관성 대 현재 수치).

## 앞 회차 기록 (2026-10-06 21:15Z 시작분)
- 상태: 끝남
- 마지막 갱신: 2026-10-06T21:16Z (21:15Z 시작분)
- 이번 회차 결론: **단구역 밀집 95% 꼬리(camL)의 원인은 밀집 단계가 아니라 포즈 — 출력 좌표계 전체가 동쪽 축 둘레 약 0.75° 기울고(북-남 거리에 비례한 수직 오차), 카메라 하나에 기울기 편향이 실행마다 다르게 붙음(켬 camL, 끔 camR). 중심을 강체 정렬하면 95% 1.188 → 0.637 m, 정답 포즈로 밀집 재실행 0.351 m(카메라 차이 없음). F-199 처리(#98, 감독 닫음), F-439 처리(#99). 초벌 누적 재정렬 불일치는 사실이나 적용하면 수치가 나빠져 PR 없음.** 4코어 측정 기계라 동시 묶음 4개.
- 묶음별 결과:
  | 묶음 | 제품 | 연구 | 결과 |
  |---|---|---|---|
  | dense-caml-tail (단구역 95%) | `feat/dense-caml-tail` c3e0725 (dense.rs 진단 필드 + 무시 시험, 기본 동작 변화 없음, PR 없음) | experiment/dense-caml-tail 0c4c6e2, PR #174 | 카메라별 회전 중앙 켬 F 0.457·R 0.259·L 0.726°(기울기 편향 +0.59), 끔 F 0.456·R 0.765·L 0.362°. 깊이 지도 상대 오차 중앙 0.68~0.77%·95% 1.9~2.4%(camL 나쁘지 않음), 정답 포즈 재실행 0.39~0.44%·1.4~1.9%. 거리 정의별 95%: 수직 1.188·근사 3D 1.053·광선 깊이 1.480 m. 강체 정렬 뒤 켬 0.637(camL 0.713)·끔 0.790 m. 총괄: fmt 통과, clippy 0, `--test dense_caml_tail --ignored` 통과(93 s), 출력이 보고와 같음 |
  | align-pair-spread (정렬 잔차·F-439) | `feat/align-pair-spread` 1b8d0ad, PR #99 (base feat/preview-align-stages), 라벨 | experiment/align-pair-spread 8470572, PR #173 | 최종 초벌 좌표를 정하는 자기 구역 정렬은 구역 전체 점 쌍(구역 1 3523 쌍 중앙 1.21 m·축척 1.0024, 구역 2 1651 쌍 0.117 m·1.178), 겹침 띠 점 쌍은 미리보기 재정렬용 — "점 잔차는 작다" 서술은 구역 1 에 틀림. 사진 중심 대용: 띠 8장만 맞추면 띠 밖 중앙 3.10/최대 5.19 m·축척 +12.6%, 전체 0.71 m, 띠+전체 0.726 m. 점 쌍 좌표 진단은 제품 쪽 필요(못 함). F-439 네 항목 처리. 총괄: fmt 통과, clippy 0, `--test pipeline_poses` 2 통과·2 무시(150 s) |
  | preview-realign (초벌 누적 재정렬) | `feat/preview-realign` a8c2da1(시험)·0014241(pipeline.rs 한 줄 + `preview_final_sim`), PR 없음 | experiment/preview-realign 74922a5, PR #175 | 불일치 사실: `pipeline.rs` 최종 단계에서 정밀은 누적 `rsim`, 초벌은 자기 구역 `own` 만 받음. `rsim ∘ own` 적용 시 구역 1 초벌→정답 3.678/6.348 → 3.991/6.463 m, 초벌→정밀 3.371 → 3.558 m(나빠짐), 구역 2 변화 없음. 좌표계 일관성은 맞지만 `own` 정렬 오차가 드러남 → 채택 보류. 총괄: 다시 돌리지 않음(PR 안 엶) |
  | align-roll-check (F-199) | `feat/align-roll-check` d54592d, PR #98, 라벨 | experiment/align-roll-check 92f0a1d, PR #172 | `up_cross_check` 문턱 0.3°: 한 기체 구름 0/0.5/1/2° 어긋남 최대 0.051/0.307/0.580/1.116°, 1° 이상 20 시드 모두 초과. 총괄: fmt 통과, clippy 0, `--lib align` 31 통과(17 s). 감독 20:30 닫음 |
- 끝까지 흐름 진척: main 에서 전부 연결, 변화 없음.
- 다음 할 일:
  1. 단구역 95%: 출력 좌표계 0.75° 기울기 원인 — GPS 정렬의 위 방향(`up_from_rotations`·카메라별 기울기 편향이 위 방향을 끄는지, `up_cross_check` 를 단구역 실행에 적용), 같은 카메라 사진끼리 기울기 편향을 묶는 번들 조정 시험.
  2. 초벌 정렬: 자기 구역 정렬(구역 1 점 쌍 잔차 1.21 m, 강건 안쪽 63%) 원인 — 점 쌍 좌표·잔차 진단을 제품에 넣고, 그 뒤 누적 재정렬 적용(feat/preview-realign) 재평가.
  3. 구역 2 정렬 축척 1.178 과 구역 간 스케일 검사 관계 확인.
- 막힌 점:
  - 소유자 병합 필요: #93(2bd2213), #54 → #58 → #87, #55(→ #97 → #99), #90, #76(#77 포함), #89, #98.
  - feat/preview-realign 은 결정 필요: 좌표계 일관성(누적 변환 적용) 대 현재 수치(우연한 상쇄).
