# pipeline-cli: 흐름 설정의 `run` 옵션화, 트랙 경로 단일화, 스냅샷 단조 판정 통일

## 결론
- 흐름 설정 가운데 `dense_method`, `position`, `preview_ba_iters`, `gps_sigma_h/v`, `tri_*` 를 `skylens-stream run` 옵션으로 열었다. 시험의 `PIPE_POSITION` 환경변수는 시험 함수 인자로 바꿨다.
- 위치 평균 경로의 트랙 생성을 `tracks::build_tracks` 로 바꿨고 `stand_in::build_tracks` 는 지웠다(호출 0곳). 머리 주석도 고쳤다.
- 스냅샷 점 수 단조 판정은 흐름 issue(`stream::check_snapshots`)와 `verify` 가 같은 함수(`stream::snapshot_count_decreased`)를 쓴다. 규칙: 정수 step 사이만 보고, 같은 수는 허용, 감소만 위반. final 은 정밀 점군 간격 추출 결과라 판정에서 뺀다.

## 옵션 표
| 옵션 | 값 | 설정 필드 |
|---|---|---|
| --dense-method | sweep, patchmatch | dense_method |
| --position | gps, translation-averaging(별칭 ta) | position |
| --preview-ba-iters | 자연수(0 이면 BA 없음) | preview_ba_iters |
| --gps-sigma-h / --gps-sigma-v | 0 보다 큰 수(m) | gps_sigma_h / gps_sigma_v |
| --tri-loose-frac / --tri-median-k / --tri-min-px | 0 보다 큰 수 | tri_* |

잘못된 값은 오류 메시지와 종료 코드 2.

## 재현 명령(노트의 PatchMatch·위치 평균·초벌 BA 조합)
`skylens-stream run <입력> <출력> --stride 2 --span 48 --ovl 2 --max-skip-run 2 --max-features 800 --dense-width 96 --hfov 65 --ba-iters 15 --preview-ba-iters 8 --dense-method patchmatch --position translation-averaging`

## 수치
| 항목 | 전 | 후 |
|---|---|---|
| PatchMatch 단구역 issue | "스냅샷 점 수 단조 증가 아님 13068 → 12979" (final 포함 판정) | 없음(정수 step 사이만 판정, verify 와 일치) |
| 단위 시험 | - | core stream 20 통과, cli 옵션 파싱 3 통과 |
| 위치 평균 경로 등록 수·중심 오차 전/후 | 측정 못 함 | 측정 못 함(아래 남은 문제) |

## 방법
`stream.rs` 에 `snapshot_count_decreased` 를 두고 `check_snapshots`(정수 step 만 필터) 와 `verify.rs` 가 호출. 시험은 13068/13068/12979(final) 은 통과, 13068→12979(정수 step 사이)는 위반 1건.

## 남은 문제
- 마감 때문에 `cargo clippy --all-targets -D warnings`, 전체 `cargo test --release`, 위치 평균 경로(`synthetic_single_region_translation_averaging`, 측정만 출력) 전/후 수치를 돌리지 못했다. 측정 기계가 다른 작업과 겹쳐 빌드가 느렸다.
- 트랙 규칙이 바뀌어(합집합-찾기 → 충돌 분할) 위치 평균 경로의 등록 수·중심 오차는 달라질 수 있다.

## 제품 브랜치
feat/pipeline-cli
