---
title: "인텔리제이로 스프링부트 예외처리(Exception Handling) 구현하기"
categories:
  - Spring Boot
tags:
  - Spring Boot
  - Postman
  - Exception
last_modified_at:
---

# 인텔리제이로 스프링부트 예외처리(Exception Handling) 구현하기

Spring Boot에서 전역 예외 처리 구조를 구현하여 일관된 에러 응답을 반환하는 방법을 정리함

GitHub 저장소: https://github.com/tletle7102/spring-exception-project

커밋 로그를 순서대로 따라가면서 학습하면 프로젝트 구성 흐름을 이해할 수 있음


## 목차

1. 프로젝트 초기 설정
2. .gitignore 설정
3. .env.sample 환경변수 샘플
4. build.gradle 설정
5. application.yml 설정
6. JPA Auditing 설정 클래스
7. 공통 엔티티 추상 클래스 (BaseEntity)
8. 에러 도메인 열거형 (Domain)
9. 에러 코드 열거형 (ErrorCode)
10. API 성공 응답 래퍼 클래스 (Response)
11. 에러 응답 구조 클래스 (ErrorResponse)
12. 커스텀 예외 클래스 (AppException)
13. 전역 예외 처리기 (CustomExceptionHandler)
14. 예외 처리 테스트용 컨트롤러 (TestController)
15. 애플리케이션 실행 및 테스트
16. 커밋 히스토리
17. 운영 환경 고려사항
18. 참고 자료


## 1. 프로젝트 초기 설정

> 커밋: `feat(init): Spring Boot 프로젝트 초기 설정`

### 1-1. 인텔리제이에서 새 프로젝트 생성

1. File > New > Project 선택
2. 좌측에서 "Spring Boot" 선택
3. 다음 설정 입력
   - Name: spring-exception-handling
   - Language: Java
   - Type: Gradle - Groovy
   - JDK: 21
   - Packaging: Jar
4. Next 클릭

### 1-2. 의존성 선택

1. Spring Boot 버전: 3.4.1
2. Dependencies에서 다음 항목 선택
   - Spring Web
   - Spring Data JPA
   - H2 Database
   - Lombok
3. Create 클릭

프로젝트가 생성되면 다음 파일들이 자동 생성됨:
- settings.gradle: 프로젝트 이름 설정
- gradlew, gradlew.bat: Gradle Wrapper 실행 스크립트
- gradle/wrapper/: Gradle Wrapper 설정 파일

### 1-3. 메인 클래스

src/main/java/com/example/exception/ExceptionHandlingApplication.java

```java
package com.example.exception;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * Spring Boot 애플리케이션의 시작점(Entry Point)
 *
 * @SpringBootApplication 어노테이션은 다음 3가지를 포함함:
 * - @Configuration: 이 클래스가 설정 클래스임을 표시
 * - @EnableAutoConfiguration: Spring Boot의 자동 설정 기능 활성화
 * - @ComponentScan: 현재 패키지와 하위 패키지의 컴포넌트를 자동 스캔
 */
@SpringBootApplication
public class ExceptionHandlingApplication {

    public static void main(String[] args) {
        // Spring Boot 애플리케이션 실행
        // 내장 톰캣 서버가 시작되고, 모든 빈(Bean)이 초기화됨
        SpringApplication.run(ExceptionHandlingApplication.class, args);
    }
}
```

@SpringBootApplication 어노테이션은 @Configuration, @EnableAutoConfiguration, @ComponentScan을 합친 것임
이 클래스가 있는 패키지(com.example.exception)와 그 하위 패키지의 모든 컴포넌트가 자동으로 스캔됨


## 2. .gitignore 설정

> 커밋: `feat(config): Git 버전관리 제외 파일 설정`

.gitignore 파일에 환경변수 파일과 빌드 결과물이 저장소에 올라가지 않도록 설정함
이 설정을 하지 않으면 API 키 같은 민감 정보가 GitHub에 노출될 수 있음

```
# Gradle
.gradle/
build/
!gradle/wrapper/gradle-wrapper.jar

# IDE
.idea/
*.iml
*.ipr
*.iws
.vscode/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# 환경변수 파일 (민감한 정보 포함 가능)
.env
*.env

# 로그 파일
*.log
logs/

# 테스트 결과
test-results/
```

주요 설정 설명:
- .gradle/, build/: Gradle 빌드 관련 파일은 빌드 시 자동 생성되므로 제외함
- .idea/, *.iml: 인텔리제이 IDE 설정 파일은 개인 환경에 따라 다르므로 제외함
- .env, *.env: 환경변수 파일은 민감 정보를 포함하므로 반드시 제외해야 함

.gitignore 파일은 반드시 프로젝트 초기에 설정해야 함
실수로 .env 파일을 한번이라도 커밋하면 git 히스토리에 남아서 삭제하기 어려움


## 3. .env.sample 환경변수 샘플

> 커밋: `feat(config): 환경변수 샘플 파일 추가`

프로젝트 루트에 .env.sample 파일을 생성함
이 파일은 다른 개발자나 프로젝트 사용자에게 어떤 환경변수가 필요한지 안내하는 역할을 함

```
# 환경변수 샘플 파일
# 이 파일을 복사하여 .env 파일을 생성하고 실제 값을 입력하세요
# cp .env.sample .env

# 서버 포트 (기본값: 8080)
SERVER_PORT=8080

# 데이터베이스 설정 (H2 인메모리 DB 사용 시)
DB_URL=jdbc:h2:mem:testdb
DB_USERNAME=sa
DB_PASSWORD=
```

.env.sample 파일의 역할:
- 필요한 환경변수 목록을 문서화함
- 환경변수 값의 형식을 예시로 보여줌
- 실제 비밀번호가 아닌 플레이스홀더를 사용하므로 GitHub에 올려도 안전함

### 3-1. .env 파일 생성

프로젝트를 클론한 후 .env.sample을 복사하여 .env 파일을 생성함

```bash
cp .env.sample .env
```

.env 파일을 열고 필요에 따라 값을 수정함:
```
SERVER_PORT=8080
DB_URL=jdbc:h2:mem:testdb
DB_USERNAME=sa
DB_PASSWORD=
```

.env 파일은 .gitignore에 등록되어 있어서 git에 올라가지 않음
따라서 실제 값을 입력해도 GitHub에 노출되지 않음


## 4. build.gradle 설정

> 커밋: `feat(config): Gradle 빌드 설정 및 의존성 정의`

인텔리제이가 생성한 build.gradle을 다음과 같이 수정함
Spring Boot Web, JPA, H2, Lombok 의존성과 .env 파일 로드 기능을 설정함

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.4.1'
    id 'io.spring.dependency-management' version '1.1.7'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    // Spring Boot 웹 스타터 - REST API 개발에 필요한 의존성
    implementation 'org.springframework.boot:spring-boot-starter-web'

    // Spring Boot JPA 스타터 - 데이터베이스 연동에 필요한 의존성
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'

    // H2 데이터베이스 - 개발/테스트용 인메모리 데이터베이스
    runtimeOnly 'com.h2database:h2'

    // Lombok - 반복적인 코드(getter, setter 등)를 자동 생성해주는 라이브러리
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    // 테스트 의존성
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}

