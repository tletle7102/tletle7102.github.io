---
title: "Spring Boot에서 이메일 발송 기능 만들기"
categories:
  - Spring Boot
tags:
  - Spring Boot
  - Email
last_modified_at:
---

#### 📌 용어 설명
- JavaMailSender: 스프링에서 이메일 발송을 처리하는 인터페이스
- SMTP: 이메일 전송 프로토콜. Gmail, Naver 등에서 제공
- application.yml: 스프링부트 설정 파일. 이메일 서버 정보 관리
- MimeMessage: 복잡한 형식(HTML, 첨부파일 등)의 이메일 메시지
  
---
#### 📌 이메일 발송 기능이란
스프링부트에서 이메일 발송은  
회원가입 인증, 비밀번호 재설정, 알림 전송 등에 자주 사용  
JavaMailSender를 활용해 간단히 구현 가능  

외부 SMTP 서버(Gmail 등)를 연동해  
애플리케이션에서 이메일을 보내는 구조  

---
#### 📌 이메일 발송 설정 예시
스프링부트에서 Gmail SMTP를 사용한 설정  

application.yml에 SMTP 정보 추가  
```yml  
spring:
  mail:
    host: smtp.gmail.com
    port: 587
    username: your-email@gmail.com
    password: your-app-password
    properties:
      mail:
        smtp:
          auth: true
          starttls:
            enable: true
```  

Gmail은 앱 비밀번호 필요. 2단계 인증 활성화 후 생성  

---
#### 📌 이메일 발송 코드 구현
##### ✅ 의존성 추가
build.gradle에 spring-boot-starter-mail 추가  
```groovy  
implementation 'org.springframework.boot:spring-boot-starter-mail'
```  

##### ✅ 서비스 클래스 구현
이메일 발송 로직을 서비스로 분리  
```java
@Service
public class EmailService {
    private final JavaMailSender mailSender;

    public EmailService(JavaMailSender mailSender) {
        this.mailSender = mailSender;
    }

    public void sendSimpleEmail(String to, String subject, String text) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setTo(to);
        message.setSubject(subject);
        message.setText(text);
        message.setFrom("your-email@gmail.com");
        mailSender.send(message);
    }

    public void sendHtmlEmail(String to, String subject, String htmlContent) throws MessagingException {
        MimeMessage message = mailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(message, true, "UTF-8");
        helper.setTo(to);
        helper.setSubject(subject);
        helper.setText(htmlContent, true);
        helper.setFrom("your-email@gmail.com");
        mailSender.send(message);
    }
}
```  

sendSimpleEmail은 텍스트 이메일  
sendHtmlEmail은 HTML 포맷 이메일 전송  

##### ✅ 컨트롤러에서 호출
```java  
@RestController
@RequestMapping("/api/email")
public class EmailController {
    private final EmailService emailService;

    public EmailController(EmailService emailService) {
        this.emailService = emailService;
    }

    @PostMapping("/send")
    public String sendEmail(@RequestParam String to, @RequestParam String subject, @RequestParam String text) {
        emailService.sendSimpleEmail(to, subject, text);
        return "Email sent successfully";
    }
}
```

---
#### 📌 주의사항
1. Gmail 앱 비밀번호는 보안상 민감. 환경 변수로 관리 권장  
2. SMTP 포트(587)와 TLS 설정 확인  
3. 대량 이메일 발송 시 SMTP 서버 제한 확인  
4. HTML 이메일은 XSS 공격 방지 위해 입력 검증 필수  

---
#### 📌 이메일 발송의 의미
이메일 기능은 단순한 전송 이상  
사용자 경험 개선, 시스템 신뢰도 향상에 기여  
스프링부트는 복잡한 설정을 간소화해  
핵심 로직(메시지 내용, 수신자 관리)에 집중 가능  

---
#### ✍ 알아보면서
이메일 발송 설정은 처음엔 복잡해 보였지만  
JavaMailSender와 application.yml 설정으로  
쉽게 구조화 가능  

외부 서비스(Gmail SMTP)와 연동하며  
환경 설정의 중요성과 보안 고려사항을 깨달음  
이메일 하나 보내는 데도  
설정, 예외 처리, 사용자 경험까지 고민해야 함   

---