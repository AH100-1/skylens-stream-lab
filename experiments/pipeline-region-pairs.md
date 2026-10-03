# pipeline-region-pairs (미완, 시간 한도로 중단)

## 결론
- 제품 기준(merge-1717 + region-sim3 21b2496) 병합 완료. 충돌은 pipeline.rs 의 run_pipeline 도입부 한 곳(run_pipeline_with 유지, 미사용 import 제거).
- 2구역 점쌍 324 의 원인 측정, 점쌍 확대, sim3 모드 전/후 수치, 시험 기대값 수정은 미수행: 릴리스 빌드 6분 + 2구역 시험이 부하(load 16~19)로 한도 안에 끝나지 않았다.

## 한 일
- `SKYLENS_PAIR_DEBUG=1` 로 own_align 마다 창 안 이미지 수, 초벌·정밀 트랙/관측 수, 같은 (이미지, 특징 번호) 로 맺어지는 관측 비율, 점쌍 수를 stderr 로 출력(PAIRDBG).
- 참고: 시험 설정 ovl=2 이므로 구역 1 창 [46,50) 은 4 위치 x 3 카메라 = 12 이미지뿐이다(가설: 점쌍 324 의 주원인).

## 남은 문제
- `SKYLENS_PAIR_DEBUG=1 cargo test --release --test pipeline synthetic_two_region -- --nocapture` 로 PAIRDBG 수집, SKYLENS_REGION_SIM3=0/1/2 비교, 기대값 수정.
- 점쌍 확대 후보: 정밀 점 트랙의 모든 관측 이미지를 경유한 대응(창 안 관측 기준 유지).

## 제품 브랜치
feat/pipeline-region-pairs