tasks.named('test') {
    useJUnitPlatform()
}

// .env 파일에서 환경변수 로드
def loadEnv() {
    def envFile = file('.env')
    if (envFile.exists()) {
        envFile.readLines().each { line ->
            if (line && !line.startsWith('#') && line.contains('=')) {
                def (key, value) = line.split('=', 2)
                if (System.getenv(key.trim()) == null) {
                    System.setProperty(key.trim(), value.trim())
                }
            }
        }
    }
}

loadEnv()

bootRun {
    def envFile = file('.env')
    if (envFile.exists()) {
        envFile.readLines().each { line ->
            if (line && !line.startsWith('#') && line.contains('=')) {
                def parts = line.split('=', 2)
                if (parts.length == 2) {
                    environment parts[0].trim(), parts[1].trim()
                }
            }
        }
    }
}
```

주요 의존성 설명:
- spring-boot-starter-web: REST API 개발에 필요한 Spring MVC, 내장 톰캣 등을 포함함
- spring-boot-starter-data-jpa: JPA를 사용한 데이터베이스 연동 기능을 제공함
- h2: 별도 설치 없이 사용 가능한 인메모리 데이터베이스로, 개발 및 테스트에 적합함
- lombok: getter, setter, 생성자 등의 반복적인 코드를 어노테이션으로 자동 생성함

build.gradle 하단의 환경변수 로드 설정 설명:
- loadEnv() 함수: .env 파일을 읽어서 System.setProperty()로 시스템 프로퍼티에 등록
- loadEnv() 호출: 스크립트 실행 시점에 환경변수를 로드
- bootRun 블록: ./gradlew bootRun 실행 시 환경변수를 실행 환경에 주입

이 설정이 있어야 `./gradlew bootRun` 명령어로 실행할 때 .env 파일의 값을 사용할 수 있음

Gradle 코끼리 아이콘을 클릭하여 의존성을 다운로드함


## 5. application.yml 설정

> 커밋: `feat(config): Spring Boot 애플리케이션 설정 파일`

src/main/resources/application.yml

```yaml
# Spring Boot 애플리케이션 설정 파일
# YAML 형식으로 작성 (application.properties 대신 사용)

spring:
  application:
    name: spring-exception-handling  # 애플리케이션 이름

  # H2 데이터베이스 설정
  datasource:
    url: jdbc:h2:mem:testdb         # 인메모리 데이터베이스 URL
    driver-class-name: org.h2.Driver
    username: sa                     # 기본 사용자명
    password:                        # 비밀번호 없음

  # H2 콘솔 설정 (브라우저에서 DB 확인 가능)
  h2:
    console:
      enabled: true                  # H2 콘솔 활성화
      path: /h2-console              # 접속 경로: http://localhost:8080/h2-console

  # JPA 설정
  jpa:
    hibernate:
      ddl-auto: create               # 애플리케이션 시작 시 테이블 자동 생성
    show-sql: true                   # SQL 쿼리 콘솔 출력
    properties:
      hibernate:
        format_sql: true             # SQL 쿼리 포맷팅 (가독성 향상)
    open-in-view: false              # 지연 로딩 관련 설정 (서비스 레이어에서 처리)

# 서버 설정
server:
  port: 8080                         # 서버 포트 (기본값)
```

설정 항목 설명:
- spring.datasource: H2 인메모리 데이터베이스 연결 정보. 애플리케이션 종료 시 데이터가 사라짐
- spring.h2.console: H2 콘솔을 활성화하면 브라우저에서 http://localhost:8080/h2-console로 접속하여 데이터베이스를 직접 확인할 수 있음
- spring.jpa.hibernate.ddl-auto: create로 설정하면 애플리케이션 시작 시 기존 테이블을 삭제하고 새로 생성함
- spring.jpa.open-in-view: false로 설정하면 지연 로딩을 서비스 레이어에서만 처리하도록 강제하여 성능 문제를 방지함


## 6. JPA Auditing 설정 클래스

> 커밋: `feat(config): JPA Auditing 설정 클래스`

JPA Auditing 기능을 활성화하는 설정 클래스임
이 설정이 있어야 엔티티의 생성일시와 수정일시가 자동으로 관리됨

src/main/java/com/example/exception/common/config/JpaAuditingConfig.java

```java
package com.example.exception.common.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;

/**
 * JPA Auditing 설정 클래스
 *
 * JPA Auditing이란?
 * - 엔티티의 생성일시, 수정일시 등을 자동으로 관리해주는 기능
 * - @CreatedDate, @LastModifiedDate 어노테이션과 함께 사용
 *
 * @EnableJpaAuditing 어노테이션이 없으면:
 * - BaseEntity의 createdDttm, updatedDttm 필드가 자동으로 채워지지 않음
 * - 데이터 저장 시 해당 필드들이 null로 남게 됨
 *
 * 왜 별도 설정 클래스로 분리하는가?
 * - 메인 애플리케이션 클래스를 깔끔하게 유지
 * - 설정의 역할과 책임을 명확히 분리
 * - 테스트 시 필요에 따라 이 설정만 제외할 수 있음
 */
@Configuration
@EnableJpaAuditing
public class JpaAuditingConfig {
    // 설정만 활성화하면 되므로 메서드 내용은 비어있음
}
```

@Configuration은 이 클래스가 스프링 설정 클래스임을 나타냄
@EnableJpaAuditing은 JPA Auditing 기능을 활성화함
이 두 어노테이션이 함께 있어야 @CreatedDate, @LastModifiedDate가 동작함

메인 애플리케이션 클래스에 직접 @EnableJpaAuditing을 붙여도 되지만, 설정의 책임을 분리하기 위해 별도 클래스로 관리하는 것이 좋음


## 7. 공통 엔티티 추상 클래스 (BaseEntity)

> 커밋: `feat(entity): 공통 엔티티 추상 클래스 (BaseEntity)`

모든 엔티티가 공통으로 가지는 생성일시, 수정일시 필드를 추상 클래스로 정의함
이 클래스를 상속받으면 생성일시와 수정일시가 자동으로 관리됨

src/main/java/com/example/exception/common/entity/BaseEntity.java

```java
package com.example.exception.common.entity;

import jakarta.persistence.Column;
import jakarta.persistence.EntityListeners;
import jakarta.persistence.MappedSuperclass;
import lombok.Getter;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import java.time.LocalDateTime;

/**
 * 모든 엔티티의 공통 필드를 정의하는 추상 클래스 (기본 엔티티)
 *
 * 이 클래스를 상속받으면 자동으로 생성일시와 수정일시가 관리됨
 *
 * 왜 사용하는가?
 * - 모든 엔티티에 createdDttm, updatedDttm을 반복적으로 작성할 필요가 없음
 * - 코드 중복을 줄이고 일관성을 유지
 * - 생성일시와 수정일시가 자동으로 채워지므로 실수 방지
 *
 * 사용 방법:
 * public class User extends BaseEntity {
 *     // User 엔티티만의 필드들...
 * }
 *
 * 어노테이션 설명:
 * - @MappedSuperclass: 이 클래스 자체는 테이블이 아니지만, 상속받는 클래스에 필드를 물려줌
 * - @EntityListeners: 엔티티의 변경사항을 감지하는 리스너 지정
 * - @CreatedDate: 엔티티가 처음 저장될 때 자동으로 현재 시간 저장
 * - @LastModifiedDate: 엔티티가 수정될 때마다 자동으로 현재 시간 갱신
 */
