# 인텔리제이로 스프링부트 파일 업로드/다운로드 REST API 구현하기

Spring Boot와 JPA를 사용하여 이미지 파일 업로드/다운로드 기능을 구현하는 방법을 정리함

GitHub 저장소: https://github.com/tletle7102/spring-file-upload-project

커밋 로그를 순서대로 따라가면서 학습하면 프로젝트 구성 흐름을 이해할 수 있음


## 목차

1. 프로젝트 초기 설정
2. .gitignore 설정
3. .env.sample 환경변수 샘플
4. build.gradle 설정
5. application.yml 설정
6. JPA Auditing 설정 클래스
7. FileEntity 엔티티
8. FileRepository
9. FileResponseDTO
10. FileService 구현
11. FileRestController 구현
12. 애플리케이션 실행 및 테스트


## 1. 프로젝트 초기 설정

> 커밋: `feat(init): Spring Boot 프로젝트 초기 설정`

### 1-1. 인텔리제이에서 새 프로젝트 생성

1. File > New > Project 선택
2. 좌측에서 "Spring Boot" 선택
3. 다음 설정 입력
   - Name: spring-file-upload-project
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

src/main/java/com/example/fileupload/SpringFileUploadProjectApplication.java

```java
package com.example.fileupload;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringFileUploadProjectApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringFileUploadProjectApplication.class, args);
    }

}
```

@SpringBootApplication 어노테이션은 @Configuration, @EnableAutoConfiguration, @ComponentScan을 포함하며 스프링부트 자동 설정을 활성화함

### 1-4. 초기 build.gradle

인텔리제이가 생성한 초기 build.gradle에 springdoc-openapi 의존성을 수동으로 추가함
springdoc-openapi는 Spring Initializr에서 제공하지 않으므로 직접 build.gradle에 추가해야 함

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
    implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:2.8.8' // 수동 추가

    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    runtimeOnly 'com.h2database:h2'

    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

주요 의존성 설명:
- spring-boot-starter-web: 웹 애플리케이션 개발에 필요한 의존성을 포함함 (내장 톰캣, Spring MVC 등)
- spring-boot-starter-data-jpa: JPA를 사용한 데이터베이스 접근을 지원함
- springdoc-openapi-starter-webmvc-ui: Swagger UI로 API 문서를 자동 생성함
- h2: 인메모리 데이터베이스로 별도 DB 설치 없이 개발/테스트가 가능함
- lombok: 반복적인 보일러플레이트 코드(Getter, Setter, 생성자 등)를 자동 생성함


## 2. .gitignore 설정

> 커밋: `chore(git): .gitignore에 환경변수 파일 제외 설정 추가`

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

### macOS ###
.DS_Store

### Environment Variables ###
.env
*.env
```

마지막에 추가된 환경변수 관련 설정:
- .env: 환경변수 파일을 git 추적에서 제외함
- *.env: .env 확장자를 가진 모든 파일을 제외함

.gitignore 파일은 반드시 프로젝트 초기에 설정해야 함
실수로 .env 파일을 한번이라도 커밋하면 git 히스토리에 남아서 삭제하기 어려움


## 3. .env.sample 환경변수 샘플

> 커밋: `chore(env): 환경변수 샘플 파일 추가`

프로젝트 루트에 .env.sample 파일을 생성함
이 파일은 다른 개발자나 프로젝트 사용자에게 어떤 환경변수가 필요한지 안내하는 역할을 함

```
# 환경변수 샘플 파일
# 이 파일을 복사하여 .env 파일로 만들고 실제 값을 입력하세요
# cp .env.sample .env

# 현재 이 프로젝트는 환경변수가 필요하지 않지만
# 향후 확장 시 아래와 같은 형식으로 환경변수를 추가할 수 있습니다

# 예시: 데이터베이스 설정
# DB_URL=jdbc:h2:mem:filedb
# DB_USERNAME=sa
# DB_PASSWORD=

