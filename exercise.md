# exercise.md — Git 브랜치·병합 실습 가이드

이 문서는 이 저장소에서 지금까지 진행한 실습을 **처음부터 다시 재현하며 공부**할 수 있도록 정리한 가이드입니다.
각 Lab에는 실행 명령, 확인 방법, 예상 결과가 있습니다.

> **팁**
> - 재현 실습은 이 저장소를 직접 건드리지 말고 **새 디렉터리**에서 진행하세요.
> - 명령은 Git Bash 기준입니다. PowerShell에서는 파일 생성/내용 작성 명령만 바꾸면 됩니다.
> - 커밋 해시는 실행 시점에 따라 달라집니다. 해시가 아니라 **그래프 모양과 파일 내용**을 비교하세요.
> - `main`에 이미 모든 이력이 있으므로, 현재 저장소의 과거를 확인할 때는 `git show <해시>`를 사용합니다.

---

## Lab 0. 실습 환경 만들기

빈 디렉터리에서 새 Git 저장소를 만듭니다.

```bash
mkdir git-merge-lab
cd git-merge-lab
git init
git branch -M main
git config user.name "Your Name"
git config user.email "you@example.com"
```

확인:

```bash
git status
git branch --show-current
```

예상 결과:

```text
On branch main
nothing to commit (create/copy files and use "git add" to track)
main
```

> 이 저장소 자체를 복제해서 확인하려면:
>
> ```bash
> git clone https://github.com/ilovejava26/merge.git
> cd merge
> git log --graph --oneline --all --decorate
> ```

---

## Lab 1. 첫 두 개의 커밋 만들기

빈 파일 `1.txt`, `2.txt`를 차례로 커밋합니다.

```bash
touch 1.txt
git add 1.txt
git commit -m "1.txt"

touch 2.txt
git add 2.txt
git commit -m "2.txt"
```

PowerShell이라면 `touch` 대신 다음을 사용합니다.

```powershell
New-Item 1.txt
New-Item 2.txt
```

확인:

```bash
git log --oneline --graph --decorate
git show --stat HEAD~1
git show --stat HEAD
```

예상 결과:

```text
* <hash> (HEAD -> main) 2.txt
* <hash> 1.txt
```

`1.txt`, `2.txt`는 내용이 0바이트인 빈 파일이지만 Git은 정상적으로 추적합니다. 빈 파일의 blob 해시는 `e69de29`입니다.

### 생각해 보기

- 빈 파일도 커밋 대상이 될까요?
- `git add`와 `git commit`은 각각 무엇을 할까요?

---

## Lab 2. `feat/login`과 `fix` 브랜치 만들기

`main`에서 `fix` 브랜치를 만들고 빈 `3.txt`를 커밋합니다.

```bash
git switch -c fix
touch 3.txt
git add 3.txt
git commit -m "3.txt"
```

다시 `main`으로 돌아와 `feat/login` 브랜치를 만들고 빈 `4.txt`를 커밋합니다.

```bash
git switch main
git switch -c feat/login
touch 4.txt
git add 4.txt
git commit -m "4.txt"
```

확인:

```bash
git log --oneline --graph --all --decorate
git branch -vv
```

예상 구조:

```text
* <hash> (HEAD -> feat/login) 4.txt
| * <hash> (fix) 3.txt
|/
* <hash> (main) 2.txt
* <hash> 1.txt
```

### 포인트

- `git switch -c <브랜치>` = 브랜치 생성 + 이동
- 두 브랜치는 같은 `2.txt` 커밋에서 갈라졌습니다.
- `git branch -vv`의 `*`는 현재 브랜치입니다.

---

## Lab 3. `fix`를 `feat/login`에 병합하기

`feat/login`으로 이동해 `fix`를 병합합니다.

```bash
git switch feat/login
git merge fix
```

서로 다른 파일(`4.txt`와 `3.txt`)만 바꿨기 때문에 충돌 없이 **merge commit**이 만들어집니다.

확인:

```bash
git log --oneline --graph --decorate -5
git show --stat HEAD
```

예상 결과:

```text
*   <hash> (HEAD -> feat/login) Merge branch 'fix'
|\
| * <hash> (fix) 3.txt
* | <hash> 4.txt
|/
* <hash> (main) 2.txt
* <hash> 1.txt
```

