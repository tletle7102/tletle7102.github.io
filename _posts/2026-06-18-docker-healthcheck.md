---
title: "Healthcheck (컨테이너가 정상인지 자동으로 확인하기)"
categories:
  - Docker
tags:
  - Docker
  - Healthcheck
last_modified_at:
---

# Healthcheck

컨테이너가 떠 있다고 해서 정상으로 일하는 건 아니라는 걸 알게 됨. "떠 있음"과 "제대로 동작함"은 다른 얘기였고, 그걸 구분해주는 게 Healthcheck임


## 1. 컨테이너가 떠 있어도 죽어있을 수 있다

`docker ps`로 보면 컨테이너가 "Up(실행 중)"이라고 나옴. 근데 이게 함정임.

- 프로세스는 떠 있는데, 안에서 앱이 멈춰서 응답을 못 할 수 있음.
- DB 연결이 끊겨서 요청을 다 실패시키고 있을 수 있음.

겉으론 "Up"인데 실제론 일을 못 하는 상태. 사람으로 치면 자리에 앉아는 있는데 일은 안 하는 거임. 이걸 자동으로 잡아내야 함.


## 2. Healthcheck = 주기적으로 건강 체크

Healthcheck는 **컨테이너한테 주기적으로 "너 살아있어? 일할 수 있어?" 하고 물어보는 기능**임.

보통 앱에 건강 확인용 주소(예: `/actuator/health`)를 두고, 거기에 요청을 날려봄. 잘 응답하면 healthy, 응답 안 하면 unhealthy로 표시됨.

```dockerfile
HEALTHCHECK --interval=30s --timeout=10s --retries=3 --start-period=30s \
  CMD curl -fsS -o /dev/null http://localhost/ || exit 1
```

- `--interval=30s` : 30초마다 한 번씩 체크.
- `--timeout=10s` : 10초 안에 응답 없으면 실패로 침.
- `--retries=3` : 3번 연속 실패해야 unhealthy로 판정.
- `--start-period=30s` : 처음 뜰 때 30초는 봐줌(기동 시간 고려).
- `CMD curl ...` : 실제로 체크하는 명령. localhost에 요청해서 응답 보는 거임.


## 3. unhealthy면 자동으로 살린다

Healthcheck가 unhealthy를 감지하면, 보통 자동으로 컨테이너를 재시작하게 묶어둠(autoheal 같은 도구). 사람이 안 보고 있어도 알아서 살아남.

```
컨테이너 멈춤 → healthcheck 실패 → unhealthy 판정 → 자동 재시작 → 복구
```


## 4. 작은 함정: 체크 도구 고르기

체크할 때 `curl`을 쓰는 게 안전함. `wget`(특히 가벼운 alpine 이미지 안의 것)은 `localhost`를 IPv6로 먼저 해석하려다 실패하는 함정이 있음. 앱이 IPv4로만 듣고 있으면 연결이 거부돼서, 멀쩡한 컨테이너가 계속 재시작되는 사고가 남.

> 그래서 Healthcheck는 `curl`로 하는 걸 기본으로 잡는 게 좋음. 정 wget을 써야 하면 `127.0.0.1`로 IPv4를 명시.


## 5. 정리

- "떠 있음(Up)"과 "제대로 동작함"은 다름.
- Healthcheck는 주기적으로 요청을 날려 진짜 동작 여부를 확인함.
- unhealthy면 자동 재시작 등으로 복구할 수 있음.
- 체크는 `curl`로 하는 게 안전함(wget IPv6 함정 회피).