# 예시: 파일 업로드 설정
# FILE_UPLOAD_DIR=/path/to/upload/directory
# FILE_MAX_SIZE=10MB
```

.env.sample 파일의 역할:
- 필요한 환경변수 목록을 문서화함
- 환경변수 값의 형식을 예시로 보여줌
- 실제 값이 아닌 플레이스홀더를 사용하므로 GitHub에 올려도 안전함

현재 이 프로젝트는 H2 인메모리 데이터베이스를 사용하고 별도 외부 서비스 연동이 없어서 환경변수가 필수는 아님
향후 외부 스토리지(AWS S3 등)나 외부 DB를 연동할 때 환경변수를 추가할 수 있음

### 3-1. .env 파일 생성

프로젝트를 클론한 후 .env.sample을 복사하여 .env 파일을 생성함

```bash
cp .env.sample .env
```

.env 파일은 .gitignore에 등록되어 있어서 git에 올라가지 않음
따라서 실제 값을 입력해도 GitHub에 노출되지 않음


## 4. build.gradle 설정

> 커밋: `feat(gradle): .env 파일 로드 스크립트 추가`

초기 build.gradle에 .env 파일 로드 기능을 추가함
이 설정이 있어야 `./gradlew bootRun` 명령어로 실행할 때 .env 파일의 값을 사용할 수 있음

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

build.gradle 하단의 환경변수 로드 설정 설명:
- loadEnv() 함수: .env 파일을 읽어서 System.setProperty()로 시스템 프로퍼티에 등록
- loadEnv() 호출: 스크립트 실행 시점에 환경변수를 로드
- bootRun 블록: ./gradlew bootRun 실행 시 환경변수를 실행 환경에 주입

Gradle 코끼리 아이콘을 클릭하여 의존성을 다운로드함


## 5. application.yml 설정

> 커밋: `feat(config): application.yml 환경 설정 파일 추가`

src/main/resources/application.yml

```yaml
server:
  port: 8080

spring:
  application:
    name: file-upload

  datasource:
    url: jdbc:h2:mem:filedb
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

  servlet:
    multipart:
      enabled: true
      max-file-size: 10MB
      max-request-size: 10MB

file:
  upload-dir: ${user.home}/file-uploads

springdoc:
  swagger-ui:
    path: /swagger-ui.html
  api-docs:
    path: /api-docs
```

설정 항목 설명:
- server.port: 애플리케이션 실행 포트 (8080)
- spring.datasource: H2 인메모리 데이터베이스 연결 정보. jdbc:h2:mem:filedb는 메모리 기반 DB를 사용한다는 의미임
- spring.h2.console: H2 콘솔을 활성화하여 브라우저에서 DB를 직접 확인할 수 있음 (/h2-console)
- spring.jpa.hibernate.ddl-auto: create로 설정하면 애플리케이션 시작 시 테이블을 새로 생성함
- spring.jpa.show-sql: 실행되는 SQL 쿼리를 콘솔에 출력함
- spring.servlet.multipart: 멀티파트 파일 업로드를 활성화하고 최대 파일 크기를 10MB로 제한함
- file.upload-dir: 파일이 저장될 디렉토리 경로. ${user.home}은 사용자 홈 디렉토리를 의미함
- springdoc: Swagger UI와 API 문서 경로를 설정함


## 6. JPA Auditing 설정 클래스

> 커밋: `feat(config): JPA Auditing 활성화를 위한 설정 클래스 추가`

src/main/java/com/example/fileupload/config/JpaConfig.java

```java
package com.example.fileupload.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;

