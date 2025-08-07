
## Standard API

### NodeJS에서 json 파일 읽기

Node.js에서 특정 경로의 JSON 파일을 읽기 위해 `fs` 모듈의 `readFile` 또는 `readFileSync` 메서드를 사용하고, 읽은 데이터를 `JSON.parse`로 파싱하면 됩니다.

예시 코드(비동기 방식):

```javascript
const fs = require('fs');

fs.readFile('경로/파일명.json', 'utf8', (err, data) => {
  if (err) {
    console.error(err);
    return;
  }
  const jsonData = JSON.parse(data);
  console.log(jsonData);
});
```

예시 코드(동기 방식):

```javascript
const fs = require('fs');

try {
  const data = fs.readFileSync('경로/파일명.json', 'utf8');
  const jsonData = JSON.parse(data);
  console.log(jsonData);
} catch (err) {
  console.error(err);
}
```

- `'utf8'` 인코딩을 지정해줘야 Buffer가 아닌 문자열로 읽습니다.
- `JSON.parse`를 이용해 문자열을 자바스크립트 객체로 변환합니다.
