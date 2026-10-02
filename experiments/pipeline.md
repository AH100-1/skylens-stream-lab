# pipeline: 데이터셋에서 PLY 까지 잇기

## 결론
`skylens-stream run <입력> <출력>` 이 이미지 폴더(+GPS)에서 구역별 초벌·정밀 점군, 스냅샷, manifest.json,
report.json, poses.txt 까지 끝까지 돈다. 합성 장면(18위치, 320x180)에서 `verify` 는 7항목 중 3항목 통과
(종료 코드 1). 통과하지 못하는 항목은 아래 표. 아직 위치당 카메라 1대분(18/54장)만 등록된다.

## 수치 (4코어 측정 기계, 합성 18위치 x 3대, 320x180, span 8, ovl 2, 특징 800개, 깊이 폭 96)
| 항목 | 값 |
|---|---|
| 구역 | 2개 (위치 10곳, 12곳) |
| 시간(구역 0) | 특징 2.9 s, 매칭 22.9 s, 희소 0.0 s, BA+밀집 0.2 s |
| 등록 | 사진 18/54 (위치마다 1장) |
| 재투영 오차(초벌→정밀) | 1.46→0.19 px, 1.78→0.18 px |
| 카메라 중심 오차(정답 대비) | 중앙 1.62 m, 최대 3.72 m (GPS 잡음 1.5 m/축) |
| 점 수 (최종 스냅샷) | 3856 (구역당 초벌 7.8k~10.5k, 정밀 10.8k~12.3k, 6:1 추출 전) |
| 점군 → 정답 표면 수직 거리 중앙 | 3.67 m |
| verify | registered FAIL(18/54), region_images PASS, refined_reprojection PASS, preview_align FAIL(점쌍 < 1000, 스케일 차), preview_vs_refined FAIL, refined_overlap FAIL, snapshots PASS |

## 방법
구역(span/ovl)마다: 특징 → 후보 짝(시간 이웃·카메라 간) → 비율 매칭 → 본질 행렬 RANSAC·상대 자세 → 회전 평균
→ 이동 방향으로 좌표계 맞춤(Kabsch + 비행 축 둘레 회전은 보는 방향이 아래인 쪽) → GPS 사전 + 방향 제약 선형
최소제곱으로 중심 → 트랙(합집합-찾기) → 점-광선 삼각측량 = 초벌. 초벌에 번들 조정 = 정밀. 두 모델 각각 희소 점
보간 깊이 → 융합. 초벌을 공유 3D 점 닮음 변환으로 정밀에 정렬한 뒤 스냅샷·manifest.

## 남은 문제
- 카메라 3대 중 1대만 등록: 카메라 사이 시야가 거의 겹치지 않아 짝이 서지 않는다. 장비 고정 상대 배치 또는 GPS 사전으로 연결 필요.
- stand_in: 트랙 → feat/tracks, 위치 평균·삼각측량 → feat/translation-averaging, 깊이 → feat/patchmatch
  (희소 점 보간). feat/pipeline-sparse(`sparse::reconstruct`)·feat/pipeline-dense(`dense::region_cloud`)는 병합하지 않았다.
- 정렬 점쌍 < 1000, 구역 간 스케일 차 20%: 점이 188~373개뿐이라 짝이 모자란다.
- 구역을 병렬로 정밀화하고 다음 등록을 최신 정밀 모델 위에서 하는 순서는 아직 순차 단순판.
- 전체 시험(fmt/clippy/test 전체)은 이번 기록 시점에 돌리지 못했다.

## 제품 브랜치·커밋
feat/pipeline (마지막 커밋은 제품 저장소 로그 참고).

## 갱신: 카메라 사이 짝 일정 (구역 40위치)
### 결론
짝 목록을 편대 겹침에 맞춰 바꿨다(F(p)–R(p+12..40), F(p)–L(p+16..40), 간격 4 표본, 같은 위치 ±2, 같은 카메라 1..5).
합성 장면 40위치 x 3대(320x180, `--stride 2 --span 48 --ovl 2`)에서 등록이 18/54 → 120/120 으로 늘었고
verify 는 7항목 중 5항목 통과(이전 3). 구역 길이가 12 위치보다 짧으면 카메라 사이 짝이 서지 않으므로 구역을 40위치 이상으로 잡아야 한다.

