# 삼각측량 거절 시험 — SmallRayAngle·BehindCamera 분기 확인

## 결론
피드백 F-215 의 내용은 제품 main(32a5da4)에 이미 반영되어 있다(a5cb96e 에서 들어옴). 이번에 코드를 고치지 않고 현재 시험이 두 분기를 실제로 잡는지만 변이로 확인했다. 두 변이 모두 해당 단언에서 실패했고 원래대로 되돌렸다. 제품 쪽 새 커밋은 없다.

## 수치 표
| 변이(임시, 커밋하지 않음) | 결과 | 실패 위치 |
|---|---|---|
| 작은 각 판정 줄 무력화 (`if false && max_angle < min`) | 시험 실패 | `SmallRayAngle` 단언(257행) |
| 깊이 검사 제거 (`reprojection(..).ok_or(BehindCamera)` → `unwrap_or(0.0)`) | 시험 실패 | `BehindCamera` 단언(272행) |
| 변이 없음 | `triangulation` 필터 4개 통과 | - |

## 방법
- `look()` 은 f × z 가 거의 0 이면 위쪽 벡터를 y 로 바꾼다. 시험은 네 카메라(a, b, c, d) 자세 행렬과 이동이 모두 유한함을 단언한다.
- 작은 각 경우: 기선 0.3 m, 거리 30 m(약 0.6°). `assert_eq!(unwrap_err(), SmallRayAngle)`.
- 뒤쪽 경우: 카메라 c 는 +x 쪽을 비스듬히 보고 점 (-25,3,0) 은 c 기준 깊이가 음수, d 기준 양수임을 시험 안에서 먼저 단언한 뒤 `BehindCamera` 를 단언. 한 시점 입력은 `TooFewViews`.
- 시험 명령: `cargo test --release -p skylens-core --lib triangulation` (4 코어 측정 기계).

## 남은 문제
- 변이는 두 줄만 확인했다. `LargeReprojection`·`Degenerate` 분기는 같은 방식의 변이 확인을 하지 않았다.

## 제품 브랜치·커밋
- 제품 feat/triangulation-reject-test 는 origin/main 32a5da4 와 같다(변경 없음). 해당 시험이 들어온 커밋은 a5cb96e.
