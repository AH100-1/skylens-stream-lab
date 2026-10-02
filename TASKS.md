# 작업 목록

위에서부터 순서대로 한다. 각 작업 `<이름>` = 연구 `experiment/<이름>` + 제품 `feat/<이름>` 브랜치·PR 한 쌍.
감독이 검증 후 병합하며 체크·날짜·제품 커밋을 적는다.
막혀서 건너뛰면 `[~]` 로 표시하고 이유를 적는다. 감독이 순서를 바꿀 수 있다.

- [x] T00 `scaffold` — 2026-10-01 병합, 제품 9dc03d9 — cargo 워크스페이스, `crates/core`(수학·카메라·PLY 입출력), `crates/cli`, CI 통과. PLY 쓰기→읽기 왕복 테스트.
- [x] T01 `synthetic-scene` — 2026-10-01 병합, 제품 9dc03d9 — SPEC §6 합성 장면 생성기(이미지 렌더 + 정답 카메라·점 + GPS). 이후 모든 테스트의 기반.
- [x] T02 `camera-model` — 2026-10-01 병합, 제품 9dc03d9 — 핀홀 + k1,k2,p1,p2 투영·역투영·왜곡 보정, 야코비안. 수치 미분과 1e-6 일치 테스트.
- [x] T03 `features` — 2026-10-01 병합, 제품 cde0fb4 (실험 노트 보충 필요) — 가우시안 차분 특징점 + 128차원 기술자. 합성 장면 회전·축척 변화에서 재검출률·매칭 정확도 측정.
- [x] T04 `matching` — 2026-10-01 병합, 제품 97cf72c (실험 노트 보충 필요) — 시간 이웃 + 카메라 간 짝 생성, 비율 검사, 기하 검증 RANSAC. 정답 대비 정상 짝 비율.
- [x] T05 `two-view` — 2026-10-01 병합, 제품 7b2e6e6 (실험 노트 보충 필요) — 8점·5점 본질 행렬, 상대 자세 복원, 삼각측량. 잡음별 회전 오차(도).
- [x] T06 `rotation-averaging` — 2026-10-01 병합, 제품 bc51103 — 상대 회전 그래프 → 전역 회전. 정답 대비 각도 오차.
- [ ] T07 `translation-averaging` — 방향 제약 위치 추정 + 삼각측량 → 초벌 모델. 카메라 240대 등록 확인.
- [x] T08 `bundle-adjustment` — 2026-10-01 병합, 제품 04e4d29 (F-034~F-037 남음) — 희소 Levenberg–Marquardt + 슈어 보수, 강건 손실, 카메라별 공유 내부 파라미터. 재투영·중심 오차.
- [x] T09 `gps-align` — 2026-10-01 병합, 제품 f87549a — 위경도 → 동-북-위, Umeyama 닮음 변환 + 이상치 제외. 정답 대비 잔차.
- [ ] T10 `dense-patchmatch` — 시점별 PatchMatch 깊이·법선(CPU, rayon 병렬). 정답 깊이 대비 오차.
- [ ] T11 `depth-fusion` — 다시점 일관성 융합 → 점군 + 법선 + 색. 정답 표면 대비 최근접 거리.
- [x] T12 `progressive-stream` — 2026-10-01 병합, 제품 f2b658b (임시 닮음 변환 → P08 병합 뒤 교체, F-055~F-057 남음) — 구역 분할, 초벌/정밀, 3D 점 대응 정렬, 잔상 걸러내기, 스냅샷, manifest. SPEC §3.7~3.8.
- [x] T13 `verify` — 2026-10-02 병합, 제품 0afc818 (F-066 통합 시험·F-089·F-180·F-181 남음) — SPEC §4 검증을 `skylens-stream verify <폴더>` 로. 실패 시 종료 코드 1.
- [ ] T14 `perf` — 구간별 시간 측정, 병렬화. 합성 240장 전체 시간 기록. 이후 GPU 백엔드 설계 노트.

## 병렬 묶음 (동시에 진행)

아래 묶음은 **서로 다른 파일만 고치도록** 나눴다. 한 번에 여러 묶음을 동시에 진행한다.
묶음마다 제품 `feat/<묶음>` 브랜치(별도 작업 트리), 연구 `experiment/<묶음>` 노드(괄호 안 부모 아래) 하나씩.
README 는 P04 만 고친다. 다른 묶음은 사용법 변화를 PR 본문에 적고, P04 가 모아서 반영한다.
CLI `main.rs` 는 하위 명령 연결 한 줄씩만 추가한다(충돌 최소).

