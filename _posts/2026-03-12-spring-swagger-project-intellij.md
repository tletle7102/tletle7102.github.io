---
title: "인텔리제이로 스프링부트 Swagger API 문서 자동화 구현하기"
categories:
  - Spring Boot
tags:
  - Spring Boot
  - Postman
  - Swagger
last_modified_at:
---

# 인텔리제이로 스프링부트 Swagger API 문서 자동화 구현하기

Spring Boot와 Springdoc OpenAPI를 사용하여 Swagger UI 기반 REST API 문서를 자동 생성하는 방법을 정리함

GitHub 저장소: https://github.com/tletle7102/spring-swagger-project

커밋 로그를 순서대로 따라가면서 학습하면 프로젝트 구성 흐름을 이해할 수 있음


## 목차

1. 프로젝트 초기 설정
2. .gitignore 설정
3. .env.sample 환경변수 샘플
4. build.gradle 설정
5. application.yml 설정
6. SwaggerConfig 클래스
7. Product 엔티티
8. ProductRequestDTO
9. ProductResponseDTO
10. ProductRepository
11. ProductService
12. ProductController
13. 애플리케이션 실행 및 테스트
14. 커밋 히스토리
15. 운영 환경 고려사항
16. 참고 자료


## 1. 프로젝트 초기 설정

> 커밋: `feat(init): 스프링부트 프로젝트 초기 설정`

### 1-1. 인텔리제이에서 새 프로젝트 생성

1. File > New > Project 선택
2. 좌측에서 "Spring Boot" 선택
3. 다음 설정 입력
   - Name: spring-swagger-project
   - Language: Java
   - Type: Gradle - Groovy
   - JDK: 21
   - Packaging: Jar
4. Next 클릭

### 1-2. 의존성 선택

1. Spring Boot 버전: 3.4.5
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

src/main/java/com/example/swagger/SpringSwaggerProjectApplication.java

```java
package com.example.swagger;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringSwaggerProjectApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringSwaggerProjectApplication.class, args);
    }
}
```

@SpringBootApplication은 @Configuration, @EnableAutoConfiguration, @ComponentScan을 포함하는 복합 어노테이션임
스프링부트 애플리케이션의 진입점으로 자동 설정과 컴포넌트 스캔을 수행함


## 2. .gitignore 설정

> 커밋: `chore: .gitignore 추가`

.gitignore 파일에 환경변수 파일이 저장소에 올라가지 않도록 설정함
이 설정을 하지 않으면 데이터베이스 비밀번호나 JWT 시크릿 같은 민감 정보가 GitHub에 노출될 수 있음

```
HELP.md
.gradle
build/
!gradle/wrapper/gradle-wrapper.jar
!**/src/main/**/build/
!**/src/test/**/build/

### STS ###
.apt_generated
.classpath
.factorypath
.project
.settings
.springBeans
.sts4-cache
bin/
!**/src/main/**/bin/
!**/src/test/**/bin/

### IntelliJ IDEA ###
.idea
*.iws
*.iml
*.ipr
out/
!**/src/main/**/out/
!**/src/test/**/out/

### NetBeans ###
/nbproject/private/
/nbbuild/
/dist/
/nbdist/
/.nb-gradle/

### VS Code ###
.vscode/

### macOS ###
.DS_Store

### Environment Variables ###
.env
*.env
!.env.sample
```

주요 설정 설명:
- .gradle, build/: Gradle 빌드 관련 디렉토리를 제외함
- .idea, *.iml: 인텔리제이 프로젝트 설정 파일을 제외함
- .env, *.env: 환경변수 파일을 제외함
- !.env.sample: .env.sample 파일은 예외적으로 포함함

.gitignore 파일은 반드시 프로젝트 초기에 설정해야 함
실수로 .env 파일을 한번이라도 커밋하면 git 히스토리에 남아서 삭제하기 어려움


## 3. .env.sample 환경변수 샘플

> 커밋: `chore: .env.sample 환경변수 샘플 파일 추가`

프로젝트 루트에 .env.sample 파일을 생성함
이 파일은 다른 개발자나 프로젝트 사용자에게 어떤 환경변수가 필요한지 안내하는 역할을 함

```
# 환경변수 샘플 파일
# 이 파일을 복사하여 .env 파일로 저장한 후 실제 값을 입력하세요
# cp sample.env .env

# 서버 포트 (기본값: 8080)
SERVER_PORT=8080

# 데이터베이스 설정 (H2 인메모리 기본값)
DB_URL=jdbc:h2:mem:testdb
DB_USERNAME=sa
DB_PASSWORD=

# JWT 시크릿 키 (실제 운영 시 변경 필요)
JWT_SECRET=your-secret-key-here

# 메일 설정 (필요시 사용)
MAIL_USERNAME=your-email@example.com
MAIL_PASSWORD=your-app-password
```

