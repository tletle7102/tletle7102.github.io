---
title: "코드 변경 반영 라이브러리(LiveReload Reborn, DevTools)"
categories:
  - Spring Boot
tags:
  - Spring Boot
  - DevTools
  - LiveReload
last_modified_at: 
---

#### 코드 변경 반영 라이브러리(LiveReload Reborn, DevTools)  
웹 개발에서 코드 수정마다 서버 재시작이나 브라우저 새로고침을 수동으로 하는 건 번거로움  
코드 변경 실시간 반영으로 개발 속도 높임  

#### 📌 용어 설명  

* Spring Boot DevTools: 개발 도구 라이브러리  
코드 변경 감지해 자동 재시작 지원  
라이브 리로드로 브라우저 자동 업데이트  

* Live Reload: 파일 수정 시 브라우저 실시간 새로고침 기능  
서버와 Extension 연동  

* LiveReload Reborn: Chrome Extension  
기존 LiveReload 대체 버전  
WebSocket 연결로 변경 감지  

#### 📌 Spring Boot DevTools란?  

개발 환경 전용 라이브러리  
자동 재시작으로 코드 반영 즉시 적용  
라이브 리로드 내장 서버 제공  

개발 환경과 프로덕션 환경을 구분하기 위해 주로 애플리케이션의 실행 방식에 기반한 자동 감지 매커니즘 사용  
- 클래스패스 실행 시 활성화  
- 프로덕션에서 자동 비활성화  
- JAR 실행 시 비활성화  

💡 DevTools로 정적 파일 변경 시 브라우저 자동 리로드 가능  
Extension과 함께 사용  

#### 📌 Spring Boot DevTools 설치 및 사용 방법  

1. build.gradle에 DevTools 의존성 추가  

```groovy  
dependencies {
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
}
```  

2. IntelliJ에서 추가 설정  
   Settings > Build, Execution, Deployment > Compiler > Build project automatically 체크  
   Ctrl+Alt+Shift+/ > Registry > compiler.automake.allow.app.running = true  

3. application.yml에 필요 시 설정 추가  
```yml  
spring:
  devtools:
    restart:
      enabled: true
    livereload:
      enabled: true
  server:
    port: 8080
```  

4. 애플리케이션 실행  
   IntelliJ에서 Run 또는 Gradle 탭에서 bootRun  
   코드 변경 시 자동 재시작 확인  

#### 📌 LiveReload Reborn이란?  

Chrome Extension  
기존 LiveReload abandoned 후 대체  
로컬 서버와 WebSocket 연결  
파일 변경 감지해 브라우저 리로드  
개발 생산성 향상  

💡 Reborn으로 localhost:8080 포트 연결  
DevTools와 호환  

#### 📌 LiveReload Reborn 설치 및 사용 방법  

1. Chrome Web Store에서 "LiveReload Reborn" 검색 설치  

2. Extension 아이콘 클릭 활성화  

3. DevTools 실행 중 라이브 리로드 서버 확인  

4. 브라우저에서 페이지 열기  
   변경 시 자동 리로드  

5. 연결 문제 시 Extension 재활성화  

#### 📌 DevTools와 LiveReload Reborn 함께 사용하는 방법  

1. build.gradle DevTools 추가  

2. IntelliJ 설정 적용  

3. application.yml 라이브 리로드 활성화  

4. Extension 설치 활성화  

5. 코드 수정  
   Java 클래스: 자동 재시작  
   정적 파일: 브라우저 리로드  

6. 서버 확인 명령  
```shell  
curl http://localhost:8080
```  

💡 이 조합으로 백엔드 프론트엔드 동시 개발 효율적  

#### 📌 주의사항  

* DevTools 개발 환경만 사용  
프로덕션 배포 시 비활성화  

* 2025년 8월 Chrome 기준, LiveReload 사용 불가, 대신 LiveReload Reborn 버전 사용 가능  