@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseEntity {

    /**
     * 생성일시 - 엔티티가 처음 생성(저장)된 시간
     *
     * @CreatedDate: Spring Data JPA가 자동으로 현재 시간을 넣어줌
     * @Column(updatable = false): 한 번 저장되면 수정 불가 (생성일시는 변하면 안 됨)
     */
    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdDttm;

    /**
     * 수정일시 - 엔티티가 마지막으로 수정된 시간
     *
     * @LastModifiedDate: 엔티티가 수정될 때마다 Spring Data JPA가 자동으로 현재 시간으로 갱신
     */
    @LastModifiedDate
    private LocalDateTime updatedDttm;
}
```

주요 어노테이션 설명:
- @MappedSuperclass: 이 클래스 자체는 데이터베이스 테이블로 생성되지 않지만, 상속받는 엔티티 클래스에 필드를 물려줌
- @EntityListeners(AuditingEntityListener.class): JPA 이벤트 리스너를 등록하여 엔티티의 생성/수정 이벤트를 감지함
- @CreatedDate: 엔티티가 처음 persist될 때 자동으로 현재 시간이 설정됨
- @LastModifiedDate: 엔티티가 수정될 때마다 자동으로 현재 시간으로 갱신됨
- @Column(updatable = false): createdDttm은 한번 저장되면 수정할 수 없도록 설정함


## 8. 에러 도메인 열거형 (Domain)

> 커밋: `feat(exception): 에러 도메인 열거형 (Domain)`

에러가 발생한 도메인(영역)을 구분하는 열거형임
에러 응답에서 "어느 영역에서 문제가 발생했는지"를 명확히 알려주는 역할을 함

src/main/java/com/example/exception/global/exception/Domain.java

```java
package com.example.exception.global.exception;

/**
 * 에러가 발생한 도메인(영역)을 구분하는 열거형(Enum)
 *
 * 왜 사용하는가?
 * - 에러 응답에서 "어느 영역에서 문제가 발생했는지"를 명확히 알려줌
 * - 프론트엔드에서 도메인별로 다른 처리를 할 수 있음
 * - 로그 분석 시 문제 영역을 빠르게 파악 가능
 *
 * 예시:
 * - 사용자 로그인 실패 → Domain.USER
 * - 게시글 조회 실패 → Domain.BOARD
 * - 시스템 내부 오류 → Domain.NONE
 *
 * 열거형(Enum)이란?
 * - 미리 정의된 상수들의 집합
 * - 정해진 값들 중에서만 선택할 수 있어 타입 안전성 보장
 * - 예: Domain.USER라고 쓰면 "USER"라는 문자열 대신 타입이 보장된 값 사용
 */
public enum Domain {

    // 도메인이 없거나 특정할 수 없는 일반적인 에러
    NONE("없음"),

    // 사용자 관련 에러 (로그인, 회원가입, 프로필 등)
    USER("사용자"),

    // 게시글 관련 에러 (작성, 수정, 삭제, 조회 등)
    BOARD("게시글"),

    // 댓글 관련 에러
    COMMENT("댓글"),

    // 좋아요 관련 에러
    LIKE("좋아요"),

    // 인증/인가 관련 에러 (JWT 토큰, 권한 등)
    AUTH("인증");

    // 도메인의 한글 설명 (예: USER → "사용자")
    private final String description;

    /**
     * 열거형 생성자
     * - 각 열거형 상수가 생성될 때 호출됨
     * - 예: NONE("없음")이 정의되면 이 생성자가 "없음"을 인자로 받아 호출됨
     *
     * @param description 도메인의 한글 설명
     */
    Domain(String description) {
        this.description = description;
    }

    /**
     * 도메인의 한글 설명을 반환
     *
     * @return 도메인 설명 (예: "사용자", "게시글")
     */
    public String getDescription() {
        return description;
    }
}
```

도메인 열거형의 역할:
- 에러 발생 영역을 명확하게 구분하여 클라이언트와 개발자가 문제를 빠르게 파악할 수 있게 함
- 문자열 대신 열거형을 사용하므로 오타로 인한 버그를 방지할 수 있음
- 새로운 도메인이 추가될 때 이 열거형에만 상수를 추가하면 됨

정의된 도메인 목록:
- NONE: 특정 도메인에 속하지 않는 일반적인 에러
- USER: 사용자 관련 에러
- BOARD: 게시글 관련 에러
- COMMENT: 댓글 관련 에러
- LIKE: 좋아요 관련 에러
- AUTH: 인증/인가 관련 에러


## 9. 에러 코드 열거형 (ErrorCode)

> 커밋: `feat(exception): 에러 코드 열거형 (ErrorCode)`

애플리케이션에서 사용하는 모든 에러 코드를 중앙에서 관리하는 열거형임
각 에러 코드는 HTTP 상태 코드, 에러 식별 코드, 에러 메시지를 포함함

src/main/java/com/example/exception/global/exception/ErrorCode.java

```java
package com.example.exception.global.exception;

import org.springframework.http.HttpStatus;

/**
 * 애플리케이션에서 사용하는 모든 에러 코드를 정의하는 열거형(Enum)
 *
 * 왜 에러 코드를 중앙에서 관리하는가?
 * 1. 일관성: 모든 에러가 동일한 형식으로 관리됨
 * 2. 유지보수: 에러 메시지 변경 시 이 파일만 수정하면 됨
 * 3. 재사용: 같은 에러가 여러 곳에서 발생해도 코드 중복 없음
 * 4. 문서화: 어떤 에러들이 있는지 한눈에 파악 가능
 *
 * 각 에러 코드가 가진 정보:
 * - httpStatus: HTTP 응답 상태 코드 (404, 500 등)
 * - code: 에러 식별 코드 (예: "USER_001")
 * - message: 사용자에게 보여줄 에러 메시지
 *
 * 에러 코드 네이밍 규칙:
 * - {도메인}_{순번} 형식 (예: USER_001, BOARD_001)
 * - 일반적인 에러는 GLOBAL_ 접두사 사용
 */
public enum ErrorCode {

    // ========== 사용자 관련 에러 (USER) ==========

    // 사용자를 찾을 수 없음 (404 Not Found)
    USER_NOT_FOUND(HttpStatus.NOT_FOUND, "USER_001", "사용자를 찾을 수 없습니다."),

    // 이미 존재하는 사용자 (409 Conflict)
    USER_ALREADY_EXISTS(HttpStatus.CONFLICT, "USER_002", "이미 존재하는 사용자입니다."),

    // ========== 게시글 관련 에러 (BOARD) ==========

    // 게시글을 찾을 수 없음
    BOARD_NOT_FOUND(HttpStatus.NOT_FOUND, "BOARD_001", "게시글을 찾을 수 없습니다."),

