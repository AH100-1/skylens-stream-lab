# stream-followup — 잔상 걸러내기 격자, 출력 검증 연결, 정렬 기준 검사

## 결론
- 잔상 걸러내기 반경 격자를 칸 한 변 = 반경/√3(약 0.866 m)로 줄였다. 같은 칸에 점이 있으면 바로 참이고,
  이웃 칸은 ±2 칸 중 칸 상자까지 최소 거리가 반경 이하인 칸만 본다. 넣은 점군(구역)마다 경계 상자를 두어
  어느 상자에서도 반경 밖인 질의는 칸을 보지 않고 거짓이다. 결과는 전수 비교와 같다(`ghost_filter_exact`,
  촘촘한 판 3000점 대 질의 3000점 `ghost_filter_exact_dense`).
- `write_outputs` 결과 폴더(위치 26곳, 구역 2개)를 출력 검증 `verify_dir` 로 읽으면 snapshots 항목이 판정·PASS
  (`write_outputs_passes_verify_snapshots`). manifest step 은 정수 1, 2, … 와 "final".
- `write_outputs` 의 위반 목록에 구역 0개, 정렬 점쌍 < 1000, 구역 간 스케일 차 > 10 % 를 더했다. 문서에서
  "비어 있으면 통과"를 빼고 검사 범위를 적었다.
- 오대응 시험 주석을 현재 동작(강건 첫 추정 뒤 복구)과 수치로 고쳤다.

## 수치
| 항목 | 값 |
|---|---|
| 오대응 30 % (`prelim_alignment_outlier_fractions`) | 스케일비 1.0000, fit 0.1005 m |
| 오대응 50 % | 스케일비 1.0002, fit 0.0983 m |
| 구역 0 (`prelim_alignment_against_truth`) | 스케일비 0.99930, fit 0.1031 m |
| 정렬 표 범위(정정) | 스케일비 0.99930~1.00021, fit 0.099~0.103 m |
| 위반 입력 {pairs 200, scale 0.5}·{pairs 200, scale 1.0} | 점쌍 위반 2건 + 스케일 차 위반 1건 |
| `write_outputs(dir, [], [], [], [])` | "구역 0개" 위반 |
| 스냅샷 시간 구역 3 × 10만/20만/40만 점 | 미측정(아래 남은 문제) |
| `large_snapshot_speed` 구역 7 × 200만 점 | 미측정 |

강건 첫 추정 전(전체 짝 최소제곱 첫 추정)에는 오대응 30 % 에서 스케일이 0.41 로 무너졌다.

## 방법
- 반경 r, 칸 c = r/√3·(1−1e-9). 칸 대각선 √3·c < r 이라 같은 칸의 점은 반드시 반경 안이다.
  r < 2c 이므로 이웃 ±2 칸이면 반경 구를 모두 덮는다.
- 시간 측정용 시험 `snapshot_scaling`(무시 표시, 측정할 때만): 구역 3 × 10만/20만/40만 점, 40만/10만 시간 비 < 8
  (선형 4배, 제곱 16배).
- 단계 시험은 `cargo test --release -p skylens-core stream::` 18 통과, 측정용 2개 제외.

## 남은 문제
1. F-175 시간 측정: 4 코어 측정 기계가 다른 빌드로 부하가 높아 `snapshot_scaling`·`large_snapshot_speed` 를
   기한 안에 돌리지 못했다. 다음에 단독으로 돌려 10만→40만 시간 비와 200만 점 7구역 시간을 이 표에 적는다.
   F-056 속도 기준도 이 측정으로 판정한다.
2. F-065: 스트림 `split_regions` 와 로더 `chunk_ranges` 를 하나로 합치는 일은 이번에 하지 않았다
   (스트림 쪽 꼬리 합침은 이미 들어가 있음, 26/12/2 → (0,14),(10,26)).
3. 전체 `cargo clippy --all-targets -- -D warnings`·전체 `cargo test --release` 를 기한 안에 돌리지 못했다
   (fmt 검사는 통과).

## 제품 브랜치·커밋
- feat/stream-followup 03fa95c
