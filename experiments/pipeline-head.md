# pipeline-head: 흐름 머리 합치기와 시험

## 결론
- 흐름 머리 b50b837 에 위치 평균 가지(5b21293)를 `--no-ff` 로 합쳤다. 충돌 없음. 병합 커밋 00dbbfd.
- patchmatch 가지(61fa09a)는 b50b837 에 이미 들어 있어 "Already up to date" 였고 별도 병합 커밋은 없다.
- 확인 기준: `git diff origin/feat/translation-averaging -- crates/core/src/translation_averaging.rs` 와 `git diff origin/feat/patchmatch -- crates/core/src/patchmatch.rs` 둘 다 비어 있다(0줄).
- fmt·clippy(`-D warnings`, 전 대상) 통과. 요청한 시험 중 `pipeline_e2e::two_region_end_to_end` 한 개만 실패. 이 실패는 합치기 전 b50b837 에서도 같게 재현되어 이번 병합과 무관하다.

## 수치 표
4 코어 측정 기계, 다른 빌드가 동시에 돌아 부하가 높았다. 시간은 빌드 포함 벽시계(표의 "시험 시간"은 시험 자체).

| 시험 | 결과 | 시험 시간 | 부하(1분) |
|---|---|---|---|
| core lib `pipeline` (8 통과, 2 무시) | 통과 | 33 s | 6.1 |
| cli pipeline_arrival (2) | 통과 | 22 s | 5.6 |
| cli pipeline_e2e (2개 중 1 통과) | single_region 통과, two_region 실패 | 215 s (재실행 207 s) | 9.1 / 14.1 |
| cli pipeline_regions (3) | 통과 | 90 s | 9.1 |
| cli pipeline_stream | 통과 | 47 s | 10.1 |
| cli pipeline_stream_order | 통과 | 14 s | 9.4 |
| pipeline: 단구역 | 통과 | 20 s | 5.9 |
| pipeline: 단구역 PatchMatch | 통과 | 33 s | 6.9 |
| pipeline: 2구역 | 통과 | 46 s | 5.2 |
| pipeline: preview_ba | 통과 | 73 s | 6.4 |
| pipeline: synthetic_single_region_translation_averaging (timeout 600) | 통과 | 32 s | 5.8 |
| (대조) b50b837 에서 pipeline_e2e two_region | 같은 실패 | 43 s | 4.5 |

## 방법
- 작업 트리 b50b837 에서 위치 평균 가지, 이어서 patchmatch 가지를 차례로 병합. 시험은 이름을 걸러 `cargo test --release -j 2` 로 하나씩 실행.
- 실패 시험은 출력을 남겨 다시 돌렸고, 병합 전 b50b837 을 따로 받아 같은 시험을 돌려 원인을 갈랐다.

## 남은 문제
- `crates/cli/tests/pipeline_e2e.rs` 의 `two_region_end_to_end` 는 verify 종료 코드 1(preview_vs_refined 실패 유지, 높이 차 하한 2.0 m 초과)을 기대한다. 현재 머리에서는 verify 가 7/7 통과(종료 코드 0)이고 preview_vs_refined 높이 차 중앙 최대 1.048 m, 최근접 0.943 m 다. 머리 커밋 b50b837 이 `pipeline.rs` 시험의 기대값만 갱신하고 이 파일은 갱신하지 않은 것으로 보인다. 기대값을 7/7 통과로 바꾸는 갱신이 필요하다(이번 묶음에서는 손대지 않음).
- 실측 참고(현재 머리, 2구역): 정밀 재투영 0.284 px, 점쌍 최소 1178, 스케일 차 0.26 %, 겹침 차 중앙 최대 0.285 m, 점 20188개.

## 제품 브랜치·커밋
- feat/pipeline-head: 00dbbfd (부모 b50b837, 병합 대상 origin/feat/translation-averaging 5b21293)