    // 게시글에 대한 권한 없음 (403 Forbidden)
    BOARD_UNAUTHORIZED(HttpStatus.FORBIDDEN, "BOARD_002", "게시글에 대한 권한이 없습니다."),

    // ========== 댓글 관련 에러 (COMMENT) ==========

    // 댓글을 찾을 수 없음
    COMMENT_NOT_FOUND(HttpStatus.NOT_FOUND, "COMMENT_001", "댓글을 찾을 수 없습니다."),

    // ========== 인증/인가 관련 에러 (AUTH) ==========

    // 인증 실패 (401 Unauthorized)
    AUTHENTICATION_FAILED(HttpStatus.UNAUTHORIZED, "AUTH_001", "인증에 실패했습니다."),

    // 접근 권한 없음 (403 Forbidden)
    ACCESS_DENIED(HttpStatus.FORBIDDEN, "AUTH_002", "접근 권한이 없습니다."),

    // ========== 일반 에러 (GLOBAL) ==========

    // 잘못된 요청 (400 Bad Request)
    BAD_REQUEST(HttpStatus.BAD_REQUEST, "GLOBAL_001", "잘못된 요청입니다."),

    // 서버 내부 오류 (500 Internal Server Error)
    INTERNAL_SERVER_ERROR(HttpStatus.INTERNAL_SERVER_ERROR, "GLOBAL_002", "서버 내부 오류가 발생했습니다.");

    // HTTP 상태 코드 (예: 404, 500)
    private final HttpStatus httpStatus;

    // 에러 식별 코드 (예: "USER_001")
    private final String code;

    // 사용자에게 보여줄 에러 메시지
    private final String message;

    /**
     * 열거형 생성자
     *
     * @param httpStatus HTTP 응답 상태 코드
     * @param code       에러 식별 코드
     * @param message    에러 메시지
     */
    ErrorCode(HttpStatus httpStatus, String code, String message) {
        this.httpStatus = httpStatus;
        this.code = code;
        this.message = message;
    }

    /**
     * HTTP 상태 코드 반환
     * 예: HttpStatus.NOT_FOUND (404)
     */
    public HttpStatus getHttpStatus() {
        return httpStatus;
    }

    /**
     * 에러 식별 코드 반환
     * 예: "USER_001"
     */
    public String getCode() {
        return code;
    }

    /**
     * 에러 메시지 반환
     * 예: "사용자를 찾을 수 없습니다."
     */
    public String getMessage() {
        return message;
    }
}
```

에러 코드를 중앙 관리하는 이유:
- 에러 메시지 변경 시 이 파일만 수정하면 전체 애플리케이션에 반영됨
- 같은 에러가 여러 곳에서 발생해도 동일한 에러 코드를 재사용할 수 있음
- 전체 에러 목록을 한눈에 파악할 수 있어 문서화 역할도 함

에러 코드 네이밍 규칙:
- {도메인}_{순번} 형식을 사용함 (예: USER_001, BOARD_001)
- 일반적인 에러는 GLOBAL_ 접두사를 사용함 (예: GLOBAL_001)
- 각 에러 코드는 HTTP 상태 코드, 식별 코드, 메시지를 하나의 상수로 묶어서 관리함


## 10. API 성공 응답 래퍼 클래스 (Response)

> 커밋: `feat(exception): API 성공 응답 래퍼 클래스 (Response)`

모든 API 성공 응답을 동일한 구조로 감싸는 제네릭 래퍼 클래스임
클라이언트(프론트엔드)에서 응답 파싱 로직을 통일할 수 있음

src/main/java/com/example/exception/global/exception/Response.java

```java
package com.example.exception.global.exception;

import lombok.Getter;
import org.springframework.http.HttpStatus;

/**
 * API 성공 응답을 감싸는 제네릭 래퍼 클래스
 *
 * 왜 응답을 래핑(감싸기)하는가?
 * 1. 일관성: 모든 API 응답이 동일한 구조를 가짐
 * 2. 확장성: 나중에 공통 필드(예: 타임스탬프)를 추가하기 쉬움
 * 3. 클라이언트 편의: 프론트엔드에서 응답 파싱 로직을 통일할 수 있음
 *
 * 응답 구조 예시:
 * {
 *   "result": "SUCCESS",
 *   "data": { ... },        // 실제 응답 데이터
 *   "message": "요청이 성공적으로 처리되었습니다.",
 *   "status": 200
 * }
 *
 * 제네릭(Generic)이란?
 * - <T>는 "어떤 타입이든 될 수 있다"는 의미
 * - Response<User>면 data가 User 타입
 * - Response<List<Board>>면 data가 Board 리스트 타입
 * - 다양한 타입의 데이터를 하나의 클래스로 감쌀 수 있음
 */
@Getter  // Lombok: 모든 필드의 getter 메서드 자동 생성
public class Response<T> {

    // 요청 처리 결과 ("SUCCESS" 또는 "ERROR")
    private final String result;

    // 실제 응답 데이터 (성공 시에만 포함)
    private final T data;

    // 응답 메시지 (성공/실패 메시지)
    private final String message;

    // HTTP 상태 코드 (숫자로 저장, 예: 200, 404)
    private final int status;

    /**
     * 성공 응답을 위한 생성자
     *
     * @param data    응답 데이터
     * @param message 성공 메시지
     * @param status  HTTP 상태 코드
     */
    public Response(T data, String message, HttpStatus status) {
        this.result = "SUCCESS";
        this.data = data;
        this.message = message;
        this.status = status.value();  // HttpStatus → int 변환 (예: OK → 200)
    }

    /**
     * 에러 응답을 위한 생성자
     * - 컨트롤러에서 간단한 에러 상황을 직접 처리할 때 사용
     *
     * @param message 에러 메시지
     * @param status  HTTP 상태 코드
     */
    public Response(String message, HttpStatus status) {
        this.result = "ERROR";
        this.data = null;  // 에러 시에는 데이터 없음
        this.message = message;
        this.status = status.value();
    }

    /**
     * 성공 응답을 간편하게 생성하는 정적 메서드 (커스텀 메시지 사용)
     *
     * 사용 예시:
     * return Response.success(user, "사용자 조회 성공");
     *
     * @param data    응답 데이터
     * @param message 성공 메시지
     * @return 성공 응답 객체
     */
    public static <T> Response<T> success(T data, String message) {
        return new Response<>(data, message, HttpStatus.OK);
    }

    /**
     * 성공 응답을 간편하게 생성하는 정적 메서드 (기본 메시지 사용)
     *
     * 사용 예시:
     * return Response.success(userList);
     *
     * @param data 응답 데이터
     * @return 성공 응답 객체 (기본 메시지 포함)
     */
    public static <T> Response<T> success(T data) {
        return new Response<>(data, "요청이 성공적으로 처리되었습니다.", HttpStatus.OK);
    }