### 수치 (4코어 측정 기계, 부하 평균 12~23 상태에서 측정, 특징 800개, 깊이 폭 96)
| 항목 | 이전 | 이번 |
|---|---|---|
| 등록 | 18/54 | 120/120 |
| 카메라 중심 오차 중앙·최대 | 1.62 m·3.72 m (18장) | 3.82 m·12.76 m (120장) |
| 점군→정답 표면 수직 거리 중앙 | 3.67 m | 4.95 m (최종 정밀 6935점) |
| 재투영(초벌→정밀) | 1.46→0.19 px | 2.05→0.24 px |
| 시간 | 합계 약 30 s | 특징 6 s, 매칭 81 s, 희소 0.1 s, BA+밀집 7 s, 합계 94 s |
| verify | registered FAIL, region_images PASS, refined_reprojection PASS, preview_align FAIL, preview_vs_refined FAIL, refined_overlap FAIL, snapshots PASS | registered PASS, region_images PASS, refined_reprojection PASS, preview_align FAIL (잔차 중앙 6.16 m, 기준 < 6 m, 점쌍 1513), preview_vs_refined FAIL (최근접 중앙 3.68 m, 높이 차 중앙 6.18 m), refined_overlap PASS(구역 1개라 해당 없음), snapshots PASS |

### 방법
`pipeline.rs` 의 `formation_pairs` 가 짝 목록을 만든다(matching.rs 는 그대로). 시험은 합성 장면 기본 80위치를 stride 2 로 읽어
40위치, 구역 하나. 정답 대비 값은 시험이 숫자로 단언한다(등록 120/120, 중심 오차 중앙 < 5 m·최대 < 15 m, 표면 중앙 < 6 m, verify 통과 >= 5).

### 남은 문제
- 중심·표면 오차가 18장 때보다 큼: 좌표계 맞춤은 GPS 잡음(1.5 m/축)과 직선에 가까운 비행에서 정해지며, 카메라 사이 짝이 들어와 가로 방향 제약이 생겼는데도 중앙 3.8 m. 초벌→정밀(BA) 사이 높이 차 중앙 6 m 가 남아 preview_align·preview_vs_refined 가 실패한다. BA 가 자유 좌표계에서 움직이는 것이 원인으로 보이며 GPS 고정 항이 필요하다.
- `sparse::reconstruct` 는 짝 목록을 인자로 받지 않고 내부에서 `candidate_pairs(PAIR_TEMPORAL, PAIR_CROSS, PAIR_POW2_MAX)` 를 고정으로 부른다. 그래서 pipeline.rs 는 자체 희소 경로(`sparse_init`)를 계속 쓰고, sparse.rs 는 병합만 해 두었다(호출 안 함). 짝 목록 인자가 생기면 한 줄로 바꿀 수 있다.
- `dense::region_cloud` 가 담긴 feat/pipeline-dense 는 main 의 융합 변경(FusionView.group, FusionConfig.min_groups·same_group_views)과 맞지 않아 빌드가 깨져 병합하지 못했다. 그 브랜치가 main 을 합친 뒤 병합한다. 밀집은 계속 희소 점 보간 대체 구현을 쓴다.
- feat/tracks, feat/translation-averaging 은 시간이 모자라 병합하지 못했고 `stand_in` 이 대신한다.
- 구역 병렬·즉시 초벌 방출·최신 정밀 모델 위 다음 등록·sim3 재정렬 순서는 구현하지 못했다(구역 하나라 지금 시험에서는 순차).
- 매칭 81 s 가 시간 대부분. 부하가 큰 기계에서 측정했다.

## 제품 브랜치·커밋
feat/pipeline 69a6363
