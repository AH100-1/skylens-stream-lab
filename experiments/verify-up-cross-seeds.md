# up_cross 문턱 10° 의 거짓 실패 점검 (시드 4·5 기본 경로)

## 결론
- 시드 4 는 up_cross 가 PASS(경고 수준)이고 구역별 최대가 0.559°·1.477°·2.135° 로 모두 10° 이하다. 거짓 실패 없음.
- 시드 5 는 up_cross 가 PASS 이지만 '측정값 없음'(세 구역 diff_deg 전부 null)이라 문턱 10° 를 검증한 것이 아니다. 시드 5 는 registered 가 54/81 로 FAIL 이라 카메라 묶음이 모자란 탓으로 보인다(원인 분리는 안 함). 따라서 시드 5 는 '거짓 실패 없음'이 아니라 '판정 불가(건너뜀)'로 적는다.
- 확인 기준("시드 3 구역 0 외 모든 구역 ≤ 10°")은 측정된 구역에 한해 시드 1~4 에서 충족. 시드 5 는 측정 구역이 없어 미충족(미확정).
- up_cross 만 실패하는 시드는 없다. 시드 4 는 preview_vs_refined 하나만 FAIL(7/8), 시드 5 는 registered·preview_vs_refined FAIL(6/8). 두 시드 모두 up_cross 는 통과라 거짓 실패는 아니다. 시드 3 은 앞선 기록대로 preview_align·preview_vs_refined 도 FAIL(5/8).

## 수치 표
기본 경로(인자 없는 synth → run → verify, 시드만 변경), 구역 3개. 구역별 최대 어긋남(°).

| 시드 | verify | 구역 0 | 구역 1 | 구역 2 | up_cross | 다른 FAIL 항목 | up_cross 만 실패 |
|---|---|---|---|---|---|---|---|
| 1 | 미측정(이번) | 0.577 | 0.031 | 0.023 | PASS(경고) | 앞선 기록 | 아니오 |
| 2 | 미측정(이번) | 0.617 | 0.026 | 1.386 | PASS(경고) | 앞선 기록 | 아니오 |
| 3 | 5/8 (앞선 기록) | 69.509 | 0.250 | 1.449 | FAIL | preview_align(25.41%), preview_vs_refined | 아니오 |
| 4 | 7/8 | 0.559 | 1.477 | 2.135 | PASS(경고) | preview_vs_refined | 아니오 |
| 5 | 6/8 | 미측정 | 미측정 | 미측정 | PASS(측정값 없음) | registered, preview_vs_refined | 아니오 |

시드 1·2 의 verify 전체 결과는 이번 시험으로 돌리지 못했다(미측정). 구역별 값은 verify-up-cross 노트에서 가져왔다.

시드 4·5 항목별(verify 표):

| 항목 | 시드 4 | 시드 5 |
|---|---|---|
| registered | PASS 81/81, 81/81 | FAIL 54/81, 54/81 |
| region_images | PASS | PASS |
| refined_reprojection | PASS 0.244 px | PASS 0.232 px |
| preview_align | PASS 스케일 차 3.16%, 잔차 0.394 m | PASS 4.54%, 0.321 m |
| preview_vs_refined | FAIL 최근접 2.635 m, 높이 차 2.918 m | FAIL 최근접 3.234 m, 높이 차 3.267 m |
| refined_overlap | PASS 0.111 m | PASS 0.091 m |
| snapshots | PASS | PASS |
| up_cross | PASS 0.559/1.477/2.135° | PASS 측정값 없음 |

## 방법
- 시험 `crates/cli/tests/up_cross_seeds.rs`(무시 시험). 환경 변수 `UPX_SEEDS`(쉼표 구분, 기본 "4,5")의 시드마다 시드만 바꾼 기본 설정 장면을 쓰고 `run` → `verify` 를 돌려 항목별 판정·측정값, report.json 의 구역별 diff_deg 최대, 'up_cross 만 실패' 여부를 출력한다. 구역 최대가 모두 null 이면 미측정으로 표시. 시드 3 구역 0 외 측정된 구역이 10° 초과면 단언 실패. synth CLI 에 시드 인자가 없어 장면 쓰기는 라이브러리 호출로 했다.
- 실행: `UPX_SEEDS=4,5 cargo test --release -p skylens-stream --test up_cross_seeds -- --ignored --nocapture`. 4코어 측정 기계, 다른 작업과 동시 실행, 시드 4 run 포함 약 10 분, 시드 5 단독 7.8 분.
- 시드 4 수치는 첫 실행의 verify 표 출력에서 가져왔다. 이 실행은 report.json 파서가 공백 있는 JSON 을 못 읽어 시험이 실패했고(시드 4 구역별 값은 verify 표의 up_cross 줄에서 확인), 파서를 고친 뒤 시드 5 만 다시 돌렸다. 시드 4 는 고친 시험으로 재실행하지 못했다.
- 마지막 수정(전부 null 구역을 미측정으로 구분)은 컴파일·clippy·fmt 만 확인하고 다시 돌리지 않았다.
- 검사: `cargo fmt --all --check` 통과, `cargo clippy --all-targets -- -D warnings` 통과.

## 남은 문제
- 시드 5 의 up_cross 가 실제로 문턱 안인지 모른다. 등록 54/81 이라 묶음이 모자라 diff_deg 가 null 인데, 이 경우 verify 가 '건너뜀'으로 통과시키는 것이 맞는지 판단이 필요하다(실패 구역을 숨길 수 있음).
- 시드 1·2 의 항목별 verify 결과와 시드 4 재실행(고친 시험)이 미측정이다.
- 시드 4·5 모두 preview_vs_refined 가 FAIL 이라 기본 경로가 시드 1 외에는 8/8 이 아니다. 별도 항목으로 볼 문제.

## 제품 브랜치·커밋
- feat/verify-up-cross-seeds (시험 파일만 추가)
- 커밋 5f072f0
