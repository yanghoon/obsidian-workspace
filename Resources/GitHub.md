
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

### Git Actions with GraphQL

```js
const https = require('https');

const token = process.env['GITHUB_TOKEN'];
const owner = 'your-github-username-or-org';  // repo 소유자 이름
const repoName = 'your-repo-name';             // 생성하려는 repo 이름

if (!token) {
  console.error('GITHUB_TOKEN 환경변수가 필요합니다.');
  process.exitCode = 1;
  process.exit();
}

// GraphQL 쿼리: repo 존재 여부 확인
const queryCheckRepo = `
  query {
    repository(owner: "${owner}", name: "${repoName}") {
      id
    }
  }
`;

// GraphQL 뮤테이션: repo 생성
const mutationCreateRepo = `
  mutation {
    createRepository(input: {name: "${repoName}", visibility: PRIVATE}) {
      repository {
        id
        name
        url
      }
    }
  }
`;

// GraphQL 요청 함수
function graphqlRequest(query) {
  const options = {
    hostname: 'api.github.com',
    path: '/graphql',
    method: 'POST',
    headers: {
      'User-Agent': 'node.js',
      'Authorization': `bearer ${token}`,
      'Content-Type': 'application/json',
    },
  };

  const body = JSON.stringify({ query });

  return new Promise((resolve, reject) => {
    const req = https.request(options, (res) => {
      let data = '';
      res.on('data', chunk => data += chunk);
      res.on('end', () => {
        if (res.statusCode >= 200 && res.statusCode < 300) {
          resolve(JSON.parse(data));
        } else {
          reject(new Error(`Status Code: ${res.statusCode}, Body: ${data}`));
        }
      });
    });
    req.on('error', reject);
    req.write(body);
    req.end();
  });
}

(async () => {
  try {
    // repo 존재 체크
    const checkResult = await graphqlRequest(queryCheckRepo);
    if (checkResult.data.repository) {
      console.log(`Repository "${repoName}" already exists.`);
    } else {
      // repo 없으면 생성
      const createResult = await graphqlRequest(mutationCreateRepo);
      const repo = createResult.data.createRepository.repository;
      console.log(`Repository created: ${repo.name} (${repo.url})`);
    }
  } catch (error) {
    console.error('GitHub GraphQL 요청 실패:', error.message);
    process.exitCode = 1;
  }
})();
```