.env.sample 파일의 역할:
- 필요한 환경변수 목록을 문서화함
- 환경변수 값의 형식을 예시로 보여줌
- 실제 비밀 값이 아닌 플레이스홀더를 사용하므로 GitHub에 올려도 안전함

### 3-1. .env 파일 생성

프로젝트를 클론한 후 .env.sample을 복사하여 .env 파일을 생성함

```bash
cp .env.sample .env
```

.env 파일을 열고 실제 환경에 맞게 값을 수정함:
```
SERVER_PORT=8080
DB_URL=jdbc:h2:mem:testdb
DB_USERNAME=sa
DB_PASSWORD=
JWT_SECRET=실제시크릿키
```

.env 파일은 .gitignore에 등록되어 있어서 git에 올라가지 않음
따라서 실제 비밀 값을 입력해도 GitHub에 노출되지 않음


## 4. build.gradle 설정

> 커밋: `feat(build): build.gradle에 환경변수 로드 스크립트 추가`

인텔리제이가 생성한 build.gradle을 다음과 같이 수정함
Springdoc OpenAPI 의존성을 추가하고, .env 파일 로드 기능을 설정함

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.4.5'
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
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:2.8.8'

    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    runtimeOnly 'com.h2database:h2'

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
                    // 환경변수가 설정되지 않은 경우에만 .env 값 사용
                    System.setProperty(key.trim(), value.trim())
                }
            }
        }
    }
}

// 빌드 스크립트 실행 시 환경변수 로드
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
- spring-boot-starter-web: Spring MVC 기반 웹 애플리케이션 개발용 스타터임
- spring-boot-starter-data-jpa: JPA를 사용한 데이터 접근 계층 구현을 지원함
- springdoc-openapi-starter-webmvc-ui: Swagger UI로 API 문서를 자동 생성함
- h2: 인메모리 데이터베이스로 별도 DB 설치 없이 테스트 가능함
- lombok: 보일러플레이트 코드(getter, setter, 생성자 등)를 어노테이션으로 자동 생성함

build.gradle 하단의 환경변수 로드 설정 설명:
- loadEnv() 함수: .env 파일을 읽어서 System.setProperty()로 시스템 프로퍼티에 등록
- loadEnv() 호출: 스크립트 실행 시점에 환경변수를 로드
- bootRun 블록: ./gradlew bootRun 실행 시 환경변수를 실행 환경에 주입

이 설정이 있어야 `./gradlew bootRun` 명령어로 실행할 때 .env 파일의 값을 사용할 수 있음

Gradle 코끼리 아이콘을 클릭하여 의존성을 다운로드함


## 5. application.yml 설정

> 커밋: `feat(config): application.yml 설정 파일 추가`

src/main/resources/application.yml

```yaml
server:
  port: 8080

spring:
  application:
    name: spring-swagger-project
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
    username: sa
    password:
  h2:
    console:
      enabled: true
      path: /h2-console
  jpa:
    hibernate:
      ddl-auto: create
    show-sql: true
    properties:
      hibernate:
        format_sql: true

springdoc:
  swagger-ui:
    path: /swagger-ui.html
    tags-sorter: alpha
    operations-sorter: alpha
  api-docs:
    path: /api-docs
  default-consumes-media-type: application/json
  default-produces-media-type: application/json
```

설정 항목 설명:
- server.port: 애플리케이션 실행 포트 (기본값 8080)
- spring.datasource: H2 인메모리 데이터베이스 접속 정보
- spring.h2.console: H2 콘솔 활성화. /h2-console 경로에서 DB 내용을 확인할 수 있음
- spring.jpa.hibernate.ddl-auto: create로 설정하면 애플리케이션 시작 시 테이블을 새로 생성함
- spring.jpa.show-sql: JPA가 실행하는 SQL을 콘솔에 출력함
- springdoc.swagger-ui.path: Swagger UI 접속 경로
- springdoc.swagger-ui.tags-sorter: API 태그를 알파벳 순으로 정렬함
- springdoc.swagger-ui.operations-sorter: API 오퍼레이션을 알파벳 순으로 정렬함
- springdoc.api-docs.path: OpenAPI JSON 문서 경로
- springdoc.default-consumes-media-type: 기본 요청 미디어 타입을 application/json으로 설정함
- springdoc.default-produces-media-type: 기본 응답 미디어 타입을 application/json으로 설정함


## 6. SwaggerConfig 클래스

> 커밋: `feat(config): SwaggerConfig 클래스 추가`

src/main/java/com/example/swagger/config/SwaggerConfig.java

