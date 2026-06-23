---
title: "Thymeleaf 잔재 처리 (static, templates 디렉토리 정리)"
categories:
  - Architecture
tags:
  - Thymeleaf
  - Migration
  - Spring Boot
last_modified_at:
---

# Thymeleaf 잔재 처리

React로 다 옮기고 나니 스프링 프로젝트에 옛날 Thymeleaf 파일들이 그대로 남아 있었음. 이걸 왜, 어떻게 정리하는지 정리함


## 1. 어디에 잔재가 남나

Thymeleaf 쓰던 스프링 프로젝트엔 보통 이 두 폴더에 화면 파일이 있음.

```
src/main/resources/
 ├─ static/      ← CSS, JS, 이미지 등 정적 파일
 └─ templates/   ← Thymeleaf HTML 화면들(.html)
```

React로 화면을 다 옮기면 이 폴더의 파일들은 더 이상 안 쓰임. 화면은 이제 React(프론트)가 담당하니까.


## 2. 왜 정리해야 하나

안 지우고 둬도 당장 큰일은 안 남. 근데 남겨두면 문제가 생김.

- **헷갈림**: 다음 사람이 "화면이 templates에 있나? React에 있나?" 하고 혼란스러움.
- **충돌 위험**: 스프링은 `static` 폴더를 자동으로 서빙함. 옛날 `static/index.html`이 남아 있으면, React로 가야 할 요청을 이 옛날 파일이 가로챌 수 있음.
- **빌드 무게**: 안 쓰는 파일이 계속 빌드에 딸려 들어감.

특히 두 번째(옛날 index.html이 가로채는 것)가 실제로 잘 겪는 함정임.


## 3. 어떻게 정리하나

순서대로 안전하게 함.

1. **진짜 안 쓰는지 확인**: `templates/`, `static/`의 파일들이 코드 어디서도 참조 안 되는지 확인. (컨트롤러가 `return "home"` 식으로 templates를 가리키는 곳이 없는지)
2. **Thymeleaf 의존성 정리**: 더 이상 안 쓰면 `build.gradle`에서 `spring-boot-starter-thymeleaf` 제거.
3. **폴더 정리**: `templates/` 통째로 삭제, `static/`에서 옛 화면 파일 삭제.
4. **확인**: 지우고 빌드·실행해서 React 화면이 정상으로 뜨는지 검증.

> 한 번에 다 지우지 말고, 지운 다음 실행해서 멀쩡한지 확인하면서 진행하는 게 안전함.


## 4. 주의: 무관한 코드까지 건드리지 말기

정리한다고 옆에 있는 멀쩡한 설정이나 코드까지 손대면 안 됨. **내가 React로 옮기면서 안 쓰게 만든 것만** 지우는 게 원칙임. 기존부터 있던, 마이그레이션과 상관없는 건 그대로 둠.


## 5. 정리

- Thymeleaf 잔재는 주로 `src/main/resources/static`, `templates`에 남음.
- 안 지우면 헷갈리고, 옛 `static/index.html`이 React 요청을 가로채는 함정이 있음.
- 참조 확인 → 의존성 제거 → 폴더 삭제 → 실행 검증 순으로 정리.
- 마이그레이션으로 안 쓰게 된 것만 지우고, 무관한 건 건드리지 않음.
