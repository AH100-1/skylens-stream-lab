# P04 io-cleanup — PLY 헤더 상한·형식 줄, GrayImage 불변식, synth 인자, README 대조

## 결론
채택 제안. 입출력 정리 잔여(F-061·F-081·F-062·F-063·F-082·F-058)를 모두 처리했다.
줄바꿈 없는 300 MiB PLY 에 `ply-info` 를 돌리면 최대 상주 메모리가 574 MB(기존 측정)에서 약 10 MB 로 줄고 0.01 s 안에
"헤더 줄이 너무 김" 으로 끝난다. `format` 줄이 없거나 버전이 1.0 이 아니거나 두 번 나오는 헤더는 `InvalidData`.
`GrayImage` 필드를 비공개로 바꿔 길이가 틀린 영상 자체를 만들 수 없게 했다(구조체 리터럴은 컴파일 오류, 문서 시험으로 고정).
`synth ""`·`synth out +20 20` 은 종료 코드 2 로 아무것도 쓰지 않는다. synth 인자 시험은 검사가 빠지면 10 s 에 자식을 끝내고 실패한다.
작업공간에 `rust-version = "1.88"` 을 넣었고, README 두 절을 현재 main 의 CLI·라이브러리 API 와 다시 대조해 고쳤다.

## 수치
| 검증 | 기준 | 결과 |
|---|---|---|
| `ply\ncomment ` + `a`×300 MiB(줄바꿈 없음), `ply-info` | 종료 코드 1, 상주 메모리가 파일 크기에 비례하지 않음 | 종료 1, 최대 상주 9.9 MB, 0.012 s |
| 줄바꿈 없는 4 MiB 헤더를 세는 리더로 `read_ply` | 읽은 바이트 ≤ 4096 + 16 KiB | 통과 |
| 헤더 줄 길이 4096 바이트(줄바꿈 포함) / 4097 바이트 | Ok / Err("헤더 줄이 너무 김") | 통과 |
| 짧은 줄 1400개로 헤더 > 64 KiB | Err("헤더가 너무 큼") | 통과 |
| `format` 줄 없음 + 12 바이트 | Err("format 줄 없음") (기존 Ok(1점)) | 통과 |
| `format binary_little_endian 2.0` / `format` 두 번 | Err / Err | 통과 |
| 개수 `10000000000` / `683212743470724134000` / `1537228672809129302` | "잘림" / "해석 실패" / "너무 큼" | 통과(세 문구 각각 단언) |
| `GrayImage { width:64, height:64, data: vec![0.5;100] }` | 컴파일 실패 | 통과(compile_fail 문서 시험) |
| `GrayImage::try_from_vec(64, 64, vec![0.5;100])` | Err(64,64,1,100), 패닉 0 | 통과 |
| `synth ""`, `synth "" 64 48` (빈 임시 폴더에서) | 종료 2, 폴더 비어 있음 | 통과 |
| `synth <폴더> +20 20` / `20 +20` / `" 20" 20` / `"" 20` | 종료 2, 출력 폴더 미생성 | 통과 |
| `parse_side` 가 늘 `Some(960)` 인 변형 실행 파일 | 관련 시험이 10 s 안에 실패, 임시 폴더 0개 남음 | 2개 시험 실패(범위 시험은 첫 경우에서), 10.42 s, 남은 폴더 0 |
| `parse_side` 가 늘 64 이하 값인 변형 | 실패 | 2개 시험 즉시 실패(종료 코드 0) |
| `cargo metadata` rust_version | 1.88 | core·cli 모두 1.88 |

## 방법
- PLY: 헤더 줄을 `take(4097).read_until(b'\n')` 로 읽어 4096 바이트를 넘으면 더 읽지 않고 오류를 낸다. 줄 길이를 더해 64 KiB 를 넘어도 오류.
  `format` 줄은 `binary_little_endian 1.0` 만 받고 한 번만 허용, `end_header` 뒤에 없으면 오류. 넘침 시험의 u64 초과 개수는
  주석을 "해석 실패" 로 바로잡고 문구를 단언한다.
- `GrayImage`: 필드 비공개, `try_from_vec`·`width()`·`height()`·`data()`·`into_data()` 추가. `new` 는 `폭×높이` 넘침에서 메시지와 함께 패닉.
  다른 모듈은 모두 생성자(`from_rgb`)만 써서 고칠 사용처가 없었다.
- CLI: 빈 출력 경로는 사용법 오류. 크기는 ASCII 숫자만 받은 뒤 `u32` 로 해석하고 범위 검사. `--help`/`-h` 는 사용법을 표준 출력, 종료 0.
- 시험 고리: 자식 프로세스를 `try_wait` 로 20 ms 간격 확인, 10 s 넘으면 kill 후 실패. 임시 경로는 Drop 에서 지운다.
  오류 문구 단언은 사용법 본문에 늘 있는 "16..=8192" 대신 오류 줄("오류: 폭·높이는 16..=8192")로 바꿔 다른 원인의 종료 2 를 걸러낸다.
- README: 다음을 고쳤다(한·영 같은 내용). gps.txt 이름은 확장자 없음, synth 의 `truth/origin.txt`, synth 인자 규칙·`--help`,
  `GrayImage` 접근자, 2의 거듭제곱 간격 상한 `PAIR_POW2_MAX`(16)(기존 문구 "8, 16, 32, …" 는 틀림), 회전 평균 이상치 문턱이
  max(5°, 6σ̂) 적응형이 된 것, 구간별 시간 측정 도구(`cargo bench --bench pipeline`, `bench_stages` 예제), PLY 헤더 상한·형식 줄.
- 측정: 4 코어 측정 기계, 다른 빌드와 함께 도는 상태.

## 남은 문제
- 전체 `cargo test --release` 에서 `two_view::tests::five_point_terminates_on_many_seeds`(시간 기준, 이 묶음이 고치지 않은 파일)가
  부하 중 한 번 실패, 단독 재실행에서 통과(0.42 s). 그 밖은 lib 136 통과·3 무시, perf_structure 5, 문서 시험 1, cli 10 통과.
- `cargo +1.88 build` 는 측정 기계에 해당 도구 사슬이 없어 확인하지 못했다. 표기만 넣었다.
- F-082 확인 기준의 "시험 4개 실패" 중 인자 개수 시험 2개는 `parse_side` 를 거치지 않으므로 그 변형으로는 실패하지 않는다(정상).

## 제품 브랜치·커밋
- 브랜치 `feat/io-cleanup`(origin/main 20a2ecf 에서): 97d413a(PLY), 6b09873(GrayImage), 63f61bb(CLI·시험), f80463b(rust-version·README).
