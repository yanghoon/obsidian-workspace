#database #rdb

## Test
* https://hub.docker.com/_/postgres

```yaml
version: '3.1'

services:
  postgresql:
    image: postgres
    restart: always
    ports:
      - 5432:5432
    environment:
    . POSTGRES_USER: root
      POSTGRES_PASSWORD: root
    volumes:
      - ./data/postgres:
  adminer:
    image: adminer
    restart: always
    ports:
      - 8080:8080
```