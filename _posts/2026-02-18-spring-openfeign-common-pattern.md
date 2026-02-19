---
title: "OpenFeign을 활용한 외부 API 연동 공통 패턴 정리"
categories:
  - springboot
tags:
  - springboot
  - openfeign
  - api
last_modified_at:
---

### OpenFeign을 활용한 외부 API 연동 공통 패턴 정리

SMS 전송 API(Solapi)랑 공공데이터 날씨 API를 OpenFeign으로 각각 구현했음
다음에 또 다른 외부 API를 연동할 때 이 포스트 보고 따라할 수 있게끔 정리

#### OpenFeign이란?

Open Feign은 Netflix에 의해 처음 만들어진 Declarative(선언적인) HTTP Client 도구로, 외부 API 호출을 쉽게할 수 있도록 도와줌
여기서 “선언적인” 이란 어노테이션 사용을 의미, Open Feign은 인터페이스에 어노테이션들만 붙여주면 구현 가능

#### 공통 구현 순서

- Gradle 기반 스프링부트 프로젝트 생성
- OpenFeign 관련 의존성 추가
- .gitignore에 .env 등록
- 환경변수 파일(.env) 만들기
- application.yml에 외부 API URL, 키 값, Feign 설정 넣기
- FeignConfig 설정 클래스 만들기
- FeignClient 인터페이스 만들기
- 인증 처리 (Interceptor 또는 파라미터)
- 요청/응답 DTO 만들기
- Service 클래스 만들기
- Controller 만들고 테스트 요청 보내보기

#### 구현하기

##### 의존성 넣기

```groovy
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    implementation 'org.springframework.boot:spring-boot-starter-webmvc'
    implementation 'org.springframework.cloud:spring-cloud-starter-openfeign'
    implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:3.0.1'
```

##### .gitignore에 .env와 *.env 등록

```
.env 
*.env
```

##### build.gradle에 환경변수 로드 스크립트 추가

loadEnv() 함수: .env 파일을 읽어서 시스템 프로퍼티로 세팅하는 부분
bootRun 블록: .env 파일을 읽어서 Spring Boot 실행 시 환경변수로 전달하는 부분
이 스크립트가 있어야 ./gradlew bootRun 할 때 .env 값이 application.yml에 주입됨

```groovy
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

##### 환경변수 파일(.env) 만들기

프로젝트 루트에 .env 파일을 만들고 외부 API 인증에 필요한 값 입력

날씨 프로젝트 .env.sample:

```env
# 공공데이터에서 발급받은 API키
PUBLIC-DATA-APIKEY=helloworld

# 공공데이터 도메인
PUBLIC-DATA-URL=https://domain.com

```

외부 API가 달라도 "키 이름과 값"만 변경하면 적용됨

##### application.yml 설정

```yaml
server:
  port: 8080

spring:
  application:
    name: weather

# Feign 
feign:
  client:
    config:
      default:
        connect-timeout: 5000
        read-timeout: 5000
        logger-level: full

# PublicData API
app:
  public-data-url: ${PUBLIC-DATA-URL:public-data-domain}
  public-data-apikey: ${PUBLIC-DATA-APIKEY:your-public-data-api-key}
  public-data-weather-dataType: JSON

# 
logging:
  level:
    com.example.weather: DEBUG
    com.example.weather.client: DEBUG

# Swagger 
springdoc:
  api-docs:
    path: /api-docs
  swagger-ui:
    path: /swagger-ui.html
    tags-sorter: alpha
    operations-sorter: alpha
```

외부 API URL과 인증 키만 바꾸면 같은 구조로 쓸 수 있음

##### FeignConfig 설정 클래스 만들기

@EnableFeignClients는 메인 Application 클래스에 직접 붙이지 않고, 별도 설정 클래스에 분리해서 붙임
FeignConfig 클래스를 만들어서 @Configuration + @EnableFeignClients를 같이 달아주는 방식

```java
@Configuration
@EnableFeignClients(basePackages = "com.example.weather")
public class FeignConfig {

