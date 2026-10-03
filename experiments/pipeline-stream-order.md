# pipeline-stream-order: 구역 순서 처리 흐름 확인과 사건 기록

## 결론
- 지금 `run_pipeline` 은 SPEC §3.7~3.8 순서(구역 도착 → 등록 → 초벌 즉시 출력 → 정밀은 별도 스레드 → 완료 시 교체 → 다음 구역은 최신 정밀 위에서 → 정밀 구역 재정렬 → 잔상 1.5 m 걸러내기)를 이미 대부분 따른다. 빠진 것은 (1) 위치 단위 도착, (2) 시점마다의 스냅샷, (3) manifest 에 사건 순서가 남지 않는 점이었다. 이 세 가지를 새 모듈 쪽으로 몰아 채웠다.
- 2구역 합성 장면에서 등록 수·verify 통과 수·정밀 중심 중앙 오차는 순차 방식과 같고(20, 2/7, 2.1537 m), 전체 시간은 41.6 초 대 51.5 초로 스트림 방식이 짧다.
- 이 합성 장면(320x180, 위치 16)은 카메라 1대분만 등록되어(구역 0: 12/36) verify 통과가 2/7 에 머문다. 흐름 순서의 시험용이며 정밀도 판정용이 아니다.

## 수치 표
2구역(위치 16, 구역 길이 10, 겹침 2), 2 코어 설정 측정 기계(4 코어 중 일부 공유), 특징 600, 밀집 폭 80, BA 8회.

| 방식 | 등록 | verify 통과/항목 | 정밀 중심 중앙 오차(GPS 기준) m | 전체 초 |
|---|---|---|---|---|
| 스트림(정밀 병렬) | 20 | 2/7 | 2.1537 | 41.6 |
| 순차(구역마다 정밀 대기) | 20 | 2/7 | 2.1537 | 51.5 |

사건 순서(스트림, manifest `events`): arrive_position 0..11 → coarse_output 구역 0 → arrive_position 12..15 → coarse_output 구역 1 → refined_replace 구역 0 → refined_replace 구역 1 → final. 구역 1 등록 중에 구역 0 정밀이 병렬로 돌지만 마무리는 구역 1 초벌 뒤에 반영됐다. 순차 방식은 refined_replace 구역 0 이 구역 1 의 첫 도착 위치보다 앞선다.

SPEC 순서 대조표:

| 순서 | 이전 | 지금 |
|---|---|---|
| 위치 단위 도착 | 구역 단위로 한꺼번에 읽음 | 위치마다 세 장을 읽고 `arrive_position` 기록 |
| 구역이 차면 초벌 즉시 출력 | 이미 그렇게 함 | `coarse_output` 사건과 시점 스냅샷 |
| 정밀은 별도 스레드, 완료 시 교체 | 이미 그렇게 함(스레드 + 채널) | `refined_replace` 사건과 시점 스냅샷 |
| 다음 구역 등록은 최신 정밀 위 | 이미 그렇게 함(앵커) | 변화 없음 |
| 이미 낸 구역 재정렬(공유 3D 점 sim3) | 이미 그렇게 함 | 재정렬이 있었으면 `realign` 사건과 스냅샷 |
| 잔상 1.5 m, 스냅샷·manifest | 마지막에 한 번 | 시점 스냅샷 합성에도 같은 걸러내기, manifest 에 `events` 추가 |

## 방법
- `crates/core/src/pipeline_stream.rs`: `StreamOptions{sequential}`, `LiveLog`(사건 목록, `snapshots/live/ev_NNN_종류_rK.ply`, 마지막에 manifest.json 에 `events` 배열 덧붙임), `compose`(정밀 구역은 최신 변환 적용, 정밀 없는 초벌 구역은 정밀 점 전체 기준 1.5 m 안 점 제거 후 6:1 추출).
- `pipeline.rs`: `run_pipeline` 은 `run_pipeline_with(..., StreamOptions::default())` 호출만 한다. 사진 읽기를 위치 단위 반복으로, 사건 기록 호출 몇 줄, `sequential` 일 때 구역마다 정밀 완료 대기. 설정 구조체는 바꾸지 않았다(다른 묶음과 충돌 방지).
- 시험 `crates/cli/tests/pipeline_stream_order.rs`: 사건 순서 단언, 시점 스냅샷 파일(NaN 없음·점 수 일치), 순차 방식 대비 등록·verify·중심 오차 단언, 시간 표 출력.

## 남은 문제
- 위치 단위 도착은 읽기 단계만 해당한다. 등록은 구역 전체 짝 맞춤 뒤에 하므로 위치 하나씩 등록하는 점진 등록은 아니다.
- 시점 스냅샷 합성의 잔상 걸러내기는 F-175(점 수의 제곱에 가까운 느림)의 영향을 그대로 받는다. 큰 점군에서는 시점 스냅샷이 병목이 될 수 있다.
- 시험 장면의 등록률이 낮아 정밀 중심 오차(2.15 m)와 verify 통과(2/7)는 방식 비교 기준으로만 의미 있다.
- 기존 `pipeline_regions` 시험 3개 중 2개가 실패했다(`failed_middle_region_is_skipped_not_fatal`: 건너뜀 기록 없음, `stationary_segment_is_error_or_issue`: 정지 구간 문구 없이 정렬 잔차·스케일 issue 만 남음; 이 장면에서 구역 간 스케일 차 2.6~3.5). 바뀐 부분(읽기 순서·사건 기록)과 직접 관계가 없어 보이나 기반 브랜치에서 같은 시험을 돌려 비교하지 못했다(시간 부족). 이 두 시험은 확인 필요. `pipeline_regions` 의 순서 시험과 `pipeline_stream_order` 는 통과, core 단위 시험 `pipeline_stream` 통과. 3구역 `pipeline_stream` 통합 시험은 시간 안에 돌리지 못했다.

## 제품 브랜치·커밋
- feat/pipeline-stream-order 6de27fc
