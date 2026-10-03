# pipeline-e2e: synth → run → verify 끝까지 시험 (F-273)

## 결론
- README 의 단구역·2구역 명령을 그대로 프로세스로 돌리는 시험 `crates/cli/tests/pipeline_e2e.rs` 를 추가했다. 단구역은 기본 시험, 2구역도 일반 시험이다(부하 22 에서 run 332 초였으나 부하 약 10 에서 시험 전체 179 초라 5분 기준 안, ignore 하지 않음).
- 구역 2개에서 refined_overlap 이 실제로 판정된다(1쌍, 0.257 m, PASS). 높이 차는 하한(> 2 m, 현재 실패 유지)과 상한(실측 x 1.2)을 둘 다 단언한다.
- README 절 "합성 장면으로 한 번 돌려 보기" 의 수치 = 아래 표 = 시험 주석의 실측.
- 카메라 회전 오차는 재지 못했다: 출력 `poses.txt` 에 카메라 중심만 있고 회전이 없다(라이브러리 변경 필요, 이 묶음 범위 밖).

## 수치 표 (4코어, 부하 약 20, 제품 92429d8 기준)
| | 단구역 (120장) | 구역 2개 (240장) | 시험 상한 (단 / 2구역) |
|---|---|---|---|
| run 시간 | 92~122 초 | 332 초 (부하 22), 시험 전체 179 초 (부하 약 10) | - |
| 출력 폴더 | preview/ refined/ snapshots/ (+manifest.json) poses.txt report.json | 같음 | 존재 단언 |
| verify | 6/7 (종료 1) | 5/7 (종료 1) | 항목별 단언 |
| registered / region_images / refined_reprojection / snapshots | PASS | PASS | PASS |
| 재투영 초벌 → 정밀 | 3.026 → 0.300 px | 3.232 → 0.282 px | verify 기준 0.7 |
| preview_align | PASS (점쌍 2746, 잔차 3.932 m) | FAIL (점쌍 최소 324 < 1000, 스케일 차 8.26%, 잔차 5.130 m) | 실패 유지 + 스케일 <= 10, 잔차 < 6 |
| preview_vs_refined | FAIL (최근접 2.358, 높이 차 2.657 m) | FAIL (최근접 5.946, 높이 차 6.278 m) | 높이 차 (2, 3.2) / (2, 7.6) |
| refined_overlap | 해당 없음 | PASS (1쌍 0.257 m) | < 0.3, "해당 없음" 아님 |
| 카메라 중심 오차 중앙 / 최대 | 0.329 / 2.912 m | 0.290 / 3.115 m | 0.40 / 3.5 ; 0.35 / 3.75 |
| 정밀 점→정답 표면 중앙 / 95% | 0.475 / 1.465 m | 0.494 / 3.022 m | 0.57 / 1.76 ; 0.60 / 3.63 |
| 정밀 점 수 | 11054 | 19855 | 실측 +-20% |

상한 근거: 실측 x 1.2 를 올림(점 수는 +-20%). 느슨하게 하지 않았고 실측이 상한에 가까워지면 알 수 있는 수준이다.

## 방법
- 시험은 `synth scene 320 180` → `run scene out --stride {2|1} --span 48 --ovl 2 --max-features 800 --dense-width 96 --hfov 65 --ba-iters 15` → `verify out` 을 `CARGO_BIN_EXE_skylens-stream` 으로 실행한다(README 와 같은 옵션 열).
- 카메라 중심 오차: `truth/cameras.txt` 의 C = -Rᵀt 를 `truth/origin.txt` 와 `gps.txt` 첫 줄의 차(geodetic_to_enu)만큼 옮겨 `poses.txt` 와 비교.
- 점 오차: `refined/*.ply` 전부, 정답 표면은 `Scene::surface_height`(수직 거리 근사, 최근접 점 거리가 아님이라 경사 면에서는 약간 낮게 나옴).
- verify 항목 판정은 stdout 표를 파싱한다.
- 이전 시험(`pipeline.rs`)의 주석 "표면 거리 0.342 m" 는 이번 측정(같은 점군에서 중앙 0.475 m)과 맞지 않는다. 원인은 확인하지 못했다(이전 주석이 옛 설정의 값일 가능성).

## 남은 문제
- `poses.txt` 에 회전이 없어 회전 오차(°) 를 재지 못한다. 출력에 회전을 쓰면 시험에 추가 가능.
- 2구역 시험은 부하가 크면(20 이상) 5분을 넘는다. 측정 기계가 붐비면 `#[ignore]` 를 고려.
- preview_vs_refined(높이 차 2.657 / 6.278 m), 2구역 preview_align(점쌍 324 < 1000) 은 제품 쪽 미달로 남아 있다. 시험이 '실패'로 고정하므로 고치면 단언을 바꿔야 한다.
- 시험은 임시 폴더에 장면을 다시 만들어 단구역 약 2분, 2구역 약 5.5분이 든다.

## 제품 브랜치·커밋
feat/pipeline-e2e (기준 feat/pipeline-merge-1717 92429d8): 634844d, 5de63af (2구역 시험 ignore 해제)
