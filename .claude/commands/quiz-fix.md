---
description: 점검에서 걸린 문항 하나를 고치고, 그 문항을 다시 점검한다 (/quiz-validate의 짝)
argument-hint: [문항 id, 예: science-07]
allowed-tools: Read, Grep, Glob, Edit, Bash(node -e:*)
---

`/quiz-validate`가 짚은 문항 **하나**를 고치고, 고친 뒤 그 문항을 다시 점검합니다. `/quiz-validate`는 점검만 하고 고치지 않으므로, 고치는 일은 이 명령이 맡습니다.

인수: `$ARGUMENTS` (고칠 문항 id 하나. 비어 있거나 `questions.js`에 없는 id이면 **여기서 멈추고** 쓸 수 있는 id 형식(`history-NN`, `geography-NN`, `science-NN`, `culture-NN`)을 알려 줍니다.)

## 지킬 것

- 고치는 파일은 `questions.js` 하나이고, 인수로 받은 **그 문항만** 고칩니다. 다른 문항, 다른 파일, `.claude/commands/` 파일은 고치지 않습니다.
- `CLAUDE.md`의 "문항 작성 규칙" 10개와 "이 저장소에서 더한 규칙"을 지킵니다. 규칙은 이 파일에 옮겨 적지 않습니다. 읽고 따릅니다.
- **정답과 사실은 바꾸지 않는 것이 기본입니다.** 보기의 길이·형식, 문장, 부정문, 해설 길이처럼 모양만 고칠 수 있는 어긋남을 고칩니다. 사실을 바꿔야 고쳐지는 어긋남(정답이 둘이 될 수 있는 문항, 해설이 사실과 맞지 않는 문항)은 고치지 말고 이유와 함께 보고합니다.
- **출처를 지어내지 않습니다.** 규칙 4(출처 2곳 이상)는 출처를 실제로 열어 확인해야만 고칠 수 있습니다. 열어서 확인하지 못했으면 `source`를 바꾸지 않고 "출처 확인이 필요함"이라고 보고합니다.
- 문항 id, `category`, 문항 형식(`id`, `category`, `question`, `choices` 4개, `answer`, `explanation`, `source`)은 그대로 둡니다. `choices`의 순서를 바꾸면 `answer`를 가리키는 보기가 같은 보기를 가리키도록 맞춥니다.
- 커밋하지 않습니다. `git` 명령을 쓰지 않습니다. 커밋과 병합은 `/quiz-ship`에 맡깁니다.

## 순서

1. **문항 읽기와 점검:** `questions.js`에서 그 문항을 읽고, `.claude/commands/quiz-validate.md`의 순서와 기준으로 그 문항만 점검해서 어긴 규칙 번호를 정리합니다. 기계 점검은 아래 명령을 쓰되 `ID`를 인수로 바꿉니다(읽기만 합니다).

   ```
   node -e "const s=require('./script.js'); const q=require('./questions.js').QUESTIONS; const x=q.find(y=>y.id==='ID'); console.log(x ? JSON.stringify(x,null,1) : '없는 id'); console.log(x ? s.validateQuestions([x]).join('\n') || '검사기 위반 없음' : '')"
   ```
   어긴 규칙이 없으면 **고치지 말고** "어긋난 곳을 찾지 못했습니다."라고 보고하고 끝냅니다.
2. **고치기:** 어긴 규칙마다 가장 작은 수정으로 고칩니다. 정답 보기와 해설의 내용이 계속 맞는지(규칙 6)를 다시 대조합니다.
3. **다시 점검:** 같은 기계 점검과 아래 전체 점검을 다시 실행하고, 그 문항을 다시 읽어 규칙을 하나씩 확인합니다.

   ```
   node -e "const r=require('./script.js').selfCheck(); console.log(r.failures.length ? r.failures.join('\n') : 'OK ' + r.passed + '개 통과'); process.exit(r.failures.length ? 1 : 0)"
   ```
   통과하지 못하면 **멈추고** 어떤 점검이 왜 실패했는지 보고합니다. 되돌릴지는 사람이 정합니다.
4. **보고:** 아래 표로 고친 곳을 보여 줍니다. 고치지 못한 어긋남은 이유와 함께 따로 적습니다.

   | 문항 id | 규칙 번호 | 전 | 후 |
   |---|---|---|---|

   끝에 "커밋하지 않았습니다. 저장하려면 `/quiz-ship`을 실행하세요."라고 덧붙입니다.

## 멈추는 경우

- 인수가 비었거나 없는 id일 때(맨 앞), 점검이 통과하지 못했을 때(3단계)
- 사실이나 출처를 확인해야만 고칠 수 있는 어긋남은 고치지 않고 보고에 남깁니다.