| 묶음 | 내용 | 맡는 파일 | 부모 노드 | 선행 |
|---|---|---|---|---|
| [x] P01 `formation-scene` (2026-10-01, 제품 1609550; formation-scene-check 2026-10-01 제품 e9f7187) | F-029·F-024 합성 장면을 실측 편대 배치로 | `synth.rs` | synthetic-scene | — |
| [x] P02 `matching-hardening` (2026-10-01, 제품 08d5248; matching-followup 2026-10-02 제품 506b9d4·01c3f98, F-139·F-148·F-196·F-197·F-210·F-211 남음) | F-030·F-021·F-003·F-031 | `matching.rs` | matching | — |
| [x] P03 `two-view-hardening` (2026-10-02, 제품 2bf4795 — two-view-twin 에 포함되어 병합, F-033·F-145·F-148·F-191~F-195 남음) | F-032·F-033·F-026·F-027·F-028 | `two_view.rs` | two-view | — |
| [x] P04 `io-robustness` (2026-10-01, 제품 3b0fa9c; io-cleanup 2026-10-01 제품 d4432c7; io-followup 2026-10-02 제품 026fb3c) | F-016~F-020·F-022·F-023, README 정리(한·영) | `ply.rs`, `camera.rs`, `features.rs`(from_rgb), CLI 인자, `README.md` | scaffold | — |
| [x] P05 `rotation-averaging-hardening` (2026-10-01, 제품 9f111f2; rotation-averaging-robust 2026-10-02 제품 ca34c6f, F-047·F-138·F-171~F-173 남음) | F-006~F-009 | `rotation_averaging.rs` | rotation-averaging | — |
| P06 `translation-averaging` | T07 방향 제약 위치 추정 + 다시점 삼각측량 | 새 `translation_averaging.rs`, `triangulation.rs` | rotation-averaging | — |
| [x] P07 `bundle-adjustment` (2026-10-01, 제품 04e4d29; ba-hardening 2026-10-01 제품 ec18464, F-036·F-164~F-167 남음) | T08 희소 LM + 슈어 보수, 강건 손실, 카메라별 공유 내부 파라미터, 트랙 10만 제한 | 새 `ba.rs` | two-view | — |
| [x] P08 `similarity-align` (2026-10-01, 제품 f87549a; align-robust 2026-10-02 제품 474bcf8, F-095·F-153·F-198~F-200 남음) | T09 Umeyama 닮음 변환 + 반복 트리밍, GPS→동-북-위 정렬 | 새 `align.rs` | rotation-averaging | — |
| [x] P09 `view-selection` (2026-10-01, 제품 80f9a86; dense-prep 2026-10-02 제품 60cbc2d, F-201~F-204 남음) | T10a 이웃 8장 점수·깊이 범위·왜곡 보정(960px) | 새 `view_selection.rs`, `undistort.rs` | camera-model | — |
| P10 `patchmatch` | T10b 시점별 PatchMatch 깊이·법선(rayon) | 새 `patchmatch.rs` | camera-model | P09 인터페이스 |
| [x] P11 `depth-fusion` (2026-10-01, 제품 6c0eea7; fusion-hardening 2026-10-02 제품 bbd82d8, F-069·F-113·F-178·F-179 남음) | T11 왕복 투영 걸러내기 + 3장 동의 합치기 → 점군 | 새 `fusion.rs` | camera-model | P10 인터페이스 |
| [x] P12 `dataset-io` (2026-10-02, 제품 20a293c, F-206 남음) | 실제 데이터 읽기(`images/cam{F,R,L}`, `gps.txt`, STRIDE) + `run` 명령 뼈대 | 새 `dataset.rs`, CLI `run` | scaffold | — |
| [x] P13 `progressive-stream` (2026-10-01, 제품 f2b658b; stream-hardening 2026-10-02 제품 6f14401; stream-followup 2026-10-02 제품 c7b7287, F-055·F-056·F-065·F-066·F-175~F-177 남음) | T12 구역 분할, 초벌/정밀, 공유 관측 닮음 정렬, 잔상 1.5m 걸러내기, 스냅샷·manifest | 새 `stream.rs` | two-view | P08 인터페이스 |
| [x] P14 `verify` (2026-10-02, 제품 0afc818; verify-followup 2026-10-02 제품 50fce8a, F-089·F-180·F-181·F-187 남음) | T13 `verify <폴더>` — SPEC §4 일곱 항목, 실패 시 종료 코드 1 | 새 `verify.rs`, CLI `verify` | scaffold | — |
| [x] P15 `benchmarks` (2026-10-01, 제품 5f2e886; benchmarks-scale 2026-10-01 제품 dc9bb55; benchmarks-followup 2026-10-02 제품 25d71cf; benchmarks-args 2026-10-02 제품 70219c3, F-040·F-128·F-130·F-132·F-149·F-205 남음) | T14 구간별 시간 측정 틀(합성 240장) | 새 `benches/` | fast-matching | — |

### 묶음 사이 인터페이스 (먼저 이 모양으로 맞춘다)
- `align::Similarity { s: f64, r: Rotation3<f64>, t: Vector3<f64> }`, `align::umeyama(src, dst) -> Option<Similarity>`,
  `align::robust_similarity(src, dst, iters=5, floor_m=0.3) -> Option<(Similarity, Vec<bool>, f64 /*잔차 중앙*/)>`
- `view_selection::select_neighbors(views, points, k=8) -> Vec<Vec<usize>>`, `view_selection::depth_range(view, points) -> (f64, f64)`
- `patchmatch::DepthMap { w, h, depth: Vec<f32>, normal: Vec<[f32;3]>, cost: Vec<f32> }`, `patchmatch::estimate(ref_view, neighbors, range, cfg) -> DepthMap`
- `fusion::fuse(views, depth_maps, cfg{reproj_px=1.0, depth_rel=0.01, min_views=3}) -> PointCloud`
- 선행 묶음이 아직 병합 전이면 위 모양의 임시 구현(단순·정확)으로 시험하고, 병합 뒤 바꾼다.
