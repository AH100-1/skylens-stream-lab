# verify 에 카메라 묶음 위 방향 일치 항목 추가

## 결론
- `skylens verify` 에 여덟 번째 항목 `up_cross`(카메라 묶음 위 방향 일치)를 넣었다. report.json 의 `up_cross_check.regions[].diff_deg` 에서 구역별 최대를 구해, 10° 를 넘는 구역이 있으면 FAIL, 0.3° 초과 10° 이하는 경고(통과), 0.3° 이하는 통과. report.json 이 없거나 `up_cross_check` 가 없으면 '건너뜀'(통과로 셈, 측정값 칸에 표시). 판정 불가(종료 코드 2)와는 따로 다뤄 종료 코드에 영향이 없다.
- 기본 경로 시드 3 을 끝까지 돌려 확인했다: 구역 0 이 최대 69.509° 로 FAIL 로 잡힌다(결과 5/8). 구역 1·2 는 0.250°·1.449° 로 경고 수준.
- 문턱 10° 의 근거: 정상 실행 구역별 최대 1.449° 이하, 한 기체 짐벌 구름 1~2° 장면 1.1~1.3°, 고장 47° 이상. 정상 최대의 약 7배, 고장 최솟값의 약 1/4.

## 수치 표
기본 경로(960×540, 구역 3개) 구역별 최대 어긋남(°).

| 시드 | 구역 0 | 구역 1 | 구역 2 | 판정 |
|---|---|---|---|---|
| 1 | 0.577 | 0.031 | 0.023 | 경고(통과) |
| 2 | 0.617 | 0.026 | 1.386 | 경고(통과) |
| 3 | 69.509 | 0.250 | 1.449 | FAIL(구역 0) |

시드 3 끝까지 실행(4코어 측정 기계, 다른 작업과 같이 run 247 s) 뒤 verify:

| 항목 | 결과 | 측정값 |
|---|---|---|
| registered | PASS | 81/81, 81/81 |
| region_images | PASS | 3개 구역 일치 |
| refined_reprojection | PASS | 0.253 px |
| preview_align | FAIL | 스케일 차 25.41% (기준 10%) |
| preview_vs_refined | FAIL | 최근접 3.151 m, 높이 차 4.360 m |
| refined_overlap | PASS | 0.207 m |
| snapshots | PASS | |
| up_cross | FAIL | 구역 0 69.509° |

## 방법
- 형식은 pipeline 의 `UpCrossReport::to_json` 을 읽기만 했다(`regions[].region`, `diff_deg` 배열, null 허용). 구역마다 `diff_deg` 의 최댓값을 취해 구역별로 문턱과 비교한다. 문턱 `UP_CROSS_FAIL_DEG` = 10°, 경고 `UP_CROSS_WARN_DEG` = 0.3° 는 verify.rs 의 상수이고 근거는 주석에 있다. 값이 모두 null 인 구역은 건너뛰고, 모든 구역이 그러면 '측정값 없음'(통과). 음수·비유한·문자열 값, regions 없음 같은 형식 오류는 기존 항목처럼 FAIL. report.json 자체가 깨졌으면 기존 방식대로 FAIL.
- 시험: core 단위 시험(손으로 만든 report: 통과, 1.386° 경고, 정확히 10° 통과·10.001° FAIL, 69.509° FAIL, 항목 없음 건너뜀, 전부 null, 형식 오류)과 CLI 시험(`up_cross_item_in_cli`: 통과 8/8, 경고 종료 0, FAIL 종료 1 7/8, 건너뜀 종료 0). 기존 verify 시험의 항목 수 표기만 7→8 로 바꿨다(8/8, 0/8, report 없음 5/8).
- 시험 명령: `cargo test --release -p skylens-core --lib verify` 14개 통과, `cargo test --release -p skylens-stream --test verify` 25개 통과(0.81 s).

## 남은 문제
- 기본 경로 시드 3 은 up_cross 외에 preview_align(스케일 차 25.41%)·preview_vs_refined 도 FAIL 이다. 구역 0 모델 붕괴가 초벌 정렬에도 영향을 준 것으로 보이나 원인 분리는 하지 않았다. 따라서 "81/81 통과" 는 등록 항목 얘기이고 verify 전체로는 이번 측정에서 이미 5/8 이었다(이 점은 의뢰 설명과 다르다).
- 시드 1·2 의 0.5~1.4° 경고는 모델 오차(거짓 양성)라 구름과 구분되지 않는다(이전 노트 참고).
- default_path 시험(`crates/cli/tests/default_path.rs`)의 "7/7" 표기를 "8/8" 로 바꾸고 돌려 통과했다(시드 1, 1 개, 648 s — 다른 작업과 동시 실행).

## 제품 브랜치·커밋
- feat/verify-up-cross `102d5f0`
