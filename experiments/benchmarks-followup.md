# benchmarks-followup — 측정 틀 기본값, 구조 시험 정리, 회전 평균 진단 값 (T14, F-149·F-132·F-131·F-130·F-128·F-040)

## 결론
- **F-132: 인자 없는 `cargo bench --bench pipeline` 의 기본값을 빠른 규모로 바꿨다**(24장 = 위치 8 × 3, 480×270, 반복 3,
  번들 조정 점 3000; 예전 `--quick` 과 같음). SPEC 기준 규모(240장, 960×540, 반복 3, 점 20000)는 `--full` 을 줄 때만 돈다.
  `--quick` 은 예전 이름으로 그대로 받는다. `--mode ba-scale` 은 위치를 따로 주지 않으면 80(카메라 240)으로 둬서
  기본값 변경이 번들 조정 실제 규모 측정을 줄이지 않게 했다. 틀 머리 주석의 사용법 표에 모드별 예상 시간을 적었다.
- **F-149 (코드 부분): `회전 평균(검증 결과)` 줄 비고에 반환 시점 수/전체와 입력 간선 중 정답 상대 회전과 2° 넘게 다른 것의
  비율을 더했다.** 회전 평균 결과가 무너질 때 원인이 입력 간선(두 시점 결과)인지 평균 쪽인지 표 한 줄로 가를 수 있다.
  `tests/perf_structure.rs` 머리 주석은 `benches/pipeline.rs` 를 가리키게 고쳤다(`grep -rn bench_stages crates` 결과 없음).
- **F-131: 구조 시험을 이름과 맞췄다.**
  - 머리 주석의 "`#[ignore]` 시간 시험" 언급을 지웠다(그런 시험은 없다).
  - `feature_cap_bounds_descriptor_comparisons`: 항상 참이던 `m.len() <= cap / 2`(상호 매칭은 일대일) 단언을 지우고,
    상한 없이 검출하면 상한을 넘는 영상 세 장(F 위치 10·11, R 위치 10)에 상한 200 을 적용한 뒤 짝마다 |A|·|B| ≤ cap² 를 단언한다.
  - `ransac_iteration_counts` 는 `matching.rs` 단위 시험과 중복이라 지웠다.
  - `essential_ransac_min_iterations_fixed`: `assert_eq!(ESSENTIAL_MIN_ITERS, 300)` 과 근거 주석(E RANSAC 이 짝당 가장 비싼 구간이고,
    적응 반복 수가 w=0.8·p=0.999 에서 18 회라 대부분의 짝이 바닥값으로 시간이 정해진다). 값을 바꾸면 이 시험이 실패한다.
- **F-130: 검출 해시 시험을 `tests/perf_structure.rs` 에서 뺐다.** 같은 480×270 영상의 해시는 `feat/pixel-convention` 의
  `features.rs` 시험 모듈(745 개, 0x2a1c_72f3_9c5e_a950)에서 한 곳만 고정한다. 그 브랜치가 병합되기 전 main 에는 검출 해시 시험이
  잠시 없다(아래 남은 문제 1).
- **F-128·F-040·F-149(노트 표 재측정)·F-132(`--full` 결과)는 미달.** 이번 작업 시간 동안 4 코어 측정 기계의 부하 평균이
  17~29 였다(다른 빌드·시험 여러 개). 확인 기준이 부하 없는 단독 실행 값(부하 평균 < 2)을 요구하므로 이 조건에서 잰 값으로 표를
  바꾸지 않았다. 이전 노트(benchmarks·benchmarks-scale)의 표는 여전히 부하 아래 값이며 그대로 둔다.

## 수치

| 항목 | 값 | 조건 |
|---|---|---|
| `perf_structure` 시험 수 | 4 (짝 수 선형식, 기술자 비교 상한, 옥타브 σ 상한, E RANSAC 최소 반복) | 이전 5 → 해시·중복 RANSAC 시험 빼고 E 최소 반복 더함 |
| `grep -rn bench_stages crates` | 결과 없음 | 제품 feat/benchmarks-followup |
| 인자 없는 bench 규모 | 24장, 짝 231, 480×270, 반복 3 | 이전 benchmarks 노트 B 표(같은 규모, 부하 아래)에서 렌더 포함 1 분 안팎 |
| `--full` 규모 | 240장, 짝 3663, 960×540, 반복 3 | 미측정 |
| 240장 C 표 재측정(F-128) | 미측정 | 부하 평균 17~29 |
| 번들 조정 240대·트랙 10만 구간별 표(F-040) | 미측정 | 위와 같음, 구간 분해는 `ba.rs` 내부 계측이 필요 |

## 방법
- 기본값 판정: `parse_args` 의 초기값을 위치 8·480×270·점 3000 으로 바꾸고, `--full` 이 위치 80·960×540·반복 3·점 20000 을
  설정한다. `--positions`·`--quick`·`--full` 중 하나라도 주면 위치를 지정한 것으로 본다. 지정이 없고 `--mode ba-scale` 이면 80.
- 간선 진단: 두 시점 결과 간선 (i, j, R_ij) 마다 정답 R_j R_iᵀ 와의 사이 각 ∠(R_ij (R_j R_iᵀ)ᵀ) 를 구해 2° 초과 수/전체를 적는다.
  반환 시점 수는 `AveragingResult::rotations` 중 값이 있는 것의 수.
- 구조 시험은 `cargo test --release --test perf_structure` 로 확인했다(4 통과). a7ad157 에서 `cargo fmt --all --check`·`cargo clippy --all-targets -- -D warnings` 통과, `cargo test --release` 144 통과·0 실패·3 무시(기존 무시 시험).

## 남은 문제
1. 검출 해시 시험은 `feat/pixel-convention` 의 `features.rs` 에만 있다. 그 브랜치가 병합될 때까지 main 에 검출 해시 회귀 시험이 없다.
2. 부하 없는 때(부하 평균 < 2) 다음을 다시 재야 한다: 인자 없는 bench 전체 시간(2 분 기준), `--full`, `--positions 80 --width 320
   --height 180`(F-128 C 표, 검증 통과 짝 수 일치 확인), `--mode detect`, `--mode ba-scale --ba-iters 2 --repeat 1`.
   각 표에 측정 커밋·부하 평균·장면 배치를 적는다.
3. F-040 구간 분해(선형화·슈어·촐레스키)는 공개 API 만으로는 나눌 수 없다. `ba_scale_timing` 과 같은 문제 생성(트랙 길이 상한으로
   평균 길이 약 26)으로 맞추는 일이 남았다.

## 제품 브랜치·커밋
- `feat/benchmarks-followup` a7ad157 (origin/main dc9bb55 에서) — `crates/core/benches/pipeline.rs`, `crates/core/tests/perf_structure.rs`.
