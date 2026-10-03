# pipeline-merge-2006: 구역 전체 점쌍 합침

## 결론
- 기준 `feat/pipeline-merge-1906`(6b27a59)에 `feat/pipeline-region-pairs`(a3fc623)를 merge 커밋으로 합쳤다(b106a0f). 충돌은 README 와 `pipeline.rs` 두 곳.
- 2구역 verify 5/7 → 6/7: preview_align 이 통과로 바뀌었다(점쌍 최소 324 → 1221, 스케일 차 8.26 → 0.05%). 높이 차 6.278 → 5.829 m, 최근접 5.946 → 4.923 m. preview_vs_refined 는 여전히 미달(기준 높이 차 2 m, 최근접 3 m).
- 단구역은 변화 없음(6/7). 중심 오차·점→표면 오차도 두 구성 모두 이전 값과 같다.

## 수치 표
| | 합침 전 | 합침 후 |
|---|---|---|
| 단구역 verify | 6/7 | 6/7 (preview_vs_refined 높이 차 2.657 m 미달) |
| 2구역 verify | 5/7 | 6/7 |
| 2구역 점쌍 최소 | 324 | 1221 |
| 2구역 스케일 차 | 8.26% | 0.05% |
| 2구역 높이 차 / 최근접 | 6.278 / 5.946 m | 5.829 / 4.923 m |
| 2구역 refined_overlap | 0.257 m | 0.257 m |
| 단구역 중심 오차 중앙/최대 | 0.329 / 2.912 m | 0.329 / 2.912 m |
| 2구역 중심 오차 중앙/최대 | 0.290 / 3.115 m | 0.290 / 3.115 m |
| 단구역 점→표면 중앙/95% | 0.475 / 1.465 m (점 11054) | 같음 |
| 2구역 점→표면 중앙/95% | 0.494 / 3.022 m (점 19855) | 같음 |

시험(`cargo test --release`, 4 코어 측정 기계, 부하 높음): pipeline_e2e 단구역 통과(run 65 s), 2구역 통과(수정 후 run 276 s), pipeline_arrival 2개 통과(재정렬 전 5.9455 → 후 0.0135 m; 정밀 중심 전체 중앙 오차 2.5141 m), pipeline_stream_order 통과(스트림·순차 모두 등록 20, 2/7, 중심 1.4287 m). fmt·clippy(-D warnings) 통과.

## 방법
- `pipeline.rs` 충돌: 기준 가지는 연쇄 재정렬을 `progressive::chain_realign` 으로 분리했고, region-pairs 가지는 같은 반복문에서 `region_align`(환경 변수 모드 분기 포함)을 불렀다. 둘 다 살리려고 `chain_realign_with(items, k, align)` 을 추가해 `chain_realign` 은 `cross_align` 으로 위임(공개 시그니처·arrival 시험 유지)하고 pipeline 은 `region_align` 을 넘긴다. 기본 모드에서 동작은 이전과 같다.
- README: 기준 가지에서 지운 옛 합성 명령 문단은 되살리지 않고, 수치 표의 2구역 열만 갱신했다.
- 2구역 시험: preview_align 기대를 PASS 로, 점쌍 최소 ≥ 1200·스케일 차 ≤ 1% 로 바꿨다. 정답 대비 상한은 느슨하게 하지 않았고 높이 차 상한 7.6 → 7.0 m, 최근접 상한 7.2 → 5.9 m 로 실측에 맞춰 조였다. 중심·점 오차 상한은 그대로.

## 남은 문제
- preview_vs_refined(초벌 대 정밀 높이 차 5.829 m)는 2구역·단구역 모두 미달이며 이번 합침으로 해결되지 않았다.
- 같은 시험 실행에서 기록한 점쌍 대응 환경 변수 모드(SKYLENS_REGION_SIM3 1·2)는 기본값이 아니며 이번에 측정하지 않았다.

## 제품 브랜치·커밋
- feat/pipeline-merge-2006: b106a0f(merge), 4659a32(시험·README 수치)