그다음 `main`을 `feat/login`까지 fast-forward합니다.

```bash
git switch main
git merge --ff-only feat/login
```

> **fast-forward란?**
> 현재 브랜치가 대상 브랜치의 조상일 때, 새 merge commit 없이 브랜치 포인터만 앞으로 이동하는 병합입니다.

---

## Lab 4. `4.txt`에 내용 추가하기

`main`에서 `4.txt`에 두 줄을 추가합니다.

```bash
git switch main
printf 'hello1\nhello2\n' > 4.txt
git add 4.txt
git commit -m "4.txt main"
```

PowerShell 예시:

```powershell
@("hello1", "hello2") | Set-Content 4.txt
git add 4.txt
git commit -m "4.txt main"
```

확인:

```bash
cat 4.txt          # PowerShell: Get-Content 4.txt
git log --oneline -3
```

예상 내용:

```text
hello1
hello2
```

이제 `4.txt`는 이후 충돌 실습의 **공통 조상(merge base)** 역할을 합니다.

---

## Lab 5. `feat/awesome1`과 `feat/awesome2`로 갈라지기

먼저 `feat/awesome1`을 만들고 첫 줄을 `bye3`로 바꿉니다.

```bash
git switch -c feat/awesome1
printf 'bye3\nhello2\n' > 4.txt
git commit -am "4.txt a1"
```

다시 `main`으로 돌아와 `feat/awesome2`를 만들고, 첫 줄을 `sayonara4`로 바꾸면서 `nihao5`를 추가합니다.

```bash
git switch main
git switch -c feat/awesome2
printf 'sayonara4\nhello2\nnihao5\n' > 4.txt
git commit -am "4.txt a2"
```

확인:

```bash
git log --oneline --graph --all --decorate
cat 4.txt
```

예상 구조:

```text
* <hash> (HEAD -> feat/awesome2) 4.txt a2
| * <hash> (feat/awesome1) 4.txt a1
|/
* <hash> (main) 4.txt main
...
```

### 포인트

- 두 브랜치 모두 `4.txt`의 **같은 첫 줄**을 서로 다르게 수정했습니다.
- `feat/awesome2`는 세 번째 줄 `nihao5`도 추가했습니다.
- 이 상태에서 병합하면 충돌이 발생합니다.

---

## Lab 6. 충돌 해결하며 병합하기 (핵심)

```bash
git switch feat/awesome2
git merge feat/awesome1
```

예상 결과:

```text
Auto-merging 4.txt
CONFLICT (content): Merge conflict in 4.txt
Automatic merge failed; fix conflicts and then commit the result.
```

상태와 파일 내용을 확인합니다.

```bash
git status
cat 4.txt
```

`4.txt`에는 다음과 같은 충돌 마커가 생깁니다.

```text
<<<<<<< HEAD
sayonara4
=======
bye3
>>>>>>> feat/awesome1
hello2
nihao5
```

의미:

| 구간 | 의미 |
| --- | --- |
| `<<<<<<< HEAD` ~ `=======` | 현재 브랜치(`feat/awesome2`)의 내용 |
| `=======` ~ `>>>>>>> feat/awesome1` | 병합해 들어오는 브랜치(`feat/awesome1`)의 내용 |

### 해결 방법

이번 실습에서는 **양쪽 변경을 모두 남기는 것**을 목표로 합니다.
아래처럼 파일을 직접 수정합니다.

```text
sayonara4
bye3
hello2
nihao5
```

그다음 충돌 해결을 기록하고 merge commit을 완성합니다.

```bash
git add 4.txt
git commit
```

`git commit`을 실행하면 기본 merge 메시지 편집기가 열립니다.

```text
Merge branch 'feat/awesome1' into feat/awesome2
```

저장하고 닫으면 merge commit이 만들어집니다. (편집기를 열지 않으려면 `git commit -m "Merge branch 'feat/awesome1' into feat/awesome2"`를 사용해도 됩니다.)

확인:

```bash
git log --oneline --graph --decorate -6
git show HEAD:4.txt
git status
```

예상 결과:

```text
*   <hash> (HEAD -> feat/awesome2) Merge branch 'feat/awesome1' into feat/awesome2
|\
| * <hash> (feat/awesome1) 4.txt a1
* | <hash> 4.txt a2
|/
* <hash> (main) 4.txt main
...
```

