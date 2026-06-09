---
title: "Docker Multi-stage Build (Stage 나누는 이유와 COPY --from의 의미)"
categories:
  - Docker
tags:
  - Docker
  - Build
last_modified_at:
---

# Docker Multi-stage Build

Dockerfile을 보다가 `FROM`이 두 번 나오고 `COPY --from=build` 같은 게 있어서 당황했음. 이게 멀티 스테이지 빌드라는 건데, 왜 굳이 단계를 나누는지 정리함


## 1. 문제: 빌드 도구는 무겁다

React를 빌드하려면 Node.js랑 npm이 필요함. 근데 빌드가 끝나고 나면 결과물은 `dist` 폴더(정적 파일)뿐임. **실제 서비스할 때 Node.js는 더 이상 필요 없음.** nginx가 `dist`만 서빙하면 되니까.

그런데 한 단계로만 만들면, 다 끝난 이미지 안에 무거운 Node.js랑 빌드 찌꺼기가 다 남아 있음. 이미지가 쓸데없이 커짐.


## 2. 해결: 단계를 둘로 나눈다

멀티 스테이지 빌드는 이미지 만드는 과정을 **2단계로 쪼개는 것**임.

- **Stage 1 (빌드 단계)**: Node.js로 React를 빌드해서 `dist`를 만듦. 무거워도 상관없음. 어차피 버릴 거니까.
- **Stage 2 (실행 단계)**: 가벼운 nginx만 깔고, Stage 1에서 만든 `dist`만 가져옴.

```dockerfile
# ---- Stage 1: 빌드 ----
FROM node:20 AS build      # 이 단계 이름을 'build'라고 지음
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build          # 결과물: /app/dist

# ---- Stage 2: 실행 ----
FROM nginx:alpine          # 가벼운 nginx
COPY --from=build /app/dist /usr/share/nginx/html
#         ↑ 여기가 핵심
```


## 3. COPY --from=build 가 무슨 뜻인가

`COPY --from=build` 는 **"Stage 1(이름이 build인 단계)에서 만든 결과물을 가져와라"** 는 뜻임.

```
COPY --from=build  /app/dist            /usr/share/nginx/html
        ↑               ↑                        ↑
   build 단계에서   거기서 만든 dist를      nginx가 보는 폴더로 복사
```

즉, 무거운 Node.js가 있던 Stage 1은 통째로 버리고, **거기서 나온 알맹이(`dist`)만 쏙 빼와서** 가벼운 Stage 2에 넣는 거임.


## 4. 그래서 뭐가 좋은가

- 최종 이미지에는 nginx랑 `dist`만 있음. Node.js, npm, 소스코드 다 없음.
- 이미지가 훨씬 가벼워짐(수백 MB → 수십 MB).
- 가벼우면 배포도 빠르고, 보안상으로도 불필요한 게 없어서 좋음.

요리로 치면, Stage 1은 재료 다듬고 요리하는 지저분한 주방이고, Stage 2는 완성된 음식만 담은 깨끗한 접시임. 손님(서버)한테는 접시만 내가면 됨.


## 5. 정리

- 빌드에 필요한 도구(Node.js)와 실제 실행에 필요한 것(nginx+dist)은 다름.
- 그래서 Stage 1에서 빌드하고, Stage 2에는 결과물만 옮겨서 가볍게 만듦.
- `COPY --from=build`가 "앞 단계 결과물만 가져오기"의 핵심 문법임.
