# P04 io-robustness — 입력 검증·좌표 규약·문서 정리

## 결론
채택 제안. 잘못된 입력이 프로세스를 죽이던 세 경로(PLY 헤더의 거대한 점 개수, `synth` 크기 인자, 길이가 틀린 RGB 버퍼)를
모두 오류 반환·종료 코드로 바꿨다. 100바이트짜리 PLY(`element vertex 10000000000`)는 이제 120 GB 할당 시도 대신
`InvalidData`("vertex 데이터가 잘림")로 끝나고 `ply-info` 는 종료 코드 1 을 낸다. `synth` 는 `[]` 또는 `[폭 높이]`(16..=8192)만 받고
그 밖은 사용법과 종료 코드 2. 특징점(화소 번호) ↔ 카메라(화소 중심 +0.5) 규약은 `Intrinsics::index_to_normalized` 로 명시하고
README 두 절의 예시를 고쳤다. 검출기 출력 규약 자체를 바꾸는 일은 이 묶음 범위 밖이라 남겼다.

## 수치
| 검증 | 기준 | 결과 |
|---|---|---|
| `element vertex 10000000000` + xyz float, 데이터 0 바이트 | Err(InvalidData), 할당 없음 | 통과("잘림") |
| `element vertex 1537228672809129302` (×12 가 usize 넘침) | Err(InvalidData) | 통과("너무 큼") |
| 개수가 u64 를 넘는 헤더 | Err(InvalidData) | 통과(해석 실패) |
| 3점 헤더 + 35 바이트 / 36 바이트 | Err / 3점 | 통과 |
| CLI `ply-info big.ply` | 종료 코드 1, 패닉 0 | 통과 |
| CLI `ply-info` 4점 헤더 + 47 바이트 | 종료 코드 1, "48 바이트 필요" | 통과 |
| CLI `synth out 0 0` / `synth out 64` / `synth out 64 48 5` | 종료 코드 2, 패닉 0, 출력 폴더 미생성 | 통과 |
| CLI `synth` 15·8193·음수·문자 | 종료 코드 2 | 통과 |
| `try_from_rgb(10,10,[0;30])` | Err(len 30, 채널 3) | 통과 |
| 회색조·RGB·RGBA 같은 밝기 | 차 < 1e-6 | 통과 |
| `index_to_normalized((319,239))`, 640×480 | 주점에서 −0.5 px | 오차 < 1e-12 |
| 왜곡 0 `DistortedIntrinsics` vs `Intrinsics` 정규화 | < 1e-12 | 통과(20점) |

## 방법
- PLY 읽기: `count.checked_mul(stride)` 로 크기를 구하고, `take(total).read_to_end` 로 실제로 있는 바이트만 읽는다.
  초기 용량은 min(total, 16 MiB) 이므로 헤더의 개수만으로는 큰 할당이 일어나지 않는다. 읽힌 길이가 모자라면 `InvalidData`.
- `GrayImage::try_from_rgb/try_from_rgba/try_from_gray` 가 `폭×높이×채널` 을 넘침 검사 포함으로 대조해 `ImageSizeError` 를 돌려준다.
  기존 `from_rgb` 는 시그니처를 유지하고(다른 모듈 호출부 파급 없음) 길이가 틀리면 메시지와 함께 패닉한다.
- CLI `synth`: 인자 패턴을 `[]`·`[w, h]`·그 밖으로 나눠 검사. 하한 16 은 특징 검출 최소 크기, 상한 8192.
- 카메라 문서: 연속 픽셀 좌표(화소 중심 = i + 0.5) 규약, 왜곡 없는 `Intrinsics` 는 계수 0 인 `DistortedIntrinsics` 의 특수형이며
  `with_distortion` 으로 바꾼다는 것, 실제 렌즈 영상은 `DistortedIntrinsics::unproject` 로 정규화한다는 것을 모듈 문서에 적었다.
- 제품 주석의 깨진 유도 문서 경로 2곳(geo.rs, distortion.rs)을 짧은 식 설명으로 바꿨다. 유도는 이 저장소 `derivations/` 에 있다.
- README: 입력 구조를 합성 출력과 같은 평평한 `images/camF_0000.jpg` 로, `ply-info` 설명을 실제 출력(`points`, `nan`)으로,
  필요 Rust 를 1.88 로, 특징점 예시에 +0.5 보정과 `try_from_rgb` 를 넣고 한국어·English 절을 줄 단위로 맞췄다.
- 측정: 4 코어 측정 기계, `cargo test --release`.

## 남은 문제
- F-016 핵심: `detect_and_describe` 가 여전히 화소 번호 규약으로 내보낸다. 검출기 쪽에서 +0.5 로 바꾸거나 `Keypoint::pixel()` 을 두고
  matching·two_view·features 시험의 수동 `+ 0.5` 를 지우는 일은 해당 파일을 맡은 작업에서 해야 한다. 가우시안 덩어리 < 0.1 px 시험도 그때.
- F-020: 작업공간 `Cargo.toml` 에 `rust-version = "1.88"` 을 넣는 일은 이 묶음 파일 범위 밖이라 README 문구만 1.88 로 고쳤다.
  `cargo +1.88 build` 확인도 하지 못했다(측정 기계에 해당 도구 사슬 없음).
- F-023: 형 통합은 하지 않았다. 매칭·두 시점 경로가 `Intrinsics::to_normalized` 를 쓰는 한 실제 렌즈 영상에서 왜곡이 무시된다.
  정규화 경로를 `DistortedIntrinsics::unproject` 하나로 모으는 일과 왜곡 있는 합성 카메라 두 장의 자세 시험은 T08 전에 해야 한다.

## 제품 브랜치·커밋
- 브랜치 `feat/io-robustness`, 커밋 아래 참조.