@Configuration
@EnableJpaAuditing
public class JpaConfig {
}
```

@EnableJpaAuditing 어노테이션을 설정 클래스에 선언하면 JPA Auditing 기능이 활성화됨
Auditing 기능을 사용하면 엔티티의 생성일시, 수정일시 등을 자동으로 관리할 수 있음
이 설정이 있어야 엔티티에서 @CreatedDate 어노테이션이 동작함

@Configuration을 메인 클래스에 직접 붙이지 않고 별도 설정 클래스로 분리하면 관심사를 분리할 수 있어서 유지보수에 유리함


## 7. FileEntity 엔티티

> 커밋: `feat(entity): 파일 메타데이터를 저장할 FileEntity 엔티티 추가`

src/main/java/com/example/fileupload/model/entity/FileEntity.java

```java
package com.example.fileupload.model.entity;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.NoArgsConstructor;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import java.time.LocalDateTime;

@Entity
@Table(name = "tb_file")
@Getter
@NoArgsConstructor
@EntityListeners(AuditingEntityListener.class)
public class FileEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String originalName;

    @Column(nullable = false, unique = true)
    private String storedName;

    @Column(nullable = false)
    private String filePath;

    @Column(nullable = false)
    private String contentType;

    @Column(nullable = false)
    private Long fileSize;

    @CreatedDate
    @Column(nullable = false, updatable = false)
    private LocalDateTime createdAt;

    public FileEntity(String originalName, String storedName, String filePath,
                      String contentType, Long fileSize) {
        this.originalName = originalName;
        this.storedName = storedName;
        this.filePath = filePath;
        this.contentType = contentType;
        this.fileSize = fileSize;
    }
}
```

파일 메타데이터를 데이터베이스에 저장하기 위한 JPA 엔티티임
실제 파일은 로컬 파일 시스템에 저장하고, 파일의 메타정보만 DB에 저장하는 구조임

어노테이션 설명:
- @Entity: JPA 엔티티임을 선언함
- @Table(name = "tb_file"): 매핑될 테이블 이름을 지정함
- @Getter: Lombok이 모든 필드의 getter 메서드를 자동 생성함
- @NoArgsConstructor: JPA가 엔티티를 생성할 때 필요한 기본 생성자를 Lombok이 자동 생성함
- @EntityListeners(AuditingEntityListener.class): JPA Auditing 이벤트 리스너를 등록함
- @Id, @GeneratedValue: 기본키를 자동 증가 전략(IDENTITY)으로 생성함
- @Column(nullable = false): NOT NULL 제약조건을 설정함
- @Column(unique = true): UNIQUE 제약조건을 설정함. 저장된 파일명이 중복되지 않도록 함
- @CreatedDate: JPA Auditing에 의해 엔티티 생성 시점의 시간이 자동으로 입력됨
- @Column(updatable = false): 생성 이후 수정이 불가능하도록 설정함

필드 설명:
- originalName: 사용자가 업로드한 원본 파일명
- storedName: 서버에 저장된 파일명 (UUID 기반으로 생성됨)
- filePath: 파일이 저장된 전체 경로
- contentType: 파일의 MIME 타입 (image/jpeg, image/png 등)
- fileSize: 파일 크기 (바이트 단위)
- createdAt: 파일 업로드 시각


## 8. FileRepository

> 커밋: `feat(repository): 파일 데이터 접근을 위한 FileRepository 추가`

src/main/java/com/example/fileupload/repository/FileRepository.java

```java
package com.example.fileupload.repository;

import com.example.fileupload.model.entity.FileEntity;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface FileRepository extends JpaRepository<FileEntity, Long> {
}
```

JpaRepository를 상속하면 기본적인 CRUD 메서드가 자동으로 제공됨
- save(): 엔티티 저장 또는 수정
- findById(): ID로 엔티티 조회
- findAll(): 전체 엔티티 목록 조회
- delete(): 엔티티 삭제

@Repository 어노테이션은 스프링이 이 인터페이스를 Repository 빈으로 등록하고 데이터 접근 예외를 스프링 예외로 변환함
JpaRepository가 이미 @Repository를 포함하고 있어서 생략해도 되지만 명시적으로 선언하면 역할이 분명해짐


## 9. FileResponseDTO

> 커밋: `feat(dto): API 응답용 FileResponseDTO 추가`

src/main/java/com/example/fileupload/model/dto/FileResponseDTO.java

```java
package com.example.fileupload.model.dto;