    /**
     * 에러 응답을 간편하게 생성하는 정적 메서드
     *
     * 사용 예시:
     * return Response.error("잘못된 요청입니다.", HttpStatus.BAD_REQUEST);
     *
     * @param message 에러 메시지
     * @param status  HTTP 상태 코드
     * @return 에러 응답 객체
     */
    public static Response<?> error(String message, HttpStatus status) {
        return new Response<>(message, status);
    }
}
```

Response 클래스의 구조:
- result: 요청 처리 결과를 "SUCCESS" 또는 "ERROR"로 표시함
- data: 실제 응답 데이터. 제네릭 타입으로 어떤 데이터든 담을 수 있음
- message: 성공 또는 실패에 대한 메시지
- status: HTTP 상태 코드를 숫자로 저장함

정적 팩토리 메서드:
- Response.success(data, message): 커스텀 메시지와 함께 성공 응답을 생성함
- Response.success(data): 기본 메시지("요청이 성공적으로 처리되었습니다.")로 성공 응답을 생성함
- Response.error(message, status): 에러 응답을 생성함

정적 팩토리 메서드 패턴을 사용하면 생성자 대신 의미 있는 이름으로 객체를 생성할 수 있어 코드 가독성이 향상됨


## 11. 에러 응답 구조 클래스 (ErrorResponse)

> 커밋: `feat(exception): 에러 응답 구조 클래스 (ErrorResponse)`

AppException이 발생했을 때 클라이언트에게 반환되는 에러 응답 형식을 정의하는 클래스임
Response 클래스와 별도로 에러 전용 응답 구조를 분리함

src/main/java/com/example/exception/global/exception/ErrorResponse.java

```java
package com.example.exception.global.exception;

import lombok.Getter;

/**
 * 에러 응답의 구조를 정의하는 클래스
 *
 * AppException이 발생했을 때 클라이언트에게 반환되는 응답 형식
 * CustomExceptionHandler에서 AppException을 잡아서 이 객체로 변환함
 *
 * 응답 구조 예시:
 * {
 *   "domain": "USER",           // 어느 영역에서 에러가 발생했는지
 *   "code": "USER_001",         // 에러 식별 코드
 *   "message": "사용자를 찾을 수 없습니다.",  // 에러 메시지
 *   "status": 404               // HTTP 상태 코드
 * }
 *
 * 왜 Response<T>와 별도로 ErrorResponse를 만드는가?
 * - Response는 성공 응답에 최적화된 구조 (result, data 필드 포함)
 * - ErrorResponse는 에러 상황에 필요한 정보만 포함 (domain, code 필드 포함)
 * - 클라이언트가 성공/실패를 구분하여 처리하기 용이함
 */
@Getter  // Lombok: 모든 필드의 getter 메서드 자동 생성
public class ErrorResponse {

    // 에러가 발생한 도메인 (예: USER, BOARD 등)
    private final Domain domain;

    // 에러 식별 코드 (예: "USER_001")
    private final String code;

    // 에러 메시지 (예: "사용자를 찾을 수 없습니다.")
    private final String message;

    // HTTP 상태 코드 (예: 404, 500)
    private final int status;

    /**
     * Jackson 라이브러리를 위한 기본 생성자
     *
     * Jackson이란?
     * - JSON ↔ Java 객체 변환을 담당하는 라이브러리
     * - Spring Boot에서 자동으로 사용됨
     * - JSON을 객체로 변환할 때 기본 생성자가 필요할 수 있음
     *
     * protected로 선언한 이유:
     * - 외부에서 빈 객체를 직접 생성하는 것을 방지
     * - Jackson만 이 생성자를 사용할 수 있도록 제한
     */
    protected ErrorResponse() {
        this.domain = null;
        this.code = null;
        this.message = null;
        this.status = 0;
    }

    /**
     * 모든 필드를 받는 전체 생성자
     *
     * @param domain    에러가 발생한 도메인
     * @param errorCode 에러 코드 객체 (코드, 메시지, HTTP 상태 포함)
     * @param message   사용자 정의 메시지 (null이면 ErrorCode의 기본 메시지 사용)
     */
    public ErrorResponse(Domain domain, ErrorCode errorCode, String message) {
        this.domain = domain;
        this.code = errorCode.getCode();  // ErrorCode에서 코드 문자열 추출

        // 메시지 처리: 사용자 정의 메시지가 있으면 사용, 없으면 ErrorCode의 기본 메시지 사용
        // 삼항 연산자: (조건) ? 참일때값 : 거짓일때값
        this.message = message != null ? message : errorCode.getMessage();

        this.status = errorCode.getHttpStatus().value();  // HttpStatus → int 변환
    }

    /**
     * 메시지 없이 생성하는 간편 생성자
     * - ErrorCode에 정의된 기본 메시지를 사용
     *
     * @param domain    에러가 발생한 도메인
     * @param errorCode 에러 코드 객체
     */
    public ErrorResponse(Domain domain, ErrorCode errorCode) {
        // this()로 위의 전체 생성자를 호출하면서 message에 null 전달
        // → 전체 생성자에서 ErrorCode의 기본 메시지를 사용하게 됨
        this(domain, errorCode, null);
    }
}
```

ErrorResponse와 Response의 차이:
- Response: 성공 응답에 최적화됨. result, data, message, status 필드를 가짐
- ErrorResponse: 에러 응답에 최적화됨. domain, code, message, status 필드를 가짐

ErrorResponse의 생성자 구조:
- 전체 생성자: domain, errorCode, message를 모두 받음. 커스텀 메시지가 있으면 사용하고 없으면 ErrorCode의 기본 메시지를 사용함
- 간편 생성자: domain과 errorCode만 받음. ErrorCode의 기본 메시지를 사용함
- protected 기본 생성자: Jackson 역직렬화를 위해 존재함. 외부에서는 사용할 수 없음


## 12. 커스텀 예외 클래스 (AppException)

> 커밋: `feat(exception): 커스텀 예외 클래스 (AppException)`

애플리케이션 전체에서 사용하는 커스텀 예외 클래스임
RuntimeException을 상속받아 비즈니스 로직에서 발생하는 예외를 표현함

src/main/java/com/example/exception/global/exception/AppException.java

```java
package com.example.exception.global.exception;

/**
 * 애플리케이션 전체에서 사용하는 커스텀 예외 클래스
 *
 * 왜 커스텀 예외를 만드는가?
 * 1. 비즈니스 로직에 맞는 예외 표현: "사용자를 찾을 수 없음"은 NullPointerException보다 의미가 명확함
 * 2. 일관된 에러 처리: 모든 비즈니스 예외가 동일한 형식으로 처리됨
 * 3. 추가 정보 포함: 도메인, 에러코드 등 디버깅에 필요한 정보를 함께 전달
 *
 * RuntimeException을 상속받는 이유:
 * - RuntimeException은 "언체크 예외(Unchecked Exception)"
 * - try-catch로 감싸거나 throws 선언을 강제하지 않음
 * - 비즈니스 로직 예외는 대부분 RuntimeException으로 처리하는 것이 관례
 *
 * 사용 예시:
 * // 사용자를 찾지 못했을 때
 * throw new AppException(Domain.USER, ErrorCode.USER_NOT_FOUND);
 *
 * // 커스텀 메시지와 함께
 * throw new AppException(Domain.USER, ErrorCode.USER_NOT_FOUND, "ID가 123인 사용자를 찾을 수 없습니다.");
 */
