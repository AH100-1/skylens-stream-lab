# 기본 인자 synth → run → verify (F-343)

## 결론
원인은 기본값 불일치였다. DatasetConfig 기본 span 이 12 라서 기본 장면(960x540, 240장, stride 3 → 27위치 81장)이 3구역으로 쪼개졌고,
카메라 사이 겹침이 12~40 위치 떨어진 짝에서만 생기므로 구역마다 등록이 빠졌다(61/81). 기본 span 을 48 로 올려 1구역이 되자 7/7, 종료 코드 0.
시험(pipeline_e2e)은 --span 48 을 명시해 이 불일치를 잡지 못했다.

## 수치
| 항목 | 고치기 전 (span 12, 3구역) | 고친 뒤 (span 48, 1구역) |
|---|---|---|
| registered | FAIL 초벌 61/81, 정밀 61/81 | PASS 81/81, 81/81 |
| region_images | PASS | PASS 1구역 |
| refined_reprojection | PASS 0.237 px | PASS 0.256 px |
| preview_align | PASS 점쌍 최소 2166, 스케일 차 7.18% | PASS 점쌍 최소 13388, 0.00%, 잔차 0.459 m |
| preview_vs_refined | FAIL 최근접 > 6 m, 높이 차 inf | PASS 최근접 0.431 m, 높이 차 0.323 m |
| refined_overlap | PASS 0.070 m | PASS 해당 없음(구역 1개) |
| snapshots | PASS | PASS 2단계, final 22987 |
| 합계 | 5/7, 종료 1 | 7/7, 종료 0 |

벽시계(4코어 측정 기계, 다른 빌드 3개 동시 부하): run 고치기 전 2분 38초, 고친 뒤 2분 20초. 새 시험 전체 423초(부하 중).

## 방법
release 빌드로 기본 synth → run → verify 재현, 기본 span 만 48 로 바꿔 재실행.

## 남은 문제
- preview_vs_refined 의 '높이 차 중앙 inf' 는 등록이 빠진 구역에서 짝이 없을 때 나온 것으로 보이며, 빈 구역 처리 자체는 따로 확인하지 못했다.
- 320x240 장면(3/7, refined_overlap 41 m)은 이 묶음에서 다루지 않았다.
- 구역이 둘 이상 필요한 큰 장면은 --span 을 직접 줘야 한다.

## 제품 브랜치·커밋
feat/default-path @ a498195