import com.example.fileupload.model.entity.FileEntity;
import io.swagger.v3.oas.annotations.media.Schema;
import lombok.Getter;

import java.time.LocalDateTime;

@Schema(description = "파일 응답 정보")
@Getter
public class FileResponseDTO {

    @Schema(description = "파일 ID", example = "1")
    private Long id;

    @Schema(description = "원본 파일명", example = "profile.jpg")
    private String originalName;

    @Schema(description = "저장된 파일명", example = "550e8400-e29b-41d4-a716-446655440000.jpg")
    private String storedName;

    @Schema(description = "MIME 타입", example = "image/jpeg")
    private String contentType;

    @Schema(description = "파일 크기 (bytes)", example = "102400")
    private Long fileSize;

    @Schema(description = "다운로드 URL", example = "/api/files/1")
    private String downloadUrl;

    @Schema(description = "생성일시")
    private LocalDateTime createdAt;

    public FileResponseDTO(FileEntity entity) {
        this.id = entity.getId();
        this.originalName = entity.getOriginalName();
        this.storedName = entity.getStoredName();
        this.contentType = entity.getContentType();
        this.fileSize = entity.getFileSize();
        this.downloadUrl = "/api/files/" + entity.getId();
        this.createdAt = entity.getCreatedAt();
    }
}
```

클라이언트에게 파일 정보를 반환할 때 사용하는 응답 DTO임
엔티티를 직접 반환하지 않고 DTO로 변환하여 반환하는 것이 좋은 설계 패턴임

- 엔티티의 내부 구조가 API 응답에 직접 노출되지 않아서 변경에 유연함
- filePath 같은 서버 내부 경로는 클라이언트에 노출하지 않고, 대신 downloadUrl을 제공함
- @Schema 어노테이션으로 Swagger 문서에 각 필드의 설명과 예시를 표시함

생성자에서 FileEntity를 받아서 필요한 필드만 매핑함
downloadUrl은 "/api/files/" + ID 형태로 생성하여 클라이언트가 다운로드 경로를 알 수 있도록 함


## 10. FileService 구현

> 커밋: `feat(service): 파일 업로드/다운로드 비즈니스 로직을 담당하는 FileService 추가`

src/main/java/com/example/fileupload/service/FileService.java

```java
package com.example.fileupload.service;

import com.example.fileupload.model.dto.FileResponseDTO;
import com.example.fileupload.model.entity.FileEntity;
import com.example.fileupload.repository.FileRepository;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.io.Resource;
import org.springframework.core.io.UrlResource;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;
import java.net.MalformedURLException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardCopyOption;
import java.util.Arrays;
import java.util.List;
import java.util.UUID;

@Service
@Transactional(readOnly = true)
public class FileService {

    private static final List<String> ALLOWED_EXTENSIONS = Arrays.asList("jpg", "jpeg", "png", "gif", "webp");
    private static final List<String> ALLOWED_MIME_TYPES = Arrays.asList(
            "image/jpeg", "image/png", "image/gif", "image/webp"
    );

    private final FileRepository fileRepository;
    private final Path uploadPath;

    public FileService(FileRepository fileRepository,
                       @Value("${file.upload-dir}") String uploadDir) {
        this.fileRepository = fileRepository;
        this.uploadPath = Paths.get(uploadDir).toAbsolutePath().normalize();
        createUploadDirectory();
    }

    private void createUploadDirectory() {
        try {
            Files.createDirectories(uploadPath);
        } catch (IOException e) {
            throw new RuntimeException("업로드 디렉토리를 생성할 수 없습니다: " + uploadPath, e);
        }
    }