```java
package com.example.swagger.config;

import io.swagger.v3.oas.models.Components;
import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Contact;
import io.swagger.v3.oas.models.info.Info;
import io.swagger.v3.oas.models.info.License;
import io.swagger.v3.oas.models.security.SecurityRequirement;
import io.swagger.v3.oas.models.security.SecurityScheme;
import org.springdoc.core.models.GroupedOpenApi;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SwaggerConfig {

    private static final String JWT_SCHEME_NAME = "JWT";

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
                .info(new Info()
                        .title("Spring Swagger 학습용 API")
                        .version("1.0.0")
                        .description("Swagger 학습을 위한 상품 관리 REST API 명세서")
                        .contact(new Contact()
                                .name("개발자")
                                .email("developer@example.com")
                                .url("https://example.com"))
                        .license(new License()
                                .name("Apache 2.0")
                                .url("http://www.apache.org/licenses/LICENSE-2.0.html")))
                .addSecurityItem(new SecurityRequirement().addList(JWT_SCHEME_NAME))
                .components(new Components()
                        .addSecuritySchemes(JWT_SCHEME_NAME, new SecurityScheme()
                                .name(JWT_SCHEME_NAME)
                                .type(SecurityScheme.Type.HTTP)
                                .scheme("bearer")
                                .bearerFormat("JWT")
                                .description("JWT 토큰을 입력하세요")));
    }

    @Bean
    public GroupedOpenApi productApi() {
        return GroupedOpenApi.builder()
                .group("상품 API")
                .pathsToMatch("/api/products/**")
                .build();
    }

    @Bean
    public GroupedOpenApi allApi() {
        return GroupedOpenApi.builder()
                .group("전체 API")
                .pathsToMatch("/api/**")
                .build();
    }
}
```

### 6-1. OpenAPI 빈 설정

customOpenAPI() 메서드는 Swagger 문서의 전체적인 메타 정보를 설정함
- Info: API 제목, 버전, 설명, 연락처, 라이선스 정보를 포함함
- Contact: API 개발자의 이름, 이메일, 웹사이트를 설정함
- License: API 라이선스 정보를 설정함
- SecurityRequirement: 전역 보안 요구사항을 설정함. 모든 API에 JWT 인증을 적용함
- SecurityScheme: JWT Bearer 토큰 인증 방식을 정의함. Swagger UI에서 Authorize 버튼으로 토큰을 입력할 수 있음

### 6-2. GroupedOpenApi 설정