public class AppException extends RuntimeException {

    // 예외가 발생한 도메인 (예: USER, BOARD 등)
    private final Domain domain;

    // 예외의 상세 정보를 담고 있는 ErrorCode
    private final ErrorCode errorCode;

    /**
     * 사용자 정의 메시지를 포함하는 생성자
     *
     * @param domain    예외가 발생한 도메인
     * @param errorCode 에러 코드 (에러 메시지, HTTP 상태 포함)
     * @param message   사용자 정의 에러 메시지 (null이면 ErrorCode의 기본 메시지 사용)
     */
    public AppException(Domain domain, ErrorCode errorCode, String message) {
        // super()로 부모 클래스(RuntimeException)의 생성자 호출
        // RuntimeException에 메시지를 전달하면 getMessage()로 조회 가능
        // message가 null이면 ErrorCode의 기본 메시지 사용
        super(message != null ? message : errorCode.getMessage());

        this.domain = domain;
        this.errorCode = errorCode;
    }

    /**
     * 메시지 없이 ErrorCode의 기본 메시지를 사용하는 생성자
     *
     * @param domain    예외가 발생한 도메인
     * @param errorCode 에러 코드
     */
    public AppException(Domain domain, ErrorCode errorCode) {
        // this()로 위의 생성자를 호출하면서 message에 null 전달
        this(domain, errorCode, null);
    }

    /**
     * 도메인 반환
     *
     * @return 예외가 발생한 도메인
     */
    public Domain getDomain() {
        return domain;
    }

    /**
     * 에러 코드 반환
     *
     * @return 에러 코드 객체
     */
    public ErrorCode getErrorCode() {
        return errorCode;
    }
}
```

AppException을 사용하는 이유:
- NullPointerException, IllegalArgumentException 같은 자바 기본 예외보다 비즈니스 의미가 명확함
- Domain과 ErrorCode를 함께 전달하므로 예외 발생 영역과 원인을 정확히 파악할 수 있음
- 전역 예외 처리기(CustomExceptionHandler)에서 이 예외를 잡아서 일관된 에러 응답으로 변환함

RuntimeException을 상속받는 이유:
- RuntimeException은 "언체크 예외(Unchecked Exception)"로, try-catch나 throws 선언을 강제하지 않음
- 비즈니스 로직 예외는 대부분 RuntimeException으로 처리하는 것이 스프링 생태계의 관례임
- 서비스 레이어에서 throw만 하면 전역 예외 처리기가 자동으로 처리함


## 13. 전역 예외 처리기 (CustomExceptionHandler)

> 커밋: `feat(exception): 전역 예외 처리기 (CustomExceptionHandler)`

@RestControllerAdvice를 사용하여 모든 컨트롤러에서 발생하는 예외를 한 곳에서 처리하는 클래스임
각 컨트롤러마다 try-catch를 작성할 필요 없이 중앙에서 예외를 처리함

src/main/java/com/example/exception/global/exception/CustomExceptionHandler.java

```java
package com.example.exception.global.exception;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

/**
 * 전역 예외 처리기 (Global Exception Handler)
 *
 * @RestControllerAdvice란?
 * - 모든 @RestController에서 발생하는 예외를 한 곳에서 처리
 * - 각 컨트롤러마다 try-catch를 작성할 필요 없음
 * - 예외 처리 로직을 중앙에서 관리하여 코드 중복 방지
 *
 * 동작 방식:
 * 1. 컨트롤러에서 예외 발생 (throw new AppException(...))
 * 2. Spring이 예외를 감지하고 이 클래스의 핸들러 메서드를 찾음
 * 3. 예외 타입에 맞는 @ExceptionHandler 메서드가 실행됨
 * 4. ErrorResponse 객체가 JSON으로 변환되어 클라이언트에게 반환됨
 *
 * 예외 처리 우선순위:
 * - 더 구체적인 예외 타입의 핸들러가 먼저 실행됨
 * - AppException 핸들러 → Exception 핸들러 순서로 매칭 시도
 */
@RestControllerAdvice  // 전역 예외 처리 + JSON 응답
public class CustomExceptionHandler {

    // 로깅을 위한 Logger 객체
    // 로그 레벨: TRACE < DEBUG < INFO < WARN < ERROR
    private static final Logger log = LoggerFactory.getLogger(CustomExceptionHandler.class);

    /**
     * AppException 처리 핸들러
     *
     * 개발자가 의도적으로 발생시킨 비즈니스 예외를 처리
     * 예: 사용자 없음, 권한 없음, 중복 데이터 등
     *
     * @ExceptionHandler: 이 메서드가 처리할 예외 타입 지정
     *
     * @param ex 발생한 AppException 객체
     * @return HTTP 응답 (상태 코드 + ErrorResponse 본문)
     */
    @ExceptionHandler(AppException.class)
    public ResponseEntity<ErrorResponse> handleAppException(AppException ex) {
        // 로그에 예외 정보 출력 (운영 환경에서 디버깅에 활용)
        // log.error()는 에러 레벨 로그 (가장 심각한 수준)
        log.error("AppException 발생: domain={}, code={}, message={}",
                ex.getDomain(),           // 어느 도메인에서 발생했는지
                ex.getErrorCode().getCode(),  // 에러 코드
                ex.getMessage());         // 에러 메시지

        // ErrorResponse 객체 생성
        ErrorResponse errorResponse = new ErrorResponse(
                ex.getDomain(),
                ex.getErrorCode(),
                ex.getMessage()  // 사용자 정의 메시지가 있으면 사용
        );

        // ResponseEntity로 HTTP 응답 생성
        // - status(): HTTP 상태 코드 설정 (예: 404, 500)
        // - body(): 응답 본문 설정 (JSON으로 변환됨)
        return ResponseEntity
                .status(ex.getErrorCode().getHttpStatus())
                .body(errorResponse);
    }