    @Transactional
    public FileResponseDTO uploadFile(MultipartFile file) {
        validateFile(file);

        String originalName = file.getOriginalFilename();
        String extension = getExtension(originalName);
        String storedName = UUID.randomUUID().toString() + "." + extension;

        Path targetPath = uploadPath.resolve(storedName);
        try {
            Files.copy(file.getInputStream(), targetPath, StandardCopyOption.REPLACE_EXISTING);
        } catch (IOException e) {
            throw new RuntimeException("파일 저장 중 오류가 발생했습니다: " + e.getMessage(), e);
        }

        FileEntity fileEntity = new FileEntity(
                originalName,
                storedName,
                targetPath.toString(),
                file.getContentType(),
                file.getSize()
        );
        fileRepository.save(fileEntity);

        return new FileResponseDTO(fileEntity);
    }

    public Resource downloadFile(Long fileId) {
        FileEntity fileEntity = fileRepository.findById(fileId)
                .orElseThrow(() -> new RuntimeException("파일을 찾을 수 없습니다: id=" + fileId));

        try {
            Path filePath = Paths.get(fileEntity.getFilePath());
            Resource resource = new UrlResource(filePath.toUri());

            if (resource.exists() && resource.isReadable()) {
                return resource;
            } else {
                throw new RuntimeException("파일을 읽을 수 없습니다: " + fileEntity.getStoredName());
            }
        } catch (MalformedURLException e) {
            throw new RuntimeException("잘못된 파일 경로입니다: " + fileEntity.getFilePath(), e);
        }
    }

    public FileResponseDTO getFileInfo(Long fileId) {
        FileEntity fileEntity = fileRepository.findById(fileId)
                .orElseThrow(() -> new RuntimeException("파일을 찾을 수 없습니다: id=" + fileId));
        return new FileResponseDTO(fileEntity);
    }

    public List<FileResponseDTO> getAllFiles() {
        return fileRepository.findAll().stream()
                .map(FileResponseDTO::new)
                .toList();
    }

    @Transactional
    public void deleteFile(Long fileId) {
        FileEntity fileEntity = fileRepository.findById(fileId)
                .orElseThrow(() -> new RuntimeException("파일을 찾을 수 없습니다: id=" + fileId));

        try {
            Path filePath = Paths.get(fileEntity.getFilePath());
            Files.deleteIfExists(filePath);
        } catch (IOException e) {
            // 물리적 파일 삭제 실패는 로그만 남기고 진행
        }

        fileRepository.delete(fileEntity);
    }

    private void validateFile(MultipartFile file) {
        if (file.isEmpty()) {
            throw new IllegalArgumentException("파일이 비어있습니다");
        }

        String originalName = file.getOriginalFilename();
        String extension = getExtension(originalName).toLowerCase();
        String contentType = file.getContentType();

        if (!ALLOWED_EXTENSIONS.contains(extension)) {
            throw new IllegalArgumentException(
                    "허용되지 않은 파일 형식입니다. 허용된 형식: " + String.join(", ", ALLOWED_EXTENSIONS));
        }

        if (contentType == null || !ALLOWED_MIME_TYPES.contains(contentType)) {
            throw new IllegalArgumentException("허용되지 않은 파일 형식입니다. 이미지 파일만 업로드 가능합니다");
        }
    }

