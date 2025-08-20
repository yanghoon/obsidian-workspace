
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

### Act를 이용한 GitHub Actions 로컬 테스트

#### Act 소개 - [Guide](https://nektosact.com/introduction.html)

* Act는 GitHub Actions 워크플로우를 로컬에서 실행하고 테스트할 수 있는 CLI 도구로, Docker를 이용해 GitHub 환경을 시뮬레이션합니다.
* 이를 통해 변경사항을 GitHub에 푸시하지 않고도 빠르게 디버깅과 반복 작업이 가능합니다.
* 개발 시간 단축과 정확한 워크플로우 검증에 효과적입니다.

주요 기능

- **로컬 워크플로우 실행**: `.github/workflows` 내 워크플로우를 인식하여 로컬에서 바로 실행합니다.
- **이벤트 시뮬레이션**: 기본 `push` 외에도 `pull_request`, `workflow_dispatch` 등 다양한 이벤트를 지정해 실행할 수 있습니다.
- **Docker 이미지 관리**: 러너 환경(`runs-on`)에 맞는 Docker 이미지를 자동으로 다운로드하고 컨테이너에서 실행합니다.
- **사용자 설정 지원**: `.actrc` 파일로 기본 옵션(이벤트, 플랫폼 등)을 미리 설정할 수 있습니다.
- **환경 변수 및 시크릿 전달**: GitHub Actions 환경과 동일하게 시크릿과 환경 변수를 전달해 테스트할 수 있습니다.

#### 1. Act 설치 (install.sh 스크립트 사용)

* [Installation](https://nektosact.com/installation/index.html#installation)*

```bash
curl --proto '=https' --tlsv1.2 -sSf https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash
```

#### 2. workflow_dispatch inputs 설정: payload.json 활용 및 예제

- `-e, --eventpath` 옵션을 통해 워크플로우 실행에 필요한 값을 JSON 파일로 제공할 수 있습니다.
- 수동 트리거(`workflow_dispatch`) 워크플로우에 정의된 입력값들도 로컬에서 테스트할 수 있습니다.
- 예를 들어, `payload.json`에 다음과 같이 입력합니다.

```json
{
  "inputs": {
    "input-name": "Hello, Act!"
  }
}
```

- Act 실행 시 `-e` 옵션에 `payload.json` 파일 경로를 지정합니다.

```bash
act workflow_dispatch -e payload.json
```

#### 3. .actrc 기반 설정

- 워크플로우 실행 시 사용할 기본 옵션을 `.actrc`에 작성하여 환경을 미리 설정 할 수 있습니다.
- 명령어 입력 시 매번 이벤트 정보, 플랫폼 등을 직접 명시하지 않아도 됩니다.
- 작업 디렉터리 루트에 `.actrc` 파일을 만들고 다음과 내용을 추가합니다.

```bash
-P ubuntu-latest=nektos/act-environments-ubuntu:18.04
-e payload.json
```

### 4. Workflow 목록 조회 및 Workflow 실행 방법

- 기본적으로 Act는 `.github/workflows/` 내 모든 워크플로우를 인식합니다.
- 다음 명령어로 실행 가능한 이벤트 목록을 확인할 수 있습니다.

```bash
act -l
```

- 특정 워크플로우 파일을 지정해 실행하려면,

```bash
act -W .github/workflows/main.yml
```

- 워크플로우 내 특정 Job만 실행하려면,

```bash
act -j test
```

- 이벤트 지정(ex: push, pull_request, workflow_dispatch)을 함께 사용 가능합니다.

```bash
act push
	act workflow_dispatch -e payload.json -W .github/workflows/main.yml
```
