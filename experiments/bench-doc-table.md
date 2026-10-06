# bench 사용법 표 순서 규칙 위치, dataset 시험 개수 정정

## 결론
- 제품 `crates/core/benches/pipeline.rs` 머리 문서에서 '순서 규칙' 문단이 표 중간에 끼어 있던 것을 표 뒤로 옮겼다. main(32a5da4)에서 아직 고쳐지지 않은 상태였다.
- `experiments/dataset-io.md` 의 'dataset 단위 시험 25개' 두 곳을 23개로 고쳤다.

## 수치 표
| 항목 | 고치기 전 | 고친 뒤 |
|---|---|---|
| 사용법 표 연속 줄 수 | 8줄 뒤 끊김, 4줄은 문단으로 렌더 | 12줄 연속, 문단은 표 뒤 |
| dataset 단위 시험 수 | 25 | 23 |
| fmt / clippy(`--all-targets -D warnings`) | - | 통과 / 통과 |

## 방법
- 개수: 제품 main 에서 `cargo test --release -p skylens-core --lib dataset:: -- --list` 의 끝줄 `23 tests, 0 benchmarks`. `dataset.rs` 의 `#[test]` 도 23개로 같다.
- 표 확인: `//! |` 줄이 `--ba-iters` 줄까지 끊김 없이 이어지고 빈 `//!` 줄 다음에 순서 규칙 3줄이 오는지 눈으로 확인. 문서 생성은 하지 않았다.

## 남은 문제
- 노트의 다른 수치(시험 시간 0.40 초 등)는 다시 재지 않았다.

## 제품 브랜치·커밋
- feat/bench-doc-table, ffce49a97020bc8b834086068150515c22f7e5bb
