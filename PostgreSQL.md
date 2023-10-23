#database #rdb

## Test
* https://hub.docker.com/_/postgres

```yaml
version: '3.1'

services:
  postgresql:
    image: postgres:9.5
    restart: always
    container_name: postgres
    ports:
      - 5432:5432
    environment:
    . POSTGRES_USER: root
      POSTGRES_PASSWORD: root
    volumes:
      - ./data/postgres/:/var/lib/postgresql/
  adminer:
    image: adminer
    restart: always
    ports:
      - 8080:8080
```