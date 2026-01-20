---
title: "인텔리제이로 스프링부트 프로젝트 Mail 구현"
categories:
  - springboot
tags:
  - springboot
  - mail
last_modified_at:
---

### 인텔리제이로 스프링부트 프로젝트 Mail 구현

스프링부트로 메일 보내는 기능 만들어본 내용임  
처음부터 완벽하게 하려고 하기보다는 돌려보면서 조금씩 맞춰보는 느낌으로 진행함  

#### 스프링부트 메일이란?

스프링부트에서 메일 관련 스타터를 사용해서 SMTP 서버로 메일을 보내는 기능을 말함  
자바 코드에서 메일 제목, 본문, 수신자 주소를 넣어서 서버를 통해 발송할 수 있게 해주는 기능임 

#### 구현 순서  

- Gradle 기반 스프링부트 프로젝트 생성
- .gitignore에 .env와 *.env를 등록  
- 환경변수 파일(.env) 만들기
- 메일, 롬복, 타임리프 의존성 추가  
- application.yml 설정 값 넣기  
- DTO 클래스 만들기  
- 서비스 클래스 만들기  
- 컨트롤러 만들고 테스트 요청 보내보기 

#### 구현하기

##### .gitignore에 .env와 *.env를 등록  

<img width="792" height="501" src="https://github.com/user-attachments/assets/6d79e1c3-ccd9-470f-8588-47226593280e" />  

프로젝트 루트 디렉토리에 있는 .gitignore에 .env와 *.env를 등록하여 github에 환경변수 파일에 대한 작업내용이 올라가지 않게 조치

##### 환경변수 파일(.env) 만들기

<img width="813" height="457" src="https://github.com/user-attachments/assets/9ad84c1a-b211-45bd-8efa-4691a2c1e1f5" />   
  
환경변수 파일(.env) 만들고, 메일 기능에 필요한 환경변수값 입력

##### 의존성 넣기

<img width="792" height="712" src="https://github.com/user-attachments/assets/e738fceb-37c7-4994-8cd4-10aabc3586e4" />  

Java 21 기준으로 진행함  
메일 전송, 롬복, 타임리프를 같이 쓸 예정임  

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-mail'
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
    implementation 'org.springframework.boot:spring-boot-starter-webmvc'
    implementation 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-mail-test'
    testImplementation 'org.springframework.boot:spring-boot-starter-thymeleaf-test'
    testImplementation 'org.springframework.boot:spring-boot-starter-webmvc-test'
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}
```

```groovy
// .env 파일에서 환경변수 로드
def loadEnv() {
    def envFile = file('.env')
    if (envFile.exists()) {
        envFile.readLines().each { line ->
            if (line && !line.startsWith('#') && line.contains('=')) {
                def (key, value) = line.split('=', 2)
                if (System.getenv(key.trim()) == null) {
                    // 환경변수가 설정되지 않은 경우에만 .env 값 사용
                }
            }
        }
    }
}

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

의존성 추가 후에 한 번 빌드나 프로젝트 동기화를 돌려서 문제가 없는지 확인함  

##### 필요값 넣기

메일 서버 연결을 위해 application.yml 에 설정을 넣어야 함  
사용할 SMTP 서비스에 맞게 host, port, 계정 정보를 채워야 함  

```yml
spring:  
  mail:  
    host: smtp.gmail.com
    port: 587
    username: ${MAIL_USERNAME:test@example.com}
    password: ${MAIL_PASSWORD:test-password}
    properties:  
      mail:  
        smtp:  
          auth: true  
          starttls:  
            enable: true
```  

테스트용 계정이나 별도 테스트 서비스를 쓰면 부담이 조금 줄어드는 편임  

#### Spring Boot Mail 구조

JavaMailSender라는 Spring Boot Mail 라이브러리의 클래스를 사용하여 이메일을 전송
이메일 전송을 위해 MailMessage 인터페이스를 호출하여 두 개의 인터페이스 구현체(SimpleMailMessage, MimeMessage) 사용이 가능
SimpleMailMessage의 경우는 간단한 텍스트 기반의 이메일을 보내는 용도이며, 
MimeMessage의 경우는 HTML, 텍스트, 멀티파트, 첨부 파일 등 고급 기능을 이용하는 데 사용되는 용도

##### DTO 생성

<img width="981" height="308" alt="Image" src="https://github.com/user-attachments/assets/13cc5c8f-a3d4-456d-8a1b-4a28c801e159" />

컨트롤러에서 요청을 받을 때 사용할 DTO 를 하나 만듦  
필드는 최소한으로만 두고 시작함  

- to  
- subject  
- content  

롬복을 사용할 예정이라서 생성자나 게터 세터 코드는 최대한 줄일 생각임  

###### SimpleMailMessage용 DTO

```java
@Getter
public class MailTxtDto {
    private String mailAddr;
    private String subject;
    private String content;
}
```  

###### MimeMessage용 DTO

```java
@Getter
public class MailHtmlSendDto {
    private String emailAddr;
    private String subject;
    private String content;
}
```

##### 서비스 생성

실제로 메일을 보내는 역할을 하는 서비스 클래스를 만듦  
DTO 에 들어온 값을 꺼내서 메일 객체에 담고, 메일 전송 메서드를 호출하는 구조로 갈 예정임  
Service는 SimpleMailMessage 보내는 메서드와 MimeMessage 보내는 메서드를 모두 구현

- 발신자, 수신자, 제목, 본문 설정  
- 예외가 발생했을 때 로그로 정도만 확인  
- 나중에 템플릿 메일을 쓰고 싶으면 타임리프를 같이 사용하는 방향으로 확장  

