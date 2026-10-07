# dataset-gap-ranges: 사진 없는 격자를 구간으로 담아 공백 길이와 무관하게 처리한다

## 결론
- 사진이 하나도 없는 격자 프레임을 한 칸씩 목록으로 풀지 않고 구간(첫 프레임, 간격, 개수)
  하나로 `Dataset::skipped` 에 담는다. 메모리·시간·CLI 출력이 공백 길이와 무관하다.
- 프레임 0·5000000 만 두고 `--stride 1 --max-skip-run 10000000` 으로 목록만 돌리면 0.01 초,
  최대 상주 메모리 약 10 MB, 표준 출력 164 바이트다(이전은 12.8 초, 약 804 MB, 248 MB 출력).
- 기존 동작은 그대로다: 위치 목록, 오류 판정(허용 초과 → SkipRun), `skipped_frames() == [6]`,
  CLI `skipped 1 (frames 6)` / `skip frame 6 missing camR`.
- 연속이 모두 사진 없는 격자이면 오류 문구가 "사진이 하나도 없는 프레임 N곳 연속(… STRIDE·번호 간격 확인)"
  으로 갈린다. `SkipRun` 에 `absent_only` 가 늘었다.

## 수치 표
| 입력 | 이전 | 이후 |
|---|---|---|
| 프레임 0·5000000, STRIDE 1, 허용 10^7 (CLI 목록만) | 12.8 s, RSS 약 804 MB, 출력 248 MB | 0.01 s, RSS 약 10 MB, 출력 164 B |
| 같은 입력의 `skipped` | 4999999 개 (카메라 이름 Vec 포함) | 구간 1개 {frame 1, step 1, count 4999999} |
| 프레임 0·4000000000, 허용 u32 최대 | 메모리 고갈 | 구간 1개, 1 초 미만(단위 시험) |
| 6·9 전체 누락 (허용 2) | 시험 없음 | Ok, skipped == [6, 9], 구간 1개 |
| 6·9·12 전체 누락 | 시험 없음 | SkipRun{6, 12, 3, 2}, absent_only |
| camR 3 + 6·9 전체 누락 | 시험 없음 | SkipRun{3, 9, 3, 2}, absent_only 아님 |
| 0..34, 30·33 전체 누락·34 는 camF 만 | 시험 없음 | 위치 10곳, skipped == [30, 33] |
| CLI 에서 세 카메라 0006 삭제 | 시험 없음 | `positions 13`, `skipped 1 (frames 6)`, `skip frame 6 missing camF,camR,camL` |

시험: skylens-core dataset 29개 통과, skylens-stream `run` 시험 8개 통과.

## 방법
- `SkippedFrame` 에 `step`, `count` 를 더했다(카메라가 일부 빠진 프레임은 `count == 1`).
  `add_absent` 는 산술로 개수를 세어 진행 중 연속에 합치고 구간을 하나만 넣는다. 허용을 넘는
  연속은 오류이므로 이전처럼 개수 검사가 끝난 뒤 오류로 끝난다.
- `skipped_count()` 는 산술, `skipped_frames()` 는 필요할 때 푸는 목록(크기가 건너뛴 수에 비례함을 문서에 적음).
- CLI 는 구간을 `1..=4999999 step 1` 로 요약하고 한 줄 `skip frames … missing …` 을 낸다.
  개수 1 은 이전 출력 그대로다.
- 진행 중 연속 이어받기와 끝 격자 `+1` 은 새 시험(섞인 연속, 끝 격자)이 단언한다.

## 남은 문제
- `skipped_frames()` 는 여전히 모두 푸는 함수라 큰 공백에서 호출하면 비용이 크다(현재 호출처는 시험뿐).
- 번호가 성기게 저장된 폴더(0, 4, 8, …)를 `--stride 1` 로 읽으면 이제 "사진이 하나도 없는 프레임 N곳 연속
  (STRIDE·번호 간격 확인)" 으로 구분되어 실패한다. 자동으로 간격을 추정하지는 않는다.
- 연속 판정의 `count` 는 usize 포화 덧셈이다. 32 비트 환경에서 허용 u32 최대는 따로 시험하지 않았다.
- 한 번에 돌린 것은 core `dataset` 시험과 cli `run` 시험까지이며 워크스페이스 전체는 돌리지 않았다.

## 제품 브랜치·커밋
feat/dataset-gap-ranges 4f902dae956e9d8432475047e23eb8628beea564
