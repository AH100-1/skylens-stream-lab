# pipeline-so-merge: 위치 단위 도착 가지를 흐름 가지에 합침

## 결론
- `feat/pipeline-stream-order` 를 `feat/pipeline`(e0438ca) 에 합쳤다(5d3e3c4). 충돌은 구역 루프 한 곳: 흐름 가지의 초벌 BA 인자와 도착 가지의 자기 위치 정지 검사·등록 3장 미만 건너뜀을 모두 살렸다.
- pipeline_regions 3개 시험 통과(regions_in_order..., failed_middle_region_is_skipped_not_fatal, stationary_segment_is_error_or_issue), 4 코어 측정 기계 부하 중 436 s.
- Kabsch 짝 수(<3)·최대 특이값 0 검사를 sparse_init 에 추가(커밋 아래). 직선 비행은 특이값 하나만 크므로 비율 검사는 쓰지 않았다(정상 입력이 깨진다).

## 방법
충돌 해소 후 `cargo build --release --all-targets`, 통합 시험 실행, fmt·clippy(-D warnings) 통과 확인.

## 남은 문제
- 특이값 검사 추가 이후의 pipeline_regions 재실행, core 전체·cli 전체 시험, 단구역 CLI(synth → run → verify) 전후 수치 비교는 시간이 모자라 하지 못했다.
- 흐름 순서 표(도착 → 등록 → 초벌 → 정밀 교체 → 최신 정밀 위 등록 → sim3 재정렬) 중 사건 시험은 pipeline_stream_order 가 앞 단계를 다루나 이번에 실행하지 못했다.

## 제품 브랜치·커밋
feat/pipeline-so-merge: 5d3e3c4(합침), 이후 특이값 검사 커밋.