API를 그룹별로 분류하여 Swagger UI에서 선택적으로 조회할 수 있도록 설정함
- productApi(): /api/products/** 경로의 API만 "상품 API" 그룹으로 묶음
- allApi(): /api/** 경로의 모든 API를 "전체 API" 그룹으로 묶음

Swagger UI 상단의 드롭다운에서 그룹을 선택하면 해당 그룹의 API만 표시됨
대규모 프로젝트에서 도메인별로 API를 분리해서 관리할 때 유용함


## 7. Product 엔티티

> 커밋: `feat(entity): Product 엔티티 추가`

src/main/java/com/example/swagger/domain/product/model/entity/Product.java

```java
package com.example.swagger.domain.product.model.entity;

import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

import java.time.LocalDateTime;

@Entity
@Table(name = "tb_product")
@Getter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(length = 1000)
    private String description;

    @Column(nullable = false)
    private Integer price;

    @Column(nullable = false)
    private Integer stock;

    @Column(nullable = false, updatable = false)
    private LocalDateTime createdAt;

    private LocalDateTime updatedAt;

    @PrePersist
    protected void onCreate() {
        this.createdAt = LocalDateTime.now();
    }

    @PreUpdate
    protected void onUpdate() {
        this.updatedAt = LocalDateTime.now();
    }

    public void update(String name, String description, Integer price, Integer stock) {
        this.name = name;
        this.description = description;
        this.price = price;
        this.stock = stock;
    }
}
```

### 7-1. 어노테이션 설명

- @Entity: JPA 엔티티 클래스임을 선언함. 데이터베이스 테이블과 매핑됨
- @Table(name = "tb_product"): 매핑할 테이블 이름을 tb_product로 지정함
- @Getter: Lombok이 모든 필드에 대한 getter 메서드를 자동 생성함
- @NoArgsConstructor: 기본 생성자를 자동 생성함. JPA 엔티티에 필수임
- @AllArgsConstructor: 모든 필드를 파라미터로 받는 생성자를 자동 생성함
- @Builder: 빌더 패턴을 자동 생성함. 객체 생성 시 가독성이 좋음

### 7-2. 필드 설명

- @Id, @GeneratedValue(strategy = GenerationType.IDENTITY): 기본키 자동 생성. IDENTITY 전략은 데이터베이스의 auto_increment를 사용함
- @Column(nullable = false): NOT NULL 제약조건을 설정함
- @Column(length = 1000): 컬럼의 최대 길이를 1000으로 설정함
- @Column(updatable = false): 엔티티 수정 시 해당 컬럼은 업데이트하지 않음

### 7-3. 생명주기 콜백

- @PrePersist: 엔티티가 처음 저장되기 전에 실행됨. createdAt에 현재 시간을 자동 설정함
- @PreUpdate: 엔티티가 수정되기 전에 실행됨. updatedAt에 현재 시간을 자동 설정함

### 7-4. update 메서드

엔티티의 상태를 변경하는 메서드임
JPA의 더티 체킹(dirty checking)으로 트랜잭션 커밋 시 변경된 필드가 자동으로 UPDATE 쿼리에 반영됨
setter를 노출하지 않고 의미 있는 메서드명으로 상태 변경을 수행하는 것이 객체지향적인 설계임


## 8. ProductRequestDTO

> 커밋: `feat(dto): ProductRequestDTO 추가`

src/main/java/com/example/swagger/domain/product/model/dto/ProductRequestDTO.java

```java
package com.example.swagger.domain.product.model.dto;

import io.swagger.v3.oas.annotations.media.Schema;
import jakarta.validation.constraints.Min;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Schema(description = "상품 등록/수정 요청 DTO")
@Getter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class ProductRequestDTO {

    @Schema(description = "상품명", example = "맥북 프로 14인치", requiredMode = Schema.RequiredMode.REQUIRED)
    @NotBlank(message = "상품명은 필수입니다")
    private String name;

    @Schema(description = "상품 설명", example = "Apple M3 Pro 칩셋 탑재")
    private String description;

    @Schema(description = "가격", example = "2990000", minimum = "0", requiredMode = Schema.RequiredMode.REQUIRED)
    @NotNull(message = "가격은 필수입니다")
    @Min(value = 0, message = "가격은 0 이상이어야 합니다")
    private Integer price;

    @Schema(description = "재고 수량", example = "100", minimum = "0", requiredMode = Schema.RequiredMode.REQUIRED)
    @NotNull(message = "재고 수량은 필수입니다")
    @Min(value = 0, message = "재고 수량은 0 이상이어야 합니다")
    private Integer stock;
}
```

### 8-1. Swagger 어노테이션

- @Schema(description): 클래스 레벨에서 DTO 전체에 대한 설명을 추가함
- @Schema(description, example): 필드 레벨에서 설명과 예시 값을 지정함. Swagger UI에서 Try it out 시 예시 값이 자동으로 채워짐
- @Schema(requiredMode = Schema.RequiredMode.REQUIRED): 해당 필드가 필수임을 Swagger 문서에 표시함
- @Schema(minimum = "0"): 최소값 제약조건을 Swagger 문서에 표시함

### 8-2. Bean Validation 어노테이션

- @NotBlank: null, 빈 문자열, 공백만 있는 문자열을 거부함. 문자열 필드에 적합함
- @NotNull: null만 거부함. 숫자 타입 필드에 적합함
- @Min(value = 0): 0 이상의 값만 허용함

Bean Validation 어노테이션을 사용하면 컨트롤러에서 @Valid와 함께 사용할 때 자동으로 유효성 검사가 수행됨


## 9. ProductResponseDTO

> 커밋: `feat(dto): ProductResponseDTO 추가`

src/main/java/com/example/swagger/domain/product/model/dto/ProductResponseDTO.java

```java
package com.example.swagger.domain.product.model.dto;

import com.example.swagger.domain.product.model.entity.Product;
import io.swagger.v3.oas.annotations.media.Schema;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

import java.time.LocalDateTime;

@Schema(description = "상품 응답 DTO")
@Getter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class ProductResponseDTO {

    @Schema(description = "상품 ID", example = "1")
    private Long id;

    @Schema(description = "상품명", example = "맥북 프로 14인치")
    private String name;

    @Schema(description = "상품 설명", example = "Apple M3 Pro 칩셋 탑재")
    private String description;

    @Schema(description = "가격", example = "2990000")
    private Integer price;

    @Schema(description = "재고 수량", example = "100")
    private Integer stock;

    @Schema(description = "생성일시", example = "2024-01-15T10:30:00")
    private LocalDateTime createdAt;

    @Schema(description = "수정일시", example = "2024-01-16T14:20:00")
    private LocalDateTime updatedAt;

    public static ProductResponseDTO from(Product product) {
        return ProductResponseDTO.builder()
                .id(product.getId())
                .name(product.getName())
                .description(product.getDescription())
                .price(product.getPrice())
                .stock(product.getStock())
                .createdAt(product.getCreatedAt())
                .updatedAt(product.getUpdatedAt())
                .build();
    }
}
```

### 9-1. 정적 팩토리 메서드

from() 메서드는 Product 엔티티를 ProductResponseDTO로 변환하는 정적 팩토리 메서드임
- 엔티티를 직접 API 응답으로 반환하지 않고 DTO로 변환하는 것이 보안과 유지보수에 유리함
- 엔티티의 모든 필드가 아닌 필요한 필드만 선택적으로 노출할 수 있음
- 빌더 패턴을 사용하여 가독성 있게 객체를 생성함

### 9-2. Swagger 문서에서의 역할

@Schema 어노테이션으로 각 필드에 설명과 예시 값을 지정하면 Swagger UI에서 응답 형식을 명확하게 확인할 수 있음
프론트엔드 개발자나 API 사용자가 응답 구조를 이해하는 데 도움이 됨


## 10. ProductRepository

> 커밋: `feat(repository): ProductRepository 추가`

src/main/java/com/example/swagger/domain/product/repository/ProductRepository.java

```java
package com.example.swagger.domain.product.repository;

import com.example.swagger.domain.product.model.entity.Product;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {

    List<Product> findByNameContaining(String name);
}
```

### 10-1. JpaRepository 상속

JpaRepository<Product, Long>을 상속하면 기본적인 CRUD 메서드가 자동으로 제공됨
- save(): 엔티티 저장 및 수정
- findById(): ID로 단건 조회
- findAll(): 전체 조회
- deleteById(): ID로 삭제
- existsById(): ID 존재 여부 확인

### 10-2. 쿼리 메서드

findByNameContaining(String name)은 Spring Data JPA의 쿼리 메서드 기능을 활용함
메서드 이름만으로 SQL 쿼리가 자동 생성됨
- findBy: SELECT 쿼리 실행
- Name: name 컬럼을 조건으로 사용
- Containing: LIKE '%keyword%' 조건 (부분 일치 검색)

실행되는 SQL: `SELECT * FROM tb_product WHERE name LIKE '%keyword%'`


## 11. ProductService

> 커밋: `feat(service): ProductService 추가`

src/main/java/com/example/swagger/domain/product/service/ProductService.java

```java
package com.example.swagger.domain.product.service;

import com.example.swagger.domain.product.model.dto.ProductRequestDTO;
import com.example.swagger.domain.product.model.dto.ProductResponseDTO;
import com.example.swagger.domain.product.model.entity.Product;
import com.example.swagger.domain.product.repository.ProductRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.stream.Collectors;

@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class ProductService {

    private final ProductRepository productRepository;

    public List<ProductResponseDTO> findAll() {
        return productRepository.findAll().stream()
                .map(ProductResponseDTO::from)
                .collect(Collectors.toList());
    }

    public ProductResponseDTO findById(Long id) {
        Product product = productRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("상품을 찾을 수 없습니다. ID: " + id));
        return ProductResponseDTO.from(product);
    }

    public List<ProductResponseDTO> searchByName(String name) {
        return productRepository.findByNameContaining(name).stream()
                .map(ProductResponseDTO::from)
                .collect(Collectors.toList());
    }

    @Transactional
    public ProductResponseDTO create(ProductRequestDTO requestDTO) {
        Product product = Product.builder()
                .name(requestDTO.getName())
                .description(requestDTO.getDescription())
                .price(requestDTO.getPrice())
                .stock(requestDTO.getStock())
                .build();

        Product savedProduct = productRepository.save(product);
        return ProductResponseDTO.from(savedProduct);
    }

    @Transactional
    public ProductResponseDTO update(Long id, ProductRequestDTO requestDTO) {
        Product product = productRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("상품을 찾을 수 없습니다. ID: " + id));

        product.update(
                requestDTO.getName(),
                requestDTO.getDescription(),
                requestDTO.getPrice(),
                requestDTO.getStock()
        );

        return ProductResponseDTO.from(product);
    }

    @Transactional
    public void delete(Long id) {
        if (!productRepository.existsById(id)) {
            throw new IllegalArgumentException("상품을 찾을 수 없습니다. ID: " + id);
        }
        productRepository.deleteById(id);
    }
}
```

### 11-1. 어노테이션 설명

- @Service: 서비스 계층의 스프링 빈으로 등록함
- @RequiredArgsConstructor: final 필드에 대한 생성자를 Lombok이 자동 생성함. 생성자 주입 방식으로 의존성을 주입함
- @Transactional(readOnly = true): 클래스 레벨에서 읽기 전용 트랜잭션을 기본으로 설정함. 읽기 전용 트랜잭션은 JPA 더티 체킹을 수행하지 않아 성능이 향상됨
- @Transactional: 메서드 레벨에서 쓰기 가능 트랜잭션을 설정함. 클래스 레벨의 readOnly=true를 오버라이드함

### 11-2. CRUD 메서드

- findAll(): 전체 상품을 조회하고 DTO 리스트로 변환함
- findById(): ID로 상품을 조회함. 존재하지 않으면 IllegalArgumentException을 발생시킴
- searchByName(): 상품명으로 부분 일치 검색을 수행함
- create(): 요청 DTO를 엔티티로 변환하여 저장함. 빌더 패턴으로 객체를 생성함
- update(): ID로 기존 상품을 조회한 후 엔티티의 update 메서드로 상태를 변경함. JPA 더티 체킹으로 자동 UPDATE됨
- delete(): 존재 여부를 확인한 후 삭제함. 존재하지 않으면 예외를 발생시킴


## 12. ProductController

> 커밋: `feat(controller): ProductController 추가`

src/main/java/com/example/swagger/domain/product/controller/ProductController.java

```java
package com.example.swagger.domain.product.controller;

import com.example.swagger.domain.product.model.dto.ProductRequestDTO;
import com.example.swagger.domain.product.model.dto.ProductResponseDTO;
import com.example.swagger.domain.product.service.ProductService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.media.Content;
import io.swagger.v3.oas.annotations.media.Schema;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@Tag(name = "상품 API", description = "상품 CRUD 관련 API")
@RestController
@RequestMapping("/api/products")
@RequiredArgsConstructor
public class ProductController {

    private final ProductService productService;

    @Operation(summary = "전체 상품 조회", description = "등록된 모든 상품 목록을 조회합니다")
    @ApiResponses(value = {
            @ApiResponse(responseCode = "200", description = "조회 성공",
                    content = @Content(schema = @Schema(implementation = ProductResponseDTO.class)))
    })
    @GetMapping
    public ResponseEntity<List<ProductResponseDTO>> getAllProducts() {
        return ResponseEntity.ok(productService.findAll());
    }

    @Operation(summary = "상품 상세 조회", description = "상품 ID로 특정 상품의 상세 정보를 조회합니다")
    @ApiResponses(value = {
            @ApiResponse(responseCode = "200", description = "조회 성공",
                    content = @Content(schema = @Schema(implementation = ProductResponseDTO.class))),
            @ApiResponse(responseCode = "404", description = "상품을 찾을 수 없음",
                    content = @Content)
    })
    @GetMapping("/{id}")
    public ResponseEntity<ProductResponseDTO> getProduct(
            @Parameter(description = "상품 ID", required = true, example = "1")
            @PathVariable Long id) {
        return ResponseEntity.ok(productService.findById(id));
    }

    @Operation(summary = "상품 검색", description = "상품명으로 상품을 검색합니다")
    @ApiResponses(value = {
            @ApiResponse(responseCode = "200", description = "검색 성공",
                    content = @Content(schema = @Schema(implementation = ProductResponseDTO.class)))
    })
    @GetMapping("/search")
    public ResponseEntity<List<ProductResponseDTO>> searchProducts(
            @Parameter(description = "검색할 상품명", required = true, example = "맥북")
            @RequestParam String name) {
        return ResponseEntity.ok(productService.searchByName(name));
    }

    @Operation(summary = "상품 등록", description = "새로운 상품을 등록합니다")
    @ApiResponses(value = {
            @ApiResponse(responseCode = "201", description = "등록 성공",
                    content = @Content(schema = @Schema(implementation = ProductResponseDTO.class))),
            @ApiResponse(responseCode = "400", description = "잘못된 요청",
                    content = @Content)
    })
    @PostMapping
    public ResponseEntity<ProductResponseDTO> createProduct(
            @Parameter(description = "상품 등록 정보", required = true)
            @Valid @RequestBody ProductRequestDTO requestDTO) {
        return ResponseEntity.status(HttpStatus.CREATED)
                .body(productService.create(requestDTO));
    }

    @Operation(summary = "상품 수정", description = "기존 상품 정보를 수정합니다")
    @ApiResponses(value = {
            @ApiResponse(responseCode = "200", description = "수정 성공",
                    content = @Content(schema = @Schema(implementation = ProductResponseDTO.class))),
            @ApiResponse(responseCode = "404", description = "상품을 찾을 수 없음",
                    content = @Content),
            @ApiResponse(responseCode = "400", description = "잘못된 요청",
                    content = @Content)
    })
    @PutMapping("/{id}")
    public ResponseEntity<ProductResponseDTO> updateProduct(
            @Parameter(description = "상품 ID", required = true, example = "1")
            @PathVariable Long id,
            @Parameter(description = "상품 수정 정보", required = true)
            @Valid @RequestBody ProductRequestDTO requestDTO) {
        return ResponseEntity.ok(productService.update(id, requestDTO));
    }

    @Operation(summary = "상품 삭제", description = "상품을 삭제합니다")
    @ApiResponses(value = {
            @ApiResponse(responseCode = "204", description = "삭제 성공"),
            @ApiResponse(responseCode = "404", description = "상품을 찾을 수 없음",
                    content = @Content)
    })
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(
            @Parameter(description = "상품 ID", required = true, example = "1")
            @PathVariable Long id) {
        productService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

### 12-1. 클래스 레벨 어노테이션

- @Tag(name, description): Swagger UI에서 API를 그룹화하는 태그를 설정함. 같은 태그를 가진 API가 하나의 그룹으로 표시됨
- @RestController: @Controller + @ResponseBody. JSON 응답을 반환함
- @RequestMapping("/api/products"): 공통 URL 접두사를 설정함
- @RequiredArgsConstructor: final 필드에 대한 생성자를 Lombok이 자동 생성함

### 12-2. Swagger 어노테이션 설명

- @Operation(summary, description): API 엔드포인트의 요약과 상세 설명을 지정함. summary는 API 목록에 표시되고, description은 상세 보기에 표시됨
- @ApiResponses: 가능한 응답 코드와 설명을 정의함
- @ApiResponse(responseCode, description): 각 HTTP 상태 코드별 응답 설명을 지정함
- @Content(schema = @Schema(implementation)): 응답 바디의 스키마를 지정함. Swagger UI에서 응답 구조를 확인할 수 있음
- @Parameter(description, required, example): 요청 파라미터의 설명, 필수 여부, 예시 값을 지정함

### 12-3. API 엔드포인트

| HTTP 메서드 | 경로 | 설명 | 응답 코드 |
|-------------|------|------|-----------|
| GET | /api/products | 전체 상품 조회 | 200 |
| GET | /api/products/{id} | 상품 상세 조회 | 200, 404 |
| GET | /api/products/search?name= | 상품명 검색 | 200 |
| POST | /api/products | 상품 등록 | 201, 400 |
| PUT | /api/products/{id} | 상품 수정 | 200, 404, 400 |
| DELETE | /api/products/{id} | 상품 삭제 | 204, 404 |

### 12-4. @Valid와 Bean Validation

@Valid 어노테이션이 @RequestBody와 함께 사용되면 요청 바디의 유효성 검사를 수행함
ProductRequestDTO에 정의된 @NotBlank, @NotNull, @Min 어노테이션에 의해 잘못된 요청이 자동으로 거부됨
유효성 검사에 실패하면 400 Bad Request 응답이 반환됨


## 애플리케이션 실행 및 테스트

### 13-1. 애플리케이션 실행

터미널에서 다음 명령어로 실행함

```bash
./gradlew bootRun
```

build.gradle에 설정한 bootRun 태스크가 .env 파일을 읽어서 환경변수로 등록함
인텔리제이의 Run 버튼 대신 터미널에서 실행해야 .env 파일이 적용됨

콘솔에 다음과 같은 로그가 출력되면 성공임

```
Started SpringSwaggerProjectApplication in X.XXX seconds
```

### 13-2. Swagger UI 접속

브라우저에서 http://localhost:8080/swagger-ui.html 접속
상품 API 목록이 표시됨

Swagger UI 상단의 드롭다운에서 "상품 API" 또는 "전체 API" 그룹을 선택할 수 있음
Authorize 버튼을 클릭하면 JWT 토큰을 입력할 수 있는 팝업이 표시됨

### 13-3. H2 콘솔 접속

브라우저에서 http://localhost:8080/h2-console 접속
- JDBC URL: jdbc:h2:mem:testdb
- User Name: sa
- Password: (비어 있음)

Connect 클릭 후 TB_PRODUCT 테이블의 데이터를 확인할 수 있음

### 13-4. 상품 등록 테스트

1. Swagger UI에서 POST /api/products 엔드포인트 선택
2. Try it out 클릭
3. Request body에 다음 입력

```json
{
  "name": "맥북 프로 14인치",
  "description": "Apple M3 Pro 칩셋 탑재",
  "price": 2990000,
  "stock": 100
}
```

4. Execute 클릭
5. 성공 시 201 Created와 함께 다음과 같은 응답이 반환됨

```json
{
  "id": 1,
  "name": "맥북 프로 14인치",
  "description": "Apple M3 Pro 칩셋 탑재",
  "price": 2990000,
  "stock": 100,
  "createdAt": "2026-03-05T10:30:00",
  "updatedAt": null
}
```

### 13-5. 전체 상품 조회 테스트

1. GET /api/products 엔드포인트 선택
2. Try it out 클릭
3. Execute 클릭
4. 등록된 모든 상품 목록이 배열로 반환됨

### 13-6. 상품 검색 테스트

1. GET /api/products/search 엔드포인트 선택
2. Try it out 클릭
3. name 파라미터에 "맥북" 입력
4. Execute 클릭
5. 상품명에 "맥북"이 포함된 상품 목록이 반환됨

### 13-7. 상품 수정 테스트

1. PUT /api/products/{id} 엔드포인트 선택
2. Try it out 클릭
3. id에 1 입력
4. Request body에 다음 입력

```json
{
  "name": "맥북 프로 16인치",
  "description": "Apple M3 Max 칩셋 탑재",
  "price": 3990000,
  "stock": 50
}
```

5. Execute 클릭
6. 수정된 상품 정보가 반환됨

### 13-8. 상품 삭제 테스트

1. DELETE /api/products/{id} 엔드포인트 선택
2. Try it out 클릭
3. id에 1 입력
4. Execute 클릭
5. 204 No Content 응답이 반환됨

### 13-9. Postman으로 테스트

Postman을 사용해서 테스트할 수도 있음

상품 등록:
- Method: POST
- URL: http://localhost:8080/api/products
- Headers: Content-Type: application/json
- Body (raw JSON):
```json
{
  "name": "맥북 프로 14인치",
  "description": "Apple M3 Pro 칩셋 탑재",
  "price": 2990000,
  "stock": 100
}
```

전체 상품 조회:
- Method: GET
- URL: http://localhost:8080/api/products

상품 검색:
- Method: GET
- URL: http://localhost:8080/api/products/search?name=맥북

상품 수정:
- Method: PUT
- URL: http://localhost:8080/api/products/1
- Headers: Content-Type: application/json
- Body (raw JSON):
```json
{
  "name": "맥북 프로 16인치",
  "description": "Apple M3 Max 칩셋 탑재",
  "price": 3990000,
  "stock": 50
}
```

상품 삭제:
- Method: DELETE
- URL: http://localhost:8080/api/products/1


## 커밋 히스토리

이 프로젝트의 커밋 로그를 순서대로 따라가면 스프링부트 프로젝트 구성 흐름을 이해할 수 있음

```
feat(init): 스프링부트 프로젝트 초기 설정
chore: .gitignore 추가
chore: .env.sample 환경변수 샘플 파일 추가
feat(build): build.gradle에 환경변수 로드 스크립트 추가
feat(config): application.yml 설정 파일 추가
feat(config): SwaggerConfig 클래스 추가
feat(entity): Product 엔티티 추가
feat(dto): ProductRequestDTO 추가
feat(dto): ProductResponseDTO 추가
feat(repository): ProductRepository 추가
feat(service): ProductService 추가
feat(controller): ProductController 추가
```

각 커밋 메시지에 해당 파일의 역할과 코드 설명이 포함되어 있음

첫 번째 커밋(feat(init))에 build.gradle, Gradle Wrapper, settings.gradle, 메인 클래스가 모두 포함됨
이후 .gitignore, .env.sample, 환경변수 스크립트 등이 순차적으로 추가됨


## 운영 환경 고려사항

실제 서비스에 적용할 때 추가로 고려해야 할 사항:

- Swagger UI 접근 제한: 운영 환경에서는 springdoc.swagger-ui.enabled=false로 비활성화하거나 Spring Security로 접근을 제한해야 함
- 데이터베이스 변경: H2 인메모리 대신 MySQL, PostgreSQL 등 운영용 RDBMS를 사용해야 함
- JWT 인증 실제 구현: SwaggerConfig에 JWT SecurityScheme이 정의되어 있으나, 실제 인증 로직(Spring Security + JWT 필터)은 별도로 구현해야 함
- 에러 핸들링 강화: @RestControllerAdvice를 활용하여 전역 예외 처리를 구현해야 함
- 페이지네이션 적용: 상품 목록 조회 시 Spring Data JPA의 Pageable을 활용하여 페이지네이션을 적용해야 함
- API 버전 관리: URL 경로에 /api/v1/ 등의 버전 정보를 포함하여 하위 호환성을 유지해야 함
- 환경별 설정 분리: application-dev.yml, application-prod.yml 등으로 환경별 설정을 분리해야 함


## 참고 자료

- Springdoc OpenAPI: https://springdoc.org
- Swagger 공식 문서: https://swagger.io/docs
- Spring Data JPA 문서: https://docs.spring.io/spring-data/jpa/reference
- Spring Boot 공식 문서: https://docs.spring.io/spring-boot/docs/current/reference/html