    @Value("${app.public-data-apikey}")  // application.yml에 있는 커스텀 환경변수값
    private String publicDataApikey;

    @Value("${app.public-data-weather-dataType}")  // application.yml에 있는 커스텀 환경변수값
    private String publicDataWeatherDataType;

    @Bean
    public RequestInterceptor requestInterceptor() {
        return requestTemplate -> {
            requestTemplate.query("ServiceKey", publicDataApikey);
            requestTemplate.query("pageNo", "1");
            requestTemplate.query("numOfRows", "1000");
            requestTemplate.query("dataType", publicDataWeatherDataType);
        };
    }
}
```

##### FeignClient 인터페이스 만들기

날씨 프로젝트 FeignClient:

```java
@FeignClient(  // OpenFeign의 강점인 `선언적 어노테이션 사용`으로 쉽게 외부 API 호출 가능
        name = "weatherClient",
        url = "${app.public-data-url}"
)
public interface WeatherClient {
    @GetMapping("/1360000/VilageFcstInfoService_2.0/getUltraSrtNcst")
    Map<String, Object> getWeather(
            @RequestParam("base_date") String baseDate,
            @RequestParam("base_time") String baseTime,
            @RequestParam("nx") String nx,
            @RequestParam("ny") String ny

    );
}
```

name: 클라이언트 식별 이름
url: application.yml에서 ${...}로 주입
메서드에 @GetMapping 또는 @PostMapping 붙이고, 매개변수랑 반환 타입 정의

##### 인증 처리

외부 API마다 인증 방식이 다름
OpenFeign 라이브러리에서는 RequestInterceptor에다가 
헤더 요소를 추가하거나, 쿼리 파라미터 요소로 추가해서 반환하게끔 구현하면, 
OpenFeign에 의한 외부 API 요청에 반영됨

날씨 프로젝트 (쿼리 파라미터 인증 방식):

```java
@Configuration
@EnableFeignClients(basePackages = "com.example.weather")
public class FeignConfig {

    @Value("${app.public-data-apikey}")
    private String publicDataApikey;

    @Value("${app.public-data-weather-dataType}")
    private String publicDataWeatherDataType;

    @Bean
    public RequestInterceptor requestInterceptor() {
        return requestTemplate -> {
            requestTemplate.query("ServiceKey", publicDataApikey);
            requestTemplate.query("pageNo", "1");
            requestTemplate.query("numOfRows", "1000");
            requestTemplate.query("dataType", publicDataWeatherDataType);
        };
    }
}

