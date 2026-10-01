# T00 scaffold — 워크스페이스·핵심 타입·PLY 입출력

## 결론
채택 제안. cargo 워크스페이스(`crates/core`, `crates/cli`)를 만들고 카메라(핀홀)·자세 규약과
SPEC §2 형식의 이진 PLY 입출력을 넣었다. PLY 쓰기→읽기 왕복이 비트 단위로 일치하고,
`cargo fmt --check`, `cargo clippy --all-targets -D warnings`, `cargo test --release` 모두 통과한다.

## 수치
| 검증 | 기준 | 결과 |
|---|---|---|
| PLY 왕복 (10,000점) | 비트 일치 | 일치 (`assert_eq!`) |
| PLY 데이터 크기 (1,234점) | 점당 27바이트 = 33,318B | 33,318B |
| 투영↔역투영 왕복 (50점, 1920×1080, 깊이 5~54m) | < 1e-9 px | 통과 |
| 카메라 중심 왕복 C → t → C | < 1e-12 m | 통과 |
| 화각 90° 초점거리 | f = w/2 | 500.0 (w=1000) |
| 반대칭 행렬 = 외적 | < 1e-12 | 통과 |
| CLI `ply-info` 점 개수 | 42 | 42 |

테스트 총 15개(core 13, cli 통합 2).

## 방법
- 의존성: `nalgebra 0.33` 하나 (범용 선형대수; 회전·벡터·행렬 타입).
- 카메라 규약: X_cam = R·X + t, 중심 C = −Rᵀt, 카메라 축은 x 오른쪽·y 아래·z 앞.
- PLY 쓰기: `binary_little_endian`, x y z nx ny nz (float32) + red green blue (uint8).
- PLY 읽기: vertex 원소의 스칼라 속성을 이름으로 찾으므로 순서가 달라도, double 이어도 읽는다.
  법선·색이 없으면 0. ASCII·빅엔디언·잘린 파일은 오류.
- CLI: `skylens-stream ply-info <파일>` → 점 수와 NaN 여부 출력.

## 남은 문제
- 왜곡 모델(k1,k2,p1,p2)과 야코비안은 T02.
- PLY 읽기는 vertex 가 첫 원소여야 한다. 다른 원소(면 등)가 앞에 오는 파일은 지금은 거부한다.

## 제품 저장소
- PR AH100-1/skylens-stream-rs#1 (브랜치 `feat/scaffold`)
