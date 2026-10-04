# register-issues: 등록 누락을 issues 에 남기고 verify 기준 열의 사진 수를 실제 값으로

## 결론
- 구역에서 등록 안 된 사진이 있으면 report `issues` 에 구역별 한 줄을 남긴다. 전부 등록되면 이슈 없음.
  형식: `region 0: registered 27/81 (F 0/27, R 0/27, L 27/27), pair graph components 3`
- verify 의 첫 항목 기준 열이 고정 `(240/240)` 대신 report 의 `registered.total` 로 표시된다.

## 수치 표
| 장면 | 항목 | 전 | 후 |
|---|---|---|---|
| 320x240, stride 3 (81장) | report issues | 등록 누락 언급 없음 | `region 0: registered 27/81 (F 0/27, R 0/27, L 27/27), pair graph components 3` |
| 320x240, stride 3 (81장) | verify registered 기준 열 | `(240/240)` | `(81/81)` |
| 240장 장면 | verify registered 기준 열 | `(240/240)` | `(240/240)` |

시험: skylens-core --lib verify 13 통과, register_issue 3 통과, skylens-stream --test verify 25 통과, --test run 6 통과.

## 방법
- 구역 초벌 등록 직후 구역 자기 사진(도우미 제외)의 등록 여부를 카메라 번호(3·위치+카메라 % 3 = 0/1/2 → F/R/L)별로 센다.
- 짝 그래프 덩어리 수: 매칭된 짝을 변으로 한 합집합-찾기, 구역 자기 사진을 하나라도 가진 덩어리만 센다.
- 실제 합성 장면 확인(320x240 `synth`, `run --stride 3`, 옵션은 README 공통): 등록 27/81, 덩어리 3.
  이 장면에서는 0번 카메라와 1번 카메라가 0 장, 2번 카메라가 27 장 등록되었다(예시 문구의 F/R/L 순서와 숫자 배치는 다르지만 같은 27/81).

## 남은 문제
- 카메라 0/1/2 와 F/R/L 이름의 대응은 코드에 따로 정의가 없어 순서대로 가정했다.
- 정밀 단계에서 추가로 빠진 사진은 이슈에 넣지 않았다(초벌 등록 기준).
- 실제 run 결과 확인은 시험으로 넣지 않았다.

## 제품 브랜치·커밋
feat/register-issues, feb05e5