```

헤더에 넣는 API면 RequestInterceptor, 쿼리 파라미터로 넣는 API면 @RequestParam으로 처리

##### 요청/응답 DTO 만들기

날씨 프로젝트 DTO: DTO는 아니지만, 
`공공데이터 포탈`에서 `날씨 조회 API`를 사용할 때 `필수적`으로 
`쿼리 파라미터`로 요청에 포함해야 하는 요소들에 대해 다음과 같이 구현

RequestInterceptor에 .query(String key, String value)로 포함
- ServiceKey
- pageNo
- numOfRows
- dataType

```java
 @Bean
    public RequestInterceptor requestInterceptor() {
        return requestTemplate -> {
            requestTemplate.query("ServiceKey", publicDataApikey);
            requestTemplate.query("pageNo", "1");
            requestTemplate.query("numOfRows", "1000");
            requestTemplate.query("dataType", publicDataWeatherDataType);
        };
```

인터페이스 호출 시 매개변수(RequestParam)로 포함
- base_date
- base_time
- nx
- ny

```java
public interface WeatherClient {
    @GetMapping("/1360000/VilageFcstInfoService_2.0/getUltraSrtNcst")
    Map<String, Object> getWeather(
            @RequestParam("base_date") String baseDate,
            @RequestParam("base_time") String baseTime,
            @RequestParam("nx") String nx,
            @RequestParam("ny") String ny

    );
}
```

##### Service 만들기

FeignClient를 주입받아서 호출하는 부분이 핵심

날씨 프로젝트 Service:

```java
@Slf4j
@Service
public class WeatherService {

    private final WeatherClient weatherClient;

    public WeatherService(WeatherClient weatherClient) {
        this.weatherClient = weatherClient;
    }

    public Map<String, Object> getCurrentWeather(String nx, String ny) {
        LocalDateTime now = LocalDateTime.now();
        String baseDate = now.format(DateTimeFormatter.ofPattern("yyyyMMdd"));
        String baseTime = now.format(DateTimeFormatter.ofPattern("HH00"));

        log.debug("날씨 요청: baseDate={}, baseTime={}, nx={}, ny={}", baseDate, baseTime, nx, ny);
        Map<String, Object> data = weatherClient.getWeather(baseDate, baseTime, nx, ny);
        log.debug("Weather: {}", data);
        return data;
    }
}
```

FeignClient를 생성자를 통해 의존성 주입받아서 사용준비(초기화)
→ feignClient.메서드() 호출

##### Controller 만들기

날씨 프로젝트 Controller:

```java
@RestController
@RequestMapping("/api/weather")
@Tag(name = "Weather", description = "기상청 단기 조회 API")
public class WeatherRestController {

    private final WeatherService weatherService;

    public WeatherRestController(WeatherService weatherService) {
        this.weatherService = weatherService;
    }

    @GetMapping
    @Operation(summary = "현재 날씨 조회", description = "기상청 데이터 조회")
    public Map<String, Object> getWeather(
            @Parameter(description = "X 좌표 (격자)", example = "55")
            @RequestParam(defaultValue = "55") String nx,
            @Parameter(description = "Y 좌표 (격자)", example = "127")
            @RequestParam(defaultValue = "127") String ny
    ) {
        return weatherService.getCurrentWeather(nx, ny);
    }
}
```

##### 실행 및 테스트

이 부분에는 .env 파일 읽어야 하니까 터미널로 실행하는 명령어(`./gradlew bootRun`)와 Postman으로 테스트하는 과정을 보여주면 돼
Postman 요청/응답 스크린샷이 있으면 SMS 프로젝트, 날씨 프로젝트 각각 넣으면 좋고 없으면 생략해도 돼

###### 터미널로 실행

터미널로 실행해야 하는 이유는 .env 파일 읽어야 하니까, 
build.gradle에 넣은 환경변수 읽는 스크립트를 터미널 명령어(`./gradlew bootRun`)로 실행시켜야 하기 때문

```sh
./gradlew bootRun
```

![스프링부트기동](/assets/images/2026-02-18-spring-openfeign-common-pattern-1.png)

![포스트맨요청](/assets/images/2026-02-18-spring-openfeign-common-pattern-2.png)

![공공데이터날씨응답](/assets/images/2026-02-18-spring-openfeign-common-pattern-3.png)

#### 정리 — 외부 API 연동 시 매번 반복되는 체크리스트

- [ ] spring-cloud-starter-openfeign 의존성 추가 + @EnableFeignClients 붙이기
- [ ] .env 파일 만들고 .gitignore에 등록
- [ ] application.yml에 외부 API URL, 인증 키, Feign 타임아웃 설정
- [ ] @FeignClient 인터페이스 선언 (name, url, 메서드에 @GetMapping/@PostMapping)
- [ ] 인증 방식에 따라 RequestInterceptor 또는 @RequestParam 처리
- [ ] 요청/응답 DTO 만들기 (내부용, 외부 API용 분리)
- [ ] Service에서 FeignClient 주입받아서 호출
- [ ] Controller에서 Service 호출
- [ ] ./gradlew bootRun으로 실행 후 Postman으로 테스트
