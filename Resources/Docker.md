#docker-compose #composen #healthchek #depends_on

# Docker Compose

## Build with Podman

Podman 환경에서 `docker compose build` 명령어를 사용할 때, 인터넷 프록시(proxy)가 설정된 환경에서 Podman의 `buildx` 기능이 오류를 일으킬 수 있습니다. 

Podman은 Docker와 호환되는 CLI를 제공하지만, 내부적으로 `buildx`라는 빌드 확장 기능을 기본 사용합니다. `buildx`는 빌드 캐시 관리, 멀티플랫폼 빌드 등을 지원하는 고도화된 도구이나, 프록시 환경에서는 네트워크 관련 설정 문제로 오류가 발생할 수 있습니다. 이러한 오류는 빌드 실패나 네트워크 연결 문제로 이어집니다. 따라서, `buildx` 기능을 비활성화하여 기본 빌드 방식으로 전환하면 이러한 문제를 회피할 수 있습니다.

1. 터미널에서 `DOCKER_BUILDKIT=0` 환경 변수를 설정하여 빌드킷(BuildKit) 기능을 끕니다. `buildx`도 빌드킷 기반이므로 함께 비활성화됩니다.  
2. 이후 `docker compose build` 명령어를 실행합니다.

예시 명령어:

```bash
DOCKER_BUILDKIT=0 docker compose build
```

- 추가 사항 (확인중):  
  - 프록시 환경의 경우 `~/.docker/config.json`에 HTTP/HTTPS 프록시 설정을 정확히 지정하는 것도 중요합니다.
  - `buildx`를 완전히 비활성화하는 옵션은 직접 제공되지 않지만, BuildKit 끄기로 우회합니다.

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

필수 설정 여부:  
* healthcheck는 반드시 지정해야 하는 설정이 아니며, 생략 시 기본값으로 자동 점검하지 않습니다.
* healthcheck를 활성화하려면 최소한 test 항목을 명시해야 하며, 나머지 옵션들은 기본값으로 설정됩니다.
* test를 제외한 기본 값은 아래와 같습니다. [3]
	- interval (간격): 30초
	- timeout (타임아웃): 30초
	- retries (재시도 횟수): 3회
	- start_period (시작 유예 기간): 0초 (버전 3.4 이상에서만 지원하며, 설정하지 않으면 기본 0초)

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
- 별도의 조건(`condition`)을 지정하지 않으면 기본값은 **`service_started`** 상태와 동일하며, 이는 컨테이너가 시작된 상태만 확인합니다. [1][2][3]

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