```text
sayonara4
bye3
hello2
nihao5
```

### 충돌을 취소하고 싶다면

merge commit을 만들기 **전**이라면 언제든 취소할 수 있습니다.

```bash
git merge --abort
git status
```

### 포인트

- 충돌은 "실패"가 아니라 Git이 자동으로 결정할 수 없는 부분을 사람에게 넘긴 상태입니다.
- 해결 후에는 반드시 `git add`로 해결 표시를 해야 합니다.
- `git commit` 전에 `git status`에서 `both modified`가 사라졌는지 확인하세요.
- 충돌 마커(`<<<<<<<`, `=======`, `>>>>>>>`)가 파일에 남아 있지 않은지 꼭 확인하세요.

---

## Lab 7. `main`을 병합 결과까지 이동하기

```bash
git switch main
git merge --ff-only feat/awesome2
```

이 시점에 `main`과 `feat/awesome2`는 같은 merge commit을 가리킵니다.

확인:

```bash
git log --oneline --graph --all --decorate
git rev-parse main feat/awesome2
```

두 해시가 같으면 성공입니다.

> `git merge feat/awesome2`를 사용해도 fast-forward가 가능하면 같은 결과가 됩니다.
> `--ff-only`는 fast-forward가 불가능할 때 병합을 중단하므로, 의도치 않은 merge commit을 막는 데 유용합니다.

---

## Lab 8. `release` 브랜치와 `5.txt`

```bash
git switch -c release
touch 5.txt
git add 5.txt
git commit -m "5.txt"
```

확인:

```bash
git log --oneline --graph --decorate -3
git show --stat HEAD
```

예상 결과:

```text
* <hash> (HEAD -> release) 5.txt
*   <hash> (main, feat/awesome2) Merge branch 'feat/awesome1' into feat/awesome2
...
```

`5.txt`도 빈 파일이므로 변경 통계에서 `0 insertions(+)`처럼 보일 수 있습니다.

---

## Lab 9. 문서 추가하기

`release` 브랜치에 `README.md`와 `exercise.md`를 추가합니다.

```bash
# 이 저장소의 문서를 참고하거나 직접 작성한 뒤
git add README.md exercise.md
git commit -m "docs: README와 exercise 학습 자료 추가"
```

확인:

```bash
git show --stat HEAD
git diff main..release --stat
```

예상 결과:

```text
 README.md   | ...
 exercise.md | ...
 5.txt       | 0
 3 files changed, ...
```

> `git diff main..release`는 두 브랜치의 끝 지점을 직접 비교합니다.
> `git diff main...release`는 **merge base**와 `release`를 비교하므로, `release`에서 새로 추가된 변경만 보고 싶을 때 더 유용합니다.

---

## Lab 10. 원격 저장소에 Push하고 Pull Request 만들기

### 10-1. 원격 저장소 연결

```bash
git remote add origin https://github.com/<사용자>/<저장소>.git
git remote -v
```

이미 `origin`이 있으면 `git remote set-url origin <URL>`로 주소를 바꿉니다.

### 10-2. 브랜치 Push

```bash
git push -u origin main
git push -u origin release
```

`-u`는 로컬 브랜치와 원격 브랜치를 추적 관계로 연결합니다. 이후에는 `git push`만 실행해도 됩니다.

확인:

```bash
git branch -vv
git status
```

예상 결과:

```text
* release <hash> [origin/release] ...
  main    <hash> [origin/main] ...
```

### 10-3. GitHub CLI로 PR 생성

```bash
gh auth status
gh pr create --base main --head release \
  --title "docs: README와 exercise 학습 자료 추가" \
  --body "저장소 개요와 지금까지의 Git 브랜치·병합 실습을 정리한 문서를 추가합니다."
```

| 옵션 | 의미 |
| --- | --- |
| `--base main` | 변경을 **받을** 대상 브랜치 |
| `--head release` | 변경을 **보내는** 브랜치 |
| `--title` | PR 제목 |
| `--body` | PR 설명 |

PR 확인:

```bash
gh pr view
gh pr view --web
gh pr list --state open
```

### 10-4. PR 병합 후 정리 (선택)

리뷰 후 GitHub에서 merge하거나 CLI로 병합합니다.

```bash
gh pr merge --merge
git switch main
git pull --ff-only
git branch -d release
git push origin --delete release
```

