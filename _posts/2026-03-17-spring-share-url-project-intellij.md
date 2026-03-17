# 인텔리제이로 스프링부트 단축 URL 공유 서비스 구현하기

Spring Boot와 Strategy 패턴을 사용하여 단축 URL 생성 및 공유 서비스를 구현하는 방법을 정리함

GitHub 저장소: https://github.com/tletle7102/spring-share-url-project

커밋 로그를 순서대로 따라가면서 학습하면 프로젝트 구성 흐름을 이해할 수 있음


## 목차

1. 프로젝트 초기 설정
2. .gitignore 설정
3. .env.sample 환경변수 샘플
4. build.gradle 설정
5. application.yml 설정
6. ShareChannelType 공유 채널 열거형
7. ShortUrl 단축 URL 엔티티
8. ShortUrlRepository 단축 URL 저장소
9. ShareRequestDto 공유 요청 DTO
10. ShareResponseDto 공유 응답 DTO
11. ShortUrlInfoDto 단축 URL 정보 DTO
12. ShortUrlStatsDto 단축 URL 통계 DTO
13. Base62Encoder 단축 코드 생성기
14. ShareChannel 공유 채널 인터페이스
15. LinkShareChannel 링크 공유 채널 구현체
16. ShareService 공유 서비스
17. ShareRestController 공유 API 컨트롤러
18. RedirectController 리다이렉트 컨트롤러
19. SwaggerConfig API 문서화 설정
20. 애플리케이션 실행 및 테스트
21. 커밋 히스토리
22. 운영 환경 고려사항
23. 참고 자료


## 1. 프로젝트 초기 설정

> 커밋: `feat(init): 스프링부트 프로젝트 초기설정`

### 1-1. 인텔리제이에서 새 프로젝트 생성

1. File > New > Project 선택
2. 좌측에서 "Spring Boot" 선택
3. 다음 설정 입력
   - Name: spring-share-url-project
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
   - Validation
   - Lombok
   - H2 Database
3. Create 클릭

프로젝트가 생성되면 다음 파일들이 자동 생성됨:
- settings.gradle: 프로젝트 이름 설정
- gradlew, gradlew.bat: Gradle Wrapper 실행 스크립트
- gradle/wrapper/: Gradle Wrapper 설정 파일

### 1-3. 메인 클래스

src/main/java/com/example/share/ShareUrlApplication.java

```java
package com.example.share;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringShareUrlProjectApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringShareUrlProjectApplication.class, args);
    }
}
```

@SpringBootApplication 어노테이션은 @Configuration, @EnableAutoConfiguration, @ComponentScan을 포함하며 스프링부트 애플리케이션의 진입점을 정의함


## 2. .gitignore 설정

> 커밋: `chore: .gitignore 추가`

.gitignore 파일에 환경변수 파일이 저장소에 올라가지 않도록 설정함
이 설정을 하지 않으면 API 키 같은 민감 정보가 GitHub에 노출될 수 있음

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

### Mac ###
.DS_Store

### Environment Variables ###
.env
*.env
```

.gitignore 파일은 반드시 프로젝트 초기에 설정해야 함
실수로 .env 파일을 한번이라도 커밋하면 git 히스토리에 남아서 삭제하기 어려움

인텔리제이가 생성한 .gitignore에 환경변수 관련 항목이 없다면 직접 추가해야 함


## 3. .env.sample 환경변수 샘플

> 커밋: `chore: .env.sample 환경변수 샘플파일 추가`

프로젝트 루트에 .env.sample 파일을 생성함
이 파일은 다른 개발자나 프로젝트 사용자에게 어떤 환경변수가 필요한지 안내하는 역할을 함

```
# 서버 설정
# 단축 URL 생성 시 사용할 기본 도메인
BASE_URL=http://localhost:8080

# 서버 포트 (선택사항, 기본값: 8080)
SERVER_PORT=8080
```

.env.sample 파일의 역할:
- 필요한 환경변수 목록을 문서화함
- 환경변수 값의 형식을 예시로 보여줌
- 실제 비밀 값이 아닌 기본값을 사용하므로 GitHub에 올려도 안전함

### 3-1. .env 파일 생성

프로젝트를 클론한 후 .env.sample을 복사하여 .env 파일을 생성함

```bash
cp .env.sample .env
```

.env 파일을 열고 필요한 경우 값을 수정함:
```
BASE_URL=http://localhost:8080
SERVER_PORT=8080
```

.env 파일은 .gitignore에 등록되어 있어서 git에 올라가지 않음
따라서 실제 환경에 맞는 값을 입력해도 GitHub에 노출되지 않음


## 4. build.gradle 설정

> 커밋: `build: build.gradle 환경변수 로드 스크립트 추가`

인텔리제이가 생성한 build.gradle을 다음과 같이 수정함
JPA, H2, Springdoc 의존성을 추가하고, .env 파일 로드 기능을 설정함

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
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:2.8.8'

    runtimeOnly 'com.h2database:h2'

    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}

tasks.named('test') {
    useJUnitPlatform()
}

// .env 파일에서 환경변수 로드 (빌드 전체에서 사용)
def loadEnv() {
    def envFile = file('.env')
    if (envFile.exists()) {
        envFile.readLines().each { line ->
            if (line && !line.startsWith('#') && line.contains('=')) {
                def parts = line.split('=', 2)
                if (parts.length == 2) {
                    def key = parts[0].trim()
                    def value = parts[1].trim()
                    if (System.getenv(key) == null) {
                        System.setProperty(key, value)
                    }
                }
            }
        }
    }
}

// 빌드 시작 시 환경변수 로드
loadEnv()

// bootRun 태스크에도 환경변수 적용
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
- spring-boot-starter-data-jpa: JPA를 사용한 데이터베이스 접근을 지원함
- h2: 인메모리 데이터베이스로 별도 DB 설치 없이 개발 및 테스트가 가능함
- springdoc-openapi-starter-webmvc-ui: Swagger UI로 API 문서를 자동 생성함
- spring-boot-starter-validation: Bean Validation으로 요청 데이터 검증함

build.gradle 하단의 환경변수 로드 설정 설명:
- loadEnv() 함수: .env 파일을 읽어서 System.setProperty()로 시스템 프로퍼티에 등록
- loadEnv() 호출: 스크립트 실행 시점에 환경변수를 로드
- bootRun 블록: ./gradlew bootRun 실행 시 환경변수를 실행 환경에 주입

이 설정이 있어야 `./gradlew bootRun` 명령어로 실행할 때 .env 파일의 값을 사용할 수 있음

Gradle 코끼리 아이콘을 클릭하여 의존성을 다운로드함


## 5. application.yml 설정

> 커밋: `config: application.yml 설정파일 추가`

src/main/resources/application.yml

```yaml
server:
  port: ${SERVER_PORT:8080}

