# pipeline-merge-1906: 끝까지 흐름 통합

## 결론
- 기준 `feat/pipeline-merge-1717`(92429d8)에 `feat/pipeline-e2e`(시험 `pipeline_e2e`, README 절)와 `feat/pipeline-arrival`(`progressive::chain_realign` 분리, 시험 `pipeline_arrival`)을 차례로 합쳤다. 두 합침 모두 충돌 없음(앞의 것은 빨리감기).
- 되돌린 부분: 이미 내보낸 정밀 구역의 **카메라 중심**에 재정렬 변환을 적용하던 부분을 제거했다. 재정렬은 점군에만 적용하고 중심·poses 는 구역 자기 좌표 그대로다. 정답(GPS) 대비 중심 중앙 오차는 전체 3.60 → 2.514 m, 구역 0 위치 3.70 → 2.748 m 로 되돌아왔다(arrival 노트의 미적용 측정 2.51/2.75 m 와 같다).
- `pipeline_stream_order` 182 행 실패의 원인은 코드 버그가 아니라 시험의 옛 가정이었다: "마지막 스냅샷 점 수 >= 구역 0 초벌 직후 점 수". 현재 흐름에서는 정밀 BA 가 이상치를 걸러 정밀 점이 초벌보다 적을 수 있다(이 16 위치 장면: 초벌 18 점, 정밀 11 점). 점 수 크기 비교 대신 구성을 정확히 단언하도록 바꿨다(아래).
- README 의 synth → run → verify 를 그대로 실행했고 값이 README 표와 일치한다(단구역 6/7, 구역 2개 5/7; 미달 항목도 같음).

## 수치 표
부하: 4 코어 측정 기계, uptime load average 15~25 (여러 작업 동시). 시간 값은 부하 포함.

### README 명령 verify (항목별)
단구역 `out`(120장, 부하 19, run 172 s):

| 항목 | 값 | 판정 |
|---|---|---|
| registered | 초벌 120/120, 정밀 120/120 | PASS |
| region_images | 1개 구역 모두 일치 | PASS |
| refined_reprojection | 정밀 0.300 px (초벌 3.026 px), 기준 ≤ 0.7 | PASS |
| preview_align | 점쌍 최소 2746, 스케일 차 0.00%, 잔차 중앙 최대 3.932 m | PASS |
| preview_vs_refined | 최근접 중앙 최대 2.358 m (< 3), 높이 차 중앙 최대 2.657 m (기준 < 2) | FAIL |
| refined_overlap | 해당 없음 (구역 1개) | PASS |
| snapshots | 2단계, 점 1889→1889, final 11054 | PASS |

결과 6/7, 종료 코드 1.

구역 2개 `out2`(240장, 부하 25, run 489 s):

| 항목 | 값 | 판정 |
|---|---|---|
| registered | 초벌 240/240, 정밀 240/240 | PASS |
| region_images | 2개 구역 모두 일치 | PASS |
| refined_reprojection | 정밀 0.282 px (초벌 3.232 px) | PASS |
| preview_align | 점쌍 최소 324 (기준 ≥ 1000), 스케일 차 8.26%, 잔차 중앙 최대 5.130 m | FAIL |
| preview_vs_refined | 최근접 중앙 최대 5.946 m, 높이 차 중앙 최대 6.278 m | FAIL |
| refined_overlap | 1쌍, 겹침 차 중앙 최대 0.257 m (< 0.3) | PASS |
| snapshots | 3단계, 점 1547→12132, final 19855, 새 영역 최소 1013 | PASS |

결과 5/7, 종료 코드 1. 이 명령은 verify 가 실패(종료 코드 1)하는 것이 현재 알려진 상태이며 기준 가지의 README 표와 같다. 합침으로 달라진 것은 없다.

### 시험 결과 (`cargo test --release -j 2`, 이름으로 지정)
| 시험 | 결과 | 비고 |
|---|---|---|
| pipeline_e2e (2개) | 통과 | 단구역 run 168.9 s, 구역 2개 run 378.4 s (부하 18~25), 두 시험 합 707 s |
| pipeline_arrival (2개) | 통과 | `chain_realign` 합성 3구역 재정렬 전 5.9455 m → 후 0.0135 m (점 기준, 점쌍 40/40); 3구역 실행 정밀 중심 중앙 오차 구역 0 위치 2.7480 m, 전체 2.5141 m (56 장) |
| pipeline_stream_order | 통과 (1개, 101 s, 부하 18). 스트림 대 순차: 등록 20 = 20, verify 2/7 = 2/7, 중심 중앙 오차 1.4287 m = 1.4287 m, 44.1 s 대 45.0 s |

