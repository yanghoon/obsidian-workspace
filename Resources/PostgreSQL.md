#docker #container 

# PostgreSQL Container Healthcheck 설정

이 설정은 PostgreSQL 컨테이너가 준비(ready)되었는지 확인하고, 다른 서비스가 정상 연결할 수 있게 돕습니다.

```yaml
services:
  db:
    image: postgres:17
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
    ports:
      - 5432:5432
    volumes:
      - ./data/pgdata:/var/lib/postgresql/data
```

- `healthcheck`는 `pg_isready` 명령으로 DB가 연결 가능 상태인지 주기적으로 테스트합니다.  
- `retries`가 지정된 횟수 내에 성공하지 못하면 컨테이너가 unhealthy 상태가 됩니다.  
- 이를 통해 `depends_on` 옵션에서 `condition: service_healthy`를 활용해 다른 서비스가 준비된 DB에만 의존하도록 할 수 있습니다