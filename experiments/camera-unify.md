# camera-unify — 픽셀 ↔ 정규 좌표 경로를 왜곡 포함 하나로 (F-023)

## 결론
채택 제안. `Intrinsics` 가 왜곡 계수 `dist`(k1,k2,p1,p2, 기본 0)를 갖고, 픽셀 → 정규 좌표(역왜곡 포함)와
정규 좌표 → 픽셀(왜곡 적용)은 `distortion::pixel_to_normalized`·`normalized_to_pixel` 두 함수만 거친다.
`Intrinsics::to_normalized`·`to_pixel`·`unproject`, `Camera::project`·`unproject`, 번들 조정 형태
`DistortedIntrinsics::unproject`·`project_camera` 가 모두 이 둘을 부르므로 매칭·두 시점·번들 조정이
같은 관측을 같은 규약으로 정규화한다. 왜곡 0 이면 기존 핀홀 식과 비트 단위로 같다(`==` 비교 시험).
왜곡 렌즈(k1 −0.1, k2 0.01, p1 1.2e-3, p2 −0.9e-3)로 투영한 두 장은 핀홀과 같은 자세 기준을 통과하고,
같은 영상을 왜곡을 무시하고 정규화하면 기준을 넘는다.
기존 공개 API 는 그대로이고 다른 파일 호출부는 바뀌지 않았다.

## 수치
두 시점: 960×540, 화각 70°, 고도 ~40 m, 기선 3 m, 점 200개(두 영상 안), σ 0.5 px, 시드 1–100,
8점 + `refine_relative_pose`(50 반복). 기준은 two_view 의 σ0.5 정밀화 기준 그대로.

| 경우 | 회전 오차 중앙값·90%·최대 (도) | 방향 오차 중앙값·90%·최대 (도) | 관측 불가 | 기준 |
|---|---|---|---|---|
| 핀홀 렌즈, 핀홀 정규화 | 0.093·0.175·0.327 | 0.63·1.15·2.25 | 0 | 통과 |
| 왜곡 렌즈, 새 정규화 경로 | 0.092·0.156·0.260 | 0.62·1.03·2.30 | 0 | 통과 |
| 왜곡 렌즈, 왜곡 무시(대조) | 0.178·0.313·0.521 | 1.42·1.98·2.80 | 0 | 실패(회전 전 분위·방향 중앙값) |
| 기준(상한) | 0.14·0.24·0.4 | 1.05·2.4·3.0 | 0 | |

| 검증 | 기준 | 결과 |
|---|---|---|
| 왜곡 카메라 픽셀 → 세계 → 픽셀 왕복, 1920×1080 격자 33×19(모서리 포함) | < 1e-9 px | 최대 1.27e-11 px |
| 세계 → 픽셀 → 정규 좌표 왕복(200점, |x|≤0.65, |y|≤0.37) | fx·오차 < 1e-9 px | 정규 좌표 최대 8.0e-15 |
| `Camera::project_with_jacobian` vs 중앙 차분(h=1e-6), 3점(모서리 포함) × 17 열 × 2 행 | 상대 < 1e-6 | 최대 8.2e-8 |
| 왜곡 0: `to_normalized`·`to_pixel`·`unproject`·`DistortedIntrinsics` 가 핀홀 식과 같음 | 비트 단위(==) | 통과 |
| 1920×1080 화각 70° 모서리에서 왜곡 유무 차 | > 5 px | 통과 |

## 방법
- `Distortion::is_zero`, `undistort_best`(마지막 뉴턴 반복값과 수렴 여부). 계수 0 이면 역왜곡을 건너뛴다.
- `Intrinsics::to_normalized` 는 총함수 유지(수렴 실패 시 마지막 반복값), 수렴 여부가 필요하면 `Intrinsics::unproject`(Option).
- 새 경로: `Intrinsics::distorted(dist)`(계수 바꾼 Intrinsics), `Intrinsics::to_distorted()`(번들 조정 형태),
  `DistortedIntrinsics::with_size(w, h)`, `Camera::project_with_jacobian(x)`.
- 대조 실험(왜곡 무시)은 시험이 왜곡에 민감함을 보이려고 넣었다.

## 남은 문제
- 합성 렌더러는 여전히 왜곡 없이 렌더한다(`from_hfov` 기본 계수 0). 왜곡 렌더는 synth 쪽 일.
- 매칭·두 시점이 `to_normalized` 를 쓰므로 실제 렌즈 계수를 넣으면 자동으로 보정되지만, 계수를 어디서 읽어 `Intrinsics::dist` 에 넣을지(설정·자기 보정)는 정하지 않았다.
- 역왜곡은 왜곡이 단조인 범위에서만 보장한다. 강한 배럴 왜곡 가장자리에서 `to_normalized` 는 수렴하지 않은 값을 돌려줄 수 있다(`unproject` 는 None).

## 제품 저장소
- 브랜치 `feat/camera-unify` (AH100-1/skylens-stream-rs), 커밋 e0e7e43