    /**
     * 일반 Exception 처리 핸들러 (Fallback Handler)
     *
     * AppException이 아닌 예상치 못한 예외를 처리
     * 예: NullPointerException, IllegalArgumentException 등
     *
     * 왜 필요한가?
     * - 예상치 못한 예외도 깔끔한 JSON 형식으로 응답
     * - 민감한 에러 정보(스택 트레이스 등)가 클라이언트에 노출되지 않도록 방지
     * - 500 Internal Server Error로 일관되게 처리
     *
     * @param ex 발생한 Exception 객체
     * @return HTTP 500 응답
     */
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneralException(Exception ex) {
        // 예외 로그 출력 (스택 트레이스 포함)
        // 세 번째 인자로 ex를 전달하면 전체 스택 트레이스가 로그에 기록됨
        log.error("예상치 못한 예외 발생: message={}", ex.getMessage(), ex);

        // 일반적인 서버 에러 응답 생성
        // 도메인은 NONE (특정 도메인에서 발생한 것이 아니므로)
        ErrorResponse errorResponse = new ErrorResponse(
                Domain.NONE,
                ErrorCode.INTERNAL_SERVER_ERROR
        );

        // 500 Internal Server Error 응답 반환
        return ResponseEntity
                .status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(errorResponse);
    }
}
```

전역 예외 처리기의 동작 흐름:
1. 컨트롤러에서 예외가 발생함 (throw new AppException(...) 또는 예상치 못한 예외)
2. Spring이 @RestControllerAdvice가 붙은 클래스에서 해당 예외를 처리할 수 있는 메서드를 찾음
3. 예외 타입에 맞는 @ExceptionHandler 메서드가 실행됨
4. ErrorResponse 객체가 생성되어 JSON으로 변환된 후 클라이언트에게 반환됨

두 가지 핸들러 메서드:
- handleAppException(): AppException을 처리함. ErrorCode에 정의된 HTTP 상태 코드로 응답함
- handleGeneralException(): AppException이 아닌 모든 예외를 처리함. 항상 500 Internal Server Error로 응답하여 민감한 정보 노출을 방지함

@RestControllerAdvice와 @ControllerAdvice의 차이:
- @RestControllerAdvice: @ControllerAdvice + @ResponseBody. 반환 객체를 자동으로 JSON으로 변환함
- @ControllerAdvice: 뷰(HTML)를 반환할 때 사용함


## 14. 예외 처리 테스트용 컨트롤러 (TestController)

> 커밋: `feat(controller): 예외 처리 테스트용 컨트롤러 (TestController)`

예외 처리 시스템이 정상 동작하는지 확인하기 위한 테스트 엔드포인트를 제공하는 컨트롤러임
성공 응답, 404 에러, 400 에러, 500 에러, 예상치 못한 에러 등 다양한 시나리오를 테스트할 수 있음

src/main/java/com/example/exception/api/controller/TestController.java

```java
package com.example.exception.api.controller;

import com.example.exception.global.exception.*;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.HashMap;
import java.util.Map;

/**
 * 예외 처리 테스트용 컨트롤러
 *
 * 이 컨트롤러는 예외 처리 시스템이 정상 동작하는지 확인하기 위한 테스트 엔드포인트를 제공
 *
 * 테스트 시나리오:
 * 1. 성공 응답 - Response 클래스 사용
 * 2. 404 에러 - NOT_FOUND ErrorCode로 예외 발생
 * 3. 400 에러 - BAD_REQUEST ErrorCode로 예외 발생
 * 4. 500 에러 - INTERNAL_SERVER_ERROR ErrorCode로 예외 발생
 *
 * @RestController란?
 * - @Controller + @ResponseBody의 조합
 * - 메서드의 반환값이 자동으로 JSON으로 변환되어 응답 본문에 포함됨
 * - View(HTML)를 반환하지 않고 데이터(JSON)를 직접 반환
 *
 * @RequestMapping("/api/test")란?
 * - 이 컨트롤러의 모든 엔드포인트 앞에 "/api/test" 경로가 붙음
 * - 예: @GetMapping("/success") → GET /api/test/success
 */
@RestController
@RequestMapping("/api/test")
public class TestController {

    /**
     * 성공 응답 테스트 엔드포인트
     *
     * URL: GET /api/test/success
     *
     * 동작:
     * - Response.success()를 사용하여 성공 응답 반환
     * - 예시 데이터와 함께 200 OK 응답
     *
     * 응답 예시:
     * {
     *   "result": "SUCCESS",
     *   "data": {
     *     "message": "안녕하세요!",
     *     "timestamp": "2024-01-15T10:30:00"
     *   },
     *   "message": "요청이 성공적으로 처리되었습니다.",
     *   "status": 200
     * }
     */
    @GetMapping("/success")
    public Response<Map<String, Object>> testSuccess() {
        // 응답으로 보낼 데이터 생성
        Map<String, Object> data = new HashMap<>();
        data.put("message", "안녕하세요!");
        data.put("timestamp", java.time.LocalDateTime.now().toString());

        // Response.success()로 성공 응답 생성하여 반환
        return Response.success(data, "테스트 성공 응답입니다.");
    }

    /**
     * 404 Not Found 에러 테스트 엔드포인트
     *
     * URL: GET /api/test/not-found
     *
     * 동작:
     * - AppException을 발생시켜 404 에러 응답 유도
     * - CustomExceptionHandler가 이 예외를 잡아서 ErrorResponse로 변환
     *
     * 응답 예시:
     * {
     *   "domain": "USER",
     *   "code": "USER_001",
     *   "message": "사용자를 찾을 수 없습니다.",
     *   "status": 404
     * }
     */
    @GetMapping("/not-found")
    public Response<?> testNotFound() {
        // AppException을 throw하면 CustomExceptionHandler가 처리함
        // Domain.USER: 사용자 관련 에러
        // ErrorCode.USER_NOT_FOUND: 404 상태 코드와 메시지를 포함
        throw new AppException(Domain.USER, ErrorCode.USER_NOT_FOUND);

        // 주의: throw 뒤의 코드는 실행되지 않음 (Unreachable code)
        // return 문이 없어도 되는 이유: 예외가 발생하면 메서드가 즉시 종료됨
    }

    /**
     * 400 Bad Request 에러 테스트 엔드포인트
     *
     * URL: GET /api/test/bad-request
     *
     * 동작:
     * - 잘못된 요청 에러(400)를 발생시킴
     * - 커스텀 메시지를 포함하여 예외 발생
     *
     * 응답 예시:
     * {
     *   "domain": "NONE",
     *   "code": "GLOBAL_001",
     *   "message": "필수 파라미터가 누락되었습니다.",
     *   "status": 400
     * }
     */
    @GetMapping("/bad-request")
    public Response<?> testBadRequest() {
        // 세 번째 인자로 커스텀 메시지 전달
        // ErrorCode의 기본 메시지 대신 이 메시지가 사용됨
        throw new AppException(
                Domain.NONE,              // 특정 도메인이 아닌 일반적인 에러
                ErrorCode.BAD_REQUEST,    // 400 Bad Request
                "필수 파라미터가 누락되었습니다."  // 커스텀 메시지
        );
    }

    /**
     * 500 Internal Server Error 테스트 엔드포인트
     *
     * URL: GET /api/test/server-error
     *
     * 동작:
     * - 서버 내부 에러(500)를 발생시킴
     * - 실제 운영 환경에서는 예상치 못한 에러 상황을 시뮬레이션
     *
     * 응답 예시:
     * {
     *   "domain": "NONE",
     *   "code": "GLOBAL_002",
     *   "message": "서버 내부 오류가 발생했습니다.",
     *   "status": 500
     * }
     */
    @GetMapping("/server-error")
    public Response<?> testServerError() {
        throw new AppException(
                Domain.NONE,
                ErrorCode.INTERNAL_SERVER_ERROR
        );
    }

