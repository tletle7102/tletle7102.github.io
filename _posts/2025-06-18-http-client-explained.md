---
title: "HTTP 클라이언트란 무엇인가? RestTemplate, WebClient 등"
categories:
  - CS Basics
tags:
  - CS Basics
last_modified_at:
---

#### 📌 HTTP 클라이언트란?
- HTTP 클라이언트는 서버에 요청을 보내고, 서버로부터 응답을 받는 프로그램이나 라이브러리  
- 실생활 예시:  
  - 음식 배달 앱에서 '피자 주문' 버튼을 눌렀을 때  
  - 앱 내부에서 피자가게 서버에 HTTP 요청 전송  
  - 이때 HTTP 요청을 만드는 역할을 하는 것이 HTTP 클라이언트 (RestTemplate, WebClient 등)  
  - 앱이 피자 가게 서버에 주문 요청을 보내고, 서버로부터 주문 확인 응답을 받는 과정  

즉, HTTP 클라이언트는 인터넷을 통해 데이터를 주고받기 위해 사용하는 도구  

---
#### 📌 흐름 단계
1. HTTP 요청 보내기  
주문 정보(메뉴, 수량, 주소 등)를 HTTP 요청의 본문(body)에 담아서 서버로 보냄  
2. 서버 처리  
피자가게 서버가 요청을 받아서 주문을 처리하고 응답을 준비  
3. HTTP 응답 받기  
서버가 '주문이 정상 접수되었습니다'라는 응답(HTTP Response)을 보냄  
4. 앱에서 응답 처리  
앱 화면에 '주문 완료' 메시지를 표시하거나 주문 내역 화면으로 이동  

---
#### 📌동기/비동기 차이 예시로 다시 보기
1. 동기적 처리  
- 앱이 HTTP 요청을 보낸 후, 서버의 응답을 받을 때까지 기다림
- 그동안 다른 화면으로 넘어가지 않음 (로딩 화면 등 표시)
2. 비동기적 처리
- 앱이 HTTP 요청을 보낸 후, 바로 다른 화면으로 이동 가능
- 서버 응답이 오면 알림이나 화면 업데이트로 결과 표시
- 사용자 경험(UX)이 부드러움

정리하면, **HTTP 클라이언트는 서버와 '말을 주고받는 도구'**  이고,
말을 주고받는 방법(동기/비동기)은 상황에 따라 선택하는 것

---
#### 📌 주요 HTTP 클라이언트 도구들과 예시 코드 (자바 중심)

##### 1. RestTemplate (스프링 동기 HTTP 클라이언트)
- 서버에 요청 보내고 응답을 받을 때까지 기다림 (동기 방식)  
- 간단한 REST API 호출에 적합

```java  
import org.springframework.web.client.RestTemplate;

public class RestTemplateExample {
    public static void main(String[] args) {
        RestTemplate restTemplate = new RestTemplate();

        // GET 요청 보내기 (동기)
        String url = "https://jsonplaceholder.typicode.com/posts/1";
        String response = restTemplate.getForObject(url, String.class);

        // 응답 출력
        System.out.println("Response: " + response);
    }  
}  
```  

##### 2. WebClient (스프링 5 이상 비동기 및 논블로킹 클라이언트)
- 요청 후 바로 다음 작업 진행, 응답은 나중에 받음 (비동기 방식)  
- 리액티브 프로그래밍에 적합

```java  
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Mono;

public class WebClientExample {
    public static void main(String[] args) {
        WebClient client = WebClient.create("https://jsonplaceholder.typicode.com");

        Mono<String> responseMono = client.get()
            .uri("/posts/1")
            .retrieve()
            .bodyToMono(String.class);

        // 비동기적으로 응답 받기
        responseMono.subscribe(response -> System.out.println("Response: " + response));

        System.out.println("요청 후 다른 작업 수행 중...");
        try {
            Thread.sleep(2000); // 비동기 결과 기다리기 위해 잠시 대기
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```  

##### 3. Apache HttpClient (자바에서 많이 쓰이는 HTTP 클라이언트 라이브러리)
- 동기 방식 지원, 커넥션 관리 기능 강력함

```java  
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;

public class ApacheHttpClientExample {
    public static void main(String[] args) throws Exception {
        try (CloseableHttpClient client = HttpClients.createDefault()) {
            HttpGet request = new HttpGet("https://jsonplaceholder.typicode.com/posts/1");
            try (CloseableHttpResponse response = client.execute(request)) {
                String responseBody = EntityUtils.toString(response.getEntity());
                System.out.println("Response: " + responseBody);
            }
        }
    }
}
```  

##### 4. OkHttp (안드로이드와 자바에서 널리 쓰이는 HTTP 클라이언트)
- 동기 및 비동기 모두 지원  
- HTTP/2, 웹소켓 지원

```java  
import okhttp3.Call;
import okhttp3.Callback;
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.Response;

import java.io.IOException;

public class OkHttpExample {
    public static void main(String[] args) {
        OkHttpClient client = new OkHttpClient();

        Request request = new Request.Builder()
            .url("https://jsonplaceholder.typicode.com/posts/1")
            .build();

        // 비동기 요청
        client.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                System.out.println("Request failed: " + e.getMessage());
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                System.out.println("Response: " + response.body().string());
            }
        });

        System.out.println("요청 후 다른 작업 수행 중...");
        try {
            Thread.sleep(2000); // 비동기 응답 기다리기 위해 잠시 대기
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```  

##### 5. Feign (스프링 클라우드에서 사용하는 선언적 REST 클라이언트)
- 인터페이스로 REST API 호출 정의  
- 내부적으로 HTTP 클라이언트를 사용하여 호출  

```java  
// 인터페이스 정의
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@FeignClient(name = "jsonPlaceholder", url = "https://jsonplaceholder.typicode.com")
public interface JsonPlaceholderClient {
    @GetMapping("/posts/{id}")
    String getPostById(@PathVariable("id") Long id);
}

// 사용 예
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class FeignExample implements CommandLineRunner {

    @Autowired
    private JsonPlaceholderClient client;

    @Override
    public void run(String... args) throws Exception {
        String response = client.getPostById(1L);
        System.out.println("Response: " + response);
    }
}
```  

---
#### ✍ 알아보면서
HTTP 클라이언트는 서버에 요청을 보내고, 답을 받아오는 과정을 쉽게 도와주는 쉽게 예를 들자면 서버와 대화하는 '전화기' 같은 도구  

RestTemplate은 가장 간단하고 익숙한 방식이라 많이 쓰임  
그러나 대규모 트래픽이나 비동기 처리가 필요한 상황에서는 WebClient가 훨씬 효율적임  

Apache HttpClient, OkHttp는 좀 더 세밀한 제어가 필요할 때 선택하는 라이브러리이며  
Feign은 인터페이스 선언만으로 API 호출을 쉽게 만들 수 있어 편리함  

---