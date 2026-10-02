# P19 readme-sync — README 를 현재 main 의 실제 동작에 맞춤

## 결론
채택 제안. README 두 절(한국어·English)을 main(작업 중 main 이 08d5248 → 6c0eea7 로 바뀌어 합침: #1 번들 조정, #2 PLY·synth 인자·이미지 버퍼 검증,
#9 매칭 보강, #3 사진 이웃 선택, #5 점진 스트림, #8 깊이 맵 융합, 왜곡 보정 병합 상태)의
실제 명령행 출력·합성 출력·공개 API 에 맞췄다. 아직 main 에 없는 `run`·`verify` 명령과 그 출력 폴더 구조(preview/refined/snapshots)는
문서에서 뺐다(명령행에 아직 없다). 번들 조정(`skylens_core::ba`) 사용법 절, 밀집 준비·융합·점진 스트림 라이브러리 절
(`undistort`·`view_selection`·`fusion`·`stream`, 출력 폴더 구조는 `stream::write_outputs` 기준), PLY 형식 절을 새로 넣었다. README 의 명령은 모두 실제로 실행해
문서의 출력·종료 코드와 같음을 확인했고, Rust 예시 6개 × 2 절 = 12 조각은 임시 크레이트에서 그대로 컴파일된다.
두 절은 소제목 10개가 같은 순서로 대응하고, 코드 블록은 주석·자리표시 이름 말고는 줄 단위로 같다.

## 수치
| 확인 | 기대(README) | 결과 |
|---|---|---|
| 인자 없음 / 모르는 명령 `bogus` | 사용법(표준 오류), 종료 코드 2 | 같음 |
| `--version` | `skylens-stream 0.1.0`, 0 | 같음 |
| `ply-info ok.ply`(2점, 쓰기 형식과 같은 헤더) | `points 2`, `nan false`, 0 | 같음 |
| `ply-info cut.ply`(2점 헤더 + 1점 반) | 오류 메시지, 1 | "vertex 데이터가 잘림: 54 바이트 필요, 32 바이트 있음", 1 |
| `synth <폴더> 320` | 사용법, 2 | 같음 |
| `synth <폴더> 8 8` | 사용법, 2 | 같음 |
| `synth <폴더> 64 48`, `synth <폴더> 320 180` | `views 240`, 0 | 같음 |
| `synth <폴더>`(기본 960×540) | `views 240`, 0 | 같음, JPEG 960×540 240장 |
| synth 출력 트리 | `images/cam{F,R,L}_NNNN.jpg`(평평), `gps.txt`, `truth/cameras.txt` | 같음, 영상 240장 |
| `gps.txt` 한 줄 | 확장자 없는 이름, 위도, 경도, 고도 | `camF_0000 37.499992088 127.000020169 77.380` |
| `truth/cameras.txt` 한 줄 | 이름 fx fy cx cy 폭 높이 + R 9 + t 3 | 16 칸 |
| 두 절 소제목 수 | 같음 | 10 / 10, 같은 순서 |
| Rust 예시 컴파일 | 12 조각 오류 0 | 오류 0, 경고는 조각이 만든 변수를 쓰지 않는다는 unused 만 |

## 방법
- 실행: 작업 트리에서 `cargo run --release -p skylens-stream -- <명령>`(바이너리 패키지 이름은 `skylens-stream`; `skylens-cli` 라는 패키지는 없다).
  PLY 시험 파일은 쓰기 형식(이진 little-endian, x y z nx ny nz float32 + red green blue uint8) 헤더로 2점을 만든 것과,
  같은 헤더에 1.5 점 분량만 붙인 잘린 파일. 4 코어 측정 기계, 동시 부하 있음.
- 예시 코드 컴파일 확인: 저장소 밖 임시 크레이트(`/tmp/readme-examples`, `skylens-core` 를 작업 트리 경로로 의존, nalgebra 0.33).
  README 의 ```rust 블록을 정규식으로 그대로 뽑아 각 블록을 함수 본문에 넣고, 블록이 앞 단계에서 받는다고 가정하는 변수
  (width·height·rgb_bytes, views·feats_a·feats_b, x1·x2·f·inliers·k, n1·n2·k, num_views·pose01,
  groups·poses·camera_group·points)만 함수 인자로 준 뒤 `cargo build --release --offline`. 블록 본문은 손대지 않으므로
  README 를 고친 뒤 같은 스크립트를 다시 돌리면 된다. 생성 스크립트:
  ```python
  import re
  s=open('README.md').read()
  blocks=re.findall(r"```rust\n(.*?)```", s, re.S)   # 12개(절마다 6개)
  P=[...블록별 인자 목록, 위 변수 6묶음...]
  for i,b in enumerate(blocks):
      out.append(f"pub fn ex{i}({P[i%6]}) -> Result<(), Box<dyn std::error::Error>> {{\n{b}Ok(())\n}}")
  ```
  `cargo test --doc` 은 README 가 크레이트 문서에 포함되지 않아 쓰지 않았다.
- 두 절 대응: 소제목 수·순서 비교, 각 코드 블록에서 명령·`use`·`let` 줄만 뽑아 두 절을 줄 단위로 비교. 차이는 주석과
  자리표시 이름(`<파일.ply>`/`<file.ply>` 등)뿐.
- 고친 점
  - 개발 중 안내: 지금 명령행 도구에 `ply-info`·`synth` 만 있다고 명시. `run`·`verify` 명령과 '출력' 절(미병합 기능) 삭제.
  - 설치: `cargo run --release -p skylens-stream -- <명령>` 추가.
  - 사용법: `synth` 의 표준 출력(`views 240`)과 쓰기 실패 종료 코드 1, `--version`, 인자 없음·모르는 명령의 종료 코드 2.
  - 입력: `gps.txt` 이름은 확장자 없음(`camF_0000`), 위도·경도 단위(도), 고도(m); `truth/cameras.txt` 의 R 은 세계→카메라.
  - 본질 행렬 RANSAC 조각: 쓰지 않는 `recover_pose` 가져오기를 빼고 쓰는 `RansacConfig` 가져오기를 넣었다(단독으로 컴파일되게).
  - 번들 조정 절 신설: `BaProblem`·`Observation`·`BaOptions`(손실, 내부 파라미터 마스크 순서 `INTRINSIC_NAMES`, 게이지 카메라,
    `max_tracks`, `max_iterations = 0` 의 평가만 동작)·`BaReport` 필드.
  - 밀집 준비·융합·점진 스트림 절 신설(코드 조각 없이 함수 이름·역할, `write_outputs` 출력 파일 이름은 `preview_name`·`refined_name`·`snapshot_name` 과 대조).
  - PLY 형식 절 신설: 쓰기 형식과 읽기 규칙(vertex 첫 원소, 이름으로 속성 찾기, 없는 법선·색은 0).

## 남은 문제
- F-020 의 `Cargo.toml` `rust-version = "1.88"` 표기는 이 묶음의 맡은 파일(README.md) 밖이라 넣지 않았다. README 두 절은 1.88 이상으로
  적혀 있다. `cargo +1.88 build` 확인도 하지 않았다(측정 기계에 1.88 도구 사슬 없음 확인 안 함).
- 새 라이브러리 절(undistort·view_selection·fusion·stream)은 코드 조각을 넣지 않아 컴파일 확인 대상이 아니다.
- 검토 중 PR(편대 합성, 정렬, verify, run 등)이 병합되면 README 의 사용법·출력 절을 다시 맞춰야 한다.
- README 예시 컴파일 확인이 저장소 안에서 자동으로 돌지 않는다. 문서 시험으로 묶으려면 core 크레이트에
  `#[doc = include_str!(...)]` 류 연결이 필요한데 맡은 파일 밖이다.

## 제품 브랜치·커밋
- feat/readme-sync: 4c4920a(README 맞춤), a45d58b(origin/main 합침), d4804c4(새 병합 모듈 절)