    private String getExtension(String filename) {
        if (filename == null || !filename.contains(".")) {
            return "";
        }
        return filename.substring(filename.lastIndexOf(".") + 1);
    }
}
```

파일 업로드/다운로드/삭제의 핵심 비즈니스 로직을 담당하는 서비스 클래스임

어노테이션 설명:
- @Service: 스프링이 서비스 빈으로 등록함
- @Transactional(readOnly = true): 클래스 레벨에 읽기 전용 트랜잭션을 기본으로 설정함. 조회 메서드의 성능이 최적화됨
- @Transactional: 쓰기 작업이 필요한 메서드에 개별적으로 설정하여 읽기/쓰기 트랜잭션을 구분함

주요 기능 설명:

**생성자**
- @Value("${file.upload-dir}")로 application.yml에서 업로드 디렉토리 경로를 주입받음
- Paths.get().toAbsolutePath().normalize()로 경로를 절대 경로로 변환하고 정규화함
- createUploadDirectory()로 디렉토리가 존재하지 않으면 생성함

**uploadFile()**
- validateFile()로 파일 형식을 검증함
- UUID.randomUUID()로 저장 파일명을 생성하여 파일명 충돌을 방지함
- Files.copy()로 파일을 디스크에 저장함
- FileEntity를 생성하여 DB에 메타데이터를 저장함

**downloadFile()**
- DB에서 파일 정보를 조회한 뒤 UrlResource로 파일을 읽어서 반환함
- 파일이 존재하지 않거나 읽을 수 없으면 예외를 발생시킴

**deleteFile()**
- DB에서 파일 정보를 조회한 뒤 물리적 파일을 삭제하고 DB 레코드도 삭제함
- 물리적 파일 삭제 실패는 무시하고 DB 레코드는 삭제하도록 처리함

**validateFile()**
- 파일이 비어있는지 확인함
- 파일 확장자가 허용된 목록(jpg, jpeg, png, gif, webp)에 있는지 확인함
- MIME 타입이 허용된 이미지 타입인지 확인함
- 확장자와 MIME 타입을 동시에 검증하여 확장자 위변조를 방지함


## 11. FileRestController 구현

> 커밋: `feat(controller): 파일 업로드/다운로드 REST API 컨트롤러 추가`

src/main/java/com/example/fileupload/controller/FileRestController.java

```java
package com.example.fileupload.controller;

import com.example.fileupload.model.dto.FileResponseDTO;
import com.example.fileupload.service.FileService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.tags.Tag;
import org.springframework.core.io.Resource;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.net.URLEncoder;
import java.nio.charset.StandardCharsets;
import java.util.List;

@Tag(name = "파일 API", description = "파일 업로드/다운로드 REST API")
@RestController
@RequestMapping("/api/files")
public class FileRestController {

    private final FileService fileService;

    public FileRestController(FileService fileService) {
        this.fileService = fileService;
    }

