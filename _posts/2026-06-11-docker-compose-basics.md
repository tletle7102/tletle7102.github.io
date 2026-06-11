---
title: "Docker Compose (여러 컨테이너를 한 번에 띄우는 도구)"
categories:
  - Docker
tags:
  - Docker
  - Docker Compose
last_modified_at:
---

# Docker Compose

컨테이너 하나 띄우는 건 `docker run`으로 한다는데, 서비스 하나 돌리는데 컨테이너가 여러 개 필요하다고 함. 그걸 한 번에 관리하는 게 Docker Compose라고 해서 정리함


## 1. 문제: 컨테이너가 한두 개가 아니다

웹 서비스 하나를 돌리려면 보통 이런 컨테이너들이 같이 필요함.

- 프론트엔드(nginx)
- 백엔드(스프링)
- 데이터베이스(PostgreSQL)

이걸 `docker run`으로 하나씩 띄우면, 명령어가 길고 옵션도 많아서 매번 치기 힘듦. 순서도 신경 써야 하고, 서로 연결하는 것도 복잡함.


## 2. 해결: 설정 파일에 다 적어두고 한 번에 띄운다

Docker Compose는 **띄울 컨테이너들을 파일 하나(`docker-compose.yml`)에 다 적어두고, 명령어 한 줄로 전부 띄우는 도구**임.

```yaml
# docker-compose.yml
services:
  frontend:
    image: my-frontend
    ports:
      - "80:80"

  backend:
    image: my-backend
    ports:
      - "8080:8080"

  db:
    image: postgres:16
```

이렇게 적어두면, 어떤 컨테이너를 어떤 설정으로 띄울지가 파일에 다 정리됨.


## 3. docker compose up -d --build

가장 많이 쓰는 명령어임. 옵션을 하나씩 뜯어보면 별거 아님.

```
docker compose up -d --build
```

- `up` : `docker-compose.yml`에 적힌 컨테이너들을 전부 띄워라.
- `-d` : 백그라운드로 띄워라(detached). 터미널을 안 잡아먹음.
- `--build` : 띄우기 전에 이미지를 새로 빌드해라. (코드 바뀌었을 때 필요)

즉 "코드 새로 빌드해서, 파일에 적힌 컨테이너들 전부, 백그라운드로 띄워라"는 한 줄임.

내릴 때는 `docker compose down`임.


## 4. 그래서 뭐가 좋은가

- 명령어 한 줄로 여러 컨테이너를 한 번에 관리함.
- 설정이 파일로 남아서, 누가 보든 "이 서비스는 컨테이너 3개로 돈다"가 한눈에 보임.
- 컨테이너끼리 연결도 자동으로 해줌(같은 네트워크로 묶어줌).


## 5. 정리

- 서비스 하나에 컨테이너 여러 개가 필요한 경우가 많음.
- Docker Compose는 그걸 `docker-compose.yml` 한 파일에 적어두고 한 번에 띄우는 도구.
- `docker compose up -d --build` = 빌드해서, 다 띄우되, 백그라운드로.

> Dockerfile이 "컨테이너 하나를 어떻게 만들지"라면, docker-compose.yml은 "그 컨테이너들을 어떻게 같이 굴릴지"를 적는 파일임.
