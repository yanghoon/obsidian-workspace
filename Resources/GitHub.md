
## Actions

### Custom Github Actions 개발 (NodeJS)

#### 1. 액션 메타데이터 (`action.yml`) 작성

- 입력(inputs)과 출력(outputs)을 정의해야 워크플로우가 인식하고 값을 전달할 수 있습니다.
- 기본 실행 환경과 메인 스크립트를 지정합니다.

```yaml
name: 'Example Action'
description: 'npm 없이 순수 JS 액션 예시'

inputs:
  input-name:
    description: '입력 예시'
    required: true

outputs:
  output-name:
    description: '출력 예시'

runs:
  using: 'node12'
  main: 'index.js'
```

#### 2. 워크플로우 예제 (`.github/workflows/main.yml`)

- 이 YAML 파일은 언제, 어떤 작업 환경에서, 어떤 액션을 실행할지를 정의합니다.
- 반드시 `.github/workflows` 폴더에 위치해야 인식됩니다.

```yaml
name: Test Action

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run custom JS action
        uses: ./
        with:
          input-name: 'Hello'
```

#### 3. 순수 Node.js로 입력 읽기 (input)

- `@actions/core` 미사용 시 입력은 환경변수를 통해 전달됩니다.
- 입력명 `input-name`은 대문자로 변환되고 하이픈은 언더바로 바뀌어 `INPUT_INPUT-NAME` 환경변수가 됩니다.

```js
const input = process.env['INPUT_INPUT-NAME'] || '';
console.log(`입력값: ${input}`);
```

#### 4. 순수 Node.js로 출력 설정 (output)

- 출력은 환경변수 `GITHUB_OUTPUT`가 가리키는 파일에 `key=value` 형태로 쓰면 됩니다.

```js
const fs = require('fs');
const outputPath = process.env['GITHUB_OUTPUT'];
if (outputPath) {
  fs.appendFileSync(outputPath, `output-name=Processed ${input}\n`);
} else {
  console.error('GITHUB_OUTPUT 환경변수가 설정되어 있지 않습니다.');
}
```

#### 5. 실패 상태 표시 (setFailed) 구현

- 실패를 알리는 기본 방법은 오류 메시지를 `console.error`로 출력하고, `process.exitCode`를 1 이상으로 설정하는 것입니다.

```js
try {
  // 작업 코드
  throw new Error('에러 발생');
} catch (error) {
  console.error(`Action failed: ${error.message}`);
  process.exitCode = 1;
}
```