    @Operation(summary = "파일 업로드", description = "이미지 파일을 업로드합니다 (jpg, jpeg, png, gif, webp)")
    @PostMapping(consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public ResponseEntity<FileResponseDTO> uploadFile(
            @Parameter(description = "업로드할 이미지 파일")
            @RequestParam("file") MultipartFile file) {
        FileResponseDTO response = fileService.uploadFile(file);
        return ResponseEntity.ok(response);
    }

    @Operation(summary = "파일 목록 조회", description = "업로드된 모든 파일 목록을 조회합니다")
    @GetMapping
    public ResponseEntity<List<FileResponseDTO>> getAllFiles() {
        List<FileResponseDTO> files = fileService.getAllFiles();
        return ResponseEntity.ok(files);
    }

    @Operation(summary = "파일 다운로드", description = "파일을 다운로드합니다")
    @GetMapping("/{id}")
    public ResponseEntity<Resource> downloadFile(
            @Parameter(description = "파일 ID")
            @PathVariable Long id) {
        Resource resource = fileService.downloadFile(id);
        FileResponseDTO fileInfo = fileService.getFileInfo(id);

        String encodedFileName = URLEncoder.encode(fileInfo.getOriginalName(), StandardCharsets.UTF_8)
                .replaceAll("\\+", "%20");

        return ResponseEntity.ok()
                .contentType(MediaType.parseMediaType(fileInfo.getContentType()))
                .header(HttpHeaders.CONTENT_DISPOSITION,
                        "inline; filename*=UTF-8''" + encodedFileName)
                .body(resource);
    }

    @Operation(summary = "파일 삭제", description = "파일을 삭제합니다")
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteFile(
            @Parameter(description = "삭제할 파일 ID")
            @PathVariable Long id) {
        fileService.deleteFile(id);
        return ResponseEntity.noContent().build();
    }
}
```

파일 업로드/다운로드/목록 조회/삭제 기능을 제공하는 REST API 컨트롤러임

API 엔드포인트:
- POST /api/files: 이미지 파일 업로드
- GET /api/files: 업로드된 파일 목록 조회
- GET /api/files/{id}: 파일 다운로드
- DELETE /api/files/{id}: 파일 삭제

어노테이션 설명:
- @RestController: @Controller + @ResponseBody. JSON 응답을 반환함
- @RequestMapping("/api/files"): 공통 URL 접두사
- @Tag, @Operation, @Parameter: Swagger 문서에 API 설명을 추가함
- @PostMapping(consumes = MediaType.MULTIPART_FORM_DATA_VALUE): multipart/form-data 형식의 요청만 수신함
- @RequestParam("file"): 요청 파라미터에서 "file" 이름의 멀티파트 파일을 추출함
- @PathVariable: URL 경로에서 변수를 추출함

주요 구현 설명:

**uploadFile()**
- MultipartFile 객체로 업로드된 파일을 받음
- FileService의 uploadFile()을 호출하여 파일을 저장하고 메타데이터를 반환함

**downloadFile()**
- Resource 객체로 파일을 반환하여 스트리밍 다운로드를 지원함
- Content-Disposition 헤더를 "inline"으로 설정하여 브라우저에서 바로 표시되도록 함
- 한글 파일명을 URLEncoder로 인코딩하고 RFC 5987 규격(filename*=UTF-8'')을 따라 설정함
- replaceAll("\\+", "%20")으로 공백 처리를 함 (URLEncoder는 공백을 +로 변환하지만, 파일명에서는 %20이 표준임)

**deleteFile()**
- 삭제 성공 시 204 No Content 응답을 반환함
- ResponseEntity.noContent().build()로 빈 응답 바디와 함께 204 상태 코드를 반환함


## 12. 애플리케이션 실행 및 테스트

### 12-1. 애플리케이션 실행

터미널에서 다음 명령어로 실행함

```bash
./gradlew bootRun
```

build.gradle에 설정한 bootRun 태스크가 .env 파일을 읽어서 환경변수로 등록함
인텔리제이의 Run 버튼 대신 터미널에서 실행해야 .env 파일이 적용됨

콘솔에 다음과 같은 로그가 출력되면 성공임

```
Started SpringFileUploadProjectApplication in X.XXX seconds
```

### 12-2. Swagger UI 접속

브라우저에서 http://localhost:8080/swagger-ui.html 접속
파일 API 목록이 표시됨

### 12-3. 파일 업로드 테스트

1. POST /api/files 엔드포인트 선택
2. Try it out 클릭
3. Choose File 버튼으로 이미지 파일(jpg, png 등)을 선택
4. Execute 클릭
5. 성공 시 다음과 같은 응답이 반환됨

```json
{
  "id": 1,
  "originalName": "profile.jpg",
  "storedName": "550e8400-e29b-41d4-a716-446655440000.jpg",
  "contentType": "image/jpeg",
  "fileSize": 102400,
  "downloadUrl": "/api/files/1",
  "createdAt": "2026-03-05T12:00:00"
}
```

### 12-4. 파일 목록 조회 테스트

1. GET /api/files 엔드포인트 선택
2. Try it out 클릭
3. Execute 클릭
4. 업로드된 모든 파일의 목록이 배열로 반환됨

```json
[
  {
    "id": 1,
    "originalName": "profile.jpg",
    "storedName": "550e8400-e29b-41d4-a716-446655440000.jpg",
    "contentType": "image/jpeg",
    "fileSize": 102400,
    "downloadUrl": "/api/files/1",
    "createdAt": "2026-03-05T12:00:00"
  }
]
```

### 12-5. 파일 다운로드 테스트

1. GET /api/files/{id} 엔드포인트 선택
2. Try it out 클릭
3. id에 1 입력
4. Execute 클릭
5. 이미지가 브라우저에 표시되거나 다운로드됨

또는 브라우저 주소창에 직접 http://localhost:8080/api/files/1 을 입력하면 이미지가 표시됨

### 12-6. 파일 삭제 테스트

1. DELETE /api/files/{id} 엔드포인트 선택
2. Try it out 클릭
3. id에 1 입력
4. Execute 클릭
5. 204 No Content 응답이 반환되면 삭제 성공임

### 12-7. H2 콘솔 접속

브라우저에서 http://localhost:8080/h2-console 접속
- JDBC URL: jdbc:h2:mem:filedb
- User Name: sa
- Password: (빈칸)
- Connect 클릭

tb_file 테이블에서 업로드된 파일의 메타데이터를 직접 조회할 수 있음

### 12-8. curl로 테스트

curl 명령어를 사용해서 테스트할 수도 있음

파일 업로드:
```bash
curl -X POST http://localhost:8080/api/files \
  -F "file=@/path/to/image.jpg"
```

파일 목록 조회:
```bash
curl http://localhost:8080/api/files
```

파일 다운로드:
```bash
curl -O http://localhost:8080/api/files/1
```

파일 삭제:
```bash
curl -X DELETE http://localhost:8080/api/files/1
```


## 커밋 히스토리

이 프로젝트의 커밋 로그를 순서대로 따라가면 스프링부트 프로젝트 구성 흐름을 이해할 수 있음

```
feat(init): Spring Boot 프로젝트 초기 설정
chore(git): .gitignore에 환경변수 파일 제외 설정 추가
chore(env): 환경변수 샘플 파일 추가
feat(gradle): .env 파일 로드 스크립트 추가
feat(config): application.yml 환경 설정 파일 추가
feat(config): JPA Auditing 활성화를 위한 설정 클래스 추가
feat(entity): 파일 메타데이터를 저장할 FileEntity 엔티티 추가
feat(repository): 파일 데이터 접근을 위한 FileRepository 추가
feat(dto): API 응답용 FileResponseDTO 추가
feat(service): 파일 업로드/다운로드 비즈니스 로직을 담당하는 FileService 추가
feat(controller): 파일 업로드/다운로드 REST API 컨트롤러 추가
```

각 커밋 메시지에 해당 파일의 역할과 코드 설명이 포함되어 있음

첫 번째 커밋(feat(init))에 build.gradle, Gradle Wrapper, settings.gradle, 메인 클래스가 모두 포함됨
이후 .gitignore, .env.sample, 환경변수 스크립트 등이 순차적으로 추가됨


## 운영 환경 고려사항

실제 서비스에 적용할 때 추가로 고려해야 할 사항:

- 파일 저장소를 AWS S3 등 클라우드 스토리지로 변경 (서버 확장 시 필수)
- 데이터베이스를 MySQL, PostgreSQL 등 운영용 RDBMS로 변경
- 파일 크기 및 업로드 횟수 제한 (DDoS 방지)
- 허용 파일 형식 확장 또는 제한 (보안 정책에 따라 결정)
- 에러 핸들링 강화 (@RestControllerAdvice 활용)
- 파일 바이러스/악성코드 스캔 적용
- CDN을 통한 파일 배포로 응답 속도 개선
- 대용량 파일 업로드 시 청크 업로드 방식 도입
- 파일 접근 권한 관리 (인증/인가 적용)


## 참고 자료

- Spring Boot 공식 문서: https://docs.spring.io/spring-boot
- Spring Data JPA 문서: https://docs.spring.io/spring-data/jpa
- Springdoc OpenAPI: https://springdoc.org
- H2 Database: https://h2database.com