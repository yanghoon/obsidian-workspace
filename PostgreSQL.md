#database #rdb

## Test

### Docker Compose

```yaml
# https://hub.docker.com/_/postgres
version: '3.1'
services:
  postgresql:
    image: postgres:9.5
    restart: always
    container_name: postgres
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: reviewdb
      POSTGRES_USER: gerrit
      POSTGRES_PASSWORD: gerrit
    volumes:
      - ./data/postgres/:/var/lib/postgresql/data
      - ./data/init.sql:/docker-entrypoint-initdb.d
  #adminer:
  #  image: adminer
  #  restart: always
  #  ports:
  #    - "8080:8080"
```

### Kubernetes

#todo 
