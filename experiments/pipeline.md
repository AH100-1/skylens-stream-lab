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
