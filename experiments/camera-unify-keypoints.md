# P19 camera-unify — 카메라 형 하나로 합치기, 특징점 좌표 규약 통일 (F-016, F-023)

## 결론
- **F-016**: 특징점 `Keypoint::x, y` 를 카메라 모형과 같은 연속 좌표(화소 (i, j) 중심 = (i + 0.5, j + 0.5))로
  내보낸다. 검출기 내부는 화소 번호로 계산하고 출력 직전에 + 0.5 한다(옥타브 축소가 짝수 화소를 고르므로
  옥타브 o 의 표본 i 는 원본 화소 i·2^o 의 중심). `describe()` 도 같은 연속 좌표를 받는다.
  카메라 규약으로 렌더한 덩어리 격자에서 보정 없이 평균 치우침 (0.0011, 0.0013) px, 최대 0.015 px.
- **F-023**: 카메라 형을 `camera::Intrinsics` 하나로 합쳤다. 필드 `dist: Distortion` 이 생겼고 계수 0 이면 핀홀.
  `to_pixel`·`Camera::project` 는 왜곡을 적용하고 `to_normalized`·`unproject`·`Camera::unproject` 는 왜곡을
  되돌린다. 따라서 매칭·두 시점 경로가 `to_normalized` 를 그대로 쓰면 번들 조정 초기값에도 왜곡 보정된 정규 좌표가
  들어간다(왜곡 0 이면 이전과 비트 단위로 같은 값). `DistortedIntrinsics` 는 번들 조정이 고치는 8개 값 묶음으로
  남겼고(`Intrinsics::params`/`from_params`/`From`), 다른 모듈은 고치지 않았다.
- 다른 모듈 시험 하나가 깨진다: `matching::tests::ransac_on_cross_camera_views`. 원인은 그 시험이 특징점 좌표에
  손으로 + 0.5 를 더하는 것(matching.rs:990)이라 이제 반 화소 이중 보정이 된다. 같은 수동 보정이 two_view.rs:1118
  에도 있다(그 시험은 기준 안이라 통과). 두 줄에서 `+ 0.5` 를 지우면 된다(이 묶음 범위 밖).

## 수치
| 검증 | 기준 | 결과 |
|---|---|---|
| 덩어리 24개(σ 3 px, 부화소 위치 0~0.9) 검출 평균 치우침 x, y | 성분마다 < 0.05 px (규약 어긋남 0.5 px 의 1/10) | 0.0011, 0.0013 px |
| 같은 시험 최대 위치 오차 | < 0.1 px (부화소 정밀화 한계) | 0.0147 px |
| 같은 시험 검출 수 | ≥ 22/24 | 23/24 |
| 왜곡 카메라(k1 −0.12, k2 0.03, p1 0.001, p2 −0.0015, 1920×1080, 화각 70°) 투영 → `to_normalized` 왕복, 40점 | < 1e-9 px | 통과 |
| 같은 점의 왜곡 무시 선형 정규화 오차(가장자리) | > 20 px (r³·k1·f ≈ 69 px 추정) | 통과 |
| `Camera` 깊이 역투영 → 투영 왕복(왜곡 포함) | < 1e-9 px | 통과 |
| 왜곡 0 이면 핀홀과 일치 | < 1e-12 | 통과 |

## 방법
- `crates/core/src/camera.rs`: `Intrinsics { fx, fy, cx, cy, width, height, dist }`.
  새 함수 `unproject -> Option`, `pinhole_normalized`(왜곡 무시 선형, 따로 이름 붙임), `is_pinhole`,
  `params`, `from_params`. `with_distortion` 은 이제 `Intrinsics` 를 돌려준다. 모듈 문서 갱신.
- `to_normalized` 는 뉴턴 역왜곡이 수렴하지 않는 드문 경우 선형 값을 돌려준다(실패를 구별하려면 `unproject`).
- `crates/core/src/features.rs`: 출력 좌표 + 0.5, `describe` 입력 − 0.5, 시험의 렌더·정답을 카메라 규약으로
  바꾸고 수동 `+ 0.5` 를 지움. 회전 텍스처 시험은 화소 번호로 w/2 둘레를 돌리므로 연속 좌표 회전 중심을
  w/2 + 0.5 로 적었다.
- 측정: 4 코어 측정 기계, 릴리스 빌드.

## 남은 문제
- matching.rs:990·two_view.rs:1118 시험의 수동 `+ 0.5` 제거(맡은 파일 밖). 제거 전까지
  `ransac_on_cross_camera_views` 실패.
- matching.rs:627 이 `to_normalized` 를 아핀으로 보고 세 점으로 K⁻¹ 를 되살린다. 왜곡이 0 이 아니면 틀리므로
  왜곡 있는 카메라에서는 `pinhole_normalized` 또는 점별 `to_normalized` 로 바꿔야 한다.
- F-023 확인 기준(왜곡 카메라로 렌더한 두 장의 자세 오차 시험)은 합성 렌더러가 아직 왜곡 없이 렌더해서 못 했다.
  렌더러가 `Camera::unproject`(이제 왜곡 포함)를 쓰므로 synth 쪽에 왜곡 계수 설정만 열면 된다.
- 덩어리 격자 시험에서 24개 중 하나((226.13, 140.00))가 검출되지 않는다. 좌표 규약과 무관한 극값 정밀화 문제로 보이며 따로 본다.
- README 예시(+0.5 보정)는 이 묶음 범위 밖이라 그대로다. 이제 보정 없이 `kp.x as f64` 가 맞다.

## 제품 저장소
- 브랜치 `feat/camera-unify-keypoints`, 커밋 155d613 (main 08d5248 기준). 같은 시기 `feat/camera-unify` 에 들어간 e0e7e43 과 camera.rs 가 겹쳐 합치지 않았다. 둘 중 하나로 정리할 때 features.rs 의 + 0.5 출력 규약과 덩어리 격자 시험은 그대로 옮길 수 있다.
- 시험: fmt 통과, clippy 통과, 코어 시험 95 통과·1 실패(ransac_on_cross_camera_views, 위 원인)·2 무시.
