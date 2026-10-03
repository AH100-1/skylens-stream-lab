# pipeline-merge-2306

## 결론
- 짝 맞춤 RANSAC 가속(77d9489)은 충돌 없이, F-294 수정(f8d677a)은 pipeline.rs 한 곳에서 충돌해 합쳤다. 초벌 BA 는 광축 높이 분산 롤 시작점(coarse_start)에서, 정밀 BA 는 예전 롤 규칙 시작점에서 출발하도록 F-294 쪽 의도를 살리고, 거기에 가속 쪽의 단계 시간 측정(ba_preview)을 그대로 씌웠다.
- 전체 시험은 마감 시각(23:40 UTC)까지 끝나지 않았다. core lib 은 끝났고(324 통과·1 실패·25 무시) F-294 시험은 통과했다. 새로 드러난 실패 둘 중 하나(perf_structure 의 최소 반복 수 고정 시험)는 가속 브랜치가 상수를 300 에서 100 으로 바꾸고 시험을 갱신하지 않은 것이라 시험의 기대값을 100 으로 고쳤다. 다른 하나(verify 의 nn_median 속도 시험)는 부하 중 1.017 s 로 1 s 상한을 살짝 넘은 것으로 보이며 단독 재실행은 못 했다.
- 합성 장면 README 명령(synth → run → verify)은 시간이 없어 돌리지 못했다.

## 수치 표 (4 코어 측정 기계, 다른 작업과 공유)
| 시험 묶음 | 통과 | 실패 | 무시 |
|---|---|---|---|
| core lib | 324 | 1 (verify::tests::nn_median_many_identical_points_is_fast_and_exact, 1.017 s > 1.0 s) | 25 |
| core dataset_synth | 2 | 0 | 0 |
| core perf_structure | 5 | 1 (essential_ransac_min_iterations_fixed, left 100 right 300) → 수정 후 재실행 못 함 | 0 |
| cli 단위 | 3 | 0 | 0 |
| cli pipeline (끝난 것) | single_region·patchmatch·two_region·preview_ba 통과 | 0 | - |

core lib 은 653 초 걸렸다. 나머지 cli 통합 시험(pipeline_e2e 등)은 마감 때까지 결과가 나오지 않았다.

## 방법
- feat/pipeline-merge-2106(e064aab)에서 새 브랜치를 만들어 두 브랜치를 차례로 --no-ff 병합. fmt·clippy(-D warnings) 통과.
- 전체 시험: cargo test --release --workspace --no-fail-fast -j 2, 로그 /tmp/merge-2306-test.log.

## 남은 문제
- nn_median 시험은 시간 상한 1 s 라 부하에 민감하다. 단독 실행으로 확인 필요.
- 수정한 perf_structure 시험, pipeline_e2e·pipeline_arrival·pipeline_regions·pipeline_stream* 결과, 합성 장면 단구역 verify·timing.json 총 시간은 미확인.
- 2구역 e2e 의 구역 간 스케일 차(알려진 문제)는 건드리지 않았다.

## 제품 브랜치·커밋
- feat/pipeline-merge-2306: 병합 cf044d9, 시험 기대값 수정 03f6009
