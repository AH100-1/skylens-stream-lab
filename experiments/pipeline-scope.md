# 흐름 PR 범위 정리 (F-296)

## 결론
- 흐름 브랜치(03f6009) 위에 위치 평균 머리(aaff35f)와 PatchMatch 머리(61fa09a)를 합침 커밋으로 올렸다. 충돌은 `crates/core/src/lib.rs` 모듈 목록 한 곳뿐이며 두 쪽을 모두 남겼다.
- `translation_averaging.rs` 는 위치 평균 머리와, `patchmatch.rs` 는 PatchMatch 머리와 차이가 없다(diff 비어 있음). 흐름 코드는 고칠 필요가 없었다. `triangulation.rs` 는 두 머리 어디에도 없어 흐름 판을 그대로 둔다.
- 미해결: 위치 평균 경로 시험(`synthetic_single_region_translation_averaging`)이 14분 넘게 끝나지 않아 직접 중단했다. 중단 시점 호출 스택은 `pipeline::averaged_centers` → `average_translations_with_points` → `gp_solve` 였다. 위치 평균 머리의 `gp_solve` 가 흐름 입력 크기에서 매우 느린 것으로 보인다. 이전 판과의 시간 비교는 하지 못했다.

## 수치 표
| 항목 | 결과 |
|---|---|
| fmt / clippy -D warnings | 통과 / 통과 |
| core `pipeline` 시험 | 7 통과, 0 실패, 2 무시 |
| cli: 단구역 e2e, PatchMatch e2e, 2구역 e2e, preview_ba | 통과 |
| cli: 위치 평균 단구역 | 14분 넘게 미완, 중단 |
| 03f6009 대비 등록 수·초벌 스케일 차·표면 중앙·높이 차 | 미측정 |
| 작업 공간 전체 시험 | 미실행 |

## 방법
`feat/pipeline` 에서 `feat/pipeline-scope` 를 따고 `git merge --no-ff` 로 두 머리를 차례로 합침. 4 코어 측정 기계, 다른 빌드와 동시 실행.

## 남은 문제
- 위치 평균 경로의 `gp_solve` 소요 시간.
- 03f6009 판과의 수치 비교, 전체 시험, cli 나머지 시험(`pipeline_arrival`, `pipeline_regions`, `pipeline_stream`, `pipeline_stream_order`)은 미실행.

## 제품 브랜치·커밋
feat/pipeline-scope (원격 푸시됨)
