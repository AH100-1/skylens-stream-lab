# pipeline-pm: 흐름 밀집 단계에 패치매치 연결

## 결론
- 기존 밀집(region_cloud)의 깊이는 평면 스윕(NCC, 이웃 3장)이었다. 패치매치(이웃 8장)를 같은 융합 경로에 꽂아 `DenseMethod::{Sweep, PatchMatch}` 로 고르게 했다(기본 Sweep, 처리 해상도는 `dense_width` 긴 변, 시험 96 px).
- 단구역 합성 장면에서 점→표면 중앙은 두 방식이 비슷하고(0.333 / 0.319 m) 점 수는 패치매치가 19% 많지만 95% 값은 나쁘고 밀집 시간은 5.5배 길다. 현 설정에서 패치매치가 낫다고 말할 근거는 없다.
- verify 통과는 두 방식 모두 5/7 (앞 노트 6/7 에서 한 칸 내려감, 원인 미확인).

## 수치 표 (4 코어 측정 기계, 40 위치 120 장, 폭 96 px, 2 시험 동시 실행)
| 방식 | 점→표면 중앙 | 95% | 1 m 초과 | 점 수 | 밀집 시간 | verify |
|---|---|---|---|---|---|---|
| 스윕 | 0.333 m | 1.523 m | 7.6% | 10650 | 12.0 s | 5/7 |
| 패치매치 | 0.319 m | 2.619 m | 9.0% | 12710 | 66.1 s | 5/7 |

## 방법
- dense.rs: `patchmatch_depth`(DepthView → patchmatch::View, 기본 설정), `region_cloud_patchmatch`(이웃 cfg.neighbors=8 전부 사용). 이웃 선택·깊이 범위·융합은 기존 경로 그대로.
- pipeline.rs: PipelineConfig.dense_method. cli/tests/pipeline.rs: 방식별 시험, 점→표면 중앙 ≤ 1.0 m·점 수 하한 단언.

## 남은 문제
- 해상도 320/480/960 측정 못 함(시간). fmt 통과, clippy·전체 시험 못 돌림. CLI 옵션 없음.
- 패치매치 95% 가 나쁨: 설정·융합 문턱 조정 필요.

## 제품 브랜치·커밋
feat/pipeline-pm f34be87
