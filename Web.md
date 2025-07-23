#jwt

## JWT Test 방법

### 1. 테스트용 JWT 토큰 생성

테스트용 JWT 토큰은 JWT 생성 도구 (온라인 JWT 생성기, OpenSSL, JWT 라이브러리 등)를 사용하거나, 개발 환경에서 간단히 생성할 수 있습니다. 예를 들어, 서명 없이 payload만 Base64로 인코딩한 토큰을 간단히 만들 수도 있습니다.

* [JSON Web Tokens - jwt.io](https://jwt.io/) (Encode/Decode)

### 2. curl 명령을 통한 JWT 인증 테스트

- **JWT 토큰 없이 요청 시**

```bash
curl http://jwt.example.com
```

응답: 401 Unauthorized (인증 실패)

- **잘못된 JWT 토큰 사용 시**

```bash
curl -I -H "token: InvalidTokenValue" http://jwt.example.com
```

응답: 401 Unauthorized, `error="invalid_token"` 포함

- **유효한 JWT 토큰 사용 시**

```bash
JWT="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.e30.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"
curl -H "Authorization: Bearer ${JWT}" http://jwt.example.com
```

응답: 200 OK (인증 성공)
