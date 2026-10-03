# pipeline-arrival: 도착 → 등록 → 초벌 → 정밀 → 재정렬 사건 순서

## 결론
- 여섯 단계 중 도착·등록·초벌 즉시 출력·정밀 병렬 교체·초벌 재정렬은 이미 구현돼 있었고 사건 순서 시험(신규 `pipeline_arrival.rs`)으로 고정했다.
- 빠져 있던 것: (a) 이미 내보낸 정밀 구역의 **카메라 중심(poses.txt)** 은 새 정밀 모델 좌표계로 옮겨지지 않았다(점군만 이동) → 최종 출력 직전에 `rsim` 을 중심에 적용. (b) 등록 사건이 사건 기록(manifest events)에 없었다 → `register` 사건 추가. (c) 연쇄 재정렬 논리가 `run_pipeline_with` 안에 인라인이라 단독 시험이 불가 → `progressive::chain_realign` 으로 추출.
- 확인되지 않았던 두 질문의 답: **등록은 위치 하나씩이 아니라 구역 단위**다(사진 읽기만 위치 단위로 도착, 등록 `sparse_init` 은 구역 전체를 한 번에). 그리고 **다음 등록은 최신 정밀 모델 위에서 하지 않는다**: `ANCHOR_NEXT_REGION=false`(3구역 측정에서 초벌 기반 변환 잔차 1.7~3.1 m 로 오히려 나빠져 꺼 둠). 최신 정밀 모델은 (i) 초벌 출력 정렬, (ii) 정밀 BA 의 닻 후보에만 쓰인다.

## 단계별 코드 위치 (crates/core/src/pipeline.rs, `run_pipeline_with`)
| # | 단계 | 함수/위치 | 순서·비고 |
|---|---|---|---|
| 1 | 위치 단위 도착 | 구역 루프 안 `by_pos` 반복, `live.note("arrive_position")` | 새 위치만(캐시된 겹침 위치 제외), 오름차순 한 번씩 |
| 2 | 등록 | `check_motion` → `sparse_init` (구역 전체 한 번), 이어서 `live.note("register")`(신규) | 구역 단위. 위치별 점진 등록 아님 |
| 3 | 초벌 즉시 출력 | `dense_cloud(init)` → 최신 정밀 `latest_ref` 와 `cross_align` 겹침 있으면 그 좌표계로 → `write_decimated` → `live.snapshot("coarse_output")` | 다음 구역 위치 도착 전에 일어남(시험 단언) |
| 4 | 정밀 병렬 | `std::thread::spawn`: `run_ba` + `gps_align_refined` + `dense_cloud`, 결과 `RefinedMsg` | `in_flight` ≤ 2, 닻(`make_anchor`)은 꺼짐 |
| 5 | 정밀 교체 | `handle` 클로저: 파일 쓰기, 자기 구역 초벌→정밀 정렬, `snapshot("refined_replace")` | 다음 구역 시작 때 `try_recv` 로 먼저 반영 |
| 6 | 재정렬 | 같은 `handle`: 정밀 없는 초벌 구역 `cross_align`; 정밀 구역은 `progressive::chain_realign`(공유 3D 점 → `robust_fit`; 비겹침 구역은 이웃 경유 연쇄 합성) → 점군 쓰기, `rsim` 저장, `snapshot("realign")`; 끝에서 중심에도 `rsim` 적용(신규) | |

## 수치
부하: uptime load average 15~21 (4코어, 여러 묶음 동시).

| 항목 | 값 |
|---|---|
| `chain_realign` 합성 3구역(0..10, 8..18, 16..26; 구역 0·2 비겹침, 잡음 0.01 m, 변환 스케일 1.04/0.97) 구역 0 점 RMS(정답 최신 좌표계 대비) | 재정렬 전 5.9455 m → 후 0.0135 m (점쌍 40/40) |
| 3구역 전체 실행(26 위치, 320x180) 실행 시간 | 82~106 s (부하 15) |
| 같은 실행 정밀 재정렬 잔차 중앙값 | 구역 1→2: 0.280 m (22 쌍), 단언 < 0.6 m |
| 정밀 중심 정답(GPS) 대비 중앙 오차, 중심에 rsim 적용 | 구역 0 위치 3.70 m, 전체 3.60 m |
| 같은 실행, 중심에 rsim 미적용(이전 동작) | 구역 0 위치 2.75 m, 전체 2.51 m |

## 방법
- 신규 시험 `crates/cli/tests/pipeline_arrival.rs`: (1) `chain_realign_recovers_latest_frame_through_neighbour` 는 합성 트랙으로 오차 전/후를 단언(전 > 5 m, 후 < 0.03 m, 구역 1 경유 연쇄 단계 순서). (2) `arrival_order_and_realigned_centers` 는 3구역 실행에서 구역마다 마지막 위치 도착 < 등록 < 초벌 출력 < 정밀 교체, 초벌 출력 < 다음 구역 새 위치 도착, 마지막 정밀 교체 뒤 realign, 재정렬 잔차·중심 오차 상한을 단언.

## 남은 문제
- 이 합성 장면은 약하다: 구역 0 은 카메라 1 만(12/36) 등록돼 구역 0↔1 은 공유 사진이 없어 **구역 0 의 정밀 점은 재정렬되지 않았다**(`chain_realign` 단계가 구역 1 하나뿐). 점군·중심 모두 구역 0 은 GPS 정렬 상태 그대로.
- 중심에 재정렬을 적용하니 GPS 정답 대비 오차가 늘었다(2.51 → 3.60 m): 최신 구역(2) 정밀 모델의 스케일이 정답에서 벗어나 있어(구역 1 → 2 스케일 1.126, 22 쌍) 최신 좌표계가 정답이 아니기 때문. 점군과 중심의 일관성을 택했으나 정확도 면에서는 후퇴이므로 소유자 판단이 필요(대안: 재정렬 변환을 GPS 로 다시 잡거나 스케일 고정).
- 등록 자체를 최신 정밀 위에서 하려면 `ANCHOR_NEXT_REGION` 을 켜야 하는데 측정상 나쁨. 위치별 점진 등록은 미구현(큰 변경).
- 기존 `pipeline_stream_order` 시험은 기준 브랜치(feat/pipeline-merge-1717)에서도 182 행에서 실패한다: 마지막 스냅샷 점 수(11, 정밀 점이 거의 없음)가 구역 0 초벌 출력 직후 점 수보다 적다. 단언을 느슨하게 하지 않고 그대로 둠.

## 제품 브랜치·커밋
feat/pipeline-arrival 63d15a9 (기준 origin/feat/pipeline-merge-1717 92429d8)
