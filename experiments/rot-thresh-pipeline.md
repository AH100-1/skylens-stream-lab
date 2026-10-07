# 종류별 문턱·GM IRLS·덩어리 잇기: 파이프라인 단위 비교 (시드 3 구역 0)

## 결론
- 파이프라인에서 직접 잰 것은 끔과 덩어리 잇기뿐이다. 종류별 문턱(`SKYLENS_ROT_CLASS_THRESH=1`)과 GM IRLS(`SKYLENS_ROT_GM=1`)는 마감까지 실행이 끝나지 않아 **미측정**이다(아래 표 참고, 측정은 4 코어 측정 기계에서 세 개를 동시에 돌려 느려졌다).
- 측정된 범위에서 기본 켬 후보는 덩어리 잇기다: 회전 평균 직후 L 오차 84.72° → 0.78°, 정밀 BA 뒤 L 74.30° → 0.14°, verify 5/8 → 7/8.
- 종류별 문턱·GM 과의 우열은 숫자로 가를 수 없다. 시드 1 표면 오차도 미측정.

## 수치 표 (시드 3, 기본 경로, 구역 0 의 68 대)
| 설정 | 회전 평균 직후 F/R/L 오차 중앙° | 정밀 BA 뒤 F/R/L 회전 중앙° | verify |
|---|---|---|---|
| 끔 | 1.27 / 0.33 / 84.72 | 3.46 / 1.10 / 74.30 | 5/8 (preview_align, preview_vs_refined, up_cross 실패) |
| 다리(덩어리 잇기) | 0.81 / 0.20 / 0.78 | 0.96 / 0.55 / 0.14 | 7/8 (preview_vs_refined 실패) |
| 종류별 문턱 | 미측정 | 미측정 | 미측정 |
| GM IRLS | 미측정 | 미측정 | 미측정 |

시드 1 표면 오차 중앙: 네 설정 모두 미측정.

## 방법
- 시험 `crates/cli/tests/rot_thresh_pipeline.rs`(기존 `zone0_up.rs` 측정을 설정별로 반복하게 고침). 합성 장면 시드 3 을 `run` 으로 돌리고 `verify` 판정 수와, 회전 평균 직후 덤프(`SKYLENS_DUMP_ROTAVG`)·정밀 BA 뒤 포즈를 정답과 비교. 오차는 절반 덜어내기 Q 로 맞춘 묶음별 중앙.
- 환경 변수 `ROT_SETTINGS`(off,bridge,class,gm)로 설정을 고른다.

## 남은 문제
- 종류별 문턱·GM 의 파이프라인 측정, 시드 1 표면 오차(덩어리 잇기 보류 사유) 비교.
- 덩어리 잇기에서도 verify 의 preview_vs_refined 는 계속 실패(구역 간 정렬 쪽 문제로 보임).

## 제품 브랜치·커밋
제품 feat/rot-thresh-pipeline @ 09b7214 (pipeline.rs 환경 변수 연결 `SKYLENS_ROT_CLASS_THRESH`·`SKYLENS_ROT_GM`, 기본 끔).
