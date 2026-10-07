# 시험 상한·단언 정리 (번호 이동 시간 단언, 포즈 목표, 가중 확인)

## 결론
- F-311: 번호 이동(1,000,000) 시험의 "시간 차 10% 이내" 단언은 부하에 흔들리는 벽시계 비교였다. 점 번호를 옮겨도 계산이 달라지지 않는다는 성질은 RMS 오차의 비트 일치로 단언하도록 바꾸고, 시간은 출력만 한다. 4 코어 측정 기계에서 부하 평균 30 이상일 때도 통과했다(릴리스, 117.9 s).
- F-321, F-323: 이번 묶음의 기반(origin/main)에는 대상 코드가 없어 손대지 못했다. F-321 의 `crates/cli/tests/pipeline_poses.rs`(와 `poses_io`)는 PR #55 가 병합되기 전이라 main 에 파일이 없고, F-323 의 `weighted_position_uniform_when_cost_not_finite` 와 가중 위치 식은 feat/fusion-freespace 에만 있고 main 의 fusion.rs 에는 가중평균·중앙값 위치 계산 자체가 없다. 목표 상한으로 바꿨을 때의 실측 비교는 하지 못했으므로 노트에 적을 실패 수치는 없다.

## 수치 표
| 항목 | 이전 | 이후 |
| --- | --- | --- |
| 번호 이동 0 / 1e6 시간 | 단언(차 10% 이내) | 출력만: 56.68 s / 60.43 s (부하 30 이상, 차 6.6%) |
| 두 경우 RMS | 차 1e-4 미만 | 비트 일치(0.0002) |
| 최대 메모리 | 1024 MB 미만 | 변경 없음, 203 MB |
| 절대 시간 한도 60 s | 단언 | 제거(출력만) |

## 방법
- 대상: `sparse_scaling_240_cameras_4000_obs`(무시 시험, 240장 x 장당 4000 관측). 시간 비율·절대 시간 단언을 지우고 `rms0.to_bits() == rms1.to_bits()` 로 대체. 점 수(100만 초과)·메모리·RMS 0.05 미만 단언은 유지.
- 실행: `cargo test --release -p skylens-core --lib sparse_scaling_240 -- --ignored --nocapture` 1 통과. fmt, clippy(`-p skylens-core --all-targets -D warnings`) 통과.
- 연속 5회 반복은 하지 못했다(1회 2분 안팎, 마감).

## 남은 문제
- F-321: 목표 회전(중앙 0.2°, 최대 1°) 상수와 2구역 파일 시험은 feat/pose-test-bounds(6b8ecaf)에 이미 있고, 그쪽이 main 에 오른 뒤 목표 상한 대비 실측과 겹침 사진 구역 간 차(중앙 0.90°, 최대 7.73°)를 다시 봐야 한다.
- F-323: 가중평균·중앙값 위치 시험은 feat/fusion-freespace 가 main 에 오른 뒤 가중 지수를 1 로 바꿨을 때 실패하는지 확인하는 시험으로 보강해야 한다.
- 이 기계는 부하가 커서 부하 10 안팎 5회 연속 통과 기준은 미확인.
- 같은 파일의 광선 점 시험(`ray_point_sampled_hypotheses_are_fast_and_accurate`)에는 아직 0.2 s 시간 단언(3회 최소값)이 있다.

## 제품 브랜치·커밋
- feat/ta-test-bounds 29ac911 (기반 origin/main 105bdb7)