spring:
  application:
    name: spring-share-url-project
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
    open-in-view: false

app:
  base-url: ${BASE_URL:http://localhost:8080}

springdoc:
  api-docs:
    path: /api-docs
  swagger-ui:
    path: /swagger-ui.html
    tags-sorter: alpha
    operations-sorter: alpha
```

설정 항목 설명:
- server.port: 서버 포트. 환경변수 SERVER_PORT가 있으면 그 값을 사용하고, 없으면 기본값 8080을 사용함
- spring.datasource: H2 인메모리 데이터베이스 연결 설정
- spring.h2.console: H2 콘솔을 /h2-console 경로에서 활성화함
- spring.jpa.hibernate.ddl-auto: create로 설정하여 애플리케이션 시작 시 테이블을 자동 생성함
- spring.jpa.open-in-view: false로 설정하여 영속성 컨텍스트를 서비스 계층까지만 유지함
- app.base-url: 단축 URL 생성 시 사용할 기본 도메인. 환경변수 BASE_URL에서 읽어옴

`${BASE_URL:http://localhost:8080}` 형식은 환경변수 BASE_URL이 있으면 그 값을 사용하고, 없으면 기본값 http://localhost:8080을 사용한다는 의미임


## 6. ShareChannelType 공유 채널 열거형

> 커밋: `feat(domain): ShareChannelType 공유 채널 열거형 추가`

src/main/java/com/example/share/domain/shorturl/model/enums/ShareChannelType.java

```java
package com.example.share.domain.shorturl.model.enums;

public enum ShareChannelType {
    LINK("링크 복사"),
    KAKAO("카카오톡"),
    TWITTER("트위터");

    private final String description;

    ShareChannelType(String description) {
        this.description = description;
    }

    public String getDescription() {
        return description;
    }
}
```

공유 채널의 종류를 정의하는 열거형임
- LINK: 단축 URL을 생성하여 링크를 복사하는 채널
- KAKAO: 카카오톡 공유 채널 (확장용)
- TWITTER: 트위터 공유 채널 (확장용)

각 열거형 상수에 한글 설명을 포함하여 UI에서 채널명으로 사용할 수 있음
새로운 공유 채널을 추가할 때 이 열거형에 상수를 추가하면 됨


## 7. ShortUrl 단축 URL 엔티티

> 커밋: `feat(domain): ShortUrl 단축 URL 엔티티 추가`

src/main/java/com/example/share/domain/shorturl/model/entity/ShortUrl.java

```java
package com.example.share.domain.shorturl.model.entity;

import jakarta.persistence.*;
import lombok.AccessLevel;
import lombok.Getter;
import lombok.NoArgsConstructor;

import java.time.LocalDateTime;

@Entity
@Table(name = "tb_short_url")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class ShortUrl {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true, length = 8)
    private String shortCode;

    @Column(nullable = false, length = 2048)
    private String originalUrl;

    @Column(nullable = false)
    private LocalDateTime createdAt;

    @Column(nullable = false)
    private Long clickCount;

    public ShortUrl(String shortCode, String originalUrl) {
        this.shortCode = shortCode;
        this.originalUrl = originalUrl;
        this.createdAt = LocalDateTime.now();
        this.clickCount = 0L;
    }

    public void incrementClickCount() {
        this.clickCount++;
    }
}
```

단축 URL 정보를 저장하는 JPA 엔티티임

어노테이션 설명:
- @Entity: JPA 엔티티로 등록함
- @Table(name = "tb_short_url"): 매핑할 테이블명을 지정함
- @NoArgsConstructor(access = AccessLevel.PROTECTED): JPA가 요구하는 기본 생성자를 protected로 제한하여 외부에서 직접 호출하지 못하게 함
- @GeneratedValue(strategy = GenerationType.IDENTITY): 데이터베이스의 AUTO_INCREMENT를 사용하여 기본키를 자동 생성함

필드 설명:
- shortCode: 8자리 단축 코드. unique 제약조건으로 중복을 방지함
- originalUrl: 원본 URL. 최대 2048자까지 저장 가능함
- createdAt: 생성 시간. 생성자에서 현재 시간으로 자동 설정됨
- clickCount: 클릭 횟수. 생성 시 0으로 초기화됨

incrementClickCount() 메서드로 클릭 수를 1씩 증가시킴


## 8. ShortUrlRepository 단축 URL 저장소

> 커밋: `feat(domain): ShortUrlRepository 단축 URL 저장소 추가`

src/main/java/com/example/share/domain/shorturl/repository/ShortUrlRepository.java

```java
package com.example.share.domain.shorturl.repository;

import com.example.share.domain.shorturl.model.entity.ShortUrl;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface ShortUrlRepository extends JpaRepository<ShortUrl, Long> {

    Optional<ShortUrl> findByShortCode(String shortCode);

    Optional<ShortUrl> findByOriginalUrl(String originalUrl);

    boolean existsByShortCode(String shortCode);
}
```

Spring Data JPA의 JpaRepository를 상속하여 단축 URL 데이터 접근을 담당함

메서드 네이밍 규칙을 따르면 Spring Data JPA가 자동으로 쿼리를 생성함:
- findByShortCode: 단축 코드로 ShortUrl 엔티티를 조회함
- findByOriginalUrl: 원본 URL로 기존에 생성된 단축 URL이 있는지 확인함
- existsByShortCode: 단축 코드의 존재 여부를 boolean으로 반환함

Optional을 반환 타입으로 사용하여 null 안전성을 확보함


## 9. ShareRequestDto 공유 요청 DTO

> 커밋: `feat(dto): ShareRequestDto 공유 요청 DTO 추가`

src/main/java/com/example/share/domain/shorturl/model/dto/ShareRequestDto.java

```java
package com.example.share.domain.shorturl.model.dto;

import com.example.share.domain.shorturl.model.enums.ShareChannelType;
import io.swagger.v3.oas.annotations.media.Schema;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import org.hibernate.validator.constraints.URL;

@Schema(description = "공유 요청 DTO")
@Getter
@Setter
@NoArgsConstructor
public class ShareRequestDto {

    @Schema(description = "원본 URL", example = "https://www.example.com/very/long/url/path")
    @NotBlank(message = "URL은 필수 입력값입니다")
    @URL(message = "올바른 URL 형식이 아닙니다")
    private String originalUrl;

    @Schema(description = "공유 채널 타입", example = "LINK")
    @NotNull(message = "채널 타입은 필수 입력값입니다")
    private ShareChannelType channelType;

    public ShareRequestDto(String originalUrl, ShareChannelType channelType) {
        this.originalUrl = originalUrl;
        this.channelType = channelType;
    }
}
```

클라이언트가 단축 URL 생성을 요청할 때 사용하는 DTO임
- @NotBlank: null, 빈 문자열, 공백만 있는 문자열을 거부함
- @URL: Hibernate Validator에서 제공하는 URL 형식 검증 어노테이션임
- @NotNull: channelType이 반드시 전달되어야 함을 보장함
- @Schema: Swagger 문서에 필드 설명과 예시를 표시함


## 10. ShareResponseDto 공유 응답 DTO

> 커밋: `feat(dto): ShareResponseDto 공유 응답 DTO 추가`

src/main/java/com/example/share/domain/shorturl/model/dto/ShareResponseDto.java

```java
package com.example.share.domain.shorturl.model.dto;

import com.example.share.domain.shorturl.model.enums.ShareChannelType;
import io.swagger.v3.oas.annotations.media.Schema;
import lombok.Getter;

@Schema(description = "공유 응답 DTO")
@Getter
public class ShareResponseDto {

    @Schema(description = "공유 채널 타입")
    private final ShareChannelType channelType;

    @Schema(description = "성공 여부")
    private final boolean success;

    @Schema(description = "응답 메시지")
    private final String message;

    @Schema(description = "생성된 단축 URL")
    private final String shortUrl;

    private ShareResponseDto(ShareChannelType channelType, boolean success, String message, String shortUrl) {
        this.channelType = channelType;
        this.success = success;
        this.message = message;
        this.shortUrl = shortUrl;
    }

    public static ShareResponseDto success(ShareChannelType type, String message, String shortUrl) {
        return new ShareResponseDto(type, true, message, shortUrl);
    }

    public static ShareResponseDto success(ShareChannelType type, String message) {
        return new ShareResponseDto(type, true, message, null);
    }

    public static ShareResponseDto fail(ShareChannelType type, String message) {
        return new ShareResponseDto(type, false, message, null);
    }
}
```

클라이언트에게 반환하는 통일된 응답 형식임
- 생성자를 private으로 선언하여 외부에서 직접 생성하지 못하게 함
- 정적 팩토리 메서드 success(), fail()을 통해 일관된 응답을 생성함
- success 메서드가 오버로딩되어 있어 shortUrl 포함 여부를 선택할 수 있음
- final 필드를 사용하여 불변 객체로 설계됨


## 11. ShortUrlInfoDto 단축 URL 정보 DTO

> 커밋: `feat(dto): ShortUrlInfoDto 단축 URL 정보 DTO 추가`

src/main/java/com/example/share/domain/shorturl/model/dto/ShortUrlInfoDto.java

```java
package com.example.share.domain.shorturl.model.dto;

import com.example.share.domain.shorturl.model.entity.ShortUrl;
import io.swagger.v3.oas.annotations.media.Schema;
import lombok.Getter;

import java.time.LocalDateTime;

@Schema(description = "단축 URL 정보 DTO")
@Getter
public class ShortUrlInfoDto {

    @Schema(description = "단축 코드", example = "abc12345")
    private final String shortCode;

    @Schema(description = "원본 URL")
    private final String originalUrl;

    @Schema(description = "단축 URL")
    private final String shortUrl;

    @Schema(description = "생성 일시")
    private final LocalDateTime createdAt;

    @Schema(description = "클릭 수")
    private final Long clickCount;

    public ShortUrlInfoDto(ShortUrl shortUrl, String baseUrl) {
        this.shortCode = shortUrl.getShortCode();
        this.originalUrl = shortUrl.getOriginalUrl();
        this.shortUrl = baseUrl + "/s/" + shortUrl.getShortCode();
        this.createdAt = shortUrl.getCreatedAt();
        this.clickCount = shortUrl.getClickCount();
    }
}
```

단축 URL의 상세 정보를 조회할 때 반환하는 DTO임
- 생성자에서 ShortUrl 엔티티와 baseUrl을 받아 단축 URL 전체 경로를 조합함
- 엔티티를 직접 반환하지 않고 DTO로 변환하여 계층 간 의존성을 분리함
- shortUrl 필드는 baseUrl + "/s/" + shortCode 형식으로 완전한 URL을 구성함


## 12. ShortUrlStatsDto 단축 URL 통계 DTO

> 커밋: `feat(dto): ShortUrlStatsDto 단축 URL 통계 DTO 추가`

src/main/java/com/example/share/domain/shorturl/model/dto/ShortUrlStatsDto.java

```java
package com.example.share.domain.shorturl.model.dto;

import com.example.share.domain.shorturl.model.entity.ShortUrl;
import io.swagger.v3.oas.annotations.media.Schema;
import lombok.Getter;

import java.time.LocalDateTime;

@Schema(description = "단축 URL 통계 DTO")
@Getter
public class ShortUrlStatsDto {

    @Schema(description = "단축 코드", example = "abc12345")
    private final String shortCode;

    @Schema(description = "클릭 수")
    private final Long clickCount;

    @Schema(description = "생성 일시")
    private final LocalDateTime createdAt;

    @Schema(description = "원본 URL")
    private final String originalUrl;

    public ShortUrlStatsDto(ShortUrl shortUrl) {
        this.shortCode = shortUrl.getShortCode();
        this.clickCount = shortUrl.getClickCount();
        this.createdAt = shortUrl.getCreatedAt();
        this.originalUrl = shortUrl.getOriginalUrl();
    }
}
```

단축 URL의 클릭 통계를 조회할 때 반환하는 DTO임
- ShortUrl 엔티티에서 통계에 필요한 필드만 추출하여 반환함
- clickCount를 통해 해당 단축 URL이 몇 번 클릭되었는지 확인할 수 있음


## 13. Base62Encoder 단축 코드 생성기

> 커밋: `feat(util): Base62Encoder 단축 코드 생성기 추가`

src/main/java/com/example/share/global/util/Base62Encoder.java

```java
package com.example.share.global.util;

import org.springframework.stereotype.Component;

import java.security.SecureRandom;

@Component
public class Base62Encoder {

    private static final String BASE62_CHARS = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
    private static final int CODE_LENGTH = 8;
    private static final SecureRandom RANDOM = new SecureRandom();

    /**
     * 8자리 랜덤 Base62 코드를 생성함
     * Base62는 0-9, a-z, A-Z 총 62개의 문자를 사용함
     * 8자리로 62^8 = 약 218조 개의 고유 코드 생성 가능
     */
    public String generateRandomCode() {
        StringBuilder sb = new StringBuilder(CODE_LENGTH);
        for (int i = 0; i < CODE_LENGTH; i++) {
            int index = RANDOM.nextInt(BASE62_CHARS.length());
            sb.append(BASE62_CHARS.charAt(index));
        }
        return sb.toString();
    }

    /**
     * 숫자를 Base62 문자열로 인코딩함
     * ID 기반 단축 URL 생성 시 사용 가능
     */
    public String encode(long number) {
        if (number == 0) {
            return String.valueOf(BASE62_CHARS.charAt(0));
        }

        StringBuilder sb = new StringBuilder();
        while (number > 0) {
            sb.append(BASE62_CHARS.charAt((int) (number % 62)));
            number /= 62;
        }
        return sb.reverse().toString();
    }

    /**
     * Base62 문자열을 숫자로 디코딩함
     */
    public long decode(String base62) {
        long result = 0;
        for (char c : base62.toCharArray()) {
            result = result * 62 + BASE62_CHARS.indexOf(c);
        }
        return result;
    }
}
```

단축 URL에 사용할 고유 코드를 생성하는 유틸리티 클래스임

주요 기능:
- generateRandomCode(): SecureRandom을 사용하여 8자리 랜덤 Base62 코드를 생성함. 62^8 = 약 218조 개의 고유 코드 생성이 가능하여 충돌 확률이 매우 낮음
- encode(): 숫자를 Base62 문자열로 변환함. ID 기반 단축 URL 생성 시 활용 가능함
- decode(): Base62 문자열을 숫자로 역변환함

SecureRandom을 사용하여 예측 불가능한 난수를 생성함
일반 Random보다 보안성이 높아 URL 추측 공격을 방지할 수 있음


## 14. ShareChannel 공유 채널 인터페이스

> 커밋: `feat(service): ShareChannel 공유 채널 인터페이스 추가`

src/main/java/com/example/share/domain/shorturl/service/ShareChannel.java

```java
package com.example.share.domain.shorturl.service;

import com.example.share.domain.shorturl.model.dto.ShareRequestDto;
import com.example.share.domain.shorturl.model.dto.ShareResponseDto;
import com.example.share.domain.shorturl.model.enums.ShareChannelType;

/**
 * Strategy 패턴의 전략 인터페이스
 * 각 공유 채널(링크, 카카오, 트위터 등)은 이 인터페이스를 구현함
 * 새로운 공유 채널 추가 시 이 인터페이스만 구현하면 됨
 */
public interface ShareChannel {

    /**
     * 이 채널이 지원하는 공유 타입을 반환함
     */
    ShareChannelType getType();

    /**
     * 주어진 타입을 이 채널이 지원하는지 확인함
     */
    default boolean supports(ShareChannelType type) {
        return getType() == type;
    }

    /**
     * 공유 작업을 수행하고 결과를 반환함
     */
    ShareResponseDto share(ShareRequestDto request);
}
```

Strategy 패턴의 전략 인터페이스임
- getType(): 해당 채널이 지원하는 ShareChannelType을 반환함
- supports(): default 메서드로 주어진 타입을 지원하는지 확인함. 별도 구현 없이도 기본 동작을 제공함
- share(): 실제 공유 작업을 수행하고 결과를 반환하는 핵심 메서드임

새로운 공유 채널(카카오톡, 트위터 등)을 추가할 때 이 인터페이스를 구현하면 됨
기존 코드를 수정하지 않고 기능을 확장할 수 있어 개방-폐쇄 원칙(OCP)을 준수함


## 15. LinkShareChannel 링크 공유 채널 구현체

> 커밋: `feat(service): LinkShareChannel 링크 공유 채널 구현체 추가`

src/main/java/com/example/share/domain/shorturl/service/LinkShareChannel.java

```java
package com.example.share.domain.shorturl.service;

import com.example.share.domain.shorturl.model.dto.ShareRequestDto;
import com.example.share.domain.shorturl.model.dto.ShareResponseDto;
import com.example.share.domain.shorturl.model.entity.ShortUrl;
import com.example.share.domain.shorturl.model.enums.ShareChannelType;
import com.example.share.domain.shorturl.repository.ShortUrlRepository;
import com.example.share.global.util.Base62Encoder;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

import java.util.Optional;

/**
 * Strategy 패턴의 구체적인 전략 구현체
 * 링크 공유 채널 - 단축 URL을 생성하여 반환함
 */
@Component
@RequiredArgsConstructor
public class LinkShareChannel implements ShareChannel {

    private final ShortUrlRepository shortUrlRepository;
    private final Base62Encoder base62Encoder;

    @Value("${app.base-url}")
    private String baseUrl;

    private static final int MAX_RETRY = 10;

    @Override
    public ShareChannelType getType() {
        return ShareChannelType.LINK;
    }

    @Override
    @Transactional
    public ShareResponseDto share(ShareRequestDto request) {
        String originalUrl = request.getOriginalUrl();

        // 이미 동일한 URL로 생성된 단축 URL이 있는지 확인
        Optional<ShortUrl> existingShortUrl = shortUrlRepository.findByOriginalUrl(originalUrl);
        if (existingShortUrl.isPresent()) {
            String shortUrl = baseUrl + "/s/" + existingShortUrl.get().getShortCode();
            return ShareResponseDto.success(
                    ShareChannelType.LINK,
                    "기존 단축 URL을 반환합니다",
                    shortUrl
            );
        }

        // 새로운 단축 코드 생성 (충돌 시 재시도)
        String shortCode = generateUniqueShortCode();
        if (shortCode == null) {
            return ShareResponseDto.fail(
                    ShareChannelType.LINK,
                    "단축 URL 생성에 실패했습니다. 잠시 후 다시 시도해주세요"
            );
        }

        // 단축 URL 저장
        ShortUrl shortUrl = new ShortUrl(shortCode, originalUrl);
        shortUrlRepository.save(shortUrl);

        String generatedUrl = baseUrl + "/s/" + shortCode;
        return ShareResponseDto.success(
                ShareChannelType.LINK,
                "단축 URL이 생성되었습니다",
                generatedUrl
        );
    }

    /**
     * 고유한 단축 코드를 생성함
     * 충돌 발생 시 최대 MAX_RETRY 횟수만큼 재시도함
     */
    private String generateUniqueShortCode() {
        for (int i = 0; i < MAX_RETRY; i++) {
            String shortCode = base62Encoder.generateRandomCode();
            if (!shortUrlRepository.existsByShortCode(shortCode)) {
                return shortCode;
            }
        }
        return null;
    }
}
```

Strategy 패턴의 구체적인 전략 구현체로, 링크 복사 채널을 담당함

주요 로직:
1. 동일한 원본 URL로 이미 생성된 단축 URL이 있으면 기존 것을 반환함 (중복 방지)
2. 새로운 단축 코드를 Base62Encoder로 생성함
3. 코드 충돌 시 최대 10회까지 재시도함
4. 생성된 단축 URL을 데이터베이스에 저장하고 결과를 반환함

@Transactional 어노테이션으로 share() 메서드의 데이터베이스 작업을 하나의 트랜잭션으로 관리함
@Value("${app.base-url}")로 application.yml에서 기본 도메인을 주입받음


## 16. ShareService 공유 서비스

> 커밋: `feat(service): ShareService 공유 서비스 추가`

src/main/java/com/example/share/domain/shorturl/service/ShareService.java

```java
package com.example.share.domain.shorturl.service;

import com.example.share.domain.shorturl.model.dto.*;
import com.example.share.domain.shorturl.model.entity.ShortUrl;
import com.example.share.domain.shorturl.model.enums.ShareChannelType;
import com.example.share.domain.shorturl.repository.ShortUrlRepository;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.Map;
import java.util.function.Function;
import java.util.stream.Collectors;

/**
 * Strategy 패턴의 Context 역할
 * 다양한 공유 채널 전략을 관리하고 적절한 전략에 위임함
 */
@Service
@Transactional(readOnly = true)
public class ShareService {

    private final Map<ShareChannelType, ShareChannel> channels;
    private final ShortUrlRepository shortUrlRepository;
    private final String baseUrl;

    /**
     * 생성자에서 모든 ShareChannel 구현체를 Map으로 변환하여 저장
     * Spring이 ShareChannel 인터페이스의 모든 구현체를 자동으로 주입함
     */
    public ShareService(List<ShareChannel> channelList,
                        ShortUrlRepository shortUrlRepository,
                        @Value("${app.base-url}") String baseUrl) {
        this.channels = channelList.stream()
                .collect(Collectors.toMap(ShareChannel::getType, Function.identity()));
        this.shortUrlRepository = shortUrlRepository;
        this.baseUrl = baseUrl;
    }

    /**
     * 요청된 채널 타입에 맞는 전략을 선택하여 공유 작업 수행
     */
    @Transactional
    public ShareResponseDto share(ShareRequestDto request) {
        ShareChannel channel = getChannel(request.getChannelType());
        return channel.share(request);
    }

    /**
     * 단축 코드로 URL 정보 조회
     */
    public ShortUrlInfoDto getShortUrlInfo(String shortCode) {
        ShortUrl shortUrl = shortUrlRepository.findByShortCode(shortCode)
                .orElseThrow(() -> new IllegalArgumentException("존재하지 않는 단축 URL입니다: " + shortCode));
        return new ShortUrlInfoDto(shortUrl, baseUrl);
    }

    /**
     * 단축 코드로 통계 정보 조회
     */
    public ShortUrlStatsDto getShortUrlStats(String shortCode) {
        ShortUrl shortUrl = shortUrlRepository.findByShortCode(shortCode)
                .orElseThrow(() -> new IllegalArgumentException("존재하지 않는 단축 URL입니다: " + shortCode));
        return new ShortUrlStatsDto(shortUrl);
    }

    /**
     * 단축 URL 클릭 시 원본 URL 반환 및 클릭 카운트 증가
     */
    @Transactional
    public String redirectAndCount(String shortCode) {
        ShortUrl shortUrl = shortUrlRepository.findByShortCode(shortCode)
                .orElseThrow(() -> new IllegalArgumentException("존재하지 않는 단축 URL입니다: " + shortCode));
        shortUrl.incrementClickCount();
        return shortUrl.getOriginalUrl();
    }

    /**
     * 지원되는 채널 타입 목록 반환
     */
    public List<ShareChannelType> getSupportedChannels() {
        return List.copyOf(channels.keySet());
    }

    /**
     * 채널 타입으로 해당 채널 전략 조회
     */
    private ShareChannel getChannel(ShareChannelType type) {
        ShareChannel channel = channels.get(type);
        if (channel == null) {
            throw new IllegalArgumentException("지원하지 않는 공유 채널입니다: " + type);
        }
        return channel;
    }
}
```

Strategy 패턴에서 Context 역할을 담당하는 서비스 클래스임

주요 설계:
- 생성자에서 Spring이 자동 주입한 모든 ShareChannel 구현체를 `Map<ShareChannelType, ShareChannel>`로 변환하여 저장함
- share() 메서드에서 요청된 채널 타입에 맞는 전략을 선택하여 위임함
- 새로운 공유 채널이 추가되어도 이 클래스의 코드를 수정할 필요가 없음

@Transactional(readOnly = true)를 클래스 레벨에 선언하여 기본적으로 읽기 전용 트랜잭션을 적용함
데이터를 변경하는 share(), redirectAndCount() 메서드에는 @Transactional을 별도로 선언하여 쓰기 트랜잭션을 사용함

주요 기능:
- share(): 채널 타입에 맞는 전략을 선택하여 공유 작업을 수행함
- getShortUrlInfo(): 단축 코드로 URL 정보를 조회함
- getShortUrlStats(): 단축 코드로 클릭 통계를 조회함
- redirectAndCount(): 단축 URL 클릭 시 원본 URL을 반환하고 클릭 카운트를 증가시킴
- getSupportedChannels(): 현재 지원 가능한 공유 채널 목록을 반환함


## 17. ShareRestController 공유 API 컨트롤러

> 커밋: `feat(controller): ShareRestController 공유 API 컨트롤러 추가`

src/main/java/com/example/share/domain/shorturl/controller/ShareRestController.java

```java
package com.example.share.domain.shorturl.controller;

import com.example.share.domain.shorturl.model.dto.*;
import com.example.share.domain.shorturl.model.enums.ShareChannelType;
import com.example.share.domain.shorturl.service.ShareService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@Tag(name = "공유 API", description = "단축 URL 생성 및 조회 API")
@RestController
@RequestMapping("/api/share")
@RequiredArgsConstructor
public class ShareRestController {

    private final ShareService shareService;

    @Operation(summary = "단축 URL 생성", description = "원본 URL을 받아 단축 URL을 생성합니다")
    @PostMapping
    public ResponseEntity<ShareResponseDto> createShortUrl(
            @Valid @RequestBody ShareRequestDto request) {
        ShareResponseDto response = shareService.share(request);
        return ResponseEntity.ok(response);
    }

    @Operation(summary = "단축 URL 정보 조회", description = "단축 코드로 URL 정보를 조회합니다")
    @GetMapping("/{shortCode}")
    public ResponseEntity<ShortUrlInfoDto> getShortUrlInfo(
            @Parameter(description = "단축 코드", example = "abc12345")
            @PathVariable String shortCode) {
        ShortUrlInfoDto info = shareService.getShortUrlInfo(shortCode);
        return ResponseEntity.ok(info);
    }

    @Operation(summary = "클릭 통계 조회", description = "단축 URL의 클릭 통계를 조회합니다")
    @GetMapping("/{shortCode}/stats")
    public ResponseEntity<ShortUrlStatsDto> getShortUrlStats(
            @Parameter(description = "단축 코드", example = "abc12345")
            @PathVariable String shortCode) {
        ShortUrlStatsDto stats = shareService.getShortUrlStats(shortCode);
        return ResponseEntity.ok(stats);
    }

    @Operation(summary = "지원 채널 목록", description = "지원하는 공유 채널 목록을 조회합니다")
    @GetMapping("/channels")
    public ResponseEntity<List<ShareChannelType>> getSupportedChannels() {
        List<ShareChannelType> channels = shareService.getSupportedChannels();
        return ResponseEntity.ok(channels);
    }
}
```

API 엔드포인트:
- POST /api/share: 단축 URL 생성
- GET /api/share/{shortCode}: 단축 URL 정보 조회
- GET /api/share/{shortCode}/stats: 클릭 통계 조회
- GET /api/share/channels: 지원 채널 목록 조회

어노테이션 설명:
- @RestController: @Controller + @ResponseBody. JSON 응답을 반환함
- @RequestMapping("/api/share"): 공통 URL 접두사
- @RequiredArgsConstructor: final 필드에 대한 생성자를 Lombok이 자동 생성함
- @Valid: 요청 DTO의 Bean Validation 어노테이션을 검사함
- @Tag, @Operation, @Parameter: Swagger 문서에 API 설명을 추가함
- @PathVariable: URL 경로의 변수를 메서드 파라미터에 바인딩함


## 18. RedirectController 리다이렉트 컨트롤러

> 커밋: `feat(controller): RedirectController 리다이렉트 컨트롤러 추가`

src/main/java/com/example/share/domain/shorturl/controller/RedirectController.java

```java
package com.example.share.domain.shorturl.controller;

import com.example.share.domain.shorturl.service.ShareService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.tags.Tag;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.servlet.view.RedirectView;

@Tag(name = "리다이렉트", description = "단축 URL 리다이렉트")
@Controller
@RequiredArgsConstructor
public class RedirectController {

    private final ShareService shareService;

    @Operation(summary = "원본 URL로 리다이렉트", description = "단축 URL 접속 시 원본 URL로 리다이렉트하고 클릭 수를 증가시킵니다")
    @GetMapping("/s/{shortCode}")
    public RedirectView redirect(
            @Parameter(description = "단축 코드", example = "abc12345")
            @PathVariable String shortCode) {
        String originalUrl = shareService.redirectAndCount(shortCode);
        RedirectView redirectView = new RedirectView(originalUrl);
        redirectView.setStatusCode(HttpStatus.MOVED_PERMANENTLY);
        return redirectView;
    }
}
```

단축 URL로 접속했을 때 원본 URL로 리다이렉트하는 컨트롤러임

주요 특징:
- @Controller를 사용함. @RestController가 아닌 이유는 JSON 응답이 아니라 리다이렉트 응답을 반환해야 하기 때문임
- /s/{shortCode} 경로로 접속하면 원본 URL로 301 리다이렉트됨
- RedirectView를 사용하여 리다이렉트 응답을 생성함
- HttpStatus.MOVED_PERMANENTLY (301)로 설정하여 브라우저가 리다이렉트 결과를 캐싱할 수 있게 함
- shareService.redirectAndCount()를 호출하여 리다이렉트와 동시에 클릭 카운트를 증가시킴


## 19. SwaggerConfig API 문서화 설정

> 커밋: `feat(config): SwaggerConfig API 문서화 설정 추가`

src/main/java/com/example/share/global/config/SwaggerConfig.java

```java
package com.example.share.global.config;

import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Info;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SwaggerConfig {

    @Bean
    public OpenAPI openAPI() {
        return new OpenAPI()
                .info(new Info()
                        .title("단축 URL 공유 서비스 API")
                        .description("Strategy 패턴을 적용한 단축 URL 생성 및 공유 서비스")
                        .version("1.0.0"));
    }
}
```

OpenAPI 빈을 등록하면 /swagger-ui.html에서 API 문서를 확인할 수 있음
프론트엔드 개발자나 다른 팀원이 API 스펙을 쉽게 파악하고 테스트할 수 있음

@Configuration 어노테이션으로 스프링 설정 클래스임을 선언함
@Bean 메서드로 OpenAPI 객체를 빈으로 등록하여 Swagger UI에 API 정보를 표시함


## 20. 애플리케이션 실행 및 테스트

### 20-1. .env 파일 생성

프로젝트 루트에 .env 파일을 생성하고 환경변수를 입력함

```bash
cp .env.sample .env
```

.env 파일을 열고 필요한 경우 값을 수정함:
```
BASE_URL=http://localhost:8080
SERVER_PORT=8080
```

### 20-2. 애플리케이션 실행

터미널에서 다음 명령어로 실행함

```bash
./gradlew bootRun
```

build.gradle에 설정한 bootRun 태스크가 .env 파일을 읽어서 환경변수로 등록함
인텔리제이의 Run 버튼 대신 터미널에서 실행해야 .env 파일이 적용됨

콘솔에 다음과 같은 로그가 출력되면 성공임

```
Started ShareUrlApplication in X.XXX seconds
```

### 20-3. Swagger UI 접속

브라우저에서 http://localhost:8080/swagger-ui.html 접속
공유 API와 리다이렉트 API 목록이 표시됨

### 20-4. 단축 URL 생성 테스트

1. POST /api/share 엔드포인트 선택
2. Try it out 클릭
3. Request body에 다음 입력

```json
{
  "originalUrl": "https://www.example.com/very/long/url/path/that/needs/shortening",
  "channelType": "LINK"
}
```

4. Execute 클릭
5. 성공 시 다음과 같은 응답이 반환됨

```json
{
  "channelType": "LINK",
  "success": true,
  "message": "단축 URL이 생성되었습니다",
  "shortUrl": "http://localhost:8080/s/abc12345"
}
```

### 20-5. 단축 URL 정보 조회 테스트

1. GET /api/share/{shortCode} 엔드포인트 선택
2. Try it out 클릭
3. shortCode에 생성된 단축 코드 입력 (예: abc12345)
4. Execute 클릭
5. 성공 시 다음과 같은 응답이 반환됨

```json
{
  "shortCode": "abc12345",
  "originalUrl": "https://www.example.com/very/long/url/path/that/needs/shortening",
  "shortUrl": "http://localhost:8080/s/abc12345",
  "createdAt": "2026-03-05T12:00:00",
  "clickCount": 0
}
```

### 20-6. 리다이렉트 테스트

브라우저에서 생성된 단축 URL(예: http://localhost:8080/s/abc12345)에 접속하면 원본 URL로 리다이렉트됨
리다이렉트될 때마다 클릭 카운트가 1씩 증가함

### 20-7. 클릭 통계 조회 테스트

1. GET /api/share/{shortCode}/stats 엔드포인트 선택
2. Try it out 클릭
3. shortCode에 단축 코드 입력
4. Execute 클릭
5. 성공 시 다음과 같은 응답이 반환됨

```json
{
  "shortCode": "abc12345",
  "clickCount": 3,
  "createdAt": "2026-03-05T12:00:00",
  "originalUrl": "https://www.example.com/very/long/url/path/that/needs/shortening"
}
```

### 20-8. Postman으로 테스트

Postman을 사용해서 테스트할 수도 있음

단축 URL 생성:
- Method: POST
- URL: http://localhost:8080/api/share
- Headers: Content-Type: application/json
- Body (raw JSON):
```json
{
  "originalUrl": "https://www.example.com/very/long/url/path",
  "channelType": "LINK"
}
```

단축 URL 정보 조회:
- Method: GET
- URL: http://localhost:8080/api/share/{shortCode}

클릭 통계 조회:
- Method: GET
- URL: http://localhost:8080/api/share/{shortCode}/stats

지원 채널 목록:
- Method: GET
- URL: http://localhost:8080/api/share/channels

### 20-9. H2 콘솔 접속

브라우저에서 http://localhost:8080/h2-console 접속
JDBC URL에 jdbc:h2:mem:testdb를 입력하고 Connect를 클릭하면 데이터베이스를 직접 조회할 수 있음


## 21. 커밋 히스토리

이 프로젝트의 커밋 로그를 순서대로 따라가면 스프링부트 프로젝트 구성 흐름을 이해할 수 있음

```
feat(init): 스프링부트 프로젝트 초기설정
chore: .gitignore 추가
chore: .env.sample 환경변수 샘플파일 추가
build: build.gradle 환경변수 로드 스크립트 추가
config: application.yml 설정파일 추가
feat(domain): ShareChannelType 공유 채널 열거형 추가
feat(domain): ShortUrl 단축 URL 엔티티 추가
feat(domain): ShortUrlRepository 단축 URL 저장소 추가
feat(dto): ShareRequestDto 공유 요청 DTO 추가
feat(dto): ShareResponseDto 공유 응답 DTO 추가
feat(dto): ShortUrlInfoDto 단축 URL 정보 DTO 추가
feat(dto): ShortUrlStatsDto 단축 URL 통계 DTO 추가
feat(util): Base62Encoder 단축 코드 생성기 추가
feat(service): ShareChannel 공유 채널 인터페이스 추가
feat(service): LinkShareChannel 링크 공유 채널 구현체 추가
feat(service): ShareService 공유 서비스 추가
feat(controller): ShareRestController 공유 API 컨트롤러 추가
feat(controller): RedirectController 리다이렉트 컨트롤러 추가
feat(config): SwaggerConfig API 문서화 설정 추가
```

각 커밋 메시지에 해당 파일의 역할과 코드 설명이 포함되어 있음

첫 번째 커밋(feat(init))에 build.gradle, Gradle Wrapper, settings.gradle, 메인 클래스가 모두 포함됨
이후 .gitignore, .env.sample, 환경변수 스크립트 등이 순차적으로 추가됨


## 22. 운영 환경 고려사항

실제 서비스에 적용할 때 추가로 고려해야 할 사항:

- H2 대신 MySQL, PostgreSQL 등 운영용 데이터베이스로 변경 (데이터 영속성 보장)
- 단축 코드 생성 시 분산 환경에서의 동시성 처리 (Redis 기반 분산 락 활용)
- 클릭 카운트 증가 시 비동기 처리 (리다이렉트 응답 속도 향상)
- 에러 핸들링 강화 (@RestControllerAdvice 활용)
- 만료 기능 추가 (일정 기간 후 단축 URL 자동 비활성화)
- 악성 URL 필터링 (피싱, 멀웨어 URL 차단)
- 접근 로그 및 통계 모니터링 (접속 IP, User-Agent 등 수집)
- 카카오톡, 트위터 등 추가 공유 채널 구현체 개발


## 23. 참고 자료

- Spring Boot 공식 문서: https://docs.spring.io/spring-boot
- Spring Data JPA 공식 문서: https://docs.spring.io/spring-data/jpa
- Springdoc OpenAPI: https://springdoc.org
- H2 Database: https://h2database.com