```java
import jakarta.mail.MessagingException;
import jakarta.mail.internet.MimeMessage;
import org.springframework.mail.MailException;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.stereotype.Service;
import org.thymeleaf.TemplateEngine;
import org.thymeleaf.context.Context;

@Service
public class MailSendService {

    private JavaMailSender mailSender;
    private TemplateEngine templateEngine;

    public MailSendService(JavaMailSender mailSender, TemplateEngine templateEngine) {
        this.mailSender = mailSender;
        this.templateEngine = templateEngine;
    }

    /**
     * 텍스트 기반 메일 전송(SimpleMailMessage)
     */
    public void sendSimpleMailMessage(MailTxtDto mailTxtDto) {
        SimpleMailMessage smm = new SimpleMailMessage();

        smm.setTo(mailTxtDto.getMailAddr());
        smm.setSubject(mailTxtDto.getSubject());
        smm.setText(mailTxtDto.getContent());

        try {
            mailSender.send(smm);
            System.out.println("이메일 전송 성공!");
        } catch (MailException e) {
            System.out.println("[-] 이메일 전송중에 오류가 발생하였습니다 " + e.getMessage());
            throw e;
        }
    }

    /**
     * HTML 기반 메일 전송(MimeMessage)
     */
    public void sendMimeMessage(MailHtmlSendDto mailHtmlSendDto) {
        try {
            MimeMessage message = mailSender.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(message,
                    true, "UTF-8");
            Context context = new Context();
            context.setVariable("subject", mailHtmlSendDto.getSubject());
            context.setVariable("content", mailHtmlSendDto.getContent());

            String html = templateEngine.process("example-template", context);

            helper.setTo(mailHtmlSendDto.getEmailAddr());
            helper.setSubject(mailHtmlSendDto.getSubject());
            helper.setText(html, true);
            mailSender.send(message);
        } catch (MessagingException e) {
            System.out.println("[-] Thymeleaf 템플릿 이메일 전송 중 오류 발생: " + e.getMessage());
            throw new RuntimeException(e);
        }
    }
}
```  

##### 컨트롤러 생성

외부에서 메일 전송을 요청할 수 있도록 컨트롤러를 만듦  
POST 요청으로 DTO 를 받게 하고, 서비스 메서드를 호출해서 메일을 보내는 구조로 작성함  

- 예시 URL 은 `api/mail/send` 같은 식으로 하나 정해서 사용함  
- 요청 바디에 DTO 를 받고  
- 성공 여부는 일단 단순한 문자열이나 상태 코드 정도로 응답함  

```java
package com.example.springbootmail1;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.mail.MailException;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Map;

@RestController
@RequestMapping("/api/mail")
public class MailController {

    private final MailSendService mailSendService;

    public MailController(MailSendService mailSendService) {
        this.mailSendService = mailSendService;
    }

    /**
     * 텍스트 기반 메일 전송(SimpleMailMessage)
     */
    @PostMapping("/send/txt")
    public ResponseEntity<?> sendTextMail(
            @RequestBody MailTxtDto mailTxtDto) {
        try {
            mailSendService.sendSimpleMailMessage(mailTxtDto);
            return ResponseEntity.ok(
                Map.of("message", "메일 전송 완료")
            );
        } catch (MailException e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                    .body(
                        Map.of("message", "메일 전송 실패", "error", e.getMessage())
            );
        }
    }

    /**
     * HTML 기반 메일 전송(MimeMessage)
     */
    @PostMapping("/send/html")
    public ResponseEntity<?> sendHtmlEmail(
            @RequestBody MailHtmlSendDto mailHtmlSendDto
    ) {
        try {
            mailSendService.sendMimeMessage(mailHtmlSendDto);
            return ResponseEntity.ok(
                    Map.of("message", "메일 전송 완료")
            );
        } catch (MailException e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                    .body(
                        Map.of("message", "메일 전송 실패", "error", e.getMessage())
            );
        }
    }
}

```  

##### template화면(thymleaf)

나중에 화면에서 메일을 보낼 수 있도록 간단한 폼을 하나 만들 계획임  
타임리프 템플릿을 사용해서 제목, 수신자, 내용을 입력하고 전송할 수 있게 만들 예정임  

- GET 요청으로 메일 작성 폼 페이지 열기  
- POST 요청으로 폼에서 입력한 값 전달  
- 컨트롤러에서 서비스 호출해서 메일 전송  

이 부분은 실제 템플릿을 만들고 나서 따로 정리해서 아래에 더 적을 예정임

<img width="1047" height="554" alt="Image" src="https://github.com/user-attachments/assets/7669d9af-c6b9-479e-946f-2b49fb357f21" />

##### 실행

터미널로 `./gradlew bootRun` 라고 실행해야 함
(.env 파일을 읽는 명령어 스크립트가 build.gradle에 있기에 터미널로 `./gradlew bootRun` 명령어를 실행)

<img width="712" height="120" alt="Image" src="https://github.com/user-attachments/assets/07657a85-89dd-4d59-97f5-f5f82d84f692" />
<img width="532" height="48" alt="Image" src="https://github.com/user-attachments/assets/887bf43b-3443-4c45-9370-68ff741bdf41" />

##### Postman으로 API 요청 & 응답
<img width="603" height="231" alt="Image" src="https://github.com/user-attachments/assets/953faae0-789b-4323-8c7e-7d83e9a1855a" />
<img width="457" height="156" alt="Image" src="https://github.com/user-attachments/assets/bbc1c278-eefc-438d-8785-395169ed7f8f" />
<img width="324" height="485" alt="Image" src="https://github.com/user-attachments/assets/8e1745ef-ec6b-46fc-9059-91ff06c20187" />