> `git branch -d`는 병합이 끝난 브랜치만 삭제합니다. 병합하지 않은 브랜치는 `-D` 없이는 삭제되지 않습니다.

---

## Lab 11. 전체 이력 검증하기

```bash
git log --graph --oneline --all --decorate
git branch -a
git tag
git status
```

체크리스트:

- [ ] `1.txt` ~ `5.txt`가 모두 존재한다.
- [ ] `4.txt`의 최종 내용이 `sayonara4 / bye3 / hello2 / nihao5`이다.
- [ ] `feat/awesome1`과 `feat/awesome2`가 갈라졌다가 merge commit으로 합쳐졌다.
- [ ] `main`이 그 merge commit을 포함한다.
- [ ] `release`에 문서 커밋과 `5.txt` 커밋이 있다.
- [ ] `origin/main`, `origin/release`가 로컬과 같은 지점을 가리킨다.
- [ ] `release` → `main` PR이 열려 있거나 병합되어 있다.

현재 저장소에서 실제 과거 확인:

```bash
git show 5fa92a8 --stat          # awesome1 → awesome2 merge commit
git show 5fa92a8:4.txt          # 충돌 해결 후의 최종 내용
git show 8b157f3:4.txt          # awesome1 쪽 내용
git show 295eb83:4.txt          # awesome2 쪽 내용
git diff 9e74ae1 5fa92a8 -- 4.txt
```

---

## 복습 질문

먼저 스스로 답한 뒤 펼쳐서 확인하세요.

### Q1. 빈 파일도 커밋할 수 있나요?

<details>
<summary>정답 보기</summary>

네. Git은 파일 내용이 0바이트여도 blob 객체로 저장하고 추적합니다. 이 실습의 `1.txt`, `2.txt`, `3.txt`, `5.txt`가 모두 빈 파일이며, 빈 blob 해시는 `e69de29`입니다. `git show --stat`에서 변경 줄 수가 `0`으로 보이는 것은 정상입니다.

</details>

### Q2. fast-forward merge와 3-way merge의 차이는 무엇인가요?

<details>
<summary>정답 보기</summary>

- **fast-forward**: 현재 브랜치가 대상 브랜치의 조상이어서 새 commit 없이 포인터만 앞으로 이동합니다.
- **3-way merge**: 두 브랜치가 갈라진 뒤 각자 commit이 있어, merge base·현재 브랜치·대상 브랜치 세 지점을 비교해 merge commit을 만듭니다. 양쪽이 같은 부분을 다르게 바꾸면 충돌이 발생합니다.

</details>

### Q3. 이번 실습에서 충돌이 발생한 정확한 이유는 무엇인가요?

<details>
<summary>정답 보기</summary>

`feat/awesome1`과 `feat/awesome2`가 같은 merge base(`4.txt main`)에서 갈라진 뒤, **`4.txt`의 같은 첫 줄**(`hello1`)을 각각 `bye3`와 `sayonara4`로 다르게 수정했습니다. Git은 어느 쪽을 선택해야 할지 자동으로 결정할 수 없어 충돌로 표시했습니다.

</details>

### Q4. 충돌을 해결한 뒤 `git add`는 왜 필요한가요?

<details>
<summary>정답 보기</summary>

`git add`는 "이 파일의 충돌을 해결했다"는 표시입니다. Git은 index에서 충돌 상태를 해제하고, 그때부터 merge commit에 포함할 수 있습니다. `git add` 없이 `git commit`하면 Git이 아직 병합 중이라고 판단해 커밋을 거부합니다.

</details>

### Q5. `git merge --abort`와 `git reset --hard`의 차이는 무엇인가요?

<details>
<summary>정답 보기</summary>

- `git merge --abort`: 진행 중인 merge를 취소하고 merge 시작 전 상태로 돌아갑니다. merge 때문에 생긴 index/working tree 변경을 정리합니다.
- `git reset --hard`: 지정한 commit으로 현재 브랜치와 working tree를 강제로 되돌립니다. merge뿐 아니라 **모든 커밋되지 않은 변경을 잃을 수 있어** 위험합니다.

</details>

### Q6. PR에서 `base`와 `head`는 각각 무엇인가요?

<details>
<summary>정답 보기</summary>

