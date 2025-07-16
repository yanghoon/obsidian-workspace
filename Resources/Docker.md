#docker-compose #composen #healthchek #depends_on

# Docker Compose

## Healthcheck

### Docker Compose의 healthcheck 기능 설명

Docker Compose의 **healthcheck** 기능은 컨테이너 내에서 특정 명령어를 주기적으로 실행하여 서비스 상태를 자동으로 점검하는 것입니다. 컨테이너가 정상적으로 동작하는지 확인할 수 있어, 문제 발생 시 빠르게 대응할 수 있습니다. [1][2][3][4][5]

```yaml
services:
  web:
    image: nginx
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

주요 옵션과 역할은 다음과 같습니다:

- **test**: 건강 상태를 검사할 명령어를 지정합니다. 예를 들어 `["CMD", "curl", "-f", "http://localhost"]` 같은 명령을 실행하여 서비스 응답 여부를 체크합니다.
- **interval**: healthcheck를 실행하는 주기입니다(예: 30초).
- **timeout**: 검사 명령의 최대 대기 시간으로, 이 시간을 초과하면 실패로 간주합니다(예: 10초).
- **retries**: 연속 실패 횟수로, 이 횟수만큼 실패해야 `unhealthy` 상태로 변경됩니다(예: 3회).
- **start_period**: 컨테이너 시작 후 healthcheck를 시작하기 전의 유예 시간으로, 이 기간 동안은 실패가 상태에 영향을 주지 않습니다(예: 40초).

컨테이너의 health 상태는 크게 세 가지로 구분됩니다:
- **starting**: 시작 후 유예 기간, 검사 실패해도 무시
- **healthy**: 검사 명령이 정상적으로 완료된 상태
- **unhealthy**: 지정한 재시도 횟수만큼 검사에 실패한 상태

이 기능으로 서비스의 정상 여부를 자동으로 체크하고, 예를 들어 `depends_on` 설정과 결합하여 의존 서비스가 건강할 때만 다음 서비스가 시작되도록 할 수 있습니다.

### `depends_on` 과의 관계

Docker Compose에서 **healthcheck**와 **depends_on**은 서비스 간 의존 관계를 관리하는 데 상호 보완적으로 작용하는 기능입니다.

- **healthcheck**는 컨테이너의 서비스가 실제로 정상 동작하는지 주기적으로 검사하는 기능입니다. 테스트 명령이 성공적으로 실행되면 컨테이너는 `healthy` 상태가 됩니다.

- **depends_on**는 한 서비스가 다른 서비스보다 먼저 시작되어야 한다는 의존성을 Docker Compose에 알려줍니다. 기본적으로는 컨테이너가 "시작(started)" 상태인지 여부만 판단합니다.

두 기능의 관계 요약:

| 기능         | 역할                                                         | 특징 및 제한                                                      |
|-------------|------------------------------------------------------------|----------------------------------------------------------------|
| healthcheck | 서비스 상태를 검사하여 `healthy` 여부를 판단                 | 서비스가 실제 동작 가능한 상태인지를 확인 가능                       |
| depends_on  | 서비스 시작 순서 및 의존도를 정의                            | 기본은 컨테이너 실행 여부만 판단함, 상태 확인은 추가 조건 필요          |

`depends_on`에서 다음과 같은 **condition** 옵션을 사용하면 healthcheck 상태에 따라 시작 조건을 설정할 수 있습니다:

- `service_started`: 단순히 컨테이너가 시작된 상태
- `service_healthy`: 컨테이너가 healthcheck를 통과해 `healthy` 상태일 때만 의존 서비스 시작

예:

```yaml
services:
  service_a:
    depends_on:
      service_b:
        condition: service_healthy  # service_b가 정상 상태일 때만 시작
      service_c:
        condition: service_started  # service_c가 시작만 되었으면 시작
  service_b:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 5s
      timeout: 3s
      retries: 3
      start_period: 20s
  service_c:
    ...
```

이렇게 하면 의존하는 서비스가 단순히 실행 중인 것을 넘어서 실제 사용 가능한 상태가 되었을 때 다음 서비스가 시작되어 안정적인 환경을 구성할 수 있습니다.

실제 운영 환경에서는 `depends_on`과 `healthcheck`를 적절히 결합하여 서비스 간 종속성과 상태를 관리하는 것이 중요하며, 그렇지 않으면 예를 들어 데이터베이스가 준비되지 않은 상태에서 웹 서비스가 실행되어 오류가 발생할 수 있습니다[1][2][4]. 

또한, `depends_on` 조건을 사용하지 않거나 분산 환경에서는 직접 healthcheck에 의존하는 논리를 애플리케이션 내부나 외부 스크립트로 구현하기도 합니다[3][5].