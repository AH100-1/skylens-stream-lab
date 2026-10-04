# 구역별 카메라 위치 이동 창 (F-343)

## 결론
SPAN 12·OVL 2 를 그대로 두고 R·L 카메라의 구역 창을 F 창보다 `cross_offset`(기본 24 위치) 뒤로 민다(장면 끝에서 잘림).
기본 장면(27위치, 3구역)에서 구역 0 은 F 0..14 에 R·L 14..27 을 보조로 이어 붙여 F–R·F–L 겹침 짝(위치 차 20·24)이 구역 안에 생기고,
세 카메라가 한 덩어리로 등록된다. 인자 없는 synth → run → verify 가 종료 0, 7/7, 등록 81/81, 구역 3개로 바뀌었다.
다만 기존 시험 `pipeline_arrival` 의 재정렬 잔차 상한(0.6 m)이 0.698 m 로 넘는 것이 남았다(아래 남은 문제).

## 수치 표 (4 코어 측정 기계, 다른 빌드 동시 부하)
| 항목 | 전 (main 779edb7) | 후 (이동 창, 이동 24) |
|---|---|---|
| 등록(초벌/정밀) | 61/81, 61/81 | 81/81, 81/81 |
| verify | 5/7 (registered, preview_vs_refined FAIL), 종료 1 | 7/7, 종료 0 |
| registered | FAIL | PASS |
| region_images | PASS | PASS |
| refined_reprojection | PASS 0.237 px | PASS 0.247 px |
| preview_align | PASS 점쌍 최소 2166, 스케일 차 7.18% | PASS 점쌍 최소 3463, 5.55%, 잔차 0.273 m |
| preview_vs_refined | FAIL (최근접 > 6 m, 높이 차 inf) | PASS 최근접 0.419 m, 높이 차 0.241 m |
| refined_overlap | PASS 0.070 m | PASS 0.067 m |
| snapshots | PASS | PASS final 32419 |
| 구역 수 | 3 | 3 |
| run 시간 | 96 s | 88 s (시험 안 68 s) |
| 구역 0 등록 | 14/42 (R 만) | 42/42 (+보조 R·L 26장) |
| 카메라 중심 오차 중앙/최대 | (측정 안 함) | 0.307 / 0.940 m |
| 점 표면 오차 중앙/95% | | 0.482 / 1.355 m (점 32419개) |

## 방법
- 원인: 구역 0 은 앞쪽 보조 F 사진이 없고 R·L 의 짝(F 와 위치 차 +20~+40)이 구역 밖이라 F·R·L 세 줄이 따로 놀아 R 만 등록(14/42). 구역 1·2 는 앞쪽 F 보조(기존 HELPER_SPAN 40)로 이미 이어졌다.
- 창 정의(`stream::camera_window`, `dataset::camera_ranges`/`Dataset::camera_chunks`): F = [lo, hi), R·L = [lo+off, hi+off) 를 장면 끝에서 자름. `split_regions` 와 `chunk_ranges` 는 그대로(둘이 같다는 시험 유지).
- 기준 창 뒤 R·L 사슬을 밀린 창까지 끊기지 않게 잇는 보조 위치 `partner_positions` = [hi, 밀린 창 끝) 을 등록에만 쓰고(출력·`images`/`positions` 점수에 넣지 않음, verify region_images 3×위치 유지) F–R·F–L 짝을 만든다. 밀린 창이 장면 밖이면(구역 1·2) 보조 없음.
- 이동량 근거: F-197 정답 겹침 표(F→R +20 10%, +38 35%; F→L +16 11%, +30 30%; R↔L 직접 겹침 없음)와 PR #59 의 카메라 간 시작 +24(+20 은 2° 초과 7.9%). 설정 `PipelineConfig::cross_offset`, 명령행 `--cross-offset N`, 0 이면 세 카메라 같은 창.
- 짝 일정: `PairSchedule::default()`(F–R·F–L 위치 차 20..40, 4칸)은 그대로. 구역 0 의 F 0..13 × 보조 R·L 14..26 에서 차 20(F0..6)·24(F0..2) 짝이 생기며 이동 창 [24,27) 과 기준 창의 차 범위 8..40 안이다.
- 스트림 순서: 위치는 세 카메라 사진이 함께 도착하고, 구역 완료 시점은 그 구역에 필요한 마지막 위치(보조 끝)의 도착이다. 구역 0 은 위치 26 까지 받고 등록한다. 이후 구역은 이미 받은 위치를 쓴다.
- 시험: 새 `crates/cli/tests/default_path.rs`(verify 7/7·종료 0, 81/81, 구역 ≥ 2, 중심 오차 중앙 < 0.37 m, 점 표면 중앙 < 0.58 m 등 실측 x 1.2 상한), 단위 시험(`camera_windows_and_partners`, `camera_windows_match_stream`, `cross_offset_option`), `pipeline_arrival` 의 도착 순서 단언을 '필요한 마지막 위치' 기준으로 수정.

## 남은 문제
- `pipeline_arrival::arrival_order_and_realigned_centers`: 정밀 재정렬 잔차 [0.698, 0.456, 0.698] m 가 상한 REALIGN_MEDIAN_BOUND 0.6 m 를 넘는다(전에는 통과). 도착 순서 단언은 통과. 구역 0 등록에 보조 사진이 들어 구역 좌표가 바뀐 영향으로 보이나 원인 분해는 못했다. 상한을 풀지 않았다.
- 80위치 이상 장면에서는 구역마다 보조 R·L 이 24위치씩 늘고(기존 F 보조 40위치와 별개) 등록 비용이 커진다. 구역 1 이후는 기존 F 보조만으로도 이어지므로 구역 0 에만 쓰는 방안과 비교하지 못했다.
- `pipeline_e2e`(span 48, 80위치 2구역)는 구역 0 에 보조 R·L 24위치가 붙어 결과가 달라질 수 있는데 이번에 돌리지 못했다.
- 구역 1·2 의 R·L 은 F 와 겹침이 없어 F 줄과 따로 놀지만 구역 0 에서 이미 등록된다(장면이 27위치로 짧아서).

## 제품 브랜치·커밋
feat/region-camera-offset @ 0b5326e
