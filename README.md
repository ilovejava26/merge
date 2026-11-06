# merge-practice

Git의 **브랜치 생성 → 병합 → 충돌 해결 → 원격 저장소 Push → Pull Request** 흐름을 한 번에 복습하기 위한 실습 저장소입니다.

- 원격 저장소: <https://github.com/ilovejava26/merge.git>
- 기본 브랜치: `main`
- 문서 작업 브랜치: `release`

## 1. 저장소 구성

| 파일 | 설명 |
| --- | --- |
| `1.txt` | 첫 번째 커밋으로 추가한 빈 파일 |
| `2.txt` | 두 번째 커밋으로 추가한 빈 파일 |
| `3.txt` | `fix` 브랜치에서 추가한 빈 파일 |
| `4.txt` | 브랜치 분기와 병합 충돌을 실습한 파일 |
| `5.txt` | `release` 브랜치에서 추가한 빈 파일 |
| `README.md` | 저장소 개요와 실습 요약 (현재 문서) |
| `exercise.md` | 처음부터 다시 따라 해 볼 수 있는 단계별 실습 가이드 |

`4.txt`의 최종 내용은 다음과 같습니다.

```text
sayonara4
bye3
hello2
nihao5
```

## 2. 브랜치 현황

| 브랜치 | 역할 | 마지막 커밋 |
| --- | --- | --- |
| `main` | 기본 브랜치. `feat/awesome2` 병합 지점 | `5fa92a8` |
| `release` | 문서와 `5.txt`를 추가해 `main`으로 PR을 보내는 브랜치 | `5.txt` + 문서 커밋 |
| `feat/login` | 빈 `4.txt`를 만들고 `fix`를 병합한 브랜치 | `82777d6` |
| `fix` | 빈 `3.txt`를 추가한 브랜치 | `5a2f083` |
| `feat/awesome1` | `4.txt`의 첫 줄을 `bye3`로 바꾼 브랜치 | `8b157f3` |
| `feat/awesome2` | `4.txt`의 첫 줄을 `sayonara4`로 바꾸고 `nihao5`를 추가한 뒤 `feat/awesome1`을 병합한 브랜치 | `5fa92a8` |

## 3. 문서 추가 전까지의 커밋 흐름

```text
* 18cd6ca (release) 5.txt
*   5fa92a8 (main, feat/awesome2) Merge branch 'feat/awesome1' into feat/awesome2
|\
| * 8b157f3 (feat/awesome1) 4.txt a1
* | 295eb83 4.txt a2
|/
* 9e74ae1 4.txt main
*   82777d6 (feat/login) Merge branch 'fix'
|\
| * 5a2f083 (fix) 3.txt
* | 00acb58 4.txt
|/
* 9c98243 2.txt
* 5610229 1.txt
```

`README.md`와 `exercise.md`를 추가하는 문서 커밋은 위 그래프의 `release` 위에 새로 쌓입니다.

시간 순서로 정리하면 다음과 같습니다.

1. `1.txt`, `2.txt`를 차례로 커밋해 기본 이력을 만들었다.
2. `fix` 브랜치에서 빈 `3.txt`를 커밋했다.
3. `feat/login` 브랜치에서 빈 `4.txt`를 커밋했다.
4. `fix`를 `feat/login`에 병합해 첫 번째 merge commit(`82777d6`)을 만들었다.
5. `main`의 `4.txt`에 `hello1`, `hello2` 두 줄을 추가했다.
6. `feat/awesome1`과 `feat/awesome2`를 각각 `main`에서 분기했다.
   - `feat/awesome1`: `hello1` → `bye3`
   - `feat/awesome2`: `hello1` → `sayonara4`, `nihao5` 추가
7. `feat/awesome1`을 `feat/awesome2`로 병합하면서 같은 첫 줄을 서로 다르게 수정해 **충돌**이 발생했다.
8. 충돌을 직접 해결해 `sayonara4`, `bye3`를 모두 남기고 merge commit(`5fa92a8`)을 만들었다.
9. `main`을 해당 merge commit까지 fast-forward했다.
10. `release` 브랜치를 만들고 빈 `5.txt`를 추가했다.
11. `release` 브랜치에 이 문서와 `exercise.md`를 추가하고 `main` 대상 Pull Request를 생성한다.

## 4. 이번 실습에서 익힌 것

- `git switch -c`로 브랜치를 만들고 이동하기
- 빈 파일도 Git이 정상적으로 추적한다는 점 (`e69de29`는 빈 blob의 해시)
- fast-forward 병합과 3-way merge의 차이
- merge commit이 만들어지는 조건
- 같은 파일의 같은 줄을 양쪽에서 수정했을 때 충돌이 발생하는 원리
- 충돌 마커(`<<<<<<<`, `=======`, `>>>>>>>`)를 읽고 수동으로 해결하기
- `git log --graph --oneline --all --decorate`로 브랜치 구조 확인하기
- 원격 브랜치 Push와 `main` 대상 Pull Request 생성하기

## 5. 공부하는 방법

1. `README.md`의 커밋 흐름과 브랜치 표를 먼저 읽습니다.
2. `exercise.md`를 위에서부터 따라 하며 빈 디렉터리에서 동일한 이력을 재현합니다.
3. 각 Lab의 **확인** 명령으로 실제 결과를 비교합니다.
4. 마지막의 복습 질문에 먼저 답한 뒤 접힌 정답을 확인합니다.
5. 이 저장소에서는 아래 명령으로 실제 이력을 다시 확인할 수 있습니다.

```bash
git log --graph --oneline --all --decorate
git branch -a
git show 5fa92a8 --stat
git show 5fa92a8:4.txt
```

## 6. Pull Request

`release` 브랜치의 문서 및 `5.txt` 변경을 `main`에 반영하기 위해 PR을 생성합니다.

```bash
gh pr create --base main --head release \
  --title "docs: README와 exercise 학습 자료 추가" \
  --body "저장소 개요와 지금까지의 Git 브랜치·병합 실습을 정리한 문서를 추가합니다."
```

PR이 병합되면 `release`의 문서가 `main`에 들어가고, 이후 `release` 브랜치는 정리할 수 있습니다.
