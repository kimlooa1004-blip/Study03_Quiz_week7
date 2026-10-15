---
description: 고친 내용을 새 브랜치에 커밋하고 --no-ff로 master에 병합한 뒤 깃허브에 올린다
argument-hint: [브랜치 주제, 비우면 변경 내용에서 정함]
allowed-tools: Read, Grep, Glob, Bash(git status:*), Bash(git branch:*), Bash(git checkout:*), Bash(git add:*), Bash(git commit:*), Bash(git merge:*), Bash(git push:*), Bash(git log:*), Bash(git diff:*), Bash(node -e:*)
---

지금까지 고친 내용을 저장(커밋)하고, `master`에 합치고(`--no-ff`), 깃허브에 올립니다(푸시). 이 명령을 부른 것이 곧 커밋·병합·푸시를 해도 된다는 요청입니다.

인수: `$ARGUMENTS` (브랜치 주제. 비어 있으면 변경 내용에서 짧게 정합니다.)

## 지킬 것

- **파일을 고치지 않습니다.** 이 명령은 이미 고친 내용을 옮길 뿐입니다. `Edit`, `Write`를 쓰지 않습니다.
- `git config`를 바꾸지 않습니다. 커밋 이름과 이메일은 이 폴더에 이미 설정된 값을 그대로 씁니다.
- `--force`, `--force-with-lease`, `--no-verify`, `reset --hard`, 히스토리를 고쳐 쓰는 명령을 쓰지 않습니다.
- 커밋 메시지는 `feat:`, `fix:`, `docs:`, `content:`, `refactor:` 중 하나로 시작하는 영어 한 줄과, 필요하면 본문으로 씁니다.
- 병합은 항상 `git merge --no-ff`입니다. 합친 기록이 따로 남게 하기 위해서입니다.

## 순서

1. **상태 밝히기:** `git status -sb`와 `git branch`로 지금 브랜치, 바뀐 파일, 있는 브랜치 이름을 보여 줍니다. 바뀐 파일이 없으면 **여기서 멈추고** "커밋할 변경이 없습니다."라고 알립니다.
2. **점검:** 저장소 루트에서 아래 점검 명령을 실행합니다. 통과하지 못하면(`OK N개 통과`가 아니면) **여기서 멈추고** 실패한 항목을 보여 줍니다. 커밋하지 않습니다.

   ```
   node -e "const r=require('./script.js').selfCheck(); console.log(r.failures.length ? r.failures.join('\n') : 'OK ' + r.passed + '개 통과'); process.exit(r.failures.length ? 1 : 0)"
   ```
3. **작업 브랜치 정하기:** `master`에서 시작합니다. 현재 브랜치가 `master`가 아니고 병합되지 않은 작업이 있으면 그 브랜치를 그대로 쓰고, 그렇지 않으면 새 브랜치를 만듭니다. 이름은 주제를 영어 소문자와 `-`로 쓴 `feat-…`, `fix-…`, `docs-…`, `content-…` 꼴이며, **이미 있는 브랜치 이름과 겹치면** 끝에 `-2`, `-3`을 붙입니다. 어느 브랜치에서 작업하는지 먼저 밝힙니다.
4. **커밋:** 바뀐 파일 중 앱 파일 4개(`index.html`, `style.css`, `script.js`, `questions.js`), 문서(`PRD.md`, `IMPL-PLAN.md`, `CLAUDE.md`), `.claude/commands/`의 명령 파일만 이름을 하나씩 적어 `git add`합니다. 그 밖의 파일이 섞여 있으면 담지 말고 어떤 파일인지 보고합니다. 커밋 번호를 적어 둡니다.
5. **병합:** `master`로 옮겨 `git merge --no-ff 작업브랜치 -m "Merge branch '…': 한글 한 줄 설명"`을 실행합니다. 충돌이 나면 **여기서 멈추고** 충돌한 파일을 보고합니다(해결하지 않습니다). 병합 커밋 번호를 적어 둡니다.
6. **푸시:** `master`의 상위 저장소(upstream)로 `git push`를 실행합니다. upstream이 없으면 **멈추고** 어떤 원격이 있는지 알립니다. 실패하면 이유를 그대로 보고하고 다시 시도하지 않습니다.
7. **이력 확인:** `git log --graph --oneline -8`을 보여 주고, 커밋 번호, 병합 커밋 번호, 원격이 로컬과 같은지(`git status -sb`)를 한 줄씩 보고합니다.

## 멈추는 경우

- 바꾼 파일이 없을 때(1단계), 자체 점검이 통과하지 못했을 때(2단계), 병합 충돌이 났을 때(5단계), 푸시할 곳이 없거나 푸시가 실패했을 때(6단계)
- 멈춘 경우 어느 단계에서 왜 멈췄는지 한 줄로 알립니다.