- `base`: 변경을 받을 대상 브랜치 (이번에는 `main`)
- `head`: 변경을 보내는 브랜치 (이번에는 `release`)

즉 `release`의 커밋들을 `main`에 반영해 달라는 요청입니다.

</details>

### Q7. `git diff main..release`와 `git diff main...release`의 차이는 무엇인가요?

<details>
<summary>정답 보기</summary>

- `main..release`: `main`의 끝과 `release`의 끝을 직접 비교합니다. 그 사이에 `main`에만 추가된 변경도 차이로 보일 수 있습니다.
- `main...release`: 두 브랜치의 **merge base**와 `release`를 비교합니다. `release`에서 새로 만든 변경만 보고 싶을 때 적합합니다.

</details>

### Q8. 충돌 해결에서 "정답"은 무엇인가요?

<details>
<summary>정답 보기</summary>

Git이 정답을 정해 주지는 않습니다. 코드/문서의 의도에 따라 한쪽을 선택하거나, 이번 실습처럼 양쪽을 모두 살리거나, 새 내용으로 다시 작성할 수 있습니다. 중요한 것은 **왜 그렇게 해결했는지 설명할 수 있는 것**과, 해결 후 테스트·검토를 거치는 것입니다.

</details>

---

## 자주 만나는 오류

### `fatal: not a git repository`

Git 저장소가 아닌 디렉터리에서 명령을 실행했습니다.

```bash
pwd
ls -a
cd <저장소-경로>
```

### `error: Your local changes to the following files would be overwritten by merge`

커밋하지 않은 변경이 있어 병합이 중단된 상태입니다. 변경을 커밋하거나 stash한 뒤 다시 시도합니다.

```bash
git status
git stash push -m "wip"
git merge <브랜치>
git stash pop
```

### `CONFLICT (content): Merge conflict in ...`

정상적인 충돌 상태입니다. `git status`로 충돌 파일을 확인하고, 파일을 수정한 뒤 `git add` → `git commit` 순서로 진행합니다.

### `gh: Not logged in`

```bash
gh auth login
gh auth status
```

### Push가 거부됨 (`non-fast-forward`)

원격에 로컬에 없는 커밋이 있다는 뜻입니다. 먼저 가져와서 확인합니다.

```bash
git pull --ff-only
# 또는
git fetch origin
git log --oneline --graph --all --decorate
```

> 습관적으로 `git push --force`를 사용하지 마세요. 다른 사람의 커밋을 덮어쓸 수 있습니다. 정말 필요한 경우에는 `--force-with-lease`를 사용합니다.

---

## 명령어 치트시트

```bash
# 브랜치
git branch                     # 로컬 브랜치 목록
git branch -vv                 # 추적 브랜치 포함
git switch <브랜치>            # 브랜치 이동
git switch -c <브랜치>         # 생성 + 이동
git branch -d <브랜치>         # 병합된 브랜치 삭제

# 병합
git merge <브랜치>             # 병합
git merge --ff-only <브랜치>   # fast-forward만 허용
git merge --abort              # 병합 취소
git merge --continue           # 충돌 해결 후 병합 계속

# 상태/이력
git status
git log --oneline --graph --all --decorate
git show <커밋>:<파일>
git diff <A>..<B>
git diff <A>...<B>

# 원격/PR
git remote -v
git push -u origin <브랜치>
git branch -vv
gh pr create --base main --head <브랜치>
gh pr view --web
gh pr merge --merge
```

---

## 최종 목표 상태

```text
* <docs commit> (release) docs: README와 exercise 학습 자료 추가
* <hash> (main 후보) 5.txt
*   <hash> (main, feat/awesome2) Merge branch 'feat/awesome1' into feat/awesome2
|\
| * <hash> (feat/awesome1) 4.txt a1
* | <hash> 4.txt a2
|/
* <hash> 4.txt main
*   <hash> (feat/login) Merge branch 'fix'
|\
| * <hash> (fix) 3.txt
* | <hash> 4.txt
|/
* <hash> 2.txt
* <hash> 1.txt
```

`4.txt` 최종 내용:

```text
sayonara4
bye3
hello2
nihao5
```

이 상태를 만들 수 있다면 브랜치 생성, 분기, merge commit, 충돌 해결, fast-forward, 원격 Push, Pull Request의 기본 흐름을 모두 익힌 것입니다.