### 중심 재정렬 되돌림 전/후 (GPS 정답 대비 중심 중앙 오차, 3구역 26 위치)
| | 구역 0 위치 | 전체 |
|---|---|---|
| 중심에 sim3 적용(arrival 가지) | 3.70 m | 3.60 m |
| 점군에만 적용(이 가지) | 2.748 m | 2.514 m |

시험 상한은 측정값의 약 1.25 배로 되돌렸다(구역 0 위치 3.45 m, 전체 3.15 m; arrival 가지는 4.6/4.5 m).

## 방법
- 합침 순서: e2e → arrival. 되돌림은 `run_pipeline_with` 끝의 중심 취합 직전 반복(`recs` 의 `rsim` 을 `centers` 에 적용)을 제거했다. `chain_realign` 과 점군 쓰기(`apply_cloud`)·`rsim` 저장·최종 점군의 `rsim` 적용은 그대로다. 시험 `pipeline_arrival` 의 재정렬 전/후 수치 단언(전 > 5 m, 후 < 0.03 m)은 점군 기준 그대로 유지된다.
- `pipeline_stream_order` 단언 교체: 스냅샷 점 수가 그 시점의 출력 파일 구성과 정확히 같다. 초벌 직후 = 초벌 파일 점 수 합, 구역 0 정밀 교체 직후 = 구역 0 정밀 + 구역 1 초벌, 구역 1 정밀 교체 직후와 마지막 스냅샷 = 정밀 파일 점 수 합(> 0). 이전의 부등식은 정밀 점 >= 초벌 점일 때만 성립했고, 정밀 모델이 BA 이상치 제거로 줄어드는 정상 경우에도 실패했다. 사건 순서 단언(도착 위치 단조·14~16 개, 초벌 < 도착, 초벌 < 정밀 교체, 마지막 사건 final)과 순차 방식 대비 수치 단언은 그대로다.

## 남은 문제
- `pipeline_stream_order` 의 16 위치 장면은 약하다: 구역 0 은 카메라 1 만(12/36), 구역 1 은 8/24 만 등록되고 구역 1 의 정밀 점이 0 개다(밀집 점이 없고 희소 점만 담김). 이 장면에서는 정밀이 초벌보다 점이 적다. 더 강한 장면에서 점 수 관계를 보는 시험은 따로 없다.
- 구역 0 의 정밀 점은 구역 0↔1 에 공유 사진이 없어 재정렬되지 않는다(arrival 노트와 동일).
- 최신 구역의 정밀 스케일이 정답에서 벗어나 있어(구역 1→2 스케일 1.126) 최신 좌표계로 옮기는 점군 재정렬이 정답 좌표계와 벗어날 수 있다. 중심은 옮기지 않으므로 점군과 poses 가 서로 다른 좌표계일 수 있다(재정렬된 구역에 한해). 해결안(재정렬 변환을 GPS 로 다시 잡기·스케일 고정)은 미구현.
- README 명령의 verify 는 단구역 6/7, 구역 2개 5/7(종료 코드 1)로 기준 가지와 같다. preview_align·preview_vs_refined 미달은 이 묶음 범위 밖.
- `cargo fmt --all --check`, `cargo clippy --all-targets -j 2 -- -D warnings` 통과. 상한을 조인 뒤 pipeline_arrival 을 다시 돌려 통과(105.9 s).

## 제품 브랜치·커밋
feat/pipeline-merge-1906 6b27a59 (기준 origin/feat/pipeline-merge-1717 92429d8; 합침 커밋 5de63af = pipeline-e2e 빨리감기, fedd1cd = pipeline-arrival 합침)

합친 커밋 목록: 634844d (CLI 끝까지 시험·README 절), 5de63af (두 구역 시험 기본 실행), 63d15a9 (chain_realign·register 사건·사건 순서 시험), fedd1cd (합침), 6b27a59 (중심 재정렬 되돌림·stream_order 단언 교체).
