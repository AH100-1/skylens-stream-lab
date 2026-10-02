# P04 io-followup — README 좌표 원점·왜곡 정규화·벤치 구간·GPS 정렬, PLY 매직 우선 검사, 문서 시험 오류 코드

## 결론
채택 제안. README 두 절(한·영)을 main 0afc818 의 실제 CLI(`--help`), `synth` 출력, `verify` 명령, `align.rs`·`camera.rs`·벤치 구간 이름과 다시 대조해 고쳤다(F-134·F-135·F-155·F-170).
PLY 읽기는 헤더 줄 해석 전에 첫 4·5 바이트(`ply\n`·`ply\r\n`)를 먼저 보아, JPEG 같은 이진 파일과 줄바꿈 없는 긴 글이
"PLY 매직 없음" 으로 거절된다(F-168). `GrayImage` 문서 시험에 `compile_fail,E0451` 을 달았다. 다만 안정판 rustdoc(1.97)은 오류 코드를
확인하지 않는다는 것을 실측으로 확인해, 같은 경로를 쓰는 컴파일되는 짝 문서 시험을 더했다(F-163). F-016 은 범위가 커서 현황만 쟀다.

## 수치
| 검증 | 기준 | 결과 |
|---|---|---|
| `synth /tmp/syn 64 48` 출력 파일 종류 | README 두 절 입력 트리와 일치 | gps.txt, images/cam{F,R,L}_N.jpg, truth/cameras.txt, truth/origin.txt — 두 절 트리에 모두 있음 |
| 두 절의 출력 좌표 원점 문장(첫 GPS, 동-북-위 m, `Scene::to_first_gps_frame`) | 절마다 1개 | 1·1 |
| 두 시점 예시 주석 "왜곡 없음"·`with_distortion(d).unproject` | 0건 | 0건(`dist` 포함, `to_normalized` 역왜곡, 수렴 여부는 `k.unproject(p)`) |
| README 벤치 구간 목록 vs `pipeline.rs` `name:`(pipeline 모드) | 1:1 | 10개 모두(합성 렌더, 특징 검출, 짝 생성, 비율 매칭, RANSAC F(8점), RANSAC E(5점), 두 시점 자세, 회전 평균 2종, 번들 조정) |
| README 벤치 기본 규모 | 실제 기본값 | "합성 240장" → 기본 8위치 × 3 = 24장 480×270, `--full` 240장 960×540 로 바로잡음 |
| JPEG 머리 바이트 `ff d8 ff e0 …` | 오류에 "매직" | 통과(기존: "헤더가 UTF-8 이 아님") |
| 줄바꿈 없는 `abc`×2000(6000 B) | 오류에 "매직" | 통과(기존: "헤더 줄이 너무 김") |
| `plyx\n`, ` ply\n`, `ply`, 빈 입력, `ply\rx`, `PLY\n` | 오류에 "매직" | 통과 |
| `ply\r\n` 줄바꿈 헤더 | 읽힘 | 통과 |
| 기존 PLY 시험 | 통과 | lib `ply` 15개 통과(새 2개 포함) |
| 문서 시험 경로를 `GrayImag` 로 틀리게 바꾼 변형, `compile_fail,E0451` 만 있을 때 | 실패해야 함 | **통과해 버림** — 안정판은 오류 코드 미확인 |
| 같은 변형에 짝 문서 시험(`try_from_vec`) 추가 뒤 | 실패 | 짝 시험이 컴파일 오류로 실패(경로 고장 검출) |
| 원래 코드 | 문서 시험 통과 | compile_fail 1·짝 1 통과 |
| ply 읽기량 시험 주석 수치 | 단언 수치와 같음 | 주석 "상한 4 KiB + 여유 16 KiB = 20 KiB" = 단언 `MAX_HEADER_LINE + 16 * 1024` |

F-016 현황(main 0afc818): 검출기 출력은 아직 화소 번호 규약(화소 (i, j) 중심 = (i, j)). 시험·README 의 수동 `+ 0.5` 보정이
features.rs 2곳(1016·1022 행 부근), matching.rs 1곳, two_view.rs 1곳, README 두 절 각 1곳(매칭 예시 `px`) 남아 있다.
`Keypoint::pixel()` 없음. `feat/pixel-convention`(336621f)은 main 에 병합되지 않았다.

## 정정 — io-cleanup 노트의 lib 시험 수(F-169)
`experiments/io-cleanup.md` "남은 문제" 첫 항목의 "lib 136 통과·1 실패·3 무시"(합 140)는 측정 커밋을 적지 않았고 실제 브랜치와 맞지 않는다.
측정 커밋은 제품 `feat/io-cleanup` ff3a769 이다. 이 커밋의 `crates/core/src` 에서 `#[test]` 속성은 140개, `#[ignore` 는 3개로 세어지나,
`cargo test --release -p skylens-core --lib -- --list` 개수는 142 로 보고되었다(F-169). 정정 값은 아래 "남은 문제" 의 재측정 결과로 적는다.

## 방법
- README: `skylens-stream --help`, `synth /tmp/syn 64 48` 의 파일 목록, `verify.rs` 머리말(종료 코드 0/1/2, report.json 판정 항목),
  `align.rs`(`align_to_enu`·`gps_align`·`umeyama` 의 None 조건), `synth.rs`(`GPS_ORIGIN`, `first_gps_origin`, `to_first_gps_frame`),
  `camera.rs` 모듈 문서, `benches/pipeline.rs` 의 구간 이름과 인자 표를 대조. 한·영 같은 뜻.
  상태 문장의 "명령행 도구에는 ply-info·synth 만" 을 `verify` 포함으로 고쳤고, 사용법 절에 `verify` 를 더했다.
- GPS 정렬 절은 main 의 align.rs 기준(3 m 고정 상한, 정상 3개 미만이면 첫 추정 유지, None 조건)으로 적고,
  `gps_align` 의 `origin` 에는 첫 GPS 를 주라고 적었다(기존 "합성 장면의 원점은 truth/origin.txt" 는 출력 원점과 다른 원점을 권하는 것으로 읽힘).
- PLY: `read_magic` 이 `take(4)` 로 4 바이트를 읽고, `ply\r` 이면 1 바이트 더 읽어 `ply\n`/`ply\r\n` 인지 본다. 5 바이트 넘게 읽지 않는다.
  매직 바이트도 헤더 누적 크기에 더한다. 앞 공백(` ply`)은 이제 거절된다(기존은 trim 으로 받았음).
- 4 코어 측정 기계, 다른 빌드와 함께 도는 상태.

## 남은 문제
- F-155: 검토 대기 중인 GPS 정렬 개정(잡음 비례 기본 임계, 정상 < max(3, 50%)·좁은 경로에서 None, `threshold_m`·`spread_m`·`tilt_sigma_deg`,
  `gps_align_with`·`GpsAlignConfig`)이 main 에 들어오면 두 절의 GPS 정렬 문단 세 줄을 다시 고쳐야 한다. 지금은 main 기준.
- F-016: 검출기 출력 규약 변경은 matching.rs·two_view.rs·벤치 시험과 README 를 함께 바꿔야 해 이 묶음 범위를 넘는다. 현황만 위에 적었다.
- F-169 재측정(ff3a769 `--list` 개수)과 이 브랜치 전체 `cargo test --release` 결과는 아래 줄에 덧붙인다.

## 제품 브랜치·커밋
- 브랜치 `feat/io-followup`(origin/main 0afc818 에서): ce3d673(README), 6f045af(PLY 매직·문서 시험).