    /**
     * 예상치 못한 예외 테스트 엔드포인트 (추가)
     *
     * URL: GET /api/test/unexpected-error
     *
     * 동작:
     * - AppException이 아닌 일반 RuntimeException 발생
     * - CustomExceptionHandler의 handleGeneralException()이 처리
     * - 항상 500 에러로 응답
     *
     * 왜 테스트하는가?
     * - 예상치 못한 예외도 깔끔하게 처리되는지 확인
     * - 에러 메시지가 클라이언트에 노출되지 않는지 확인
     */
    @GetMapping("/unexpected-error")
    public Response<?> testUnexpectedError() {
        // AppException이 아닌 일반 예외 발생
        // 이 예외는 handleGeneralException()에서 처리됨
        throw new RuntimeException("이것은 예상치 못한 에러입니다!");
    }
}
```

테스트 엔드포인트 목록:
- GET /api/test/success: 성공 응답 테스트. Response.success()로 200 OK 응답을 반환함
- GET /api/test/not-found: 404 에러 테스트. AppException으로 사용자 없음 에러를 발생시킴
- GET /api/test/bad-request: 400 에러 테스트. 커스텀 메시지와 함께 잘못된 요청 에러를 발생시킴
- GET /api/test/server-error: 500 에러 테스트. 서버 내부 오류를 발생시킴
- GET /api/test/unexpected-error: 예상치 못한 예외 테스트. RuntimeException을 발생시켜 handleGeneralException()이 처리하는지 확인함

이 컨트롤러를 통해 예외 처리 시스템의 전체 흐름을 검증할 수 있음


## 15. 애플리케이션 실행 및 테스트

### 15-1. .env 파일 생성

프로젝트 루트에 .env 파일을 생성함

```bash
cp .env.sample .env
```

### 15-2. 애플리케이션 실행

터미널에서 다음 명령어로 실행함

```bash
./gradlew bootRun
```

build.gradle에 설정한 bootRun 태스크가 .env 파일을 읽어서 환경변수로 등록함
인텔리제이의 Run 버튼 대신 터미널에서 실행해야 .env 파일이 적용됨

콘솔에 다음과 같은 로그가 출력되면 성공임

```
Started ExceptionHandlingApplication in X.XXX seconds
```

### 15-3. 성공 응답 테스트

브라우저 또는 Postman에서 다음 URL에 접속함

```
GET http://localhost:8080/api/test/success
```

성공 시 다음과 같은 응답이 반환됨

```json
{
  "result": "SUCCESS",
  "data": {
    "message": "안녕하세요!",
    "timestamp": "2026-03-05T10:30:00.123456"
  },
  "message": "테스트 성공 응답입니다.",
  "status": 200
}
```

### 15-4. 404 Not Found 에러 테스트

```
GET http://localhost:8080/api/test/not-found
```

응답:

```json
{
  "domain": "USER",
  "code": "USER_001",
  "message": "사용자를 찾을 수 없습니다.",
  "status": 404
}
```

### 15-5. 400 Bad Request 에러 테스트

```
GET http://localhost:8080/api/test/bad-request
```

응답:

```json
{
  "domain": "NONE",
  "code": "GLOBAL_001",
  "message": "필수 파라미터가 누락되었습니다.",
  "status": 400
}
```

### 15-6. 500 Internal Server Error 테스트

```
GET http://localhost:8080/api/test/server-error
```

응답:

```json
{
  "domain": "NONE",
  "code": "GLOBAL_002",
  "message": "서버 내부 오류가 발생했습니다.",
  "status": 500
}
```

### 15-7. 예상치 못한 예외 테스트

```
GET http://localhost:8080/api/test/unexpected-error
```

응답:

```json
{
  "domain": "NONE",
  "code": "GLOBAL_002",
  "message": "서버 내부 오류가 발생했습니다.",
  "status": 500
}
```

이 테스트에서는 RuntimeException이 발생하지만, handleGeneralException()이 처리하여 클라이언트에게는 일반적인 서버 에러 메시지만 반환됨
실제 에러 메시지("이것은 예상치 못한 에러입니다!")는 서버 로그에만 기록되어 민감 정보 노출이 방지됨

### 15-8. curl로 테스트

터미널에서 curl 명령어로 테스트할 수도 있음

```bash
# 성공 응답
curl -s http://localhost:8080/api/test/success | python3 -m json.tool

# 404 에러
curl -s http://localhost:8080/api/test/not-found | python3 -m json.tool

# 400 에러
curl -s http://localhost:8080/api/test/bad-request | python3 -m json.tool

# 500 에러
curl -s http://localhost:8080/api/test/server-error | python3 -m json.tool

# 예상치 못한 예외
curl -s http://localhost:8080/api/test/unexpected-error | python3 -m json.tool
```


## 16. 커밋 히스토리

이 프로젝트의 커밋 로그를 순서대로 따라가면 스프링부트 예외처리 프로젝트 구성 흐름을 이해할 수 있음

```
feat(init): Spring Boot 프로젝트 초기 설정
feat(config): Git 버전관리 제외 파일 설정
feat(config): 환경변수 샘플 파일 추가
feat(config): Gradle 빌드 설정 및 의존성 정의
feat(config): Spring Boot 애플리케이션 설정 파일
feat(config): JPA Auditing 설정 클래스
feat(entity): 공통 엔티티 추상 클래스 (BaseEntity)
feat(exception): 에러 도메인 열거형 (Domain)
feat(exception): 에러 코드 열거형 (ErrorCode)
feat(exception): API 성공 응답 래퍼 클래스 (Response)
feat(exception): 에러 응답 구조 클래스 (ErrorResponse)
feat(exception): 커스텀 예외 클래스 (AppException)
feat(exception): 전역 예외 처리기 (CustomExceptionHandler)
feat(controller): 예외 처리 테스트용 컨트롤러 (TestController)
```

각 커밋 메시지에 해당 파일의 역할과 코드 설명이 포함되어 있음

첫 번째 커밋(feat(init))에 build.gradle, Gradle Wrapper, settings.gradle, 메인 클래스가 모두 포함됨
이후 .gitignore, .env.sample, 설정 파일, 예외 처리 관련 클래스들이 순차적으로 추가됨


## 17. 운영 환경 고려사항

실제 서비스에 적용할 때 추가로 고려해야 할 사항:

- H2 인메모리 데이터베이스를 MySQL, PostgreSQL 등 운영용 데이터베이스로 교체해야 함
- ddl-auto 설정을 create 대신 validate 또는 none으로 변경해야 함 (데이터 유실 방지)
- ErrorCode에 새로운 도메인이나 에러 상황이 추가될 때마다 열거형을 확장해야 함
- Bean Validation(@Valid) 예외 처리를 CustomExceptionHandler에 추가하면 입력값 검증 에러도 통일된 형식으로 응답할 수 있음
- 로그 레벨을 운영 환경에 맞게 조정해야 함 (개발: DEBUG, 운영: WARN 또는 ERROR)
- 에러 로그를 파일이나 외부 로그 수집 시스템(ELK Stack 등)에 저장하여 모니터링해야 함
- 에러 응답에 요청 추적을 위한 traceId를 추가하면 디버깅이 용이함


## 18. 참고 자료

- Spring Boot 공식 문서: https://docs.spring.io/spring-boot/docs/current/reference/html/
- Spring MVC 예외 처리: https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-ann-rest-exceptions.html
- Baeldung - Error Handling for REST: https://www.baeldung.com/exception-handling-for-rest-with-spring
- H2 Database: https://www.h2database.com
